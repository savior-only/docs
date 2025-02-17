
# git 创建新分支，查看分支，删除分支，切换分支等问题

### 一、使用背景

      总所周知，在日常开发中，我们需要把测试服务器和正式服务器分开。相应的，为了保持正式版本能正常运行，我们需要新建一个 git 分支用来专门的存放正式版 APP 的[源码](https://so.csdn.net/so/search?q=%E6%BA%90%E7%A0%81&spm=1001.2101.3001.7020)。

      这样，每当我们生成一个版本的时候，我们都可以把稳定版本的源码放到 online 这个分支上。然后在 master 分支上继续开发新功能。当需要升级版本的时候，我们只需要把 master 分支上成熟的代码推送到 online 分支即可。

### 二、创建分支及其相关命令

1、创建新分支

```bash
//新建 online 分支
git checkout -b online
```

2、查看当前所有分支

```bash
//查看当前所有的分支
git branch -a
//结果显示带*号的，而且颜色是绿色的即为我们当前所在的分支
*master
online
```

3、切换分支

```bash
//从当前的 master 分支切换到 online 分支上面
git checkout online
//此时可以查看分支，使用 git branch 即可看到 master 和 online 分支
git branch
```

4、删除分支

```bash
//删除 online 分支
git branch -d online
```

5、本地合并新分支代码

```bash
//origin 是本地默认的一个名称，自己在新建本地仓库的时候是可以改名的
//平常使用的 git pull 都是默认从 master 分支上拉去代码。这里是从 online 分支上拉取代码
git pull origin online
```

6、本地提交代码到新的分支

```bash
//这里和上面的 git pull 差不多。是提交本地代码到 online 分支
git push origin online
```

### 三、需要注意的问题

1、远程新建分支之后，本地如果立刻使用 git branch -a 查看分支的话，会看不到新建的分支。需要现在本地 git pull 一下。

2、如果我们已经通过 git checkout master，切换到了 master 分支。那么我们在本地通过 git pull 和 git push 都可以直接拉去或提交代码到 master 分支。git 会默认使用你当前所在的分支

end
