# 富士康财务工作台

这是富士康项目财务工作台的独立 GitHub Pages 仓库。

- 线上地址：<https://sinmenl.github.io/finance-dashboard/>
- GitHub 仓库：<https://github.com/sinmenl/finance-dashboard>
- 发布分支：`main`
- 发布目录：`docs/`
- 线上页面：`docs/index.html`

## 这个仓库只做什么

这个仓库只保存和发布富士康财务工作台的线上公开版。

WorkBuddy 可以：

1. 检查 Excel 是否被占用；
2. 运行现有生成器；
3. 将生成好的公开版 HTML 复制到 `docs/index.html`；
4. 仅提交 `docs/index.html`；
5. 推送本仓库并核验线上页面；
6. 用 Bark 报告成功、无变化或失败。

WorkBuddy 不可以：

- 读取或处理凭证文件；
- 修改任何 Excel；
- 生成或导出月账单；
- 删除文件或历史数据；
- 把财务 HTML 复制到其他 GitHub 仓库；
- 修改其他网页项目；
- 使用 `git add -A`、force push 或破坏历史的 Git 操作。

## 本地路径

### 财务源文件

```text
/Users/sinmenlin/Downloads/1.工作/
```

### 页面生成器

```text
/Users/sinmenlin/Downloads/测试/财务工作台生成.py
```

### 生成结果

```text
本地完整版：/Users/sinmenlin/Downloads/测试/财务工作台.html
线上公开版：/Users/sinmenlin/Downloads/测试/财务工作台_公开版.html
```

### WorkBuddy 部署仓库

```text
/Users/sinmenlin/.workbuddy/deploy/finance-dashboard
```

如果部署仓库不存在，只在首次运行时克隆：

```bash
git clone https://github.com/sinmenl/finance-dashboard.git /Users/sinmenlin/.workbuddy/deploy/finance-dashboard
```

如果目录已存在但不是 Git 仓库，必须停止，不得删除或覆盖目录。

## 同步模式

### 周同步

- 时间：每周三、周日 22:00；
- 前提：用户已经整理好凭证并更新 Excel；
- 任务：重建页面并上传公开版；
- 禁止：不读凭证、不写 Excel。

### 月同步

- 时间：每月 1 日 22:00；
- 前提：用户已经在《富士康项目月账单.xlsx》中生成上月 sheet；
- 任务：重建页面并上传含上月账单的公开版；
- 禁止：不生成、不导出、不修改月账单。

## 同步流程

### 1. 检查 Excel 锁文件

检查 `/Users/sinmenlin/Downloads/1.工作/` 中是否存在：

```text
.~富士康项目资金收支明细表.xlsx
.~富士康项目月账单.xlsx
```

只要任一锁文件存在，就不能同步：

- 不运行生成器；
- 不执行 Git 操作；
- Bark 通知“Excel 被占用，未同步”；
- 结束任务。

### 2. 核验仓库

在部署仓库执行：

```bash
git remote get-url origin
git status -sb
```

origin 必须等于：

```text
https://github.com/sinmenl/finance-dashboard.git
```

不一致时立即停止，不得复制、提交或推送文件。

### 3. 先拉取再生成

```bash
cd /Users/sinmenlin/.workbuddy/deploy/finance-dashboard
git -c http.version=HTTP/1.1 pull --rebase origin main
```

pull 失败则停止，不覆盖线上版本，不自动重试。

### 4. 重建公开版页面

```bash
cd /Users/sinmenlin/Downloads/1.工作
./.venv/bin/python "/Users/sinmenlin/Downloads/测试/财务工作台生成.py"
```

生成后必须检查公开版：

- 文件存在且非空；
- 包含 `noindex`；
- 页面标题是富士康财务工作台；
- 不包含其他网页项目的标题。

任一检查失败都不得部署。

### 5. 复制到独立财务仓库

```bash
cp "/Users/sinmenlin/Downloads/测试/财务工作台_公开版.html" \
  "/Users/sinmenlin/.workbuddy/deploy/finance-dashboard/docs/index.html"
```

如果 `docs/index.html` 没有变化：

- 不创建空提交；
- Bark 通知“线上内容已是最新”；
- 结束任务。

### 6. 只提交财务页面

```bash
cd /Users/sinmenlin/.workbuddy/deploy/finance-dashboard
git add docs/index.html
git diff --cached --stat
```

只允许出现 `docs/index.html`。如果出现其他文件，必须停止。

周同步提交：

```bash
git -c user.name="sinmenl" \
  -c user.email="sinmenl@users.noreply.github.com" \
  commit -m "finance: weekly sync"
```

月同步提交：

```bash
git -c user.name="sinmenl" \
  -c user.email="sinmenl@users.noreply.github.com" \
  commit -m "finance: monthly sync"
```

推送：

```bash
git -c http.version=HTTP/1.1 push origin main
```

push 失败时不自动重试，并通知真实失败原因。

### 7. 核验线上结果

访问：

```text
https://sinmenl.github.io/finance-dashboard/
```

成功标准：

- HTTP 状态为 200；
- 页面标题正确；
- 页面包含 `noindex`；
- 页面没有显示其他网页项目。

## Bark 通知

Bark 地址文件：

```text
/Users/sinmenlin/Downloads/测试/bark-url.txt
```

文件不存在时跳过 Bark，不得将 Bark 失败视为财务部署失败。

通知类型：

- 同步成功：“财务工作台已同步”，附上时间和线上地址；
- 没有变化：“线上内容已是最新”；
- Excel 被占用：“Excel 被占用，未同步”；
- Git 失败：写明失败在 pull、commit 还是 push；
- 线上核验失败：“已推送，但线上页面未通过核验”。

## WorkBuddy 自动化怎么写

周任务：

```text
每周三、周日 22:00 同步富士康财务工作台。

使用仓库：
/Users/sinmenlin/.workbuddy/deploy/finance-dashboard

如果仓库不存在，先克隆：
https://github.com/sinmenl/finance-dashboard.git

开始前必须完整读取仓库 README.md，然后按“周同步”模式执行。

不处理凭证，不修改 Excel，不删除文件，不访问其他网页仓库。
```

月任务：

```text
每月 1 日 22:00 同步富士康财务工作台。

使用仓库：
/Users/sinmenlin/.workbuddy/deploy/finance-dashboard

如果仓库不存在，先克隆：
https://github.com/sinmenl/finance-dashboard.git

开始前必须完整读取仓库 README.md，然后按“月同步”模式执行。

上月月账单由用户自行生成。不生成月账单，不修改 Excel，不删除文件，不访问其他网页仓库。
```

## 隐私说明

该仓库目前为公开仓库。`noindex` 和 `robots.txt` 只能降低搜索引擎收录概率，不能限制知道网址的人访问。

线上页面必须使用生成器产出的公开版，不得上传本地完整版。
