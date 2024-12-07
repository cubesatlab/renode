# Renode Emulation for Crazyflie/CubedOS/CrazyCube testing

This fork of renode was created to facilitate the changes needed to emulate the Crazyflie 2 drone hardware for testing purposes. The idea being that eventually such testing could be integrated into the build pipeline so that there could be a basic level of assurance that a given firmware build will run successfully on the hardware without having to flash it to the drone each time. Much of the work on hardware emulation had already been tackled by a team at Lund University in Sweden. There is a [blog post](https://www.bitcraze.io/2021/04/successful-emulation/) about it on the Bitcraze site, as well as their [thesis paper](https://lup.lub.lu.se/luur/download?func=downloadFile&recordOId=9052405&fileOId=9052409) on the University's site.


## State of Work

Building the version of Renode as it exists in this repository should yield a platform that emulates most of the Crazyflie 2 hardware. There are some exceptions outlined in the above linked thesis paper, as well as one notable one I discovered during my work. The main body of work to this point centered around trying to get the standard Crazyflie firmware to boot all the way up in Renode.

There appears to be an issue with the [EEPROM Emulation](src/Infrastructure/src/Emulator/Peripherals/Peripherals/Miscellaneous/EEPROM_24AA64.cs), which in the Crazyflie firmware provided by Bitcraze handles persistent storage of configuration parameters (Like radio frequencies). For the standard firmware, this causes it to enter an infinite loop trying to read information from the EEPROM and not getting expected results.

For the aid of any potential future debuggin efforts, I will outline the call stack/function paths taken to get to this infinite looping. This will include links to the files in question on Github. I will reference lines in files as `file.c:123`, as GDB does for breakpoints.

- [system.c:188](https://github.com/bitcraze/crazyflie-firmware/blob/master/src/modules/src/system.c#L188) `commInit();`
- [comm.c:64](https://github.com/bitcraze/crazyflie-firmware/blob/master/src/modules/src/comm.c#L64) `paramInit();`
- [param_task.c:63](https://github.com/bitcraze/crazyflie-firmware/blob/master/src/modules/src/param_task.c#L63) `paramLogicStorageInit();`
- [param_logic.c:833](https://github.com/bitcraze/crazyflie-firmware/blob/master/src/modules/src/param_logic.c#L833) `storageForeach(PERSISTENT_PREFIX_STRING, persistentParamFromStorage);`
- [storage.c:184](https://github.com/bitcraze/crazyflie-firmware/blob/master/src/hal/src/storage.c#L184) `bool success = kveForeach(&kve, prefix, func);`
- [kve.c:135](https://github.com/bitcraze/crazyflie-firmware/blob/master/src/utils/src/kve/kve.c#L135) `size_t itemSize = kveStorageFindItemByPrefix(kve, FIRST_ITEM_ADDRESS, prefix, keyBuffer, &itemAddress);`
- [kve_storage.c:142](https://github.com/bitcraze/crazyflie-firmware/blob/master/src/utils/src/kve/kve_storage.c#L142) `while (currentAddress < (kve->memorySize - 3)) {`
    - This is the loop that never terminates. This is caused by the following call:
    - kve_storage.c:143 `kve->read(currentAddress, searchBuffer, 3);`
    - currentAddress will cease to advance, so the loop becomes infinite.
    - `kve->read` maps to [storage.c:67](https://github.com/bitcraze/crazyflie-firmware/blob/master/src/hal/src/storage.c#L67) `static size_t readEeprom(size_t address, void* data, size_t length)`

The way the EEPROM was implemented, it relies on DMA & I2C. Based on initial debugging, it seemed like maybe there was an issue with that implementation. In searching for potential solutions/other similar issues, I found [this issue](https://github.com/renode/renode/issues/616) on github.
Using the [STM32F4_I2C_Fixed.cs](src/Infrastructure/src/Emulator/Peripherals/Peripherals/I2C/STM32F4_I2C_Fixed.cs) and [STM32DMA_Fixed.cs](src/Infrastructure/src/Emulator/Peripherals/Peripherals/DMA/STM32DMA_Fixed.cs) I tested to see if this solved the boot issue with reading the EEPROM. Unfortunately, it caused the system to hang in an infinite loop at an earlier point.

- [system.c:188](https://github.com/bitcraze/crazyflie-firmware/blob/master/src/modules/src/system.c#L187) `systemInit();`
- [system.c:138](https://github.com/bitcraze/crazyflie-firmware/blob/master/src/modules/src/system.c#L138) `storageInit();`
- [storage.c:139](https://github.com/bitcraze/crazyflie-firmware/blob/master/src/hal/src/storage.c#L139) `kveDefrag(&kve);`
- [kve.c:69](https://github.com/bitcraze/crazyflie-firmware/blob/master/src/utils/src/kve/kve.c#L69) `itemAddress = kveStorageFindNextItem(kve, holeAddress);`
- [kve_storage.c:219](https://github.com/bitcraze/crazyflie-firmware/blob/master/src/utils/src/kve/kve_storage.c#L219) `while (currentAddress < (kve->memorySize - 3)) {`
    - This is the infinite loop with the _Fixed files being used for emulation.
    - [kve_storage.c:220](https://github.com/bitcraze/crazyflie-firmware/blob/master/src/utils/src/kve/kve_storage.c#L220) `kve->read(currentAddress, &header, sizeof(header));`
    - Once again we see the call into the EEPROM failing to advance the currentAddress.

Given that both the 'Fixed' and previous versions of the DMA and I2C emulation cause issues with EEPROM reads, it seems likely there is some sort of defect in the EEPROM emulation. That, or there is some other underlying issue in DMA/I2C that presents differently depending on whether the 'Fixed' version is present or not. This seems less likely, as there are almost certainly other initialization functions that utilize the DMA/I2C APIs.

### Reverting to original DMA/I2C implementations
To revert to using the original emulation of the DMA/I2C code, there are just a few updates that need to be made to the Renode platform file: [stm32f405.repl](platforms/cpus/stm32f405.repl)
- Remove `_Fixed` from lines 58, 64, 68, & 71
- Comment out or remove the `DMATransmit -> dma1@6` and `DMAReceive -> dma1@0` statements on lines 61 & 62

## Next steps

Given that the emulation seems largely functional with the exception of the above EEPROM issues, it should be possible to build and run a firmware image for the Crazyflie that works around that issue. Perhaps as a test, the values that would normally be read from EEPROM could be hardcoded into the firmware.
It would be interesting to build and run the [certyflie firmware](https://github.com/AdaCore/Certyflie/tree/ravenscar-cf-stable) and see where, if anywhere, it throws errors with the emulation. This likely wouldn't be a great test of the validity of hardware emulation though, as it is known to have some issues with the Crazyflie 2.1 hardware.

### Integration into build pipeline

The [crazyflie_test renode script](scripts/single-node/crazyflie_test.resc) has notes about redirecting uart/usart debug output to the console, as well as setting conditions to end the emulation based on debug output. Using this, one could tie into the build pipeline in a way that evaluated the debug output of a Renode emulator running the firmware, checking to see if self-tests passed, etc.
