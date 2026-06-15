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
