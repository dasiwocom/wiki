# Linux GNOME 让 fcitx5 开机自启动教程

> 适用:GNOME 桌面(Debian/Ubuntu 等),Wayland / X11 均可。
> 已验证环境:Debian + GNOME Wayland,`/usr/bin/fcitx5` 已安装。

---

## 1. 原理:GNOME 的自启机制

- GNOME 开机登录时会扫描 **`~/.config/autostart/`** 目录下的所有 `.desktop` 文件。
- 每一个 `.desktop` 文件里的 `Exec=` 命令会被自动执行。
- 所有"开机自启软件"的通用做法 = 向这个目录放入一个合法的启动 desktop 文件。
- 想禁止某个自启:删掉文件,或删除（设空）`X-GNOME-Autostart-enabled`。

---

## 2. 两种自启方式

### 方式一(推荐):复制系统自带的 desktop
系统在 `/usr/share/applications/` 里已经内置了写好的 fcitx5 桌面项:

```bash
cp /usr/share/applications/org.fcitx.Fcitx5.desktop ~/.config/autostart/
```

它内部就是 `Exec=/usr/bin/fcitx5`,`ICON=fcitx`,直接复用最省事。
> 本质同 GNOME Tweaks「Startup Applications」图形化添加的操作。

### 方式二:手写极简 desktop
```bash
nano ~/.config/autostart/fcitx5.desktop
```
```ini
[Desktop Entry]
Type=Application
Name=fcitx5
Comment=启动输入法
Exec=/usr/bin/fcitx5
X-GNOME-Autostart-enabled=true
```

---

## 3. 关键一步:让 GNOME 接管 fcitx5 作为输入法

**光自启动不够。** GNOME 有自己独立的输入源体系(默认 ibus)。如果不把 fcitx5 接入进来,即使 daemon 启动了,应用里依然切不出中文输入法。

```bash
gsettings set org.gnome.desktop.input-sources sources "[('xkb','us'),('fcitx','fcitx5')]"
```

- `('xkb','us')` = 英文键盘输入源
- `('fcitx','fcitx5')` = 让 GNOME 把 fcitx5 作为第二个输入源
- 设置后:系统设置 → 键盘 → 输入源 里可看到 fcitx5,按 Super+空格 或默认快捷键切换中英文。
- 查看当前值:`gsettings get org.gnome.desktop.input-sources sources`

---

## 4. 常见坑

1. **用了 `fcitx5-wayland-launcher.desktop`**:系统里有这个文件,但它标注 "Experimental / OnlyShowIn=KDE",是给 KDE 的试验品,GNOME 下不要用,直接 `Exec=/usr/bin/fcitx5`。
2. **只自启、没设 input-sources**:GNOME 不接管,应用里切不了输入法(最常犯)。
3. **改了没生效**:注销重登(autostart 只在登录时读取);不要指望运行时立刻生效。
4. **想在交互会话里临时测**:直接 `fcitx5 &` 或应用列表点 Fcitx5 图标。

---

## 5. 验证

```bash
pgrep -a fcitx5     # 有进程输出 = 自启成功
```
注销重登后,屏幕右上角应出现键盘/输入法托盘图标,能切中英文即可。

---

## 6. 附:自启文件通用模板(其他软件同理)

```ini
[Desktop Entry]
Type=Application
Name=任意名字
Comment=说明
Exec=/绝对/路径/命令
X-GNOME-Autostart-enabled=true
```
放进 `~/.config/autostart/` 即开机自启动;同样能复制 `/usr/share/applications/` 下已存在的 desktop 文件复用。