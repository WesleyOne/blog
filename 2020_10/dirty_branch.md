---
sort: 88000
title: 你脏了，分支
tags: git 分支污染
---

# 你脏了，分支

> 记录一次污染分支回滚处理，仅供参考

## 先说下场景
	搜了几篇关于git分支管理的文章，发现公司的分支管理是个“非标产品”，哇的哭了出来😭，只能说有一丢丢像阿里的`AoneFlow`。

	目前使用流程如下，针对不同的部署环境有相应的发布分支，用于线上环境部署的`master`分支，测试环境部署的`test`分支,以及开发环境部署的`dev`分支，需求分支`feature/xxx`从`master`分支拉出，在开发自测时合到`dev`分支，提测时合到`test`分支，上线前先从`master`分支拉出一个预发布`prerelease/xxx`分支，然后将需求分支`feature/xxx`合到预发分支，解决冲突以及release版本打包等。

	为了`master`分支健康，`master`分支部署前都会打个上一版本`tag`用于回滚，合并到`master`需要权限，采用了`gitlab`的`merge request`，会有代码review，一定程度上拦截污染分支合并；

	但是为了方便测试发布，没有对`test`分支进行管控，所以一旦需求分支被污染了，然后提测时合到测试分支上，导致测试环境也被污染了，哦吼，所以我说的污染到底是个啥？会有什么影响吗？

	分支污染，比如不小心把`dev`分支合到了需求分支上并推到远程了，导致一些和当前需求无关的代码都合到了当前需求分支上，那么就会导致当前需求分支被污染，如果进一步合到测试分支上并推到远程，会将其他需求未提测的甚至需求中止的代码合到测试代码上，导致测试分支被污染。分支千万条，谨慎第一条。
	
## 脏了怎么办，滚

	污染也要分程度，尽早发现，尽早治疗，改动越少。

	1. 提交本地分支到远程前要check下，如果发现类似`*/dev into feature/xxx`的commit日志时，那么只是本地需求分支被污染了。
	2. 本地污染的需求分支已经推到了远程。
	3. 远程的测试分支被污染了。

	![污染分支图]({{ site.baseurl }}/assets/images/2020_10/dirty_branch.jpg){:.border.rounded}

	针对第1条，本次未推到远端的变动很少的话，可以把变动记录下来，然后删掉本地需求分支，从远端需求分支重新拉取到本地重新开发，工作量相对很少；
	已经提交到远端的，没办法，只好一顿神操作（噼里啪啦）。大致操作如下：
	1. 本地备份（checkout）污染需求分支（用于后面比较更新）
	2. 回滚需求分支到污染前commit（我的方法就是git reset --hard commitId，意思是回滚的commitId之后的改动完全删除）
	![IDEA快捷回滚操作图]({{ site.baseurl }}/assets/images/2020_10/reset1.jpg){:.border.rounded}

	![IDEA快捷回滚操作图]({{ site.baseurl }}/assets/images/2020_10/reset2.jpg){:.border.rounded}

	![回滚后分支图]({{ site.baseurl }}/assets/images/2020_10/reset2_1.jpg){:.border.rounded}
	3. 修复回滚后的需求分支
		3.1 如果污染后改动很少，可以直接重写
		3.2 改动大的话，比较回滚后分支和备份污染分支，甄别出正常的代码并复制到回滚后的需求分支上（工作量大，甚至多人开发时，别人的commit都会变成你提交的）

		![回滚后修复分支图]({{ site.baseurl }}/assets/images/2020_10/reset2_2.jpg){:.border.rounded}
		
		![回滚后修复分支图]({{ site.baseurl }}/assets/images/2020_10/reset3.jpg){:.border.rounded}

		![回滚后修复分支图]({{ site.baseurl }}/assets/images/2020_10/reset3_1.jpg){:.border.rounded}
		
		![回滚后修复分支图]({{ site.baseurl }}/assets/images/2020_10/reset4.jpg){:.border.rounded}
	4. 测试分支回滚到污染前commit，然后把污染后提测的所有需求分支再合一下😭（然后和测试同事深深道个歉）


# 参考
《在阿里，我们如何管理代码分支？》- https://developer.aliyun.com/article/573549
《A successful Git branching model》- https://nvie.com/posts/a-successful-git-branching-model/
《Git 分支管理最佳实践》- https://developer.ibm.com/zh/articles/j-lo-git-mange/
《Git Reset 三种模式》- https://www.jianshu.com/p/c2ec5f06cf1a