# Acer Aspire A315-54 Hackintosh 

[![Status](https://img.shields.io/badge/Status-Maintained-blue.svg)](https://github.com/bahaedin/Acer-Aspire-A315-54-Hackintosh)
[![OpenCore](https://img.shields.io/badge/OpenCore-0.8.3-blue.svg)](https://github.com/acidanthera/OpenCorePkg)
[![macOS](https://img.shields.io/badge/macOS-12.5-brightgreen.svg)](https://www.apple.com/macos/monterey)

:warning: **WARNING:**
This OpenCore configuration is optimized for my specific hardware, so please read carefully before doing anything or use it only as a reference. I suggest you to refer to [Dortania](https://dortania.github.io/OpenCore-Install-Guide/) for anything else. 

:information_source: **NOTE:** This Guide is a **fork** of [A315-54K-34P6-OpenCore](https://github.com/velickovicdj/A315-54K-34P6-OpenCore) with some modification to adapt it to the **10th gen** of this laptop

![System info](/Screenshots/systeminfo.png "System info")


## Hardware:

* **CPU:** Intel® Core™ i5-10210U
* **GPU:** Intel Coffee Lake GT2 (UHD620)
* **RAM:** 1x 16GB Crucial DDR4-2666MHz
* **Laptop Model:** Acer Aspire A315-54
* **Audio Codec:** Realtek ALC255
* **Ethernet Card:** Realtek RTL8111/8168
* **Wifi/BT Card:** ~~Atheros QCA9377~~ Replaced By Broadcom BCM94352Z
* **Storage:** x2 SSD 256GB
  - SK hynix BC501 256GB NVME (Windows10)
  - Micron M600 256GB M.2 (macOS12)
* **Touchpad:** Synaptic I2C (SYNA7DB5)
* **Display:** 15.6" 1920x1080 IPS
* **BIOS revision:** insyde V1.11
* **Bootloader :** OC 0.8.3

### What Works:

- [x] CPU Power Management
- [x] UHD620 QE/CI Full Graphics Acceleration
- [x] Display Brightness
- [x] Audio (internal speakers, headphone jack, mic)
- [x] HDMI video & audio (up to 4K 30Hz because of HDMI 1.4 limitations)
- [x] Webcam
- [x] Ethernet
- [x] Wifi/BT Native
- [x] Sleep/Wake
- [x] Touchpad With all gestures
- [x] Keyboard (including all media keys)
- [x] All USB ports (USB-A 2.0 + 3.1)
- [x] Battery indicator
- [x] iCloud Drive
- [x] AirDrop (Two Ways)
- [x] iMessage
- [x] FaceTime

### Not Working:

- [ ] DRM Content

### Not Tested:
- [ ] Handoff
- [ ] Continuity
- [ ] AirPlay

### :information_source: **NOTE:** Do This Before installing macOS

## BIOS:
- [x] BIOS MODE: **UEFI**
- [ ] SECURE BOOT: **Disabled**
- [x] FAST BOOT: **Enabled**
- [x] Vt-d: **Enabled** 
  - (Do not forget to set `DisableIoMapper` to `true` under `Kernel` -> `Quirks` in config.plist)
- [ ] CFG Lock (MSR_E2): **Disabled** 
  - (if you cannot find this option then set `AppleXcpmCfgLock` to `true` under `Kernel` -> `Quirks` in config.plist)
- [x] DVMT Pre-Allocated Memory: 64MB
  - (If you cannot find this option and/or you are not sure if you pre-allocated memory is >= 64MB then you should patch your VRAM ([see the details below](https://github.com/bahaedin/Acer-Aspire-A315-54-Hackintosh#vram-patching))
- [x] SATA MODE: **AHCI**
  - Launch BIOS by tapping the `F2` key repeatedly right after booting.
  - When in BIOS, go to `Advanced` and type `CTRL` + `S`  it will display a (Hidden) option change it from **Optain** to **AHCI**
  
## Config.plist:
<details>
<summary><h3>SSDTs</h3></summary>
<br>
  
Refer to [Dortania Guide](https://dortania.github.io/OpenCore-Install-Guide/ktext.html#laptop) i will only describe the SSDTs that are not essential for functioning but are present in my EFI.

|**SSDT**          |**Description**                 			 																																			   |
|------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|**SSDT-PLUG**	   |***Crucial*** 		 																																	   								   |
|**SSDT-PNLF**	   |Fixes backlight.				 																																						   |
|**SSDT-XOSI**     |Enable GPI0 for fixing I2C trackpad [see](https://dortania.github.io/Getting-Started-With-ACPI/Laptops/trackpad.html) and [**add patches below**](https://github.com/bahaedin/Acer-Aspire-A315-54-Hackintosh#ssdt-xosi). 																																					   |
|**SSDT-ALS0**     |Provides macOS with a fake Ambient Light Sensor device (ALS), so it could store the current brightness level and keep it after reboots.		 		 									   |
|**SSDT-DMAC**     |Provides macOS with a fake Direct Memory Access Controller (DMAC), because the device is present in any Intel-based Mac. The necessity for this SSDT is unknown, consider it as "cosmetic".|
|**SSDT-EC-USBX**  |***Crucial***	 																																			   							   |
|**SSDT-SBUS-MCHC**|Fixes AppleSMBus support in macOS [see](https://dortania.github.io/Getting-Started-With-ACPI/Universal/smbus.html).				 		 																																   |
|**SSDT-MEM2** 	   |Makes the iGPU use MEM2 instead of TMPX, so the IOAccelMemoryInfoUserClient is loaded correctly.			 		 																	   |
|**SSDT-GPRW**     |Fixes instant wake on USB/power state change [see](https://dortania.github.io/OpenCore-Post-Install/usb/misc/instant-wake.html) and [**add patches below**](https://github.com/bahaedin/Acer-Aspire-A315-54-Hackintosh#ssdt-gprw).																																			   |
|**SSDT-AWAC**     |Re-enable the old RTC clock that is compatible with macOS [see](https://dortania.github.io/Getting-Started-With-ACPI/Universal/awac.html).																																			   |
  
<details>
<summary><h3>Patches</h3></summary>
<br>
  
:information_source: **NOTE:** Add this patches to `ACPI` -> `Patch`

![ACPI Patches](/Screenshots/ACPI%20Patches.PNG "ACPI Patches")
 
<details>
<summary><h3>SSDT-XOSI</h3></summary>
<br>  

| **Comment** | **String** | **Change _OSI to XOSI** |
| :---         |     :---:      |          :---: |
| Enabled   | Boolean     | YES    |
| Count     | Number       | 0      |
| Limit   | Number     | 0    |
| Find     | Data       | 5f4f5349      |
| Replace     | Data       | 584f5349      |
  
</details>
  
<details>
<summary><h3>SSDT-GPRW</h3></summary>
<br>  

| **Comment** | **String** | **Change Method(GPRW,2,N) to XPRW** |
| :---         |     :---:      |          :---: |
| Enabled   | Boolean     | YES    |
| Count     | Number       | 0      |
| Limit   | Number     | 0    |
| Find     | Data       | 4750525702      |
| Replace     | Data       | 5850525702      |
  
</details>  
</details>
</details>
<details>
<summary><h3>DeviceProperties</h3></summary>
<br>

<details>
<summary><h3>iGPU Settings:</h3></summary>
<br>

![iGPU Settings](/Screenshots/iGPU%20Settings.png "iGPU Settings")

Add the following under `DeviceProperties` -> `Add` -> `PciRoot(0x0)/Pci(0x2,0x0)`

|**Key**					 |**Type**|**Value**|
|----------------------------|------- |---------|
|**AAPL,ig-platform-id**|Data	  |0900A53E	|
|**device-id**|Data	  |C49B0000	|

<details>
<summary><h3>VRAM Patching:</h3></summary>
<br>

In some cases where you cannot set the **DVMT-prealloc** of these cards to **64MB** higher in your **UEFI Setup**, you may get a **kernel panic**. Usually they're configured for **32MB** of **DVMT-prealloc**, in that case **you have to add these values to your iGPU Properties**.

|**Key**					 |**Type**|**Value**|
|----------------------------|------- |---------|
|**framebuffer-patch-enable**|Data	  |01000000	|
|**framebuffer-stolenmem**   |Data    |00003001	|
|**framebuffer-fbmem**	     |Data	  |00009000	|

</details>  
</details>

<details>
<summary><h3>Audio Settings:</h3></summary>
<br>

![Audio Settings](/Screenshots/Audio%20Settings.png "Audio Settings")

Add this `PciRoot(0x0)/Pci(0x1F,0x3)` with the child `layout-id` under `DeviceProperties` -> `Add`
  
|**Key**					 |**Type**|**Value**|
|----------------------------|------- |---------|
|**layout-id**|Data	  |56000000	|
  
Check AppleALC [Supported Codecs](https://github.com/acidanthera/AppleALC/wiki/Supported-codecs#:~:text=Realtek-,ALC255/ALC3234,-layout%203%2C%2011)

With the ALC255, we get the following
> layout 3, 11, 12, 13, 15, 17, 18, 20, 21, 27, 28, 29, 30, 31, 66, 71, 82, **86**, 96, 99, 100, 255
  
I chose layout `86` because it works well with my hack then I [converted it to hexadecimal](https://www.rapidtables.com/convert/number/decimal-to-hex.html) = `56`

Note that the final HEX/Data value should be 4 bytes in total (ie. `56 00 00 00` )
  

</details>

:information_source: **NOTE:** All other entries than `PciRoot(0x0)/Pci(0x2,0x0)` and `PciRoot(0x0)/Pci(0x1F,0x3)`, under `DeviceProperties` -> `Add` are purely cosmetic and you can safely remove them if you wish so.

</details>

<details>
<summary><h3>Kexts:</h3></summary>
<br>
  
|**Kext**         								   |**Description**                 			 																										 									   									|
|--------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|**Lilu**	   									   |***Crucial***	 																						 									   																				|
|**VirtualSMC**	   								   |***Crucial***			 																					 									   																			|
|**SMCBatteryManager**     						   |Measuring battery readouts on laptops. 																																			 									   											|
|**SMCLightSensor**     						   |Supplement to fake ambient light sensor device mentioned in ACPI.		 		 										   																					 				|
|**SMCProcessor**     							   |Allows preciser measurement of the CPU.		 		 										   																					 									   		|
|**WhateverGreen**     							   |***Crucial***																									 									   																		|
|**AppleALC**  									   | Enabling native macOS HD audio . 																				   											|
|**RealtekRTL8111**  									   |Ethernet Card Driver. 																				   											|
|**VoodooPS2Controller + VoodooPS2Keyboard plugin**|Keyboard Driver.				 		 																													 									   											|
|**VoodooI2C + plugins**   						   |Add support for I2C bus devices on macOS.			 		 																	   													 									   											|
|**VoodooI2CHID**     		   					   |Enabling touchpad gestures.																																			 									   											|
|**CPUFriend + CPUFriendDataProvider**   						   |Power Management [see](https://dortania.github.io/OpenCore-Post-Install/universal/pm.html).			 		 																	   													 									   											|
|**BrcmFirmwareData.kext**     	   						   |Used for uploading firmware on Broadcom Bluetooth chipset, required for all non-Apple/non-Fenvi Airport cards..										|
|**BrcmPatchRAM3.kext**						   |macOS driver which applies PatchRAM updates for Broadcom RAMUSB based devices.									   							|
|**BlueToolFixup.kext**						   |Patches the macOS 12+ Bluetooth stack to support third-party cards.		 									   	   										|
|**NVMeFix**     		   						   |Optimizes power and energy consumption on non-Apple SSDs.																																			 						|
|**USBPorts**     		   						   |**I mapped USB ports specifically for this laptop model**. If your model is different, you should remove this kext and do your [USB mapping](https://dortania.github.io/OpenCore-Post-Install/usb/intel-mapping/intel.html).|  
|**HoRNDIS**     		   						   |Driver for macOS that allows you to use your Android phone's native USB tethering mode to get Internet access.																																			 						|
|**BrightnessKeys**     		   						   |Enabling Function `(Fn)` keys on laptop.																																			 						| 
|**RestrictEvents**     		   						   |Blocking unwanted processes causing compatibility issues & unlocking some features [see](https://github.com/acidanthera/RestrictEvents).																																			 						| 
|**FeatureUnlock**     		   						   |Enabling Sidecar, NightShift, AirPlay to Mac, Universal Control.																																			 						|
  
</details>

<details>
<summary><h3>Boot-args:</h3></summary>
<br>
  
|**Boot-args**					 |**Description**|
|----------------------------|------- |
|`debug=0x100`|Disables macOS's watchdog which helps prevents a reboot on a kernel panic.	  |
|`keepsyms=1`|This is a companion setting to debug=0x100 that tells the OS to also print the symbols on a kernel panic.	  |
|`swd_panic=1`|Avoids issue where going to sleep results in a reboot, this should instead give us a kernel panic log.	  |
|`igfxfw=2`| boot argument to force loading of Apple GuC firmware (improves IGPU performance).	  |
|`igfxrpsc=1`|boot argument to enable RPS control patch (improves IGPU performance).	  |

</details>

<details>
<summary><h3>PlatformInfo:</h3></summary>
<br>
  
PlatformInfo section of the config.plist is left empty for security reasons. You need to generate your own SMBIOS data and change the corresponding values (`MLB`, `ROM`, `SystemSerialNumber`, `SystemUUID`) under PlatformInfo in config.plist. Luckily, [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) can take care of that for you.

Check This [guide](https://dortania.github.io/OpenCore-Post-Install/universal/iservices.html#using-gensmbios) for better understanding..
  
</details>

## Miscellaneous

<details>
<summary><h3>OpenCore beauty treatment</h3></summary>
<br>

This EFI is aesthetically configured to include OpenCore's GUI.
I also applied theme for OpenCanopy. You can change the desired theme at any time by changing the `PickerVariant` value in config.plist with respect to the path of the theme in the OC/Resources/Image folder (E.g. `PickerVariant` value `Acidanthera/GoldenGate` corresponds to the path OC/Resources/Image/Acidanthera/GoldenGate).

I prefer skipping the boot picker and going straight to macOS, but if you wish to have it on every boot set `ShowPicker` to `true` under `Misc` -> `Boot` in config.plist.

**TIP 1:** You can slightly speed up the boot time by setting `ConnectDrivers` to `false` under `UEFI` in config.plist, but you'll have to give up on the fancy boot-chime.

**TIP 2:** Even if `ShowPicker` is set to `false`, you can still access the OpenCore boot picker by holding the escape key when booting, just make sure that `PollAppleHotKeys` is set to `true`.

</details>

## Screenshots:

![About This Hack](/Screenshots/About%20This%20Hack.png "About This Hack")

![Peripherals](/Screenshots/Peripherals.png "Peripherals")

![PCIe Devices](/Screenshots/PCIe%20Devices.png "PCIe Devices")

![USBmap](/Screenshots/USBmap.png "USBmap")

![Power info](/Screenshots/Power%20info.png "Power info")

## Credits

[**Acer**](http://acer.com) for the laptop.

[**Apple**](http://apple.com) for the macOS.

[**velickovicdj**](https://github.com/velickovicdj) for his detailed guide of Acer Aspire 3 7th gen.

[**Dortania**](https://dortania.github.io/OpenCore-Install-Guide/) for the great guides.

[**Acidanthera**](https://github.com/acidanthera) for awesome kexts and first-class support for hackintosh enthusiasts.

[**ic005k**](https://github.com/ic005k/OCAuxiliaryTools) for his awesome tool **OCAuxiliaryTools**

[**Alexandre Daoud**](https://github.com/alexandred) for VoodooI2C kext and making it work with the trackpad.
