# RetrospectorOS

A customized Android 16 ROM for the Sidephone SP-01 Founders Edition, based on LineageOS 23.2 GSI. Built and maintained by RetroSpector.

### What's included
- Custom RetrospectorOS boot animation
- Library app preinstalled
- Jaketype keyboard preinstalled
- Mint Mobile APN pre-configured
- Universal VoLTE / VoWiFi carrier configuration
- MTK RIL fix for stable LTE data
- MTK IMS service included
- 240dpi display density
- OTA update support

---

## Choose Your Variant

| Variant | Description |
|---|---|
| **GAPPS** | Includes Google Play Services. |
| **VANILLA** | No Google services. Use Library or sideload apps. Recommended for privacy-focused users. |

Both variants include the same fixes and preinstalled apps.

---

**NOTICE:** Flashing custom firmware and unlocking the bootloader will **void your manufacturer's warranty** and prevent future official Sidephone software updates. This is intended for **advanced power users only.**

## Disclaimer

Flashing custom software carries inherent risks and will completely wipe your personal data. Back up everything important before proceeding. I am not responsible for any damage to your device.

---

## Prerequisites

You will need the following before starting:

**Two USB flash drives:**
- **Drive A** — 30GB or larger (for the Re LiveDVD Linux environment)
- **Drive B** — 50GB or larger, formatted exFAT (for storing your backup and ROM files)

**Software to download and copy to Drive B before starting:**
- [Re LiveDVD ISO](https://github.com/bkerler/mtkclient) — the custom Linux environment used for MTK Client
- [Rufus](https://rufus.ie) — to flash the ISO onto Drive A
- [RetrospectorOS image](https://github.com/tylerreed98/RetrospectorOS/releases) — download your preferred GAPPS or VANILLA variant and extract the `.img` from the `.xz` file
- [Magisk APK](https://github.com/topjohnwu/Magisk/releases) — latest version

---

## Phase 0: Create the Live USB Environment

1. Insert **Drive A** into your Windows PC.
2. Open **Rufus**, select Drive A as the target, select the **Re LiveDVD ISO**, and click **Start**.
3. When Rufus finishes, safely eject Drive A.
4. Copy your RetrospectorOS `.img` file and Magisk APK onto **Drive B**.

---

## Phase 1: Boot into the Live Environment

1. Shut down your Windows PC.
2. Insert **Drive A** (Re LiveDVD) and **Drive B** (your files).
3. Power on your PC and enter your BIOS/Boot Menu (usually F2, F12, or Delete at startup).
4. Select Drive A as the boot device. You may need to **disable Secure Boot** in BIOS settings first.
5. When the Linux desktop loads, Drive B should mount automatically.
6. If prompted for a password at any point, the password is `user`.

---

## Phase 2: Full System Backup & Bootloader Unlock (MTK Client)

### Step 1: Launch MTK Client

Open a terminal and run:

```
cd /opt/mtkclient
python mtk_gui.py
```

### Step 2: Full System Backup

1. Power off your Sidephone completely. Do not connect it yet.
2. Press and hold **Volume Up + Volume Down** on the phone, then plug the USB cable into your PC.
3. Release the buttons once MTK Client detects the device and loads the partition list.
4. Go to the **Read partitions** tab.
5. Select **all** partitions and click **Read**.
6. When the file browser opens, navigate to **Drive B**, create a folder called `Sidephone_Backup`, and select it.
7. Wait for the backup to complete — this takes several minutes.

> **Important:** Do not save the backup to the Live OS desktop. It will be lost when you shut down the live environment. Always save to Drive B.

### Step 3: Unlock the Bootloader

1. Still connected in BROM mode, go to the **Erase partitions** tab.
2. Check `metadata`, `userdata`, and `md_udc`, then click **Erase**.
3. Go to the **Flash Tools** tab and click **Unlock bootloader**.

### Step 4: Boot Back to Stock

Disconnect the USB cable. Hold the power button for 10 seconds to force-reboot the phone. When you see *"system is changed, can not be trusted"*, press the power button once to continue booting into stock Android.

### Step 5: How to Restore Stock (If Anything Goes Wrong)

If you ever end up in a bootloop or want to go back to the factory OS:

1. Boot back into the Re LiveDVD, plug in Drive B, and launch MTK Client.
2. Enter BROM mode (power off phone → hold Vol Up + Vol Down → plug in USB).
3. Go to **Write partitions**.
4. Find `super` in the list, select `super.bin` from your `Sidephone_Backup` folder on Drive B, and click **Write**.

---

## Phase 3: Root with Magisk (Skip if you want an unrooted system in the non Gaaps version, for Gapps you will need this to get Gaaps functional)

1. Complete the initial stock Android setup wizard and connect to Wi-Fi.
2. Enable **Developer Options**: go to **Settings → About phone** and tap **Build Number** 7 times.
3. Enable **USB Debugging** in Developer Options.
4. Copy the Magisk APK from Drive B to your phone's internal storage and install it.
5. Open a terminal in the live Linux environment and push your stock boot image to the phone:

```
adb push /media/DriveB/Sidephone_Backup/boot.bin /sdcard/Download/
```

6. Open the **Magisk** app on your phone → tap **Install** → **Select and Patch a File** → select `boot.bin` → tap **Let's Go**.
7. Pull the patched file back to Drive B:

```
adb shell "mv /sdcard/Download/magisk_patched-*.img /sdcard/Download/magisk_patched.img"
adb pull /sdcard/Download/magisk_patched.img /media/DriveB/magisk_patched.img
```

8. Reboot to bootloader:

```
adb reboot bootloader
```

9. Verify fastboot connection:

```
fastboot devices
```

10. Flash the patched boot image:

```
fastboot flash boot /media/DriveB/magisk_patched.img
fastboot reboot
```

**When you see system is changed, can not be trusted, press the power button once to continue booting**
11. When the phone boots, open Magisk and complete setup if prompted. It may reboot once more automatically.

---

## Phase 4: Flash RetrospectorOS

### Step 1: Disable AVB Verification

```
adb reboot bootloader
fastboot --disable-verity --disable-verification flash vbmeta /media/DriveB/Sidephone_Backup/vbmeta.bin
fastboot --disable-verity --disable-verification flash vbmeta_system /media/DriveB/Sidephone_Backup/vbmeta_system.bin
fastboot --disable-verity --disable-verification flash vbmeta_vendor /media/DriveB/Sidephone_Backup/vbmeta_vendor.bin
```

### Step 2: Enter Fastbootd

```
fastboot reboot fastboot
```

**When you see system is changed, can not be trusted, press the power button once to continue booting**
Wait for the fastbootd screen to appear, then verify:

```
fastboot devices
```

### Step 3: Flash the System Image

Replace the filename with whichever variant you downloaded:

```
fastboot delete-logical-partition product
fastboot flash system /media/DriveB/RetrospectorOS-v1.0.0-GAPPS.img
```

*This will take several minutes.*

### Step 4: Wipe Data

```
fastboot reboot bootloader
fastboot erase userdata
fastboot format:ext4 userdata
```

### Step 5: Reboot

```
fastboot reboot
```

When you see *"system is changed, can not be trusted"* press the power button once.

**First boot takes 10–20 minutes.** The boot animation will play during this time — do not restart the phone. This is completely normal.

---

## Phase 5: First Boot Setup

1. Complete the LineageOS setup wizard. Connect to Wi-Fi when prompted.
2. **Mobile data:** After setup you may need to manually enable mobile data. Go to **Settings → Network & Internet → SIMs → [Your SIM]** and toggle Mobile Data on. This is a known v1 issue and will be fixed in v2.
3. **Your APN is pre-configured for all major US carriers.** Mobile data should connect automatically once the toggle is enabled. If data does not connect, go to **Settings → Network & Internet → SIMs → Access Point Names** and verify your carrier's APN is selected.

---

## Phase 6: Set Up Jaketype Keyboard

Jaketype is pre-installed on RetrospectorOS. To enable it:

1. Go to **Settings → System → Languages & Input → On-screen keyboard**.
2. Tap **Manage on-screen keyboards**.
3. Enable **Jaketype**.
4. Set it as your default keyboard.

*Note: Jaketype cannot be automatically set as default in v1. This will be fixed in v2.*

---

## Post-Install Notes

### Root Access
Because we flashed the Magisk-patched boot image before RetrospectorOS, root is already active. Open the Magisk app to confirm.

### OTA Updates
Future RetrospectorOS updates can be installed directly from the phone without reflashing. Go to **Settings → System → Updater** to check for and install updates.

### VoLTE & VoWiFi
The toggle should appear in **Settings → Network & Internet → SIMs** once your carrier's IMS stack registers. If it does not appear, toggle airplane mode off and on and wait a minute.

### Signal Drops
Brief signal drops when the screen turns off are normal MTK fast dormancy behavior. RetrospectorOS includes tuned settings to minimize this. If drops are persistent, toggle airplane mode off and on to reconnect.

### Gapps Support
Must find your own fix for device cirtification via Magisk Modules. Google apps will not allow you to use them untill device cirtification is fixed.

### TWRP Recovery
A TWRP custom recovery for the Sidephone is currently in development.

---

## Known Issues (v1)

| Issue | Workaround | Status |
|---|---|---|
| Mobile data off after first boot | Enable manually in Settings | Fix planned for v2 |
| Jaketype not set as default keyboard | Enable manually in keyboard settings | Fix planned for v2 |
| VoLTE toggle may not appear immediately | Toggle airplane mode off/on | Under investigation |
| Brief signal drop when screen turns off | Normal MTK behavior | Improvement planned for v2 |
| Small lock screen PIN pad | Known issue | Fix planned for v2 |

---

## Credits & Attribution

RetrospectorOS is built on the following open source projects:

| Project | Author | License |
|---|---|---|
| LineageOS 23.2 GSI | MisterZtr | Apache 2.0 |
| LineageOS | LineageOS Project | Apache 2.0 / GPL v2 |
| AOSP | Google / AOSP Contributors | Apache 2.0 |
| PHH TrebleDroid | Phhusson | Apache 2.0 |
| MTK Client | Bkerler | MIT |
| Magisk | TopJohnWu | GPL v3 |

Modifications and packaging by Tyler (Retrospector). This project is not affiliated with Sidephone, Google, LineageOS, or any carrier.
