# Payload Mods Builder

Public GitHub Actions frontend for two private builders:

- `vn-ncvinh/OPlus_Payload_Mods` - OPPO and OnePlus, output is a recovery ZIP.
- `vn-ncvinh/Xiaomi_8E5_Global_Mods` - Xiaomi Global/EEA, output is a folder of
  fastboot images, which the workflow zips.

Each downloads a full OTA, builds, and uploads the result to Google Drive with
rclone.

## Setup

Add these repository Actions secrets:

- `PRIVATE_REPO_TOKEN`: fine-grained GitHub token with read access to the
  contents of whichever private builder you run - `OPlus_Payload_Mods`,
  `Xiaomi_8E5_Global_Mods`, or both.
- `RCLONE_CONFIG_BASE64`: base64-encoded rclone configuration.
- `GDRIVE_DESTINATION`: rclone destination such as
  `gdrive:OPlus_Payload_Mods`.

Create the rclone secret on Linux:

```bash
base64 -w0 ~/.config/rclone/rclone.conf
```

Copy the output into the `RCLONE_CONFIG_BASE64` secret without adding quotes.

## Run

Three workflows sit in **Actions**. The first two build OPlus devices, and
which one you pick is the build mode:

- **OPlus Payload Mods Builder** applies the patches you tick, plus
  DisableSafeMediaVolume, the AVB fstab patch and DisableOTA, which have no
  toggle. The device keeps its current recovery.
- **OPlus Stock Builder** applies no content mod at all. The package still
  carries the vulnerable ABL donor and the GBL Chainload EFISP, and it restores
  the stock `recovery.img` from the payload. Its form asks for three things,
  because the mod toggles would mean nothing here.

Both flash the ABL donor and the GBL Chainload mode 1 EFISP, and both share one
job defined in `_build.yml`, so they cannot drift apart. The output filename
selects the `Oppo_Find_X9_Ultra` or `OnePlus_15R` subdirectory below
`GDRIVE_DESTINATION`.

## Xiaomi

**Xiaomi 8E5 Global Mods Builder** takes a Xiaomi Global/EEA full OTA ZIP and
eight mod toggles. Only Virtual A/B devices are supported; the build stops as
soon as the partitions come out if the ROM is anything else.

It does not share `_build.yml` with the two above. That builder is a different
repository, driven by `ENABLE_*` environment variables rather than command-line
flags, and its output is a folder rather than a ZIP.

The OTA keeps the file name the vendor gave it, because the builder reads the
device codename and region out of it. `youtube_morphe_url` pins a specific
YouTube Morphe module ZIP; blank picks the latest.

Output is one ZIP per build in `<GDRIVE_DESTINATION>/Xiaomi_<codename>/`,
holding `super.img`, the firmware images, and the fastboot install scripts for
Windows, Linux and macOS. Entries are stored rather than deflated: `super.img`
is already a compressed EROFS super, so deflating it would cost minutes and
save almost nothing.

The run summary reports the model, the base version, which mods went in, and
which partitions were repacked. Anything not listed there went into `super.img`
exactly as Xiaomi shipped it.

### Extra images

Two inputs supply images the payload does not contain. Leave them blank when
they do not apply; the builder rejects an image it does not need.

| Input | When it is required |
| --- | --- |
| `ace6t_ota_url` | **Every** OnePlus 15R build, from either workflow. A OnePlus Ace 6T OTA. The odm is pulled out of it with range requests, so only that blob is fetched, and it happens before the ROM download so a link with a short expiry is used while it is still alive. The builder then checks the image is the China ODM. |
| `my_preload_url` | X9U stock builds, and offered only by the stock form. A modded build uses the profile donor as shipped. |

A stock build first tests the `my_preload` donor the private repo ships. The
OnePlus 15R donor is a real signed stock image, so nothing extra is needed.
The X9U donor is an 8 KB placeholder, so an X9U stock build stops and asks for
`my_preload_url`. In a stock build the image in use must carry a signed
`my_preload` AVB hashtree matching its contents and a project ID the device
profile accepts; a modded build does not check it, because it never supplies one.

The private repository is cloned only inside the temporary Actions runner.
Credentials are not persisted in its Git checkout. Build output is uploaded to
the configured Google Drive destination and is not committed to this public
repository.

When a build fails, the job summary names the model, the reason reported by the
builder, and — for the two inputs whose absence the builder can only report at
runtime — which input to set before re-running.
