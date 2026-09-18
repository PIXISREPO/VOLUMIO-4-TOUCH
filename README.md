# PIXIS Volumio Touch

## 1. Build your music player

Follow the picture guide for your Raspberry Pi:

- [Raspberry Pi 3A+ assembly guide (PDF)](https://github.com/PIXISREPO/PIXIS/blob/main/PIXIS_CB-1_Assembly_Raspberry-Pi-3A%2B_Beta-v7.pdf)
- [Raspberry Pi Zero 2 W assembly guide (PDF)](https://github.com/PIXISREPO/PIXIS/blob/main/PIXIS_CB-1_Assembly_Raspberry-Pi-Zero-2W_Beta-v4.pdf)

Keep the power unplugged while building.

You need a PIXIS CB-1 kit, a Raspberry Pi, the Waveshare 2.8-inch SPI touchscreen (**SKU 27579**), a microSD card, a suitable power supply and an audio system.

**This guide is still a draft.** The Pi Zero 2 W setup has been tested from a fresh card. The Pi 3A+ needs an extra first-boot step that we are still turning into a checked, beginner-friendly instruction.

## 2. Set up Volumio

Volumio plays your music. Set it up before adding the touchscreen controls.

1. Get the Raspberry Pi image from [Volumio](https://volumio.com/). Our tests used **Volumio 4.119**; newer versions are not yet covered by this guide.
2. Write the image to your microSD card using the instructions supplied by Volumio. **This erases the card**, so use a spare if you want to keep your old setup.
3. Put the card into your player and switch on the power.

**Pi 3A+ owners:** pause before first boot. Volumio 4.119 needs a boot-file change on this board. See [Troubleshooting](#troubleshooting); the complete fresh-card instructions are still being checked. The tested Zero 2 W needed no such change.

4. Open the **Volumio phone app**, or use a web browser on a phone or computer connected to the same network. Try **http://volumio.local**. You can also use the player's network address.
5. Follow Volumio's setup screens to connect to your network and choose your audio output.
6. Play some music to make sure the sound works.

## 3. Turn on SSH

**Do this before downloading the extra touchscreen software.**

SSH lets your computer send setup instructions to your player.

1. In a web browser, open **http://volumio.local/dev**. If you use the player's network address instead, add `/dev` to the end.
2. Find **SSH** and select **Enable**.
3. On your computer, open **Terminal** on a Mac, or **PowerShell** on Windows.
4. Copy this line, paste it into that window and press Enter:

```bash
ssh volumio@volumio.local
```

If you use the player's network address, replace `volumio.local` with that address. On the first connection, check that it is your player and accept the connection prompt. Enter your player's SSH password when asked. Nothing appears while you type the password—that is normal.

Keep this window open. The next commands go into this connected window, so they run on the player.

## 4. Add the touchscreen software

Copy this whole line into the connected window and press Enter:

```bash
curl -fsSL https://raw.githubusercontent.com/foonerd/rpi-waveshare28/main/scripts/install.sh | sudo bash -s runtime
```

Enter the player's password if asked, then wait for the installer to finish. If it reports an error, stop and see Troubleshooting.

This installs [@nerd's touchscreen software](https://github.com/foonerd/rpi-waveshare28). It currently downloads **runtime-v1.6.0**. This method does not add a settings page to Volumio's Installed Plugins list.

### Choose your screen layout

**Pi Zero 2 W:** the tested setup uses the default upright screen. You do not need another setup command.

**Pi 3A+:** once the first-boot preparation is complete, choose **one** of these:

Upright screen (Portrait):

```bash
sudo waveshare28-config set backend=framebuffer rotation=0 console=release
```

Wide screen (Landscape):

```bash
sudo waveshare28-config set backend=framebuffer rotation=270 console=release
```

When setup is finished, restart the player:

```bash
sudo reboot
```

Your Terminal connection will close during the restart. Once Volumio is ready, choose music using its web page or phone app. Use the touchscreen for track information, volume and Play/Pause.

## Troubleshooting

You only need this section if something goes wrong.

### I cannot open Volumio or connect with SSH

- Make sure the player and your phone or computer are on the same network.
- If `volumio.local` does not work, use the player's network address.
- For an SSH connection problem, check that you enabled SSH on the browser's `/dev` page.

### My Pi 3A+ will not start

A fresh Volumio 4.119 card needed a board-specific boot-file change in our Pi 3A+ tests. The touchscreen installer cannot fix this before the player starts.

**The complete beginner procedure is still being checked.** Do not try random boot-file edits. The tested Zero 2 W started without this change.

### The screen is blank, frozen or facing the wrong way

First, restart the player. If the problem remains, reconnect with SSH as described in step 3.

Copy these lines into the connected window:

```bash
waveshare28-config show
waveshare28-config verify
systemctl is-active waveshare28-panel
systemctl is-enabled waveshare28-panel
```

The usual healthy results are **no drift**, **active** and **enabled**. Save the output if you need help. For the wrong orientation on a Pi 3A+, repeat the matching layout command in step 4 and restart.

For more detail to include in a help request:

```bash
journalctl -u waveshare28-panel -b --no-pager
```

### How can I check the player is working?

Play **Radio Paradise v2 / RP2 Main Mix**. Check that:

- Artwork and track names appear and change with the music.
- The progress bar moves.
- Touch volume and Play/Pause work.

Some live radio stations do not supply a track length, so having no progress bar can be normal.

### I cannot find the touchscreen plugin in Volumio

That is expected with this guide. It installs the touchscreen software directly. The separate Volumio plugin and its settings page need their own installation and testing.

### Which versions were tested?

Tests on 16 September 2026 used **Volumio 4.119** and **runtime-v1.6.0**:

| Raspberry Pi | Tested screen setup |
|---|---|
| Pi 3A+ | Portrait and Landscape, Framebuffer |
| Pi Zero 2 W | Fresh-card installation, Portrait, SPI |

Artwork, track information, progress and touch controls passed in these setups. These results do not cover every music service or a fresh-card Pi 3A+ installation.

The Large + Roomy readability check is complete: Peter found the screen readable from arm's length to one metre with reasonable eyesight, suitable for desktop, side-table and bedside use.

For technical support, the tested ARMv7 program can be checked with:

```bash
sha256sum /usr/local/bin/waveshare28-panel
```

Its expected result starts with:

```text
9996c4f6eb860d474c479860bd4221ab585416ef3735c6a255ba3343d51aab86
```

The installer also downloads a setup helper from upstream's changing `main` branch. The program version and checksum alone do not identify that helper.

### Removing or restoring the screen setup

Keep a working spare card or backup before trying recovery.

Upstream provides `sudo waveshare28-config recover` to remove the active screen setup while keeping saved settings, and `sudo waveshare28-config apply` to restore it. **We have not completed the full recovery and uninstall tests for this guide.** Read the [upstream recovery instructions](https://github.com/foonerd/rpi-waveshare28/blob/main/docs/CONFIG.md#recover) before using them.

### About this project

Touchscreen software is by [@nerd / foonerd](https://github.com/foonerd/rpi-waveshare28). PIXIS provides the CB-1 guide and hardware testing. This project does not claim official Volumio plugin approval.

For advanced settings and software licences, see the upstream repository. Assembly links were checked on 18 September 2026 and point to the published Beta manuals.
