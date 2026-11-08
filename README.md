# Quest ADB Driver for Windows on ARM64

An unofficial ARM64 build of the Oculus / Meta Quest ADB USB driver, for ADB on Windows 11 on ARM (e.g. Parallels on an Apple Silicon Mac).

The official `oculus-go-adb-driver-2.0` fails on ARM64 because:

- its INF only has `NTx86` / `NTamd64` sections
- it depends on x86/x64-only WDF/WinUSB co-installer DLLs

This repo is a single INF that adds an `NTarm64` section and uses Windows' built-in, Microsoft-signed `WinUSB.sys`. No binaries.

## Supported devices

- Oculus Go and the original Oculus IDs (`VID_2833` PIDs `0081`, `0083`, `0086`, `0090`)
- Meta Quest 1 / 2 / 3 / 3S / Pro (`VID_2833` PIDs `0137`, `0183`, `0186`)
- A fallback match on the standard ADB interface class (`FF/42/01`) and fastboot class (`FF/42/03`), so other Quest PIDs should also be picked up

## Download

**[Download the latest release (zip)](https://github.com/jchin14/quest-adb-driver-arm64/releases/latest/download/quest-adb-driver-arm64.zip)**

The zip is rebuilt automatically from `main` whenever the driver or README changes.

## Install

> **Note:** The package is unsigned, because editing the INF invalidates the original signature. Don't right-click the `.inf` and choose **Install**. That fails with *"The third-party INF does not contain digital signature information."* Use the steps below instead.

1. **Boot once with driver signature enforcement disabled.**
   Go to Settings → System → Recovery → Advanced startup → **Restart now**. Then pick Troubleshoot → Advanced options → Startup Settings → **Restart**, and press **7** (*Disable driver signature enforcement*).
2. **Connect the headset to Windows.**
   In Parallels, go to Devices → USB & Bluetooth and connect the Quest to Windows, not macOS. Put the headset on and accept **Allow USB debugging**. Developer mode must be enabled on the headset.
3. **Install the driver from Device Manager.**
   Right-click the Quest / *ADB Interface* device (it is often under *Other devices* with a yellow **!**). Then choose Update driver → Browse my computer → Let me pick → **Have Disk…**, select `android_winusb.inf`, and pick **Oculus Composite ADB Interface**. On the red warning, click **Install this driver software anyway**.
4. **Reboot normally.** The driver stays installed, so you don't need to repeat step 1.

## If step 1 doesn't work: test signing (needs Secure Boot off)

If the Startup Settings menu isn't available, or Windows still refuses the driver, you can switch Windows into test-signing mode instead. Test signing can't be turned on while **Secure Boot** is enabled, so turn Secure Boot off first.

1. **Turn off Secure Boot in Parallels.** Shut Windows down completely (not suspend). In Parallels, open the VM's **Configure** → **Hardware** → **Boot**, and untick **Secure Boot** (it may be under *Advanced*). Start Windows again.
2. **Turn on test signing.** In an **administrator** PowerShell, run:
   ```
   bcdedit /set testsigning on
   ```
   If you get *"The value is protected by Secure Boot policy"*, Secure Boot is still on. Go back to step 1.
3. **Reboot.** A *Test Mode* watermark appears on the desktop.
4. **Install the driver.** Repeat steps 2–3 of [Install](#install).
5. **Optional cleanup.** Run `bcdedit /set testsigning off`, reboot, and turn Secure Boot back on in Parallels.

## Verify

Install the Android [platform-tools](https://developer.android.com/tools/releases/platform-tools) for Windows, then run:

```
adb devices
```

You should see the headset listed as `device`. If it shows as `unauthorized`, accept the prompt in the headset.

## Do you even need this?

On macOS, ADB works natively with no driver. Use the macOS platform-tools, or a native Mac sideloading tool such as SideQuest. This driver is only needed when the tool you want to use runs only on Windows.

## Disclaimer

Unofficial and not affiliated with or endorsed by Meta or Google. The INF is derived from Google's Android USB driver INF, as redistributed in Oculus's ADB driver package. Use at your own risk.

## Support

If you find this useful and can afford to, support is appreciated: [buy me a coffee](https://buymeacoffee.com/jchin14). Just getting started, the domain, hosting, Cloudflare Workers and Apple and Google developer accounts add up to a few hundred dollars. Your support helps me keep making tools that are helpful for everyone.

## License

This project is licensed under the [PolyForm Noncommercial License 1.0.0](LICENSE).

You're free to use, modify, and share it for personal, hobby, educational, research, and other **noncommercial** purposes. **Commercial use requires prior written permission** — contact Jonathan Chin via [GitHub](https://github.com/jchin14) to arrange a commercial license.

This license covers only Jonathan Chin's original work in this repository. `android_winusb.inf` is derived from Google's Android USB driver INF as redistributed in Oculus's ADB driver package; the portions taken from that material remain under their original terms and are not relicensed here.
