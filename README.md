# Big Sur Hackintosh for the Fujitsu Lifebook S752
This is an OpenCore EFI configuration made by me for the Lifebook S752 series.

![Picture of my Lifebook running macOS Big Sur 11.7.10](https://files.randomtek.lol/pictures/BigSur-S752-min.png)

> [!WARNING]
> This config uses the MacBookPro11,1 SMBIOS, which may cause issues with the graphics and power management as the chipset is Ivy Bridge and this is using a Haswell model.\
> If you want a more compatible and stable config, you're limited to installing [Catalina](https://github.com/rndmtk/OpenCore_Fujitsu_Lifebook-S752/tree/catalina).

## Config Status 
What's working:
- Battery
- Audio and 3.5mm ports
- Graphics (Intel HD 4000)
- DVD-RW drive
- SDXC Reader
- Ethernet
- Wi-Fi (Atheros AR5B93)
- USB ports (including the ones on the docking station)
- Sleep
- Keyboard, trackpad and trackpoint
- Some Continuity features

What's not working:
- Cellular modem (no kext available for the MC8305)
- BlueTooth (there's no hardware for it obviously, either use a wireless combo card or a USB dongle to make it work)
- Extra Battery Info (cycle count is always on 0)
- VGA output (WhateverGreen doesn't support it)
- Middle trackpad buttons (breaks left/right clicking, reboot or sleep&wake to temporarily fix)
- FairPlay 1.x (iTunes Trailers tested, unsure about others) (audio works, but it's just a green screen)
- AirDrop (a wireless combo card is required, USB dongles won't fix it)
- The 5 buttons on top

Untested:
- SmartCard
- ExpressCard
- DisplayPort
- Modular bay battery (will most likely not work. If you have one, remove it for now)

## Preparation
> [!IMPORTANT]
> I am not responsible for broken installations, dead storage media, thermonuclear wars or you getting fired because a kernel panic occured mid work.
> Please do some research if you have any concerns about the configuration in this EFI before installing it.
> YOU are choosing to hackintosh. If you point the finger at me for messing up your computer I will laugh at you.

> [!NOTE]
> This config is using the Qualcomm Atheros AR5B93 Wi-Fi card.
> If you have a different wireless adapter, please remove the `IO80211HighSierra.kext` from both the OC/Kexts folder and `config.plist` and then have a look at https://dortania.github.io/Wireless-Buyers-Guide/Kext.html to see what kexts you need in order to get Wi-Fi working (assuming your wireless card is even supported).

### First, let's make the USB install media (minimum 2 GB)
> [!NOTE]
> If you're using macOS to make the install media, refer to [this guide](https://dortania.github.io/OpenCore-Install-Guide/installer-guide/mac-install.html) on downloading Big Sur.
> After you're done, skip to step 4. (requires 16 GB or above USB flash drive)
1. Download v1.0.4 of the [OpenCorePkg](https://github.com/acidanthera/OpenCorePkg/releases) as you'll need `macrecovery` to download the Big Sur recovery. (you also need python3 installed)
2. Format your USB flash drive to FAT32 as it's the only format that is bootable.
3. After downloading the OpenCorePkg, open the Utilities/macrecovery folder in your terminal/command prompt and run `python3 macrecovery.py -b Mac-42FD25EABCABB274 -m 00000000000000000 download` and then copy the `com.apple.recovery.boot` onto the root of the USB flash drive.
4. Download or clone this repository, make a folder named `EFI` on the root of the EFI partition on your USB drive and copy both the `OC` and `BOOT` folders from this repo in there.

### config.plist configuration
You're not done yet, you need to configure your laptop's serial number and other data.
1. Download and install [ProperTree](https://github.com/corpnewt/ProperTree) (you can use a text editor too, but this is more recommended)
> [!CAUTION]
> DO NOT USE ANY OTHER CONFIGURATORS! Those rarely respect OpenCore's configuration and even some will add Clover properties and corrupt plists!

2. You will also need [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) to generate the needed data.
3. Run `GenSMBIOS.bat` if you're using Windows or `python3 GenSMBIOS.py` if on *NIX.
4. Press `3` to generate a new SMBIOS and for the SMBIOS model, use `MacBookPro11,1`.
5. Before adding the values to the plist, you MUST copy the `Serial` value from GenSMBIOS and go to [Apple's Check Coverage page](https://checkcoverage.apple.com/), making sure that the serial is **NOT VALID**. Otherwise, generate and check again.
6. Open the `config.plist` located inside of the EFI/OC folder and navigate to `PlatformInfo > Generic`
7. Copy the generated data from GenSMBIOS into the required fields in the plist:
   + `Serial` gets copied to `SystemSerialNumber`
   + `Board Serial` gets copied to `MLB`
   + `SmUUID` gets copied to `SystemUUID`
   + `Apple ROM` gets copied to `ROM` (or you can use your NIC's MAC address. More info [here](https://dortania.github.io/OpenCore-Post-Install/universal/iservices.html#fixing-rom))
8. (Optional) Extra settings, adjust as you wish:
   * `Misc > Boot > PickerVariant` - Boot picker icons
      + `Acidanthera\Chardonnay` (currently set)
      + `Acidanthera\GoldenGate`
      + `Acidanthera\Syrah`
   * `NVRAM > Add > 4D1EDE05-38C7-4A6A-9CC6-4BCCA8B38C14 > DefaultBackgroundColor` - Background and boot logo color
      + `BFBFBF00` - Light Gray (currently set)
      + `00000000` - Syrah Black
   * `NVRAM > Add > 7C436110-AB2A-4BBB-A880-FE41995C9F82 > StartupMute` - Mute the boot chime
      + `00` - Unmuted (currently set)
      + `01` - Muted
9. Now with the modifications done, you can now save the file and eject your flash drive.

### BIOS settings
Now before you boot the USB, you should update your BIOS and use these settings for the best results:
- Advanced
   * Boot Configurations
      + Preboot Execution Environment (PXE): `[Disabled]`
   * Serial/Parallel Port Configurations
      + Serial Port: `[Disabled]`
      + Parallel Port: `[Disabled]`
      + ^(we have no use for these anyway lol)
   * Video Features
      + Display: `[Internal Flat Panel]`
   * Internal Device Configurations
      + Serial ATA Controller Mode Selection: `[AHCI]`
      + (everything else can be set to `[Enabled]`)
   * CPU Features
      + Multi-core: `[Enabled]`
      + HT Technology: `[Enabled]`
      + SpeedStep(R) Technology `[Enabled]`
      + Virtualization Technology `[Enabled]`
      + Intel(R) VT-d: `[Enabled]` (this is safe to enable as a DMAR patch is already included)
   * USB Features:
      + (Set everything to `[Enabled]`)
   * Intel(R) Management Engine Configurations
      + Intel(R) AT Suspend Mode: `[Disabled]` (Anti-Theft is discontinued anyway so it's better to keep it disabled)
- Boot
   * Set your USB flash drive to be the first boot entry
- Exit
   * Exit Saving Changes
      + Save configuration changes and exit now? `[Yes]`
## Installation
> [!NOTE]
> If you want to dual boot with Windows, shrink your Windows NTFS (or any other volume, if any) and create an ExFAT placeholder.

Now at first glance, you might see that there's only the Windows entry showing up or even no boot entries are showing up at all. Don't worry though, all you have to do is press Space to unhide the other entries and select your macOS installer to boot. (could be labeled "EFI" or any other name you gave your flash drive)

Once booted, you'll be greeted to choose a language. After that, you'll be greeted with the macOS Recovery screen. Here's what to do now:
1. Partition your drive. Open Disk Utility and create an APFS partition.
   * Go to your menu bar and select View > Show All Devices.
   * For dual booters, select the placeholder partition you created earlier and then Erase it with APFS. Give it any name you want.
   * If you want to only use macOS, select the entire drive you want to use and Erase with APFS.
   * After you're done, exit Disk Utility.
2. Now go to "Reinstall macOS" and install it like usual. It's going to take a while to install macOS, depending on the type of installation you downloaded (Online or Offline).

## Finishing up
Aaand congrats, you successfully installed macOS Big Sur on your Lifebook!
Although there's one problem: Your flash drive has to be plugged in all the time in order to make macOS bootable. It's not hard to fix though!

### Moving the EFI onto your laptop
1. First, open up Safari and download [Hackintool](https://github.com/benbaker76/Hackintool/releases/).
2. Move the Hackintool app to your Applications folder in the sidebar.
3. You can now open up Hackintool from Spotlight (Cmd + Space, Cmd being either Win or Super) or via the Launchpad in the dock.
   * If you get a warning that the app is not trusted, simply close the pop-up and open System Preferences > Security & Privacy > General and then Open Anyway.
4. With Hackintool open, go over at the top and select Disks. Mount both your USB flash drive's and your main drive's EFIs by clicking on the triangles that are pointing down.
5. Open Finder, select your flash drive and copy the EFI folder (Cmdd + C or Right Click > Copy).
6. Now open your main drive's EFI partition and paste the EFI folder in there. If it asks you to replace something inside the BOOT folder, select Replace - it's most likely the BOOTx64.efi that needs to be replaced anyway.
7. After you're done shut down your laptop, unplug the flash drive, power it on and now you should be able to boot macOS without it.

## Ending
Well, you might as well go ahead and enjoy your new hackintosh now. :3

If you have any problems or need help, you can make an issue in this repository.
