# OPlus Payload Mods Builder

Public GitHub Actions frontend for the private
`vn-ncvinh/OPlus_Payload_Mods` builder.

The workflow downloads an OPlus full OTA or `payload.bin`, builds a recovery
ZIP, and uploads it to Google Drive with rclone.

## Setup

Add these repository Actions secrets:

- `PRIVATE_REPO_TOKEN`: fine-grained GitHub token with read access to
  `vn-ncvinh/OPlus_Payload_Mods` contents.
- `RCLONE_CONFIG_BASE64`: base64-encoded rclone configuration.
- `GDRIVE_DESTINATION`: rclone destination such as
  `gdrive:OPlus_Payload_Mods`.

Create the rclone secret on Linux:

```bash
base64 -w0 ~/.config/rclone/rclone.conf
```

Copy the output into the `RCLONE_CONFIG_BASE64` secret without adding quotes.

## Run

Two workflows sit in **Actions**, and which one you pick is the build mode:

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

### Extra images

Two inputs supply images the payload does not contain. Leave them blank when
they do not apply; the builder rejects an image it does not need.

| Input | When it is required |
| --- | --- |
| `ace6t_ota_url` | **Every** OnePlus 15R build, from either workflow. A OnePlus Ace 6T OTA, which the builder reads with range requests to pull out only the odm it needs, then checks that image is the China ODM. |
| `my_preload_url` | X9U stock builds. Offered by both forms; in a modded build it simply replaces the profile `my_preload` donor. |

A stock build first tests the `my_preload` donor the private repo ships. The
OnePlus 15R donor is a real signed stock image, so nothing extra is needed.
The X9U donor is an 8 KB placeholder, so an X9U stock build stops and asks for
`my_preload_url`. In a stock build the image in use must carry a signed
`my_preload` AVB hashtree matching its contents and a project ID the device
profile accepts; in a modded build it only has to be an EROFS image.

The private repository is cloned only inside the temporary Actions runner.
Credentials are not persisted in its Git checkout. Build output is uploaded to
the configured Google Drive destination and is not committed to this public
repository.

When a build fails, the job summary names the model, the reason reported by the
builder, and — for the two inputs whose absence the builder can only report at
runtime — which input to set before re-running.
