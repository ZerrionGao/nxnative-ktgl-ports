# KTGL Switch → Mac / iPhone 移植笔记

本仓库记录两款光荣 KTGL 主机游戏在 [nxnative](https://github.com/seedds/nxnative) 上的移植与打包经验：

| 目录 | 游戏 | Title ID | 卡带语言 |
|---|---|---|---|
| [`dw8/`](dw8/) | 真・三國無双７ with 猛将伝 DX（真三国无双 7 猛将传） | `01009D600CA88000` | 含简体 / 繁体 / 日文 |
| [`wo3/`](wo3/) | 無双OROCHI２ Ultimate（无双大蛇 2 终极版） | `0100153006300000` | 仅日文 |

仓库内**只有**说明、临时 `game.tsv` / 着色器表，以及复现所需的宿主补丁。不含游戏本体、密钥、签名证书或成品 `.app` / IPA。

> 合规说明见 [NOTICE.md](NOTICE.md)。许可证为 [MIT](LICENSE)。

## 你需要准备什么

1. **Apple 芯片 Mac**，已装 Xcode（本笔记验证时使用过 Xcode 27 一带的 Metal 编译器）。
2. **合法持有的 Switch 卡带或数字版**对应 Title ID（见上表）。西方完全版等其它 Title ID **不能**混用本仓库的 profile。
3. **自备密钥**：`prod.keys`（以及需要时的 `title.keys`）。只放在本机，永不提交。
4. **nxnative 源码**：从上游克隆后，对本仓库 `patches/` 打补丁（见下），或使用已包含等价修复的分支。
5. （可选）**Apple Developer** 账号与真机：仅当你要签名安装到 iPhone 时。Mac 包可用 ad-hoc 签名本地运行。

Python 依赖通过 `uv` 按打包命令临时拉取：`lz4`、`capstone`、`fonttools`。本机需能跑 `uv` 与 `/opt/homebrew/bin/python3.14`（或改成你机器上的 Python 3.11+ 路径）。

## 目录里有什么

```
.
├── README.md                 # 本文件
├── NOTICE.md                 # 法律与范围
├── LICENSE                   # MIT
├── patches/                  # 打到 nxnative 上的补丁（按序号）
├── docs/
│   ├── packing.md            # 共用打包步骤
│   └── faq.md                # 常见问题
├── dw8/
│   ├── README.md             # 无双 7 过程与注意点
│   └── profile/              # 临时 profile（可复制进 Games/dw8/...）
└── wo3/
    ├── README.md
    └── profile/
```

## 环境：把 nxnative 准备好

```sh
git clone https://github.com/seedds/nxnative.git
cd nxnative
# 建议在干净工作区打补丁；冲突时以补丁意图为准手工合并
git am --3way /path/to/this-repo/patches/*.patch
# 或：for p in /path/to/this-repo/patches/*.patch; do git apply --index "$p" || git apply "$p"; done
```

补丁作用简述：

| 补丁 | 作用 | 主要相关游戏 |
|---|---|---|
| `01-jump-table-…` | 跳转表在索引被重载后仍认加载旁的范围检查 | wo3 |
| `02-audren-…` | 无进程内渲染器的 SDK 本地应答 `audren`（无声） | wo3（SDK 1.6） |
| `03-nvos-…` | 把游戏图形分配器装进 `NvOs`；接受非 const 的 `movie::Buffer` | wo3 |
| `04-linkdata-…` | `LINKDATA.BIN` 配对 `LINKDATA.IDX` | dw8（wo3 同样受益） |
| `05-symptoms-…` | 症状索引条目 | 文档 |
| `06-launcher-…` | `DMLanguage` 以 `zh` 开头时启动窗中文 | 两款可选 |

更细的机制说明在 nxnative 的 `docs/reference/`（rewriter / audio / movies / shaders）。

## 游戏文件与 keys 怎么放

在 **nxnative 仓库根**下为每款游戏建目录（名称可自定，下文用 `Games/dw8`、`Games/wo3`）：

```text
nxnative/
  Games/
    dw8/
      input/
        keys/
          prod.keys          # 自备，勿提交
          title.keys         # 若有
        YourGame-[01009D600CA88000][v0].xci   # 拼接后的完整卡带
      artifacts/             # 打包输出（.app / IPA），勿提交
    wo3/
      input/
        keys/
          prod.keys
        YourGame-[0100153006300000][v0].xci
      artifacts/
```

分段卡带（`00`/`01`/`02`/`03`）先拼成一份 XCI，再删分段原件以省空间。拼接时**按文件名显式列出四段**，不要用 `"00" in name` 这类子串判断（会漏掉 `00`）。拼完核对：总字节数、偏移 `0x100` 的 `HEAD`、`0xf000` 的 HFS0。

把本仓库的 profile 拷进去，供脚本复用（也可让脚本重新猜，再对照本仓库）：

```sh
# 以 dw8 为例；wo3 同理改路径与 Title ID
mkdir -p Games/dw8/input/01009D600CA88000_build/profile/01009D600CA88000
cp -R /path/to/this-repo/dw8/profile/. \
  Games/dw8/input/01009D600CA88000_build/profile/01009D600CA88000/
```

`.gitignore` 已覆盖 `input/`、`artifacts/`、`*.keys`、`*.xci`、`*.ipa` 等；请勿强行 `git add -f` 这些文件。

## 打包（摘要）

完整命令与开关说明见 [docs/packing.md](docs/packing.md)。在 nxnative 根目录：

```sh
uv run --no-project --python /opt/homebrew/bin/python3.14 \
  --with lz4 --with 'capstone>=5,<6' --with fonttools \
  python scripts/build_ipa_from_nsp.py Games/dw8/input \
  -k Games/dw8/input/keys/prod.keys \
  --platform mac-app --allow-other-metal --allow-damaged --gpu none \
  --language zh-Hans \
  -o Games/dw8/artifacts/apps/ShinSangokuMusou7.app --progress lines
```

无双大蛇 2：把路径换成 `Games/wo3`，`--language ja`（控制数据只有日文；装好后可把 `Info.plist` 的 `DMLanguage` 改成 `zh-Hans` 做启动窗中文，游戏内文本仍是日文）。

iOS：同一命令把 `--platform mac-app` 换成 `--platform ios`，输出改为 `.ipa`。未签名 IPA 不能直接装机；签名须用**你自己的**团队、描述文件，并带上 `com.apple.developer.kernel.increased-memory-limit`。细节见各游戏 README 与 [docs/faq.md](docs/faq.md)。

## 验证过什么、没验证什么

两款都曾在 Apple 芯片 Mac 上进到出帧循环；无双大蛇 2 在真机上进过开场影片与数百帧。**均未**做完整菜单 / 战斗 / 听感 / 长时间稳定性验收。无双大蛇 2 因 SDK 1.6 无进程内音频渲染器，当前宿主路径下**没有声音**。

## 常见问题

见 [docs/faq.md](docs/faq.md)。

## 贡献与范围

欢迎修正文档笔误、补丁对更新版 nxnative 的适配说明。请不要 PR 游戏数据、密钥或签名材料。若某款游戏已正式写入 nxnative 的 `dm/games/`，以官方 profile 为准，本仓库仅作过程笔记。
