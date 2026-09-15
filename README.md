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

Open **Actions → OPlus Payload Mods Builder → Run workflow**, enter a direct
ROM URL, pick a build mode, and start the workflow. The output filename
automatically selects the `Oppo_Find_X9_Ultra` or `OnePlus_15R` subdirectory
below `GDRIVE_DESTINATION`.

### Build mode

- `mods` applies the patches selected below, plus DisableSafeMediaVolume and
  the AVB fstab patch, which have no toggle. The device keeps its current
  recovery.
- `stock` applies no content mod at all. The package still carries the
  vulnerable ABL donor and the GBL Chainload EFISP, and it restores the stock
  `recovery.img` from the payload.

Both modes flash the ABL donor and the GBL Chainload mode 1 EFISP.

### Extra images

Two inputs supply images the payload does not contain. Leave them blank when
they do not apply; the builder rejects an image it does not need.

| Input | When it is required |
| --- | --- |
| `ace6t_ota_url` | **Every** OnePlus 15R build, in either mode. A OnePlus Ace 6T OTA, which the builder reads with range requests to pull out only the odm it needs, then checks that image is the China ODM. |
| `my_preload_url` | X9U `stock` builds. Optional elsewhere, where it simply replaces the profile `my_preload` donor. |

A stock build first tests the `my_preload` donor the private repo ships. The
OnePlus 15R donor is a real signed stock image, so nothing extra is needed.
The X9U donor is an 8 KB placeholder, so an X9U stock build stops and asks for
`my_preload_url`. In `stock` mode the image in use must carry a signed
`my_preload` AVB hashtree matching its contents and a project ID the device
profile accepts; in `mods` mode it only has to be an EROFS image.

The private repository is cloned only inside the temporary Actions runner.
Credentials are not persisted in its Git checkout. Build output is uploaded to
the configured Google Drive destination and is not committed to this public
repository.

When a build fails, the job summary names the model, the reason reported by the
builder, and — for the two inputs whose absence the builder can only report at
runtime — which input to set before re-running.
