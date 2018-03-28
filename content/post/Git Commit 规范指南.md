---
date: 2018-03-28
title: "Git Commit 规范指南"
tags:
    - Git
    - 规范
categories:
    - Git
comment: false
---

## 1. Commit message 有啥用

* 提供更多的历史信息，方便快速浏览

  * 可以过滤某些commit(比如文档改动)，便于快速查找信息

  * 可以直接从commit生成Change log,Change Log 是发布新版本时，用来说明与上一个版本差异的文档.例如:
  
    ![example](http://oojbdbtdp.bkt.clowuddn.com/6632199163166779416.png?imageView/2/w/350)
    
  * 其他优点:

    *  可读性好，清晰，不必深入看代码即可了解当前commit的作用

    *  为 Code Reviewing 做准备

    *  方便跟踪工程历史

    *  让其他的开发者在运行 git blame 的时候想跪谢
    
    *  提高项目的整体质量，提高个人工程素质

## 2. 正确的 Commit message 的使用姿势 

  每次提交，Commit message 都包括三个部分：Header，Body 和 Footer.
  
  ```
    <type>(<scope>): <subject>
    <空行>
    <body>
    <空行>
    <footer>
  ```

### 1. Header

  Header部分只有一行，包括三个字段：type(必需)、scope(可选)和subject(必需)
  
*   **type**

    用于说明 commit 的类别，只允许使用下面7个标识  
    
    |      | 标识          | 作用                                |
    |:----:|:------------:|-------------------------------------|
    | 1    | feat         | 新增功能(feature)                     |
    | 2    | fix          | 修复bug                              |
    | 3    | docs         | 文档更新                              |
    | 4    | style        | 更新代码格式,不影响代码的原始逻辑和执行    |
    | 5    | refactor     | 重构(即不是新增功能，也不是修改bug的代码) |
    | 6    | test         | 新增单元测试用例                       |
    | 7    | chore        | 构建过程或辅助工具的变动                |
    |8     | perf         | 提升优化性能                          |
    |9     | deps         | 升级依赖                              |

* **scope**

    scope用于说明 commit 影响的范围，比如数据层、控制层、视图层等等，视项目不同而不同
    如果你的修改影响了不止一个scope，你可以使用*代替

* **subject**

    subject是 commit 目的的简短描述需要注意以下事项:
    
    |     | 注意点                                                     |
    |:---:|:--------------------------------------------------------- |
    |1    |不超过50个字符                                               |
    |2    |以动词开头，使用第一人称现在时，比如change，而不是changed或changes |
    |3    |第一个字母小写                                               |
    |4    |结尾不加句号（.）                                             |

### 2. Body

Body 部分是对本次 commit 的详细描述，可以分成多行,Body编写注意点:

|      | 注意点                                                 |
|:----:| :-----------------------------------------------------|
| 1    | 每行不超过72个字符                                       |
| 2    | 段落之间用空行隔开                                       |
| 3    | 分段表述时,使用悬挂缩进                                   |
| 4    | 使用第一人称现在时，比如使用change而不是changed或changes    |
| 5    | 应该说明代码变动的动机，以及与以前行为的对比                 |

### 3. Footer

通常情况下的commit是不需要编写Footer部分的说明的,仅在以下情况发生时编写

* **不兼容变动**

  如果当前代码与上一个版本不兼容，则 Footer 部分以BREAKING CHANGE开头，后面是对变动的描述、以及变动理由和迁移方法

* **关闭 Issue**

  如果当前 commit 针对某个issue，那么可以在 Footer 部分关闭这个 issue

* **Revert**

  如果当前 commit 用于撤销以前的 commit，则必须以revert:开头，后面跟着被撤销 Commit 的 Header.Body部分的格式是固定的，必须写成This reverts commit <hash>.，其中的hash是被撤销 commit 的 SHA 标识符.如果当前 commit 与被撤销的 commit，在同一个发布（release）里面，那么它们都不会出现在 Change log 里面如果两者在不同的发布，那么当前 commit，会出现在 Change log 的Reverts小标题下面

### 4.示例

* 初始化创建一个新的Git版本库

* 给初始化提交添加版本标签(0.0.1)

  ![init](http://oojbdbtdp.bkt.clouddn.com/init.png)

* 提交一个新功能

	```
	git log 0.0.1 HEAD --grep feat
	```
	![feat](http://oojbdbtdp.bkt.clouddn.com/feat.png)

* 提交一个bug修正

	```
	git log 0.0.1 HEAD --grep fix
	```
	![fix](http://oojbdbtdp.bkt.clouddn.com/fix.png)

* 提交一次文档更新

	```
	git log 0.0.1 HEAD --grep docs
	```
	![docs](http://oojbdbtdp.bkt.clouddn.com/docs.png)

* 提交一次代码格式修改

  ```
  git log 0.0.1 HEAD --grep style
  ```
  ![style](http://oojbdbtdp.bkt.clouddn.com/style.png)
  
* 提交一次代码优化重构

  ```
  git log 0.0.1 HEAD --grep refactor
  ```
  ![refactor](http://oojbdbtdp.bkt.clouddn.com/refactor.png)
    
* 提交一个单元测试

  ```
  git log 0.0.1 HEAD --grep test
  ```
  ![test](http://oojbdbtdp.bkt.clouddn.com/test.png)
  
* 查看所有git提交记录
  ```
  git log 0.0.1 HEAD --pretty=format:%s
  ```
  
  ![log](http://oojbdbtdp.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-04-19%2015.54.18.png)
 
 使用[tig](https://github.com/jonas/tig)查看到的提交记录:
  
  ![tig](http://oojbdbtdp.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-04-19%2015.59.36.png)

### 5.生成changelog

* 安装工具

	```
	npm install -g conventional-changelog-cli
	```

* 配置conventional-changelog

  ```
  cd  项目目录
  npm init
  ```
  
  修改 package.json,添加:
  
  ```javascript
  "scripts": {
      "changelog": "conventional-changelog -p angular -i CHANGELOG.md -s -r 0"
  }
  ```
  
* 执行命令

	```
	npm run changelog
	```

* 查看版本库目录下生成的CHANGELOG.md文件,如下图所示:

  ![changelog](http://oojbdbtdp.bkt.clouddn.com/changelog.png?imageView/2/w/350)
  
* 工具默认只会将feat和fix这两个type的commit信息抓取出来并生成到CHANGELOG.md文件中.

### 6.SourceTree 配置默认commit message 模板

  SourceTree提供了自定义的提交模板配置功能.
  
*   在SourceTree中点击仓库菜单->仓库设置->提交模板
    填入以下内容:  
    
    ```
    type(scope): subject  

    body  

    bodydetail  

    foot
    ```
    
    保存设置,之后每次commit代码时,这个模板就会自动的填充到commit message中,然后我们只需要替换修改必要的部分就好了.
    
## 3.参考文献:

* [AngularJS Git Commit Message Conventions](https://docs.google.com/document/d/1QrDFcIiPjSLDn3EL15IJygNPiHORgU1_OOAqWjiDU5Y/edit#heading=h.uyo6cb12dt6w)

*   [git commit 规范指南](https://segmentfault.com/a/1190000009048911)

*   [Commit message 和 Change log 编写指南](http://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html)

*   [Contributing to AngularJS](https://github.com/angular/angular.js/blob/master/CONTRIBUTING.md#toc10)
