My QMK
======

*"Keyboard is the [DZ60RGB ANSI V2](https://kbdfans.com/products/dz60rgb-ansi-mechanical-keyboard-pcb)"*

# Install

## Download Repository

Git clone your `qmk_firmware` fork repository:

```
git clone --branch "ventto/dz60rgb-ansi-v2" --recursive git@github.com:Ventto/qmk_firmware.git
cd qmk_firmware
```

## Create A Docker Container

We create a Docker container for using *qmk*, *avrdude* and *dfu-xxx* tools along with
downgraded *avr-gcc* version (see [this issue](https://github.com/qmk/qmk_firmware/issues/10764) for further reading).

Create a Docker container and run it as follows:

```
sudo docker run --privileged -d -ti \
                -v /dev:/dev \
                -v /etc/udev/rules.d:/etc/udev/rules.d \
                -v ${PWD}:/qmk_firmware \
                --name "qmk_firmware" \
                -w "/qmk_firmware" \
                "qmkfm/base_container" \
                /bin/bash

sudo docker exec -ti qmk_firmware bash
```

>  `--privileged -v /dev:/dev`: mount the `/dev` directory to access any USB and serial devices.
>
> `-v /etc/udev/rules.d:/etc/udev/rules.d`: mount `/etc/udev/rules.d:/etc/udev/rules.d` directory to let *qmk* install new Udev rules.
>
> `-v ${PWD}:/qmk_firmware`: mount your own `qmk_firmware` repository.

## Setup QMK

* Set the `qmk_firmware` directory automatically and verify that everything is correctly installed:

```
root@docker:/qmk_firmware# qmk setup
```

* Add your custom keyboard model to the QMK configuration file into `~/.config/qmk/qmk.ini` (avoiding to write `-kb <keyboard> -km <keymap>` *qmk* options to the command line):

```
root@docker:/qmk_firmware# qmk config \
    user.keyboard="dztech/dz60rgb_ansi/v2" \
    user.keymap=ventto
```

* Reload Udev rules before plugging-in anything:

```
udevadm control --reload-rules
```

**See also:**

- https://beta.docs.qmk.fm/tutorial/newbs_getting_started

# Configure

This section deals with the keyboard configuration.

## Configure Keymap

* Go to [QMK Configurator](https://config.qmk.fm/#/dztech/dz60rgb_ansi/v2/LAYOUT_60_ansi)
* Select the keyboard model and the layout
* Customize
* Flash with your `qmk_firmware` source:
  - Download `.json` keymap file (by clicking on the EXPORT button)
  - Convert `qmk json2c file.json > keymap.c`
  - Create a custom keymap directory in your source
* (or) Flash directly:
  - Download `.hex` firmware file (by clicking on the FIRMWARE button)
  - Move this file in your docker volume
  - Flash with

```
mkdir keyboards/dztech/dz60rgb_ansi/keymaps/ventto/
```

- Place the `keymap.c` in the `keyboards/<maker>/<model>/keymaps/<mykeymap>/`


# Flash

### Build & flash the firmware

Sources:

- https://beta.docs.qmk.fm/tutorial/newbs_building_firmware
- https://beta.docs.qmk.fm/tutorial/newbs_flashing

Build the firmware as follows:

```
root@docker:/qmk_firmware# qmk compile
```

Plug the keyboard and flash the `.hex` firmware file on it as follows:

```
root@docker:/qmk_firmware# qmk flash
```

### Flash the bootloader

Sources:

- https://beta.docs.qmk.fm/using-qmk/guides/keyboard-building/isp_flashing_guide
- https://drop.com/talk/9635/how-to-flash-your-planck-light-keyboard-via-isp

Requirements:
* AVR programmer (ex: https://www.pololu.com/product/3172)
* *avrdude* binary installed
* pre-built bootloader file (ex: `qmk_firmware/utils/`)

Flash the `.hex` bootloader file on a keyboard's Atmega32U4 mcu (hence `-p m32u4`) with the `/dev/ttyACM0` device's SPI interface:

```
root@docker:/qmk_firmware# avrdude -P /dev/ttyACM0 \
                            -c avrispv2 \
                            -p m32u4 \
                            -U flash:w:util/bootloader_atmega32u4_1.0.0.hex:i
```

# Test

The purpose is to know if the keyboard is fully operational according the following check list:

- All keyboard's hot switches work (check with this [online tool](https://config.qmk.fm/#/test));
- KB still works after a sleep mode (try 3x);
- KB wakes up the computer in sleep mode, if it doesn't work:
  - check the BIOS options
  - check ACPI wakeup settings (ex: `/proc/acpi/wakeup` file);
  - check the device wakeup settings in the Sysfs (ex: `grep . /sys/bus/usb/devices/*/power/wakeup`)
  - check with a non-custom keyboard;
  - check the USB cable.
- Flash the keyboard with the default configuration on [config.qmk.fm](https://config.qmk.fm/);
- Flash the keyboard with a custom configuration;
