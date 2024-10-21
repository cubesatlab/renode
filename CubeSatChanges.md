# CubeSat Change Documentation
This file will be used for documenting any changes made as part of the CubeSat project.
At a minimum, it should list the files added/modified from the base Renode project. There should be additional documentation of the reasoning for any such changes if they are not well documented in the source of the file in question.

## Added:
- [`CubeSatChanges.md`](CubeSatChanges.md)
    - *This file. Added for above reasons*
- [`CubeSatSetup.md`](CubeSatSetup.md)
    - *Added to document setup instructions*


## Modified:

- [`.gitmodules`](.gitmodules)
    - *Changed Infrastructure submodule to point at cubesatlab implementation*


## Changes of Note in Bitcraze Fork
This is a compilation of files added/differing in the Bitcraze fork from the main repo. This may differ from the end product of the added/modified list above, as we may find there are different changes we do/don't need to make as a result of updates to the main repo or issues found in the bitcraze modifications. Each item will have a description of the changes made.
**Note:** This may not be complete, but was compiled by going through git logs.

### Added/Modified in main Renode repo

- [.gitmodules](https://github.com/bitcraze/renode/blob/crazyflie/.gitmodules)
    - *points the infrastructure submodule at the modified version*
- **platforms/**
    - [boards/cf2.repl](https://github.com/bitcraze/renode/blob/crazyflie/platforms/boards/cf2.repl)
        - *board definition for the CrazyFlie 2.1*
    - [cpus/stm32f405.repl](https://github.com/bitcraze/renode/blob/crazyflie/platforms/cpus/stm32f405.repl)
        - *cpu definition for the STM32F405.*
        - I think this potentially should have been based on Renode's existing stm32f4.repl
        - That approach would have it pulling in the SVD file: STM32F40x.svd.gz
        - These SVD files seem to be how Renode stores core board definitions.
        - It's possible thise deviation was due to issues encountered while using inherited defintions, but I feel it's worth trying to stick more tightly to the base implementation.
- **scripts/single-node/**
    - [crazyflie.resc](https://github.com/bitcraze/renode/blob/crazyflie/scripts/single-node/crazyflie.resc)
        - *cf2 board init script*
    - [crazyflie_test.resc](https://github.com/bitcraze/renode/blob/crazyflie/scripts/single-node/crazyflie_test.resc)
        - *cf2 board init script for testing*
        - Running either of these above scripts requires the existence of the crazyflie's software image
            - It expects it to be named cf2.elf, and to be living in the directory you're running from.
            - This will be important to keep in mind for any SPARK/Ada images we create.

### Added/Modified in Renode-Infrastructure

- **src/**
    - **Emulator/Peripherals/**
        - **Peripherals/**
            - [Peripherals.scproj](https://github.com/bitcraze/renode-infrastructure/blob/crazyflie/src/Emulator/Peripherals/Peripherals.csproj)
                - *Adds compile directives for other added files*
            - [I2C/STM32F4_I2C.cs](https://github.com/bitcraze/renode-infrastructure/blob/crazyflie/src/Emulator/Peripherals/Peripherals/I2C/STM32F4_I2C.cs)
                - *These changes appear largely debug related*
                - These changes need to be revisited to see if actually needed.
            - **Miscellaneous/**
                - [CF_Syslink.cs](https://github.com/bitcraze/renode-infrastructure/blob/crazyflie/src/Emulator/Peripherals/Peripherals/Miscellaneous/CF_Syslink.cs)
                    - *Adds the device for the CF Syslink*
                - [EEPROM_24AA64.cs](https://github.com/bitcraze/renode-infrastructure/blob/crazyflie/src/Emulator/Peripherals/Peripherals/Miscellaneous/EEPROM_24AA64.cs)
                    - *Adds EEPROM*
            - **Sensors/**
                - [BMI088_Accelerometer.cs](https://github.com/bitcraze/renode-infrastructure/blob/crazyflie/src/Emulator/Peripherals/Peripherals/Sensors/BMI088_Accelerometer.cs)
                    - *Adds CF2 accelerometer*
                - [BMI088_Gyroscope.cs](https://github.com/bitcraze/renode-infrastructure/blob/crazyflie/src/Emulator/Peripherals/Peripherals/Sensors/BMI088_Gyroscope.cs)
                    - *Adds CF2 gyroscope*
                - [BMP388.cs](https://github.com/bitcraze/renode-infrastructure/blob/crazyflie/src/Emulator/Peripherals/Peripherals/Sensors/BMP388.cs)
                    - *Adds CF2 barometer*
            - **Timers/**
                - [STM32_Basic_Timer.cs](https://github.com/bitcraze/renode-infrastructure/blob/crazyflie/src/Emulator/Peripherals/Peripherals/Timers/STM32_Basic_Timer.cs)
                    - *This feels like a hack to avoid using the normal STM32_Timer*
                    - I think it makes sense to try and make things work with the normal timer file, this feels weird and hacky to me.
                - [STM32_Timer.cs](https://github.com/bitcraze/renode-infrastructure/blob/crazyflie/src/Emulator/Peripherals/Peripherals/Timers/STM32_Timer.cs)
                    - *Changed to allow Word to DoubleWord translation, which seems probably reasonable?*
    - **UI/**
        - [CommandLineInterface.cs](https://github.com/bitcraze/renode-infrastructure/blob/crazyflie/src/UI/CommandLineInterface.cs)
            - *One line change to shell behavior, will have to see what breaks without it*
