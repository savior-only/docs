---
title: Github Action 简单实践
url: https://blog.imkasen.com/github-action-practice/
clipped_at: 2024-03-26 17:20:34
category: default
tags: 
 - blog.imkasen.com
---


# Github Action 简单实践

## [](#%E7%9B%AE%E6%A0%87 "目标")目标

假定当前项目下有一个 `script.py` 爬虫脚本，我们想要定期执行这个 Python 爬虫，并将运行结果（JSON 文件）推送回仓库。

## [](#%E7%BC%96%E5%86%99 "编写")编写

首先需要在项目的根目录下创建 `.github/workflows` 文件夹，并且在文件夹下创建 YAML 文件用于编写执行脚本，如 `action.yml` 。

### [](#%E5%90%8D%E7%A7%B0 "名称")名称

首先先要给整个工作流程设置一个名称：

```yaml
name: action workflow practice
```

### [](#%E8%A7%A6%E5%8F%91%E6%9D%A1%E4%BB%B6 "触发条件")触发条件

接着设定该 Github Action 要在什么时候触发。我需要这个爬虫脚本每天 12 点自动运行，于是：

```yaml
on:
  schedule:
  - cron: '0 4 * * *'
```

这里要注意的是时区问题，cron 用的是 UTC 时间，所以我们通常需要用北京时间减 8 小时。

触发条件还有许多种，例如推送时运行 `push:` 等。

### [](#%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F "环境变量")环境变量

这里设定环境变量是因为接下去需要用到本地时间的设定。

```yaml
env:
  TZ: Asia/Shanghai
```

### [](#%E5%B7%A5%E4%BD%9C%E6%8C%87%E4%BB%A4 "工作指令")工作指令

接下去就要正式设定整个工作流程的具体命令了。格式如下：

```yaml
jobs:
  job1:
    ...
  job2:
    ...
```

`jobs` 下可以包含一个或多个任务，任务之间可以顺序的执行，也可以同时执行。这里我只需要执行一个任务即可。

#### [](#%E8%BF%90%E8%A1%8C%E7%8E%AF%E5%A2%83 "运行环境")运行环境

在单个任务中，首先需要设置的是虚拟的运行环境，如：

```yaml
job1:
  runs-on: ubuntu-18.04
```

#### [](#%E6%AD%A5%E9%AA%A4 "步骤")步骤

接下来就要设置每个步骤的指令，首先要 checkout the repository under `$GITHUB_WORKSPACE` :

```yaml
steps:
      - name: Checkout
        uses: actions/checkout@v2
```

因为是 Python 爬虫，所以需要安装对应的依赖：

```yaml
- name: Set up Python
  uses: actions/setup-python@v2
  with:
    python-version: 3.7
- name: Install requirements
  run: |
    python3 -m pip install --upgrade pip
    pip3 install -r ./requirements.txt
- name: Running
  run: python3 ./script.py
```

最后，需要把生成的文件提交回仓库：

```yaml
- name: Commit
  env:
    GIT_NAME: ${{ secrets.GIT_NAME }}
    GIT_EMAIL: ${{ secrets.GIT_EMAIL }}
  run: |
    git config --local user.name $GIT_NAME
    git config --local user.email $GIT_EMAIL
    git add .
    git commit -m "Github action update at `date '+%Y-%m-%d %H:%M:%S'`."
- name: Push
  uses: ad-m/github-push-action@master
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}
    branch: master
```

这里有几点需要注意：

1.  首先，很多教程里 git 的名字和邮箱都是直接明文写在了配置文件里，这样其实有隐私泄露的风险。正确的做法应该是将信息写入到仓库的 `Setting` - `Secrets` 中并在运行时读取。
    
2.  如果不想把提交设置为自己，可以配置成：
    
    ```yaml
    GIT_NAME: GitHub Actions[Bot]
    GIT_EMAIL: 41898282+github-actions[bot]@users.noreply.github.com
    ```
    

## [](#%E5%AE%8C%E6%95%B4%E7%9A%84%E9%85%8D%E7%BD%AE%E4%BB%A3%E7%A0%81 "完整的配置代码")完整的配置代码

```yaml
name: Action Practice

on:
  push:
  schedule:
    - cron: '0 0 * * *'

env:
  TZ: Asia/Shanghai

jobs:
  # a single job
  a-job-name:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2
      - name: Set up Python3
        uses: actions/setup-python@v2
        with:
          python-version: '3.x'
      - name: Install requirements
        run: |
          python3 -m pip install --upgrade pip
          pip3 install -r ./requirements.txt
      - name: Run
        run: python3 ./script.py
      - name: Commit
        env:
        GIT_NAME: ${{ secrets.GIT_NAME }}
        GIT_EMAIL: ${{ secrets.GIT_EMAIL }}
        run: |
          git config --local user.name $GIT_NAME
          git config --local user.email $GIT_EMAIL
          git add .
          git commit -m "Github action update at `date '+%Y-%m-%d %H:%M:%S'`."
      - name: Push
        uses: ad-m/github-push-action@master
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          branch: master
```
