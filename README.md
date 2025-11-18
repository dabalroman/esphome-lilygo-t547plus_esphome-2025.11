This repository contains a Display component for [ESPHome](https://esphome.io/) 2025.11+ 
to support the ESP32-S3 [LILYGO T5 4.7" Plus E-paper display](https://www.lilygo.cc/products/t5-4-7-inch-e-paper-v2-3).
For more info on the display components, see the [ESPHome display documentation](https://esphome.io/#display-components)

**(Do not confuse it with the original ESP32-based Lilygo T5 4.7 board.)**

![](readme-files/full_setup.jpg)
<p align='center'><b>Image of full setup config</b></p>


> [!TIP]
> The "dusty" marks on the image above are signs of e-ink screen fatigue.  
> This will also happen to your display, so avoid frequent updates and rendering the same stuff over and over.

## Contributions
- Huge thanks to [@hbast](https://github.com/hbast) for ESP-IDF 5.4.2+ Compatibility!


## Usage (TLDR)
To use the board with [ESPHome](https://esphome.io/), adjust your `.yaml` config:
Make note how `platformio_options`, `libraries`, `esp32`, `sdkconfig_options` and `external_components` are set up in [basic setup](#basic-setup) below, these are important for the compilation stage.
Make sure to `Clean build files` if you're editing your old `.yaml` configs. 

```yaml
external_components:
  - source: github://dabalroman/esphome-lilygo-t547plus_esphome-2025.11
    components: ["t547"]
```


## Usage (step by step)
1. Open the esphome dashboard.
2. Fill in the secrets with ssid and password of your Wi-Fi network.
3. Create new device.
4. In `New Device` wizard use `ESP32S3` board.
5. You may install the firmware now or skip and apply the config directly from [basic_setup.yaml](basic_setup.yaml).
6. Make sure to not override the `name`, `friendly_name`, `api` and `ota` sections with the example, cause the Wi-Fi update may might fail. The values there should be unique for every setup to make sure your device is secure!
7. Flash the [basic setup](basic_setup.yaml) or [full setup](full_setup.yaml) via OTA to confirm the connection does work.
8. Now you can do whatever you want :)

> [!IMPORTANT]
> EspHome aggressively uses cache. 
> Try using `Clean build files` on device card before `Install` if compilation fails.

## Basic setup
Basic setup listing is available in [basic_setup.yaml](basic_setup.yaml) file.  
You can use the [full_setup.yaml](full_setup.yaml) file to use almost test most of supported display features, including fetching time data from HomeAssistant.

```yaml
esphome:
  name: lilygo
  friendly_name: lilygo e-ink display

  platformio_options:
    upload_speed: 921600
    monitor_speed: 115200
    board_build.mcu: esp32s3
    board_build.f_cpu: 240000000L
    board_build.arduino.memory_type: qspi_opi
    board_build.flash_size: 4MB
    board_build.flash_mode: qio
    board_build.flash_type: qspi
    board_build.psram_type: opi
    board_build.memory_type: qspi_opi
    board_build.boot_freq: 80m
    build_flags:
      - "-DBOARD_HAS_PSRAM"
      - "-DARDUINO_USB_CDC_ON_BOOT=1"
      - "-mfix-esp32-psram-cache-issue"

  libraries:
    - SPI

esp32:
  board: esp32-s3-devkitc-1
  framework:
    type: arduino
    # CRITICAL for ESP-IDF 5.4.2+: Enable PSRAM initialization (thanks @hbast!)
    sdkconfig_options:
      CONFIG_SPIRAM: "y"
      CONFIG_SPIRAM_MODE_OCT: "y"
      CONFIG_SPIRAM_SPEED_80M: "y"
      CONFIG_SPIRAM_USE_MALLOC: "y"
      CONFIG_SPIRAM_MALLOC_ALWAYSINTERNAL: "16384"
      CONFIG_SPIRAM_TRY_ALLOCATE_WIFI_LWIP: "y"
      CONFIG_ESP32S3_SPIRAM_SUPPORT: "y"
      CONFIG_SPIRAM_BOOT_INIT: "y"
      CONFIG_SPIRAM_IGNORE_NOTFOUND: "n"

# Enable logging
logger:

# Enable Home Assistant API
api:
  encryption:
    key: "uji41VdMORB5966gRlVYEbBaC5LU9NZwHStB8j2d/nU="

ota:
  - platform: esphome
    password: "4ab30fdd89bd61087359d5eebf85d296"

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "Lilygo Fallback Hotspot"
    password: "zYQIOsSr8spF"

captive_portal:

external_components:
  - source: github://dabalroman/esphome-lilygo-t547plus_esphome-2025.11
    components: ["t547"]

font:
  - file: "gfonts://Roboto"
    id: font_title
    size: 52

  - file: "gfonts://Roboto"
    id: font_subtitle
    size: 32

display:
  - platform: t547
    id: my_display
    update_interval: 10s
    rotation: 180
    lambda: |-
      const int w = it.get_width();
      const int h = it.get_height();

      it.fill(COLOR_OFF);
      it.rectangle(10, 10, w - 20, h - 20);

      for (int x = 20; x < w - 20; x += 40) {
        it.line(x, 30, x + 20, 30, COLOR_ON);
      }

      for (int i = 0; i < 40; i += 4) {
        it.line(20 + i, 20, 20, 20 + i, COLOR_ON);
      }

      it.printf(w / 2, h / 2 - 40, id(font_title), TextAlign::CENTER, "ESPHome T547");
      it.printf(w / 2, h / 2 + 10, id(font_subtitle), TextAlign::CENTER, "Hello world!");
```


## Z-lib
This branch does not support z-lib compressed fonts.


## Discussion
https://github.com/esphome/feature-requests/issues/1960
