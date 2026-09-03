# OPlus Payload Mods Builder

Public GitHub Actions frontend for the private
`vn-ncvinh/OPlus_Payload_Mods` builder.

The workflow downloads an OPlus full OTA or `payload.bin`, applies the selected
mods, builds a recovery ZIP, and uploads the ZIP plus its SHA-256 file to Google
Drive with rclone.

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
ROM URL, select the desired patches, and start the workflow. The output filename
automatically selects the `x9u` or `op15r` subdirectory below
`GDRIVE_DESTINATION`.

The private repository is cloned only inside the temporary Actions runner.
Credentials are not persisted in its Git checkout. Build output is uploaded to
the configured Google Drive destination and is not committed to this public
repository.
