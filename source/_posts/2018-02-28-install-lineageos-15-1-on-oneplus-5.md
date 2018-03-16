title: "Install LineageOS 15.1 on OnePlus 5"
date: 2018-02-28 00:41:37
tags:
- Android
- LineageOS
- OnePlus 5
---
## Downloads

- (Firmware) [OxygenOS 5.0.2 Firmware & Modem for OnePlus 5](https://jamal2367.com/downloads/OnePlus%205/Firmware%20%2B%20Modems/Android%208.0.0/OxygenOS/OnePlus5_5.0.2-FIRMWARE%2BMODEM-flashable.zip)
- (ROM) [LineageOS 15.1 for OnePlus 5](https://forum.xda-developers.com/oneplus-5/development/rom-lineageos-15-1-oreo-oneplus-5-t3727228)
- (GApps) [MindTheGapps-8.1.0-arm64-20180223_195845](https://androidfilehost.com/?fid=962187416754462508)
- (SU) [Magisk v16.0](https://forum.xda-developers.com/apps/magisk/official-magisk-v7-universal-systemless-t3473445)
- (No Verity) [Disable DM Verity and Forced Encryption for OnePlus 5 v2](https://forum.xda-developers.com/showpost.php?p=75705679&postcount=602)

## Steps

1. Install TWRP.
2. Make a backup of your current system in case of failure.
3. Flash stock firmware of OxygenOS 5+ if you are upgrading from Android 7.x or below.
4. Flash ROM, then GApps. I'm using MindTheGapps and it's working correctly with LineageOS 15.1. Ground Zero's GApps also works too. Open GApps does not seem to work.
5. Flash Magisk, then disable DM verity.
6. Reboot into system.
7. Enjoy LineageOS.
