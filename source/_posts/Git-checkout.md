---
title: Git checkout 用法
date: 2016-09-18 18:38:05
categories:
tags:
    - Git
---
### check out
#### 1. git checkout [-q] [&lt;commit>] [--] &lt;paths>...
 - 有commit(提交版本号):用版本库中对应版本文件覆盖暂存区和工作区文件     
    `$ git checkout 0w3f...(版本号) -- filename`

<!--more-->

 - 无commit(提交版本号)：用暂存区对应版本文件覆盖工作区文件    
  `$ git checkout -- filename`    
  `$ git checkout .` :　用暂存区文件覆盖工作区所有修改

#### 2. git checkout [&lt;branch>]
 - 省略branch:检查工作区，暂存区，HEAD间的差别
 - 有branch:检出branch分支，分三步：将HEAD指向branch分支，然后将HEAD对应版本文件覆盖到工作区和暂存区    
  `$ git checkout branch` --filename:这种情况不会改变HEAD指向，只会把branch中对应文件覆盖到工作区和暂存区

#### 3. git checkout [-m] [[-b]--orphan] &lt;new_branch>] [&lt;start_point>]    
 - 创建和切换到新的分支（&lt;new_branch>），新的分支从<start_point>指定的提交开始创建



