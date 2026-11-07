# 共用打包步骤

在已打好本仓库 `patches/` 的 nxnative 根目录执行。

## 1. 输入目录

- `Games/<id>/input/`：拼接后的 base XCI、可选 update NSP、`keys/`。
- 可选：事先放入 `…_build/profile/<titleId>/`（本仓库 `dw8/profile`、`wo3/profile`）。
- 没有 profile 时，`build_ipa_from_nsp.py` 会 survey 并猜一份临时配置；猜完后请与本仓库对照内存键、`modules`、`romfs_marker`。

## 2. Mac 应用

```sh
uv run --no-project --python /opt/homebrew/bin/python3.14 \
  --with lz4 --with 'capstone>=5,<6' --with fonttools \
  python scripts/build_ipa_from_nsp.py Games/<id>/input \
  -k Games/<id>/input/keys/prod.keys \
  --platform mac-app \
  --allow-other-metal \
  --allow-damaged \
  --gpu none \
  --language <lang> \
  -o Games/<id>/artifacts/apps/<Name>.app \
  --progress lines
```

| 开关 | 原因 |
|---|---|
| `--allow-other-metal` | 本机 Metal 编译器版本常与仓库锁定版本不一致 |
| `--gpu none` | 不对齐的离线 PSO 归档不要打进包；首次启动本机编译 |
| `--allow-damaged` | 个别 RomFS 文件哈希损坏时仍写入读到的字节（无双 7 的两份 dlc） |
| `--language` | 写入 `Info.plist` 的 `DMLanguage`；必须是该卡 NACP 声明的语言之一 |

Mac 包为 ad-hoc 签名即可本地打开。

## 3. 未签名 IPA

同一命令，`--platform ios`，`-o …/artifacts/ipas/<Name>.ipa`。

产物仅对可执行文件做了 ad-hoc 签名，**不能**直接装到手机。Xcode DerivedData 里的 `DmIOS.app` **没有** `romfs.img`，不能当安装包。

## 4. 真机签名（自备证书）

1. 用**你自己的** Bundle ID 建一个空 iOS 工程，打开 `increased-memory-limit`，对目标设备编一次 Release，拿到描述文件。
2. 解压 IPA 得到带 `romfs.img` 的 `Payload/DmIOS.app`。
3. 改 `CFBundleIdentifier` 为你的 Bundle ID；写入 `embedded.mobileprovision` 与授权（须含提高内存上限）。
4. `codesign --force --sign "<你的开发身份>" --entitlements … --generate-entitlement-der` 签整个 `.app`。
5. `xcrun devicectl device install app --device <你的设备> …/Payload/DmIOS.app`。

不要提交描述文件、entitlements 里的 team、设备 UDID 或证书。

## 5. 打包后清理

nxnative 的 `AGENTS.md` 约定：包能玩之后删除解包缓存、着色器缓存、`patched_v0`、日志、Xcode 派生数据、分段原件；每种成品只留最新一份 Mac `.app` 与一份可安装手机包；保留拼接 XCI、`keys/`、profile、说明与验收截图。
