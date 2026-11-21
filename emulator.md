# 在 PC 上跑真固件（PrimeShim）

*不接计算器也能验证：真固件、真文件系统、真按键，画面由模拟器截图。*

## 目录

1. [这是什么](#1-这是什么)
2. [准备](#2-准备)
3. [跑起来](#3-跑起来)
4. [启动自己的应用](#4-启动自己的应用)
5. [没有图形界面时怎么判断](#5-没有图形界面时怎么判断)
6. [已知限制](#6-已知限制)

## 1. 这是什么

[PrimeShim](https://github.com/nbprime-team/PrimeShim) 把 `PRIME_APP.DAT` 里的
ARM 固件解出来，在 PC 上直接执行：固件自己的界面、自己的文件系统、自己的后门
都在，应用在里面跑和真机上几乎一样。

它分两半：

- **guest**：真固件 + 一层把固件的 SVC 接到宿主功能的胶水，用 `qemu-arm`
  跑（ARMv5TE，soft-float，正好是 G1）；
- **宿主**：一个 SDL 窗口，或者一个 memfd 帧缓冲 + 管道输入（无头）。

我们用的分支：`nbprime-team/PrimeShim` 的 `nbprime/headless-bringup`。相对上游
多了"让后门加载的应用跑起来"的四处修复，细节见该仓库
`docs/HEAP_LOADED_APPS.md`。

## 2. 准备

- 一台 x86-64 Linux（WSL2 可以）；
- `PRIME_APP.DAT`（固件包里的那个，32 MB）；
- 本地 `arm-linux-gnueabi-gcc`、`qemu-arm`、SDL2。没有 root 也能装：
  `apt-get download` 之后 `dpkg-deb -x` 到一个目录，把 `bin` 和
  `usr/lib/x86_64-linux-gnu` 加进 `PATH` / `LD_LIBRARY_PATH` 就行。

## 3. 跑起来

```bash
# 1. 从固件容器里取出 ELF
python3 tools/fw_extract.py PRIME_APP.DAT -o firmware/20250915/armfir.elf

# 2. 准备 A: 盘（见下）
mcopy -s -i a.img ::/ runtime/prime_data/A/
cp runtime/prime_data/A/programs/misc/armfir.dat runtime/prime_data/A/dummy.DAT

# 3. 无头跑一次，把画面存成 PNG
python3 tools/run_keys.py out/x.png "0xB1@8" 14
```

**A: 盘里的 `dummy.DAT` 必须有。** 官方只发布 `programs/misc/armfir.dat`，
固件却按 `A:\dummy.DAT` 去读它；少了这个文件，固件能启动但**一个字都不显示**：
没有字体、没有帮助文本，只有一个空壳界面。

A: 卷在 `PRIME_APP.DAT` 的 `0x2000` 处，是个 FAT16。它的 BPB 里磁头数、每道
扇区数、总扇区数都是 0，`mtools` 会拒绝挂载，先把这三个字段补上（63/255/实际
扇区数）再 `mcopy`。

有窗口的话用 `build/pc/primeshim-sdl`；WSL2 上 `DISPLAY=:0` 不可用，走 WSLg 的
wayland：

```bash
SDL_VIDEODRIVER=wayland SDL_RENDER_DRIVER=software \
WAYLAND_DISPLAY=wayland-0 XDG_RUNTIME_DIR=/run/user/1000 \
build/pc/primeshim-sdl <qemu-arm> ... primeshim.elf --firmware firmware/.../armfir.elf
```

## 4. 启动自己的应用

应用装进 `runtime/prime_data/C/DATA/<名字>.hpappdir/`，然后在 HOME 里执行

```
STARTAPP("<名字>")
```

模拟器里没有键盘映射表可以直接打这条命令，用 `run_keys.py` 注入就够了：

| 键 | 码 | 键 | 码 |
| --- | --- | --- | --- |
| F9（应用库） | `0xB1` | F10（HOME） | `0xE047` |
| F7（alpha） | `0xB6` | Shift | `0x8B` |
| Esc | `0x01` | Enter | `0x0D` |
| ↑↓←→ | `0x03 0x05 0x02 0x04` | `()` / `""` 键 | `0x4E` / `0x30` |

注意大小写：HOME 默认大写，`F7` 在 `空 → 单次 → 固定 → 空` 之间循环，`Shift`
只翻转当前那次的大小写。要输小写名字，先 `F7 Shift F7` 进固定小写。

## 5. 没有图形界面时怎么判断

四个工具都是纯文本输出，专门给"读不了图"的场合：

| 工具 | 输出 |
| --- | --- |
| `tools/fbview.py` | 颜色直方图 + 亮度 ASCII 图（看整体布局） |
| `tools/fbband.py` | 任意一块画面，1:1 逐像素字符（能直接读出文字） |
| `tools/fbdiff.py` | 两帧差异的像素数与行范围（判断"按键进去了没有"） |
| `tools/run_keys.py` | 按时间注入按键，存帧（PNG + raw） |

## 6. 已知限制

- 只有 PC profile 能跑；Android 那份还是半成品。
- 宿主侧没有实现 `W_GETCWD`，所以固件返回的是垃圾字符串。launcher 现在会丢掉
  不可信的答案并回落到默认目录（`nb-code` 的 `_plausible_dir`），但别的程序
  如果信了它，还是会被带偏。
- 走 wayland 时只能软件渲染（Zink 在这个环境里起不来）。
- 固件里的浮点和 VFP 都不要碰：G1 没有 VFP，模拟器也不会有。
