## 三、解压 notebook.zip（含超长路径）

- 背景：F 盘（U 盘）只有 2 个东西：`System Volume Information` + `notebook.zip`（2.6GB）。
- 现象 1：进 F 盘鼠标转圈。
  - 原因：U 盘速度慢 + 2.6GB 大文件 + Windows Defender 实时扫描大压缩包。
- 现象 2：直接对 F:\notebook.zip 用 `Expand-Archive` 解压到 D:\notebook → 超过 10 分钟未完成。
  - 原因：U 盘读速度慢；且 `Expand-Archive` 遇到深层路径会极慢。
- 解决 1：先把 zip 复制到本地磁盘（D:\notebook.zip，复制成功），再本地解压。
  - 但本地 `Expand-Archive` 仍超时。
- 解决 2：改用 `tar`：

```
Remove-Item -LiteralPath "D:\notebook" -Recurse -Force
New-Item -ItemType Directory -Path "D:\notebook" -Force
tar -xf "D:\notebook.zip" -C "D:\notebook"
```

- 报错：大量 `Can't create '...terminfo\c\c321': Invalid argument`（共 3205 行错误）。
  - 根因：zip 内含 `electron\dist\win-unpacked\resources\python\share\terminfo\...`
    这类超深嵌套路径，超过 Windows 默认 260 字符路径上限（MAX_PATH）。
- 解决 3：解压到更短的根目录 `D:\nb` 减小路径长度：

```
tar -xf "D:\notebook.zip" -C "D:\nb"
```

- 结果：成功解出 83,579 个文件；仅 Python terminfo / bin 等少量超长路径文件解压失败（不影响项目运行）。
- 收尾：把内容移入 D:\notebook，清理 `D:\notebook.zip` 与 `D:\nb`。
- 附：压缩包内还有残留的 `.git`、`.claude`、数据库文件（backend\\notebook.db 等）。
