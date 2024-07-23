# zmk-config for fingerpunch

This repository serves as an example for extending the base fingerpunch keyboards to add your own keymap and extend/override the config and/or devicetree.

Steps to create a similar set up:
1) Go to https://github.com/zmkfirmware/unified-zmk-config-template and click "Use this template", and "Create a new repository"
2) Choose a name for your repository
3) Copy the contents of the `config/west.yml` in this repository into the same file in your new repository. Please review the comments in that file to see if you want to make any changes.
4) Copy the keymap of your choice from the appropriate [zmk-fingerpunch-keyboards shield](https://github.com/sadekbaroudi/zmk-fingerpunch-keyboards/tree/main/boards/shields) for your keyboard
5) Modify your keymap
6) Update `build.yml` in your repo to build the keyboard with the controller of your choice. You can use any zmk supported controller, or one of the fingerpunch controllers found in the [zmk-fingerpunch-controllers boards](https://github.com/sadekbaroudi/zmk-fingerpunch-controllers/tree/main/boards/arm).
7) Note: You can look at the [zmk-fingerpunch-keyboards build.yml](https://github.com/sadekbaroudi/zmk-fingerpunch-keyboards/blob/main/build.yaml) to see all sorts of tested build combinations

Commit your changes, and wait for the github actions run to complete. Your firmware is ready!

> [!NOTE]  
> This example also includes adding a cirque trackpad, mostly to highlight a more complex example using the `ffkb_v2.conf` and `ffkb_v2.overlay`, or the `rock_on_v1.conf` and `rock_on_v1.overlay`. It also includes using Pete's `cirque-input-module`, defined in the `config/west.yml`. If you're simply overriding a keymap, and otherwise building a fingerpunch board, with or without a [VIK module](https://github.com/sadekbaroudi/vik), then you don't need any of these things. This will likely be the case for most people using this guide.

## example local build string

### ffkb

`west build --pristine -b "nice_nano_v2" -- -DSHIELD="ffkb_v2" -DZMK_CONFIG='/home/sadek/zmk-config-fp-example/config' -DZMK_EXTRA_MODULES='/home/sadek/zmk-fingerpunch-keyboards;/home/sadek/zmk-fingerpunch-controllers;/home/sadek/zmk-fingerpunch-vik;/home/sadek/cirque-input-module'`

### rock on

`west build --pristine -b "nice_nano_v2" -- -DSHIELD="rock_on_v1" -DZMK_CONFIG='/home/sadek/zmk-config-fp-example/config' -DZMK_EXTRA_MODULES='/home/sadek/zmk-fingerpunch-keyboards;/home/sadek/zmk-fingerpunch-controllers;/home/sadek/zmk-fingerpunch-vik;/home/sadek/cirque-input-module'`