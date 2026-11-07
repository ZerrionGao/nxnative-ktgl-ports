# Patches for nxnative

Apply in numeric order against a clean [nxnative](https://github.com/seedds/nxnative) tree:

```sh
cd /path/to/nxnative
for p in /path/to/nxnative-ktgl-ports/patches/0*.patch; do
  git apply --check "$p" && git apply "$p"
done
```

If `git apply` fails on a newer upstream, open the `.patch` and merge by hand: each file header lists the paths touched.

| File | Summary |
|---|---|
| `01-jump-table-bounds-check.patch` | Rewriter: bounds check beside the load after a reloaded index |
| `02-audren-silent-fallback.patch` | Host: SDKs without an in-process audren renderer |
| `03-nvos-allocator-and-movie-buffer.patch` | Host: forward `SetGraphicsAllocator`; non-const `movie::Buffer` |
| `04-linkdata-idx-pairing.patch` | Tools: `LINKDATA.BIN` ↔ `LINKDATA.IDX` |
| `05-symptoms-index.patch` | Docs: symptom index lines |
| `06-launcher-chinese-ui.patch` | Launcher: Chinese UI when `DMLanguage` starts with `zh` |

These patches do **not** include Unreal pak v9 / Xuan Yuan Sword work. That belongs in other notes.
