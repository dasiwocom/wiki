## 推送上线（Git Bash 依次敲）

```
git add .
```

```
git commit -m "write homepage"
```

```
git push
```
## 本地预览（修改后先看效果再上线）

在 Git Bash 里，**确保在 wiki 目录**下，输入：

```
npx quartz build --serve
```

它会启动一个本地服务器，然后浏览器打开：

```
[http://localhost:8080](http://localhost:8080)
```