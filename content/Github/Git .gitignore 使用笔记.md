> `.gitignore` 是一个特殊文件，放在**仓库根目录**，用来告诉 Git：哪些文件/文件夹不要纳入版本控制，不要提交上传到 GitHub。 注意：**已经被Git跟踪过的文件，写进.gitignore不会自动删除，只对后续新文件生效**。

## 创建 .gitignore

进入仓库根目录：

```
cd /d/everything/projects/superwin
nano .gitignore
```

粘贴下面模板：

```
/target/
*.log
```

保存退出nano：`Ctrl+O` →回车 →`Ctrl+X`
### 问题：文件已经提交到GitHub，现在想忽略它

仅仅写进 `.gitignore` 没用，需要手动从git缓存移除：

```
git rm --cached 文件名
git commit -m "remove file from git track"
git push
```

> `--cached`：只从git版本记录移除，**本地磁盘文件保留不会删掉**。

### 问题：我不想提交某文件夹

用例子看，假设仓库结构是这样：

```
superwin/
├── target/            ← 构建产物，真正想忽略的
└── src/
    └── demo/
        └── target/    ← 假如某个子项目里也有个叫 target 的文件夹
```

- 写 `target/`：两个都被忽略
- 写 `/target/`：只有根目录那个被忽略，`src/demo/target/` 不受影响

所以前面的 `/` 是一把**限定范围的锁**：「只认根目录这一个，别的地方同名的不动」。