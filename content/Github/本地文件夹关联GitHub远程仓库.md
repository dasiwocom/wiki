> 适用场景：GitHub上已经建好仓库，电脑本地已经存在项目文件夹，需要把两者绑定。 前提：本机已经配置好GitHub SSH密钥，可正常 `ssh -T git@github.com` 连通。

## 1、进入本地项目文件夹

```
cd /d/everything/projects/superwin
```

查看当前位置确认：

```
pwd
```

## 2、把文件夹初始化为Git仓库

```
git init
```
## 3、配置提交者身份（每台电脑只配一次）

> 第一次用 git 提交时会报 `Author identity unknown`，因为 git 要知道每条提交记录的「作者」是谁。配过一次就全局生效，以后换项目不用再配。

```
git config --global user.name "你的GitHub用户名"
git config --global user.email "你在GitHub上绑定的邮箱"
```

- **邮箱要和 GitHub 账号绑定的一致**，提交才能在 GitHub 上正确归属到你的账号（头像、贡献记录）。
- 不想暴露真实邮箱：GitHub → Settings → Emails → 勾选 "Keep my email addresses private"，用页面提供的 `xxxxx+用户名@users.noreply.github.com`。
- 验证配置：`git config --global user.name` 会回显刚设置的值。
- 修改 = 重新执行一遍同名命令，后写的覆盖先写的。
- 注意：这只管「提交署名」，和推送权限（SSH 密钥）是两回事。

## 4、绑定远程仓库，给远程地址设置别名 origin

```
git remote add origin git@github.com:dasiwocom/superwin.git
```

 `origin`：远程仓库的别名，只是一个代号，代表后面这一串GitHub地址。后续所有操作直接写`origin`就代表这个远程仓库。
 如果报错 `fatal: remote origin already exists.`，代表这个仓库之前绑定过远程，先删除旧绑定：
  ```bash
 git remote remove origin
  ```

查看已经绑定的远程信息：

```
git remote -v
```

## 5、修改本地分支名称为main（GitHub默认分支名）

旧版本git初始化默认分支叫master，GitHub默认main，统一名字避免冲突。

```
git branch -M main
```

## 6、分两种情况处理

### 情况A：GitHub网页新建仓库时勾选了README / LICENSE / gitignore

👉远程仓库已经有文件，本地是一套独立git历史，需要先拉取合并

```
git pull origin main --allow-unrelated-histories
```

参数解释：

- `git pull`：拉取远程文件并且自动合并
- `origin main`：拉取origin这个远程的main分支
- `--allow-unrelated-histories`：允许两套互不相关的git提交历史进行合并。

### 情况B：GitHub仓库完全空白，没有任何文件

不要执行pull，跳过这一步。

## 7、提交本地文件

### 查看当前状态（非常常用）

```
git status
```

标记含义（git 原生输出）：

- `Untracked files` 下面的：全新未跟踪文件，git 还没在管
- `Changes not staged` / `modified:`：已跟踪文件被修改

> 提交前确认 `.gitignore` 已建好（Rust 项目的 `target/` 必须挡住），否则 `git add .` 会把构建产物全部提交。建法见《Git .gitignore 使用笔记》。

把文件夹内文件交给git管理，生成本地提交记录

```
git add .
git commit -m "initial commit"
```

> 如果文件夹完全为空，git不允许空提交，需要新建至少一个文件：

```
echo "# superwin" > README.md
git add README.md
git commit -m "initial commit"
```

## 8、推送到GitHub，绑定上游

```
git push -u origin main
```

 `-u`(set-upstream)：设置上游关联，绑定完成后，后续操作就不用写完整地址分支。
> 后续日常提交只需要简短命令：

```
git add .
git commit -m "修改说明"
git push
```

> 拉取远程更新只需要：

```
git pull
```

## 9、常用排查命令

1. 查看git状态

```
git status
```

2. 查看分支

```
git branch
```

3. 查看远程仓库地址

```
git remote -v
```

## 10、切换文件夹（终端）

```
# 进入项目文件夹
cd /d/everything/projects/superwin

# 返回上一级目录
cd ..

# 回到家目录
cd

# 在两个目录来回跳转
cd -
```

## 关键概念备忘

1. `origin`：远程仓库别名，仅对当前这个本地git仓库生效，换文件夹就失效。
2. `main`：主分支名称，GitHub默认。
3. `git init`：**只需要执行一次**，不要重复执行。
4. `--allow-unrelated-histories`：只第一次合并两套独立历史时才用，以后正常pull不需要。

## 两种建仓库方案区分

- 方案1（本篇笔记）：本地文件夹已经存在 → `git init` + `git remote add origin xxx`
- 方案2：本地还没有文件夹，直接从GitHub下载仓库：

```
git clone git@github.com:dasiwocom/superwin.git
```

## 方案2 补充

- 在**上级目录**运行，clone 自动建文件夹（init / 绑远程 / 改 main / 设上游全替你做了），clone 完直接 `add / commit / push`。
- 文件夹默认叫仓库名，可自定义：`git clone 地址 名字`。事后改名也不影响——`.git` 才是仓库本体。
- 换新电脑记得先配 `user.name` / `user.email`（第 3 步），clone 不代劳。
