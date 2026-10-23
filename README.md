# proton-wrapper

一直以来，用 Proton 运行 exe 大多通过添加非 Steam 游戏或者Lutris之类的方式，难以做到`双击直接运行`般无缝的交互体验，所以我制作了 `shorin-proton-wrapper`。最大的特点是安装后可以直接运行 exe，无须任何前置操作。

>注意，这只是个包装器，原本 Proton 运行不了的软件依旧会运行不了。

![](./pics/1.png)

灵感来自 Wine 的配置过程。安装 Wine 后运行 `winecfg` 会在 `~/.wine` 目录初始化前置运行环境。我借鉴并改良了这个设计，运行 exe 的默认行为是检测 `~/.proton` 是否存在，如果不存在则静默初始化。默认运行器是 [`WE-Proton`](https://github.com/SHORiN-KiWATA/we-proton)：它基于 `GE-Proton` 和 `DW-Proton`，额外修了一些 Wine 的 bug，能运行 WeGame 和大部分国服游戏、二次元游戏，还内置了游戏加速器 TUN 模式需要的 wintun，支持雷神加速器和网易 UU 加速器（见下面的「独立网络」）。

运行器默认装在 `~/.local/share/Steam/compatibilitytools.d`。已经有时直接使用，不会重复下载；`--update` 会一并更新这几个位置里的 Latest 运行器。

首次自动安装运行器时，临时下载文件会放在 `~/.cache/shorin-proton-wrapper/downloads`，安装成功后清理。

安装失败时会弹出常驻通知说明原因，完整日志在 `~/.cache/shorin-proton-wrapper/logs/install-*.log`，前置环境初始化日志在 `~/.cache/shorin-proton-wrapper/logs/init.log`。

Proton 会放进它要求的 Steam Linux Runtime 容器里运行（WE-Proton、DW-Proton 和 GE-Proton 11 要 SLR 4.0，GE-Proton 10 要 sniper），否则自带的 GStreamer 插件等会缺库，视频和音频解码之类的功能出问题。已装 umu 或 Steam 的运行时会直接复用，没有就从 Valve 下载到 `~/.local/share/shorin-proton-wrapper/runtime`。

32 位程序默认用 Wine 的新 WoW64 模式运行（`PROTON_USE_WOW64=1`），不需要安装 32 位系统库。个别 32 位程序有问题时，可以在它的配置里把环境变量设成 `PROTON_USE_WOW64=0`。

可调环境变量：`SHORIN_PROTON_WRAPPER_DOWNLOAD_STALL`（卡死判定秒数，默认 120）、`SHORIN_PROTON_WRAPPER_DOWNLOAD_TIMEOUT`（下载总时长上限秒数，默认 0 表示不限制）、`SHORIN_PROTON_WRAPPER_RUNTIME`（设为 0 时不用 Steam 运行时，直接在宿主机上运行 Proton）。

## 安装

依赖：`bash` `python` `python-gobject` `gtk4` `curl` `icoextract` `python-pillow`

可选：`gamescope` `passt`（独立网络） `gcc`（独立网络里的 Steam 覆盖层） `mangohud` `libnotify`（通知） `wget` `steam` `lutris` `lib32-cairo`（`PROTON_USE_WOW64=0` 时）

Debian / Ubuntu / Fedora 从 [Releases](https://github.com/SHORiN-KiWATA/proton-wrapper/releases/latest) 下载安装包，依赖会一并装上：

```
sudo apt install ./shorin-proton-wrapper_*_all.deb       # Debian / Ubuntu
sudo dnf install ./shorin-proton-wrapper-*.noarch.rpm    # Fedora
```

Arch Linux：

```
yay -S shorin-proton-wrapper-git
```

其他发行版装好依赖后：

```
git clone https://github.com/SHORiN-KiWATA/proton-wrapper
cd proton-wrapper
sudo install -Dm755 -t /usr/local/bin shorin-proton-wrapper shorin-proton-wrapper-configure shorin-proton-wrapper-manager
sudo install -Dm644 -t /usr/local/share/applications *.desktop
sudo install -Dm644 icons/shorin-proton.png /usr/local/share/icons/hicolor/512x512/apps/shorin-proton.png
```

## 设置Windows程序运行环境

>`shorin-proton-wrapper-configure`

为了兼顾自由度，我额外做了一个名为 `设置Windows程序运行环境` 的打开方式。支持自定义前置环境目录、Proton 版本、MangoHud、GameScope 等常用设置项。`启动参数` 会追加在程序路径之后，例如鸣潮指定画质用的 `-krqlv=hd`。

![](./pics/2.png)

![](./pics/3.png)

第一次设置之后会将设置项缓存至 `~/.cache/shorin-proton-wrapper`，以后使用 `运行Windows软件` 打开会自动读取缓存配置，如果再次用 `设置Windows软件运行环境` 打开，则会提示 `已保存的运行环境`，可自行决定要重新设置还是直接运行。

![](./pics/4.png)

设置项里还有「以 Wayland 运行」：使用 Wine 的 Wayland 驱动，开 Gamescope 时不生效。用 Gamescope 运行时，如果桌面在用 fcitx5，会自动让 fcitx5 也接管 Gamescope 的显示，在 Gamescope 里也能打中文。

## 同一个前置环境里同时运行多个程序

前置环境里已经有程序在运行时，再打开别的程序会直接加入，不用等前面的程序退出。加入的程序沿用会话正在用的运行器和显示方式，和自己的设置不同时会弹出通知说明：

- 运行器不同时，改用会话正在用的那个（两个版本的 Wine 不能共用一个会话）；
- 会话以 X11 运行时，`以 Wayland 运行`和 `Gamescope` 不生效；会话以 Wayland 运行时，加入的程序也以 Wayland 运行；
- 会话在 Gamescope 里时，其他程序不能加入。

## 独立网络（游戏加速相关）

在管理器的前置环境一栏打开`独立网络`（或者 `shorin-proton-wrapper -p 前置环境 --netns on`）后，这个前置环境的程序会运行在独立的网络命名空间里。把 Windows 版的游戏加速器（例如雷神加速器的`模式三`或网易 UU 加速器的`路由模式`）和要加速的游戏装进同一个前置环境，先打开加速器、再打开游戏。

- 需要安装 `passt`（提供 `pasta`），运行器用 WE-Proton；
- 游戏要和加速器在同一个前置环境里；

Linux 程序也可以放进这个独立网络，让加速器接管它的流量：把 exe 换成 Linux 程序即可，例如

```
shorin-proton-wrapper steam
shorin-proton-wrapper -p 前置环境 curl https://store.steampowered.com
```

- 它以当前用户的身份运行（独立网络里其他程序看到的是 root，Steam 这类程序不肯以 root 运行）；前置环境没开独立网络时直接运行；
- 用这种方式启动的 Steam 整个在独立网络里，要先退出已经开着的 Steam（Steam 只能开一个）。它启动的 Windows 游戏照样用 `shorin-proton-wrapper %command%`，会加入同一个前置环境的会话；
- 独立网络里的 Steam 游戏照样有 Steam 覆盖层，不管 Steam 在独立网络里还是外面。覆盖层要靠一个小辅助库，wrapper 第一次用到时用系统的 C 编译器（`cc`，Arch 上装 `gcc`）编译；没有编译器时游戏照常运行，只是没有覆盖层；
- 加速器自己决定接管哪些连接。例如网易 UU 的 Steam 加速只放行商店和社区的域名，在海外使用时，和它们共用 IP 的商店图片、脚本会被断开，商店页面加载不出来。

## Steam 游戏

在 Steam 里把游戏的启动选项设为：

```
shorin-proton-wrapper %command%
```

游戏就不再用 Steam 给它建的前置环境，而是作为 `~/.proton` 里的程序运行（另一个前置环境用 `shorin-proton-wrapper -p 前置环境 %command%`）：和那里的其他程序共用一个会话、运行器和独立网络，比如让加速器接管 Steam 游戏。Linux 原生游戏用同样的启动选项，会在这个前置环境的独立网络里运行。

- 游戏的`兼容性`里仍要选一个 Proton（建议 WE-Proton Latest），否则 Steam 不会启动 Windows 游戏；实际运行用的是前置环境的运行器。wrapper 装的 Latest 运行器在 Steam 里的名字是固定的（版本号显示在括号里），更新之后这个选择不会丢；
- 存在 Windows 用户目录（AppData、文档）里的存档会在 `~/.proton` 里。Steam 云存档同步的是 Steam 自己的前置环境（`steamapps/compatdata/<AppID>`）；
- 游戏会出现在管理器的应用列表里，标着「Steam」：在那里点「运行」会交给 Steam 启动（`steam://rungameid/<AppID>`），配置、日志照常可用。

## Shorin Proton Wrapper 管理器

>`shorin-proton-wrapper-manager`

每次运行 exe 都会自动进入管理器的仓库，通过 `icoextract` `python-pillow` 生成缩略图。支持导出程序到 Linux 应用菜单、重命名显示名称和 `.desktop` 名称、修改配置等功能；配置里的 `启动参数` 会追加在程序路径之后。

应用、目录和进程页面都可以搜索：在窗口里直接打字或按 Ctrl+F 开始，Esc 清空。应用列表来自前置环境的开始菜单和桌面快捷方式，同一个程序只列一次；最近运行过的排在前面，卸载程序和目标已不存在的快捷方式排在最后。

![](./pics/5.png)

在进程卡死或者后台进程无法退出的时候可以在进程页面强制退出。

![](./pics/6.png)

## shorin-proton-wrapper

另外还提供了 CLI 工具方便从终端进行 Proton 管理、exe 运行等操作。

```
shorin-proton-wrapper 

用法:
  shorin-proton-wrapper [选项] app.exe [参数...]
  shorin-proton-wrapper [-p 目录] 命令 [参数...]     在前置环境的独立网络里运行 Linux 程序

选项:
  --init                     初始化，默认路径 ~/.proton
  -p, --prefix 目录          指定前置环境路径，若不存在则初始化
  --status                   显示前置环境信息
  --list-runner              列出已安装的运行器
  --runner 路径|ge|dw|we     指定运行器
  --install ge|dw|we         安装运行器，默认安装至 ~/.local/share/Steam/compatibilitytools.d
  --install-steam ge|dw|we   同 --install
  --install-lutris ge|dw|we  给 Lutris 安装运行器
  --download ge|dw|we        仅下载运行器，默认下载路径为 XDG 下载目录
  --list-version ge|dw|we    列出可使用的运行器版本
  --version 版本             指定特定版本，默认 latest
  --update                   升级所有已安装的 Latest 运行器
  --check-update             检查运行器是否有可用更新（不安装）
  --doctor                   检查 wrapper 运行环境并输出诊断信息
  --remove ge|dw|we          移除默认位置中已安装的运行器
  --remove-steam ge|dw|we    移除 Steam 中指定运行器，省略 ge|dw|we 则移除全部由 wrapper 安装的运行器
  --remove-lutris ge|dw|we   移除 Lutris 中指定运行器，省略 ge|dw|we 则移除全部由 wrapper 安装的运行器
  --gs, --gamescope          用 Gamescope 运行。默认全屏；需要自定义分辨率、帧率、FSR 或遇到显示兼容问题时使用
  --mangohud                 启用 MangoHud 性能叠加层
  --wayland                  以 Wayland 运行（Proton 运行器，开 Gamescope 时不生效）
  --netns on|off             给前置环境开启独立网络命名空间（加速器用），需要 passt
  --debug                    将完整调试日志写入 ~/.cache/shorin-proton-wrapper/logs
  --help-full                显示完整说明

示例:
  shorin-proton-wrapper --init
  shorin-proton-wrapper /path/to/app.exe
  shorin-proton-wrapper --runner /path/to/proton /path/to/app.exe
  shorin-proton-wrapper -p /path/to/prefix /path/to/app.exe
  shorin-proton-wrapper --install we
  shorin-proton-wrapper --install-steam we
  shorin-proton-wrapper --download ge --version GE-Proton10-34
  shorin-proton-wrapper --gamescope --res 1920x1080 --fps 30 --fsr /path/to/app.exe
  shorin-proton-wrapper --debug /path/to/app.exe
  shorin-proton-wrapper --list-version we
  shorin-proton-wrapper --doctor
  shorin-proton-wrapper --update
```
```
shorin-proton-wrapper --gamescope 

用法:
  shorin-proton-wrapper --gamescope [Gamescope 选项] app.exe [参数...]

Gamescope 选项:
  --windowed             窗口模式
  --fullscreen           全屏模式 (默认)
  --res 宽x高            设置游戏渲染分辨率，例如 1920x1080
  --fps 帧率             限制帧率
  --fsr                  启用 FSR 超分辨率
  --args 参数            追加自定义 Gamescope 参数
  --mangohud             启用 MangoHud 性能叠加层
  -h, --help             显示本帮助

示例:
  shorin-proton-wrapper --gamescope --windowed --res 1920x1080 --fps 30 --fsr --mangohud game.exe
```

## 赞助

如果这个项目帮到了你，欢迎请我喝杯咖啡 ☕ 你的支持会让它持续更新下去。

<img src="https://raw.githubusercontent.com/SHORiN-KiWATA/.github/main/images/wechat.jpg" width="200" alt="微信赞赏码"> <img src="https://raw.githubusercontent.com/SHORiN-KiWATA/.github/main/images/alipay.png" width="134" alt="支付宝收款码">

手机上看可以打开 [赞助页](https://github.com/SHORiN-KiWATA/.github/blob/main/SPONSOR.md)，支付宝能直接点链接付款。
