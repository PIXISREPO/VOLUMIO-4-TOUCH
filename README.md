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

4. On its first start, Volumio creates its own Wi-Fi hotspot called **Volumio-XXXX**. Open the Wi-Fi settings on your phone, tablet or computer and connect to that hotspot.
5. Follow the on-screen instructions to connect your player to your home Wi-Fi. If you need more help, see [Volumio's website](https://volumio.com/).
6. Once the player is connected, go back to your device's Wi-Fi settings and reconnect your phone, tablet or computer to your **home Wi-Fi**.
7. Open [**http://volumio.local**](http://volumio.local) in your web browser. Or open the **Volumio phone app** and select your player from the discovered devices.
8. Follow the remaining Volumio setup screens, including choosing your audio output. You are now ready to play some music!

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

**Learn the touchscreen controls:** see [@nerd's touchscreen guide on GitHub](https://github.com/foonerd/rpi-waveshare28/blob/main/docs/UI.md#surfaces). The **Surfaces** section explains which icons to tap to open playback controls, volume, track details, player status and larger artwork, and how to close those screens.

## 5. Install Touch Plugin Settings

This optional step adds a **Settings** page to Volumio so you can change the screen layout and appearance without typing more commands.

The plugin installs its own copy of the touchscreen software too. It is not just an extra menu.

### Download and install the plugin

1. Open Volumio in your browser or phone app and sign into your **Volumio account**. A Free account was sufficient in our tests.
2. Make sure **SSH is enabled**, as described in step 3. Reopen Terminal or PowerShell and connect to the player again:

```bash
ssh volumio@volumio.local
```

3. Copy and paste this whole block into the connected window. It downloads @nerd’s current software into a new temporary folder and opens the plugin folder:

```bash
plugin_folder=$(mktemp -d /tmp/waveshare28-plugin.XXXXXX)
git clone https://github.com/foonerd/rpi-waveshare28.git "$plugin_folder" &&
cd "$plugin_folder/plugin/waveshare28"
```

Wait until the download finishes. If you see an error, stop here. Otherwise, enter:

```bash
volumio plugin install
```

4. Volumio will warn that this is a **manually installed plugin that has not been verified by Volumio**. To continue with this installation, choose **Yes**.
5. Wait for **Plugin Successfully Installed**. Keep the player powered on while installation is running.

### Open the settings

6. Return to Volumio and open **Settings → Plugins → Installed Plugins**.
7. Find **Waveshare 2.8 SPI Panel** and enable it if it is not already enabled. If you can already see **Settings**, open that.
8. Choose your screen layout and appearance, then save your changes. For the setups used in this guide, keep **SPI / Portrait (0°)** on the Zero 2 W, or **Framebuffer / Portrait (0°) or Landscape (270°)** on the Pi 3A+.
9. Restart when Volumio asks you to. If this is your first touchscreen installation and the screen stays blank, restart the player once.

For an explanation of the options, see [@nerd’s plugin settings reference](https://github.com/foonerd/rpi-waveshare28/blob/main/docs/CONFIG.md#volumio-plugin).

**What we have tested:** this plugin installation method and its Settings page worked on our Pi 3A+ with the September 14 build. The commands above download the current upstream version, which may have changed since then. The Zero 2 W is supported by the plugin, but its plugin installation and Settings page still need our hands-on test.

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

That is expected if you have only completed step 4. It installs the touchscreen software directly. See [Install Touch Plugin Settings](#5-install-touch-plugin-settings) for the separate plugin and its Settings page.

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

For advanced settings, see the upstream repository. Assembly links were checked on 18 September 2026 and point to the published Beta manuals.

## Licensing

**PIXIS documentation:** the original PIXIS material in this repository is copyright © 2026 PIXIS and released under the [MIT License](LICENSE).

**@nerd’s software:** [rpi-waveshare28](https://github.com/foonerd/rpi-waveshare28) remains under its own upstream licences:

- The repository’s [main licence is Apache License 2.0](https://github.com/foonerd/rpi-waveshare28/blob/main/LICENSE).
- Its [README](https://github.com/foonerd/rpi-waveshare28#licence) identifies device-tree overlays under `kernel/source_files/*/overlays/*.dts` as **GPL-2.0 OR MIT**.
- The Volumio plugin’s [package metadata](https://github.com/foonerd/rpi-waveshare28/blob/main/plugin/waveshare28/package.json) separately declares **MIT**. This does not make the bundled renderer or all upstream files MIT-licensed; check the applicable upstream notices.

Our MIT licence applies to PIXIS’s original contributions here. It does not relicense @nerd’s software, Volumio, or linked assembly manuals and other external material.
