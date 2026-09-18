# PIXIS Volumio Touch

## 1. Assemble your PIXIS CB-1

Start with the illustrated instructions for your Raspberry Pi:

- [Raspberry Pi 3A+ — PIXIS CB-1 assembly instructions, Beta v7 (PDF)](https://github.com/PIXISREPO/PIXIS/blob/main/PIXIS_CB-1_Assembly_Raspberry-Pi-3A%2B_Beta-v7.pdf)
- [Raspberry Pi Zero 2 W — PIXIS CB-1 assembly instructions, Beta v4 (PDF)](https://github.com/PIXISREPO/PIXIS/blob/main/PIXIS_CB-1_Assembly_Raspberry-Pi-Zero-2W_Beta-v4.pdf)

Both public document links were checked on 18 September 2026. They are the published Beta manuals. Disconnect power before assembling or changing connections.

This project combines PIXIS CB-1, a Raspberry Pi and the **Waveshare 2.8-inch SPI capacitive touchscreen, SKU 27579**, with Volumio and [@nerd's rpi-waveshare28 software](https://github.com/foonerd/rpi-waveshare28). The screen provides artwork, track information and touch playback controls for a desktop, side-table or bedside music player.

**Review draft:** the runtime results below are verified against PIXIS's retained tests. Clean-image Pi 3A+ preparation and the separate Volumio plugin installation/recovery route still need a complete customer procedure. This guide currently describes the standalone runtime route.

## 2. Download and prepare Volumio

You need an assembled CB-1 with the specified display, a microSD card, suitable power supply, network access and an audio output configured in Volumio.

The tested system is **Volumio 4.119**. Obtain the Raspberry Pi image through [Volumio](https://volumio.com/) and write it to your microSD card using an image-writing application. Writing an image erases the selected card; use a separate card to preserve a working installation. A newer Volumio image requires its own verification and is not covered by the results below.

### Raspberry Pi Zero 2 W

The tested Zero 2 W booted a stock Volumio 4.119 image without a KMS amendment. Complete Volumio's network and audio setup in its browser interface, then confirm music plays before adding the touchscreen software.

### Raspberry Pi 3A+

The tested 4.119 systems required board-specific KMS preparation before first boot. The historical successful preparation added the following immediately before the generic `[pi3]` stanza in `volumioconfig.txt`:

```text
[0x9020e0]
dtoverlay=vc4-kms-v3d,cma-128

[0x9020e1]
dtoverlay=vc4-kms-v3d,cma-128
```

This records what made the tested boards boot; it is **not yet a complete verified fresh-card procedure**. The current upstream configurator instead maintains a board-specific backstop in `userconfig.txt` after Linux starts, and cannot repair a board that never reaches that stage. Reconcile and verify the pre-boot method before publishing this section as a customer installation sequence. Do not infer that a 128 MiB effective CMA pool was measured, or that system-managed boot-file edits survive updates.

## 3. Download and install @nerd's app

The tested release is [runtime-v1.6.0](https://github.com/foonerd/rpi-waveshare28/releases/tag/runtime-v1.6.0), also the latest published runtime when checked on 18 September 2026.

Enable SSH using the `/dev` page of your Volumio browser interface, then connect to the player's address as the `volumio` user. Run installation commands on the Raspberry Pi, not on your Mac or PC.

The route used in the retained runtime tests is the [upstream runtime installer](https://github.com/foonerd/rpi-waveshare28/blob/main/scripts/install.sh):

```bash
curl -fsSL https://raw.githubusercontent.com/foonerd/rpi-waveshare28/main/scripts/install.sh | sudo bash -s runtime
```

It downloads an architecture-appropriate renderer and checks its published SHA-256 before installation. It also installs the configurator and applies the configuration. Follow its reboot instruction.

**Version boundary:** the installer currently selects runtime-v1.6.0, but downloads the configurator from `main`. This command is therefore a changing upstream installation route, not an immutable reproduction of the September 16 test environment. Before public release, record and verify the installer/configurator revision alongside the renderer version.

This route does not install the Volumio **Waveshare 2.8 SPI Panel** settings plugin. The runtime can work normally without appearing in Installed Plugins. The plugin is a separate upstream integration whose customer installation, settings and recovery path has not been verified by these runtime tests.

## 4. Choose the tested display configuration

Inspect the current settings first:

```bash
waveshare28-config show
waveshare28-config detect
```

For the clean-image Zero 2 W test, the defaults were **SPI, Portrait, rotation 0**. No Pi 3A+ KMS setting applies to this board.

The tested Pi 3A+ configurations used **Framebuffer**:

```bash
# Portrait
sudo waveshare28-config set backend=framebuffer rotation=0 console=release

# Or Landscape
sudo waveshare28-config set backend=framebuffer rotation=270 console=release
```

Choose one orientation. Reboot when requested; firmware overlay and framebuffer orientation changes require it. An older installation without `/boot/waveshare28.conf` initially takes the installer's defaults, so explicitly restore your intended backend and orientation.

Use [upstream configuration documentation](https://github.com/foonerd/rpi-waveshare28/blob/main/docs/CONFIG.md) for other settings. `/boot/waveshare28.conf` holds the durable settings; generated files are recreated by the configurator.

## 5. Test your player

After reboot, check:

```bash
sha256sum /usr/local/bin/waveshare28-panel
waveshare28-config show
waveshare28-config verify
systemctl is-active waveshare28-panel
systemctl is-enabled waveshare28-panel
```

For the tested ARMv7 release on both boards, the renderer hash is:

```text
9996c4f6eb860d474c479860bd4221ab585416ef3735c6a255ba3343d51aab86
```

Expect `no drift`, `active` and `enabled`. The hash matches the published runtime-v1.6.0 ARMv7 checksum; it does not identify the configurator version.

Play Radio Paradise v2 / RP2 Main Mix and check artwork, artist/title changes, the moving progress strip, touch volume and Play/Pause. Check metadata again after a natural track change. A duration-bearing source is needed for the progress test; ordinary live web radio may have no track duration and no progress bar.

### Retained PIXIS results — 16 September 2026

| Hardware | Configuration | Result |
|---|---|---|
| Pi 3A+ Rev 1.1, `9020e1` | Volumio 4.119; v1.6.0; Framebuffer; Landscape 270 | Artwork, metadata, moving progress, touch volume and Play/Pause passed |
| Pi 3A+ Rev 1.0, `9020e0` | Volumio 4.119; v1.6.0; Framebuffer; Portrait 0 | Track-rollover artwork/metadata, moving progress and touch controls passed |
| Pi Zero 2 W Rev 1.0, `902120` | Clean Volumio 4.119; v1.6.0; SPI; Portrait 0 | Stock boot, installation, RP2 artwork/metadata/progress and touch controls passed |

All three configurations matched the renderer hash above and reported no configuration drift with an active/enabled service. A separate older Pi 3A+ installation also migrated successfully to v1.6.0 Landscape; that was a migration, not a fresh-image test. The Portrait Pi 3A+ reference retained 1 GiB swap from earlier investigation.

Peter accepted Large + Roomy readability on 18 September: readable from arm's length to one metre with reasonable eyesight, appropriate for the intended desktop, side-table and bedside use. This hands-on check is complete. Those setting names do not mean that normal-player metadata is enlarged: upstream documents `large` for startup-overlay text and `roomy` as an accepted setting that does not move the current player layout.

## 6. Recovery

For diagnosis, retain the output of `waveshare28-config show`, `waveshare28-config verify` and:

```bash
journalctl -u waveshare28-panel -b --no-pager
```

Upstream documents `sudo waveshare28-config recover` to stop/remove the panel service and its derived configuration while keeping `/boot/waveshare28.conf` and the Pi 3A+ KMS backstop. `sudo waveshare28-config apply` reinstates the configuration. These are upstream-documented behaviours; complete recovery/uninstall and factory-reset tests are not established by the functional passes above. Preserve a known-working card or image before testing them.

## Credits

Renderer, installer and configurator: [@nerd / foonerd](https://github.com/foonerd/rpi-waveshare28). PIXIS supplies the CB-1 project documentation and the scoped hardware test results. This is not a claim of official Volumio plugin approval.

Upstream declares Apache-2.0, with specified device-tree overlays under GPL-2.0 OR MIT; consult the upstream repository for the applicable licences. No upstream code or binary is redistributed by this documentation draft.
