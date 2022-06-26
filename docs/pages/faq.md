---
layout: page
title: Questions and Answers
permalink: /faq
---

## Do you plan publishing on F-Droid?

Project build-time dependencies make publishing on F-Droid impossible. The
most problematic parts are ones using Docker: the Alpine Linux ISO and JNI
library build environments.

Probably if F-Droid supports publishing prebuilt APKs, I will look into that
in future.

Note that I plan doing publication on GitHub Releases, in addition to Play
Store. That will help users without Google services to install the app. In
addition, I will provide a build without certain restrictions implied by
Play Store rules.

## Do you plan localization?

Localizing just few labels of UI elements is pointless. The main part, i.e
operating system image, can't be localized. I prefer to avoid any UI
inconsistencies.

Therefore the answer is no and I will not accept the pull requests for this.

## Do you plan to add multiple terminal tabs?

No, this can't be done by design. vmConsole is not a standard terminal
program that just launches a shell. Instead, this is a QEMU attached directly
to a terminal emulator screen. One QEMU instance, one console. If you need
multiple terminal tabs, please use a multiplexer like `tmux`.

Even though QEMU supports making multiple serial consoles, it was shown
that attaching each one would require a unix socket as IPC between QEMU
and vmConsole frontend. That's does not look good and experiments shown
that such implementation is prone to failure on at least some Android
devices.

## Why to use Alpine Linux?

Because it is perfectly suitable for use in virtual machines, especially
emulators.

* Very small comparing to other distributions.
* Simple by design.
* No systemd.

Base system images of Debian, Ubuntu, Kali, etc will be too big for packaging
inside APK (if publishing on Play Store). Init system is another issue.
Systemd isn't good for use on emulated, low-performance systems, and even
worse on mobile devices where unnecessary activity increases battery drain.

I don't have any plans to change the Alpine Linux to something else unless
there would be a good reason for this, such as when its development was
suddenly halted.

## Can I run other operating system?

Yes, you can. With a little effort of course. vmConsole provides several
disks:

* `/dev/cdrom`: read-only ISO image with Alpine Linux.
* `/dev/sda`: read-write user disk 1.
* `/dev/sdb`: read-write user disk 2.

So, you should be able to install a custom operating system on either
`/dev/sda` or `/dev/sdb`. Don't forget that your OS must do output over
serial console (`/dev/ttyS0`).

## Why system image is not AArch64?

Because I need x86_64 and there no particular reason to use other architecture.
vmConsole emulates hardware (no KVM), no matter which target CPU is used. Use
of same architecture will not disable the emulation. Therefore even if AArch64
will give you some better performance (say +30%), that will be related solely
to emulator implementation differences.

## Is KVM support planned?

Currently not, I generally against implementing any feature that would
require customizing firmware of device or at least rooting. In general, the
virtualization is not possible on the most of modern Android devices because
HYP mode (EL2) is already acquired and locked by ARM TrustZone. Modifying
bootloader to disable the latter may cause issues with fingerprint sensor,
HW keystore or even render device unbootable. That's for ARM 32/64 bit
devices. Not sure about the situation with x86 ones.

If Android will officially provide `/dev/kvm` accessible by regular apps,
then I will look into implementing KVM support. But remember that it would
be available only for x86_64 devices, since system image is for that CPU.

## How can I make vmConsole faster?

You can't, as vmConsole emulates a PC-like hardware. Running even a such
simple program as printing "hello world" takes significantly more power than
running it on bare metal host. Obviously, the only way to make it faster is
to eliminate the emulator layer between host and your program.

vmConsole is not about speed. If speed is your priority, please use [Termux].

## Is there any graphical output support?

vmConsole literally means: Virtual Machine Console. So main focus is the
text based terminal interface to the operating system. There no graphics
support. However in future versions I probably will look into adding VNC.

## How to access QEMU monitor?

It is disabled for security reasons. For example you shared the device with
someone. Without QEMU monitor, that person will not be able to dump the
contents of disk or RAM. As soon as your device is not rooted, it will not
be possible to extract the data if vmConsole OS login is password-protected.

You will need a custom build of vmConsole in order to use the QEMU monitor.

## How can I pass custom arguments to QEMU?

This application does not provide any way to configure QEMU by design. All
configuration is hardcoded ([see this](https://github.com/sylirre/vmConsole/blob/cf20aee3364aba654a2c65da5ac3d4ce268ee30a/app/src/main/java/sylirre/vmconsole/TerminalActivity.java#L373))
and part (such as RAM allocation) is determined automatically at runtime.

I want to keep the application as simple as possible, as I am not skilled
at Android application development. It has only necessary UI elements,
without any optional dialogs, screens, tabs, etc.

## How can I customize port forwarding?

See answer about custom QEMU arguments.

Currently vmConsole will have only fixed forwardings for `22` and `80`
ports. In future may add more fixed forwardings for common ports. As a
workaround, use external SSH client (e.g. in [Termux]) to forward your
ports.

## Can I run Aircrack-NG?

Not really.

You can run few tools of that package, like `aircrack-ng` itself to brute
force the WPA2 password. But everything else that will require Wi-Fi hardware
will not work. vmConsole does not provide access to the host hardware.

[Termux]: https://termux.dev
