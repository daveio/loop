# How to root your Loop

Download all the files in this repository. Have them to hand.

Everything we do here, as long as something doesn't explode for no good reason, is reversible. You can flash the stock firmware and lock the bootloader and nobody needs to be any the wiser if you really want to.

So let's unlock that bootloader. Connect your Loop to your computer via USB, enable developer mode by smashing the build number in `Settings > System` until it gives in, then enable USB debugging in `System > Developer Options`.

Hopefully you've got the Android SDK installed, which will give you `adb` and `fastboot`. If not, fix that.

Then we can go ahead and unlock the bootloader.

> [!WARNING]
> Unlocking the bootloader will factory reset the device and erase all data.

```sh
adb reboot bootloader

# Check that device is showing up in fastboot list
fastboot devices

# Unlock bootloader (will factory reset device)
# if this doesn't work, try `fastboot flashing unlock`
fastboot oem unlock

# Reboot back to main OS
fastboot reboot
```

That was easy. Don't worry, it gets more annoying from here.

Set your Loop back up, either online or offline, whatever you like. We're not going to factory reset it again.

Hopefully your Loop is still connected via USB. Enable developer mode and USB debugging again, as our factory reset nuked it.

Then we need to check that nothing really funky has changed in the Loop hardware or software between the time I wrote this and now. We'll do that using `TrebleInfo`.

The `TrebleInfo.apk` file is included, so get it installed.

```sh
adb install TrebleInfo.apk
```

Fire up TrebleInfo. You should see something like this:

![TrebleInfo views](images/treble.avif)

Assuming it shows Project Treble support (it should, unless they've pulled a fast one since I tested this), we're good to proceed. If not, when you've finished screaming, maybe hit me up because something's changed.

Next up, we need to sideload a GSI to get root access temporarily. This is where DSU Sideloader comes in. Grab the APK from the repo (or from its GitHub if you want the latest), and install it the same way:

```sh
adb install DSU-Sideloader.apk
```

You'll also need a GSI image. I've included a known-good one in the repo (`ANDROID-16-system-td-arm64-vanilla.img.xz`), but if you're feeling adventurous, snag the latest from TrebleDroid's releases.

> [!NOTE]
> The original instructions suggested using an Android 15 GSI image because that's what our core OS is running. When I tried this, it failed. Using an Android 16 GSI worked fine, at the cost of some UI weirdness - though it remains usable enough for our purposes.

Launch DSU Sideloader on the Loop. Point it to your GSI image, set userdata to something reasonable like 3GB (don't go nuts), and follow the prompts. It'll sideload the GSI and prompt you to reboot into it.

This process, if you're coming back to Android fresh after being a dedicated Android hacker for years and then being taken in by the lure of iOS, is pretty cool. It doesn't touch the firmware partitions; you're not even overwriting the alternate firmware. It creates a virtual partition inside a file, and boots from that. This means that **loading a GSI does not inherently make any changes to your system**.

Once you're in the GSI (it'll look bare-bones, that's fine), you've got root shell access. Time to dump those boot partitions. Connect via ADB again and run this as root:

```sh
su
mkdir -p /sdcard/img/
for PARTITION in "init_boot_a" "init_boot_b" "boot_a" "boot_b" "vendor_boot_a" "vendor_boot_b"; do
  BLOCK=$(find /dev/block \( -type b -o -type c -o -type l \) -iname "$PARTITION" -print -quit 2>/dev/null)
  if [ -n "$BLOCK" ]; then
    echo "$PARTITION" = $(readlink -f "$BLOCK")
    dd if="$BLOCK" of="/sdcard/img/$PARTITION.img"
  fi
done
```

Confirm your active slot with

```sh
getprop ro.boot.slot_suffix
```

You'll need it soon.

Pull those images to your computer for safekeeping:

```sh
adb pull /sdcard/img/
```

Now, install Magisk on the Loop (APK's in the repo, or grab from GitHub):

```sh
adb install Magisk-v30.6.apk # Or whatever version you're using
```

Use Magisk to patch the `init_boot` image for your active slot (e.g., `init_boot_b.img` if that's the one). It'll spit out a patched file in Downloads – pull that too:

```sh
adb pull /sdcard/Download/magisk_patched-30600_XXXXX.img # Adjust if needed
```

Reboot out of the GSI back to stock (DSU Sideloader has an option for this, or just reboot normally a couple times if it acts up).

Boot into fastboot mode:

```sh
adb reboot bootloader
```

Flash the patched init_boot:

```sh
fastboot flash init_boot_b magisk_patched-30600_XXXXX.img # Adjust if needed
fastboot reboot
```

Magisk should finish setting up on boot. It didn't for me. Fortunately, it's nice and simple; look for Magisk in your apps and run it. The stub will execute and replace itself with the real Magisk superuser proxy.

Congrats, you've got root.

Fire it up, grant superuser to whatever you need, go wild. Just remember, if you brick it, it's on you. But. Hey. That's half the fun.

If things go sideways, you've got those boot backups. Flash them back with fastboot to restore stock; you might as well flash them all. They're already named for their partitions.

I know this works, because after rooting it I realised I didn't actually care about having it rooted, and wanted it back to stock. I just flashed all the backups and everything was back to 'normal'.

If you also want to relock the bootloader later because of that irritating warning on boot, `fastboot flashing lock` after flashing stock everything. Note: `fastboot oem lock` *doesn't* work. Or it didn't for me. Which I found a bit weird, because I used `fastboot oem unlock` to *unlock* the thing, but Android is a magical land full of mysteries.

Happy hacking. Don't blame me if your £1 speaker starts demanding tribute.
