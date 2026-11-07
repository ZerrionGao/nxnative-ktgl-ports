# Notice

## What this repository is

Documentation, provisional game profiles, and host/tool patches for packing two Koei Tecmo KTGL Switch titles with [nxnative](https://github.com/seedds/nxnative) on Apple silicon Mac and (optionally) iPhone.

## What this repository is not

- It does **not** contain Nintendo Switch game ROMs, NSP/XCI dumps, RomFS images, or decrypted title keys.
- It does **not** contain `prod.keys`, `title.keys`, or any other console keys.
- It does **not** contain Apple Developer certificates, provisioning profiles, team IDs, device UDIDs, or signed IPA/`.app` packages.
- It does **not** grant a license to the games themselves.

## Your responsibilities

You must legally own the games you pack. Obtain keys only from hardware you own, by means allowed in your jurisdiction. Do not redistribute dumps, keys, or finished packages through this project.

Apple sideloading and increased-memory entitlements require your own Apple Developer account and devices. Replace any Bundle ID examples with IDs under **your** team.

## Upstream host

Runtime code belongs to nxnative (MIT). Apply the patches under `patches/` to a recent nxnative checkout before packing, or use a fork that already includes equivalent fixes. See `README.md`.
