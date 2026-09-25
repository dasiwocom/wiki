## 1. 安装 flatpak（Arch 官方软件源）

```bash
sudo pacman -S flatpak
```

添加 flathub 远程源（仅需执行一次）：

```bash
flatpak remote-add --if-not-exists flathub [https://flathub.org/repo/flathub.flatpakrepo](https://flathub.org/repo/flathub.flatpakrepo)
```

## 2. 搜索应用

普通搜索（文本可能会被 `...`截断）：

```bash
flatpak search wechat
flatpak search qq
```

完整原始输出，通过 `| cat` 避免截断

```bash
flatpak search wechat | cat
flatpak search qq | cat
```

- `|`：管道，将左侧命令的输出传递给右侧命令
- `cat`：打印原始文本，去除终端表格格式
## 3. 安装应用
使用搜索得到的完整**应用 ID**

```bash
flatpak install flathub com.tencent.WeChat
flatpak install flathub com.qq.QQ
```
## 4. 运行 flatpak 应用

```bash
flatpak run com.tencent.WeChat
flatpak run com.qq.QQ
```

## 5. 实用命令
列出所有已安装的 flatpak 应用：

```bash
flatpak list
```

更新全部 flatpak 应用：

```bash
flatpak update
```

卸载 flatpak 应用：

```bash
flatpak uninstall com.tencent.WeChat
```

## 核心概念
- **应用 ID**：类似 `com.tencent.WeChat` 的由点分隔的长字符串。flatpak 使用该标识，而非简短显示名称。
- flathub：flatpak 的主要软件源。