# PIXIS Volumio Touch

## 1. Build your music player

Follow the picture guide for your Raspberry Pi:

- [Raspberry Pi 3A+ assembly guide (PDF)](https://github.com/PIXISREPO/PIXIS/blob/main/PIXIS_CB-1_Assembly_Raspberry-Pi-3A%2B_Beta-v7.pdf)
- [Raspberry Pi Zero 2 W assembly guide (PDF)](https://github.com/PIXISREPO/PIXIS/blob/main/PIXIS_CB-1_Assembly_Raspberry-Pi-Zero-2W_Beta-v4.pdf)

Keep the power unplugged while building.

You need a PIXIS CB-1 kit, a Raspberry Pi, the Waveshare 2.8-inch SPI touchscreen (**SKU 27579**), a microSD card, a suitable power supply and an audio system.

**This guide is still a draft.** The Pi Zero 2 W setup has been tested from a fresh card. The Pi 3A+ needs the boot-file edit in section 2. The full fresh-card customer procedure still needs an end-to-end check.

## 2. Set up Volumio

Volumio plays your music. Set it up before adding the touchscreen controls.

1. Get the Raspberry Pi image from [Volumio](https://volumio.com/). Our tests used **Volumio 4.119**; newer versions are not yet covered by this guide.
2. Write the image to your microSD card using the instructions supplied by Volumio. **This erases the card**, so use a spare if you want to keep your old setup.
3. **Pi 3A+ owners:** Volumio 4.119 needs a boot-file change in order to boot. Make the change below **before putting the card into your player**. The tested Zero 2 W needed no such change and can skip this edit.

### Pi 3A+: edit the boot file

With the freshly written microSD card still in your computer:

- Open the card in **Finder** on a Mac or **File Explorer** on Windows. If it does not appear after writing the image, eject it and reconnect it. Open the partition containing `config.txt`, `userconfig.txt` and `volumioconfig.txt`. Do not format the card if Windows asks.
- Open the card’s **`volumioconfig.txt`** in a plain-text editor, such as Notepad on Windows or TextEdit in plain-text mode on a Mac. Do not use Word.
- Find the line **`[pi3]`**. Paste the following block **immediately above it**:

```text
[0x9020e0]
dtoverlay=vc4-kms-v3d,cma-128

[0x9020e1]
dtoverlay=vc4-kms-v3d,cma-128

```

Leave the existing `[pi3]` line and everything below it unchanged. Include both entries: they cover the two tested Pi 3A+ board revisions. If these exact entries are already present, do not add them again.

Save the file on the card as **`volumioconfig.txt`**, keeping the same name and plain-text format. Close the editor and safely eject the card.

If the edit goes wrong, download a fresh Volumio image, write it to the card again and repeat these steps.

This is the boot-file amendment that made our tested Pi 3A+ boards start with **Volumio 4.119**. A Volumio update may replace this system-managed file; do not assume the same edit is needed on a newer image.

### Start the player


4. Put the card into your player and switch on the power. On its first start, Volumio creates its own Wi-Fi hotspot called **Volumio-XXXX**. Open the Wi-Fi settings on your phone, tablet or computer and connect to that hotspot.
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

### Choose one installation route

**For normal use, go straight to [section 5: the plugin](#5-install-the-touchscreen-plugin).** It installs the touchscreen software and adds Settings in Volumio.

[Section 4](#4-lab-route-standalone-touchscreen-software) is the standalone lab route, configured with commands. You do not need to do both.

## 4. Lab route: standalone touchscreen software

Copy this whole line into the connected window and press Enter:

```bash
curl -fsSL https://raw.githubusercontent.com/foonerd/rpi-waveshare28/main/scripts/install.sh | sudo bash -s runtime
```

Enter the player's password if asked, then wait for the installer to finish. If it reports an error, stop and see Troubleshooting.

This installs [foonerd's touchscreen software](https://github.com/foonerd/rpi-waveshare28). It currently downloads **runtime-v1.6.0**. This method does not add a settings page to Volumio's Installed Plugins list.

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

**Learn the touchscreen controls:** see [foonerd's touchscreen guide on GitHub](https://github.com/foonerd/rpi-waveshare28/blob/main/docs/UI.md#surfaces). The **Surfaces** section explains which icons to tap to open playback controls, volume, track details, player status and larger artwork, and how to close those screens.

## 5. Install the touchscreen plugin

This is the customer installation route. It installs the touchscreen software and adds a **Settings** page to Volumio so you can change the layout and appearance.

**You can skip section 4.** If you already followed it, installing the plugin replaces the same binaries and adds Settings. You will have one touchscreen system, not two.

**This is not a Volumio-approved plugin.**

### Download and install the plugin

1. Open Volumio in your browser or phone app and sign into your **Volumio account**. A Free account was sufficient in our tests.
2. Make sure **SSH is enabled**, as described in step 3. Reopen Terminal or PowerShell and connect to the player again:

```bash
ssh volumio@volumio.local
```

3. Copy and paste this whole block into the connected window. It downloads a small copy of foonerd’s current software, without the full Git history, into a new temporary folder and opens the plugin folder:

```bash
plugin_folder=$(mktemp -d /tmp/waveshare28-plugin.XXXXXX)
git clone --depth 1 https://github.com/foonerd/rpi-waveshare28.git "$plugin_folder" &&
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
8. Choose your screen layout and appearance, then save your changes. The Zero 2 W starts with **SPI / Portrait (0°)**. These are defaults, not fixed restrictions—you can change them. Our Pi 3A+ setups used **Framebuffer / Portrait (0°) or Landscape (270°)**.
9. Restart when Volumio asks you to. If this is your first touchscreen installation and the screen stays blank, restart the player once.

For an explanation of the options, see [foonerd’s plugin settings reference](https://github.com/foonerd/rpi-waveshare28/blob/main/docs/CONFIG.md#volumio-plugin).

**Version reference:** use **runtime-v1.6.0** and the **16 September 2026 ARMv7 checksum** in Troubleshooting as the tested renderer reference. Foonerd confirms that the current plugin also carries that renderer. A download from `main` can change over time.

The plugin Settings page has worked on our Pi 3A+. Its installation and Settings page on the Zero 2 W still need our hands-on test. On the Zero 2 W, expect no **3A+ KMS** or **HDMI** control. **Console** should appear only if you choose **Framebuffer**.

**Learn the touchscreen controls:** see [foonerd’s touchscreen guide](https://github.com/foonerd/rpi-waveshare28/blob/main/docs/UI.md#surfaces) for what to tap and how to close each screen.

## Troubleshooting

You only need this section if something goes wrong.

### I cannot open Volumio or connect with SSH

- Make sure the player and your phone or computer are on the same network.
- If `volumio.local` does not work, use the player's network address.
- For an SSH connection problem, check that you enabled SSH on the browser's `/dev` page.

### My Pi 3A+ will not start

A fresh Volumio 4.119 card needed a board-specific boot-file change in our Pi 3A+ tests. The touchscreen installer cannot fix this before the player starts.

Check that you followed [Pi 3A+: edit the boot file](#pi-3a-edit-the-boot-file), saved the file on the card, and kept the original `[pi3]` section. The tested Zero 2 W started without this change.

### The screen is blank, frozen or facing the wrong way

First, restart the player. If the problem remains, reconnect with SSH as described in step 3.

Copy these lines into the connected window:

```bash
waveshare28-config show
waveshare28-config verify
systemctl is-active waveshare28-panel
systemctl is-enabled waveshare28-panel
```

The usual healthy results are **no drift**, **active** and **enabled**. Save the output if you need help. For the wrong orientation on a Pi 3A+, change the layout in the plugin’s Settings page, or use the matching command in section 4 for a standalone installation, then restart when required.

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

That is expected if you have only completed step 4. It installs the touchscreen software directly. See [Install the touchscreen plugin](#5-install-the-touchscreen-plugin) for the separate plugin and its Settings page.

### Which versions were tested?

Tests on 16 September 2026 used **Volumio 4.119** and **runtime-v1.6.0**:

| Raspberry Pi | Tested screen setup |
|---|---|
| Pi 3A+ | Portrait and Landscape, Framebuffer |
| Pi Zero 2 W | Fresh-card installation, Portrait, SPI |

Artwork, track information, progress and touch controls passed in these setups. These results do not cover every music service or a fresh-card Pi 3A+ installation.

Peter found the screen readable from arm's length to one metre with reasonable eyesight, suitable for desktop, side-table and bedside use. **Large** changes only the boot address screen; **Roomy** does not move the player layout.

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

Touchscreen software is by [foonerd](https://github.com/foonerd/rpi-waveshare28). PIXIS provides the CB-1 guide and hardware testing. This project does not claim official Volumio plugin approval.

For advanced settings, see the upstream repository. Assembly links were checked on 18 September 2026 and point to the published Beta manuals.

## Licensing

**PIXIS documentation:** the original PIXIS material in this repository is copyright © 2026 PIXIS and released under the [MIT License](LICENSE).

**foonerd’s software:** [rpi-waveshare28](https://github.com/foonerd/rpi-waveshare28) remains under its own upstream licences:

- The repository’s [main licence is Apache License 2.0](https://github.com/foonerd/rpi-waveshare28/blob/main/LICENSE).
- Its [README](https://github.com/foonerd/rpi-waveshare28#licence) identifies device-tree overlays under `kernel/source_files/*/overlays/*.dts` as **GPL-2.0 OR MIT**.
- The Volumio plugin’s [package metadata](https://github.com/foonerd/rpi-waveshare28/blob/main/plugin/waveshare28/package.json) separately declares **MIT**. This does not make the bundled renderer or all upstream files MIT-licensed; check the applicable upstream notices.

Our MIT licence applies to PIXIS’s original contributions here. It does not relicense foonerd’s software, Volumio, or linked assembly manuals and other external material.

Our sincere thanks go to @nerd for producing an outstanding Plugin for the Volumio community and for his help and advice in porting it to the PIXIS Platform.
