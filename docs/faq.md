# 常见问题

## 提示没有游戏 profile / no game profile

打包与 `make -C dm/host` 都必须指定游戏。临时 profile 放在  
`Games/<id>/input/<titleId>_build/profile/<titleId>/`，并通过环境变量让工具优先读到：

```sh
export DM_GAMES_DIR="$PWD/Games/<id>/input/<titleId>_build/profile"
export DM_GAMES_FIRST=1
export GAME=<titleId>
```

`make -C dm/host` 时 `DM_GAMES_DIR` 请用**绝对路径**。

## 解包停在 hash check failed

个别文件可能本身损坏。使用 `--allow-damaged`，把损坏路径记在游戏笔记里。无双 7 已知：`dlc/13169.bin.gz`、`dlc/13170.bin.gz`。

## 着色器数量少一个数量级

根目录是 `LINKDATA.BIN` + `LINKDATA.IDX` 时，需已应用本仓库 `04-linkdata-idx-pairing` 补丁。旧宿主只认 `DATA1.bin` / `LD_*.BIN` 配对。

## 改写断言 `jump table <addr>`

常见于索引先拷贝、中间又重载、真正 `cmp`/`b.hi` 在 load 旁边的情况。应用 `01-jump-table-bounds-check`。

## 点 Play 立刻 Break / 退出码 42，日志有 audren:u

SDK 1.6 一类没有进程内 `ExecuteAudioRendererRendering`。应用 `02-audren-silent-fallback`。游戏能继续，但**没有声音**——这不是配置漏开混音器，而是该 SDK 路径下宿主故意不连服务。

## 开场影片附近 `sig=11 addr=0`，PC 在 NVN 模块

宿主只记录了 `SetGraphicsAllocator`、没有调用模块原函数时，`NvOs` 仍用默认小堆。应用 `03-nvos-allocator-and-movie-buffer`。

## `FATAL movie: missing SDK symbol _ZNK5movie6Buffer4DataEv`

SDK 1.6 的 `Data` / `Capacity` / `Size` 是非 const 符号。同一补丁 `03` 会回退到非 const 名字。

## 启动窗是英文

`Info.plist` 的 `DMLanguage` 必须以 `zh` 开头（如 `zh-Hans`）才会走中文 UI（需补丁 `06`）。NACP 未声明的语言不能靠 `--language` 硬写进打包脚本；可先按卡带语言打包，再改 plist。

## 无双大蛇 2 游戏内仍是日文

该日版卡带只有日文数据；主程序不查询系统语言。改 `DMLanguage` 只影响启动窗与宿主报告的系统语言，**不会**变成中文版文本。Switch 上也没有官方中文语言 DLC。

## Metal 版本对不上

使用 `--allow-other-metal`，并用 `--gpu none`，不要打入错误编译器生成的离线 PSO。

## 装到手机后变成空壳 / 没游戏数据

不要安装 DerivedData 里的 app，也不要安装 `DM_COPY_ROMFS=NO` 打的壳。必须用含 `romfs.img` 的 Payload。

## 大内存游戏被系统杀掉

描述文件与 entitlements 都要有 `com.apple.developer.kernel.increased-memory-limit`。
