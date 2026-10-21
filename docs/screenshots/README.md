# Screenshots — `assets/screenshots/`

This folder holds the 72 numbered screenshots that visually validate the lab execution. Every
screenshot is mapped to a phase and step in the [Evidence Index](../../docs/evidence-index.md).

## Naming Convention

- File name: `screenshot-NN.png` where `NN` is the zero-padded screenshot number (`01`–`72`).
  Example: `screenshot-01.png`, `screenshot-72.png`.
- Path referenced from phase documents: `assets/screenshots/screenshot-NN.png`.
- Keep the original numbering — it matches the capture order of the build and the evidence index.

## Sanitization Rules

Before committing any screenshot:

- No passwords, DSRM passwords, LAPS secrets, recovery keys, or Kerberos tickets.
- No real public IP addresses or confidential organizational data.
- Lab-only addresses (`192.168.10.0/24`) are fine to show.
- Blur or crop credential prompts where a password field is visible.
- `Get-LapsADPassword` output must be captured without displaying the password value.

## Status

- [ ] Screenshot files `01–72` added and verified against the evidence index captions.
- [ ] Each image reviewed for sanitization before publishing.

Phase documents reference screenshots by number and relative path (rather than embedding them
inline) so the repository renders cleanly whether the image files are present or still being
added.
