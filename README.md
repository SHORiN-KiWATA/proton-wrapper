# proton-wrapper
方便命令行使用 Proton 运行 exe 的简易包装器。

## 快捷方式

运行 Windows 安装包后，wrapper 会自动扫描当前 prefix 中的 `.lnk` 快捷方式，并导出到 Linux 应用菜单：

```sh
shorin-proton-wrapper -p ~/Games/foo-prefix ~/Downloads/setup.exe
```

也可以手动重新扫描：

```sh
shorin-proton-wrapper -p ~/Games/foo-prefix --export-shortcuts
```

## Gamescope 显示适配

自定义环境选择器支持 Gamescope、全屏、游戏分辨率、帧率限制和 FSR。CLI 也可以直接使用：

```sh
shorin-proton-wrapper --gamescope --gamescope-res 1280x720 --gamescope-fps 60 --gamescope-fsr game.exe
```

FSR 会让游戏以较低分辨率渲染后放大到屏幕，通常可提升性能，但可能降低清晰度。
