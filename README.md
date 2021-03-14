## ESP32 Dev Board with latest WROOM-32E module, USB Type-C, 800mA LDO, 16MB flash and PTC fused.

After receiving too many boards that were DOA (dead-on-arrival) or that came with cheap power regulators causing power resets, I decided to design my own dev board.

### What is it?

TwilightLord-ESP32 is a custom ESP32 Dev board based on the D1 Mini32 but with many updates to it, this means it is a drop-in replacement for any design using a D1 Mini32 (same dimensions), which in term is also pin compatible with the original D1 Mini (ESP8266 based):

- ESP32-WROOM-32E which is the latest revision that includes the ESP32-D0WD-V3 chip (link) (latest on the ECO line with improved power consumption and stability over the V0 that most ESP32 dev boards include)
- 16MB flash, double what most dev boards include. Plenty of space for complex Arduino projects, with OTA slots, larger SPIFFs partitions for your data (store your HTMLs in flash and not as string literals consuming memory), etc.
- USB Type-C connector for data and power
- ESD USB protection. Most D1 Mini clones lack of electrostatic discharge protection.
- PTC Fused that trips at 750ma. This will stop the board from taking more than its traces can handle, protecting the board from early failures.
- 800mA Linear 3.3v LDO regulator. It provides plenty of current to prevent the power resets that occur with other boards with 250mA-500mA regulators, when using multiple GPIOs current spikes can cause power resets if the regulator cannot handle it. Plenty of D1 Mini32 clones come with 250mA regulators.
- Plenty of capacitors added to guarantee stable power delivery to the ESP32.
- D1 Mini and Mini32 pin compatible. 9-pin rows used instead of 10-pin ones since WROOM-32E drops a few pins (connected to internal flash) so no need for 20 pins any longer. Skip the first pin (next to USB connector) if plugging it to a 10-pin header.
- CP2104 USB-to-serial (USB to UART) controller with up to 2Mbits Baud rate. Write the WROOM flash in just over 10 seconds.
- BOOT and RESET buttons. Most compact form factor dev boards only come with RESET button, not allowing you to enter bootloader mode by just pressing buttons.
- Autoprogram circuit included. Tools like esptool and ESPHome-Flasher can restart the ESP32 in bootloader mode for programming.

### Useful environment configuration files

Here are some files that might help you configure the module in your environment:

 - Sample partitions file for the 16MB Flash, with 2 OTA Slots, EEPROM and SPIFF (copy or [download](partitions_16M.csv)) 

        ```csv
        # Name,   Type, SubType, Offset,  Size, Flags
        nvs,      data, nvs,     0x9000,  20K,
        otadata,  data, ota,     0xe000,  8K,
        app0,     app,  ota_0,   0x10000, 6M,
        app1,     app,  ota_1,          , 6M,
        eeprom,   data, 153,            , 4K,
        spiffs,   data, spiffs,         , 4028K,
        ```


 - Board definition file for PlatformIO. Copy it to your  *\<PROFILE\>/*.platformio/boards folder (copy or [download](twilightlord-esp32-16MB.json))

    ```json
    {
    "build": {
        "arduino": {
        "ldscript": "esp32_out.ld"
        },
        "core": "esp32",
        "extra_flags": "-DARDUINO_ESP32_DEV",
        "f_cpu": "240000000L",
        "f_flash": "40000000L",
        "flash_mode": "dio",
        "mcu": "esp32",
        "variant": "esp32",
        "partitions": "partitions_16M.csv"
    },
    "connectivity": [
        "wifi",
        "bluetooth",
        "ethernet",
        "can"
    ],
    "debug": {
        "openocd_board": "esp-wroom-32.cfg"
    },
    "frameworks": [
        "arduino",
        "espidf"
    ],
    "name": "TwilightLord-ESP32 16MB",
    "upload": {
        "flash_size": "16MB",
        "maximum_ram_size": 327680,
        "maximum_size": 16777216,
        "require_upload_port": true,
        "speed": 2000000
    },
    "url": "https://en.wikipedia.org/wiki/ESP32",
    "vendor": "TwilightLord"
    }
    ```


 - Sample Environment definition for your platformio.ini file (copy or [download](env_twilightlord-esp32-16MB_platformio.ini))

    ```dosini
    [env:twilightlord-esp32-16MB]
    board = twilightlord-esp32-16MB
    platform = espressif32@3.0
    build_unflags = ${env:esp32dev.build_unflags}
    build_flags = ${env:esp32dev.build_flags}
    lib_ignore = ${env:esp32dev.lib_ignore}
    upload_speed=2000000
    ```

### Pinout


  
  
Bottom View

|      |      |      |      |      |      |      |      |
| :--: | :--: | ---- | ---- | ---- | ---- | :--: | :--: |
| GND  | TXD  |      |      |      |      | RST  | GND  |
| IO27 | RXD  |      |      |      |      | SVP  | GND  |
| IO25 | IO22 |      |      |      |      | IO26 | SVN  |
| IO32 | IO21 |      |      |      |      | IO18 | IO35 |
| IO12 | IO17 |      |      |      |      | IO19 | IO33 |
| IO4  | IO16 |      |      |      |      | IO23 | IO34 |
| IO0  | GND  |      |      |      |      | IO5  | IO14 |
| IO2  | 5v0  |      |      |      |      | 3v3  | 3v3  |
| NC   | IO15 |      |      |      |      | IO13 | NC   |





Top View (component side)

|      |      |      |      |      |      |      |      |
| :--: | :--: | ---- | ---- | ---- | ---- | :--: | :--: |
| GND  | RST  |      |      |      |      | TXD  | GND  |
| GND  | SVP  |      |      |      |      | RXD  | IO27 |
| SVN  | IO26 |      |      |      |      | IO22 | IO25 |
| IO35 | IO18 |      |      |      |      | IO21 | IO32 |
| IO33 | IO19 |      |      |      |      | IO17 | IO12 |
| IO34 | IO23 |      |      |      |      | IO16 | IO4  |
| IO14 | IO5  |      |      |      |      | GND  | IO0  |
| 3v3  | 3v3  |      |      |      |      | 5v0  | IO2  |
| NC   | IO13 |      |      |      |      | IO15 | NC   |

### Board Pictures

<img src="TwilightLord-ESP32-16MB-v1.0r3-001.png" width="300"><br/>
<img src="TwilightLord-ESP32-16MB-v1.0r3-002.png" width="300"><br/>
<img src="TwilightLord-ESP32-16MB-v1.0r3-003.png" width="300"><br/>



