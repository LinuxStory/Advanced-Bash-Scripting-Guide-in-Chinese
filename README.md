# Bash脚本进阶指南

[![CC0 1.0 Universal (CC0 1.0) Public Domain Dedication](https://img.shields.io/badge/License-CC0%201.0%20Universal-blue.svg)](https://creativecommons.org/publicdomain/zero/1.0/)
[![GitBook](https://img.shields.io/badge/GitBook-read-blue)](https://linuxstory.gitbook.io/advanced-bash-scripting-guide-in-chinese/)

*Advanced Bash-Scripting Guide* in Chinese

## 翻译作品

翻译作品展示于[GitBook](https://linuxstory.gitbook.io/advanced-bash-scripting-guide-in-chinese/)，欢迎指正！

## 原著及译著

### 原著

- 链接：http://tldp.org/LDP/abs/html/
- 作者：Mendel Cooper
- 版本：Revision 10, 10 Mar 2014
- 版权：公共领域(Public Domain)

### 最新译著

- Linux Story 通告地址：http://www.linuxstory.org/asdvanced-bash-scripting-guide-in-chinese/
- 版本：Revision 10, 10 Mar 2014
- 版权：公共领域(CC0)

### 早期译著

- 链接（已失效）：http://www.linuxsir.org/bbs/thread256887.html
- 译者：杨春敏 黄毅
- 版本：Revision 3.7, 23 Oct 2005

## 翻译进度

- [x] 第一部分 初见Shell[@imcmy][@zihengcat]
	- [x] 1\. 为什么使用shell编程[@imcmy][@zihengcat]
	- [x] 2\. Sha-Bang（#!）一起出发[@imcmy][@zihengcat]
- [x] 第二部分 Shell基础[@imcmy][@zihengcat]
	- [x] 3\. 特殊字符[@imcmy][@zihengcat]
	- [x] 4\. 变量与参数[@imcmy][@zihengcat]
	- [x] 5\. 引用[@mr253727942][@zihengcat]
	- [x] 6\. 退出与退出状态[@samita2030][@zihengcat]
	- [x] 7\. 测试[@imcmy][@zihengcat]
	- [x] 8\. 运算符相关话题[@samita2030][@zihengcat]
- [x] 第三部分 Shell进阶[@imcmy]
	- [x] 9\. 换个角度看变量[@imcmy]
	- [x] 10\. 变量处理[@imcmy]
	- [x] 11\. 循环与分支[@imcmy]
	- [x] 12\. 命令替换[@imcmy]
	- [x] 13\. 算术扩展[@imcmy]
	- [x] 14\. 休息时间[@imcmy]
- [x] 第四部分. 命令[@imcmy]
	- [x] 15\. 内建命令[@imcmy]
		- [x] 15.1 任务控制命令[@Administroot]
	- [ ] 16\. 外部过滤器，程序与命令[@hsupu]
		- [x] 16.1 基本命令[@hsupu]
		- [x] 16.2 复杂命令[@Administroot]
		- [x] 16.3 时间与日期命令[@Administroot]
		- [x] 16.4 文本处理命令[@Administroot]
		- [x] 16.5 文件与归档命令[@Administroot]
		- [x] 16.6 通信命令[@Administroot]
		- [x] 16.7 终端控制命令[@Administroot]
		- [x] 16.8 数学命令[@Administroot]
		- [x] 16.9 杂项命令[@Administroot]
	- [ ] 17\. 系统与高级命令
- [ ] 第五章. 高级话题
	- [x] 18\. 正则表达式[@Zjie]
	- [x] 19\. 嵌入文档[@mingmings]
	- [x] 20\. I/O 重定向[@mingmings]
	- [x] 21\. Subshells[@ysun90]
	- [x] 22\. 限制模式的Shell[@panblack]
	- [x] 23\. 进程替换[@panblack]
	- [x] 24\. 函数[@zy416548283]
	- [x] 25\. 别名[@mingmings]
	- [x] 26\. 列表结构[@panblack]
	- [x] 27\. 数组[@zy416548283]
	- [x] 28\. 间接引用[@plutonji]
	- [x] 29\. `/dev` 和 `/proc`[@plutonji]
	- [x] 30\. 网络编程[@Zjie]
	- [ ] 31\. Of Zeros and Nulls
	- [x] 32\. 调试[@wuqichao]
	- [x] 33\. 选项[@zy416548283]
	- [x] 34\. 陷阱[@liuburn]
	- [ ] 35\. Scripting With Style
	- [ ] 36\. 杂项[@richard-ma]
	- [ ] 37\. Bash, versions 2, 3, and 4
- [x] 38\. 后记[@zy416548283]
- [ ] Bibliography
- [ ] 附录
	- [ ] A\. Contributed Scripts
	- [ ] B\. Reference Cards
	- [x] C\. A Sed and Awk Micro-Primer[@wuqichao]
		- [x] C.1 Sed[@wuqichao]
		- [x] C.2 Awk[@wuqichao]
	- [ ] D\. Parsing and Managing Pathnames
	- [x] E\. 带有特殊意义的退出代码[@ShadowRZ]
	- [x] F\. I/O 与 I/O 重定向详细介绍[@ShadowRZ]
	- [ ] G\. Command-Line Options
		- [ ] G.1 Standard Command-Line Options
		- [ ] G.2 Bash Command-Line Options
	- [x] H\. 重要文件[@ShadowRZ]
	- [ ] I\. Important System Directories
	- [ ] J\. An Introduction to Programmable Completion
	- [ ] K\. Localization
	- [ ] L\. History Commands
	- [x] M\. 示例 ~/.bashrc 与 ~/.bash_profile[@ShadowRZ]
	- [ ] N\. Converting DOS Batch Files to Shell Scripts
	- [ ] O\. Exercises
		- O.1 Analyzing Scripts
		- O.2 Writing Scripts
	- [ ] P\. Revision History
	- [ ] Q\. Download and Mirror Sites
	- [ ] R\. To Do List
	- [ ] S\. Copyright
	- [x] T\. ASCII 表[@ShadowRZ]
- [ ] Index
- [ ] List of Tables
- [ ] List of Examples

## 翻译校审流程

### 初始化

1. 首先fork项目
2. 把fork过去的项目clone到本地
3. 命令行下运行 `git checkout -b dev` 创建一个新分支
4. 运行 `git remote add upstream https://github.com/LinuxStory/Advanced-Bash-Scripting-Guide-in-Chinese.git` 添加远端库
5. 运行 `git remote update`更新
6. 运行 `git fetch upstream master` 拉取更新到本地
7. 运行 `git rebase upstream/master` 将更新合并到你的分支

初始化只需要做一遍，之后请在dev分支进行修改。

如果修改过程中项目有更新，请重复5、6、7步。

### 翻译校审流程

1. 保证在dev分支中
2. 打开README.md，在翻译进度后加上你自己的github名
	
	> 1\. Shell Programming! [@翻译人][@校审人]
3. 本地提交修改，写明提交信息
4. push到你fork的项目中，然后登录GitHub
5. 在你fork的项目的首页可以看到一个 `pull request` 按钮，点击它，填写说明信息，然后提交即可
	
	> 为了不重复工作，请等待我们确认了你的pull request(即你的名字出现在项目中时)，再进行翻译校审工作
6. 进行翻译校审，重复3-5步提交翻译校审的作品


> 新手可以参阅针对github小白的[《翻译流程详解》](https://github.com/LinuxStory/Advanced-Bash-Scripting-Guide-in-Chinese/wiki/%E7%BF%BB%E8%AF%91%E6%B5%81%E7%A8%8B%E8%AF%A6%E8%A7%A3),妹子写的呦～

## 翻译校审建议

1. 使用markdown进行翻译校审，文件名必须使用英文
2. 翻译校审后的文档请放到source文件夹下的对应章节中，然后pull request即可
3. 有任何问题随时欢迎发issue
4. 术语尽量保证和已翻译的一致，也可以查询[微软术语搜索](https://www.microsoft.com/zh-cn/language/search)或[Linux中国术语词典](https://github.com/LCTT/TranslateProject/blob/master/Dict.md)
5. 你可以将你认为是术语的词汇加入术语表`TERM.md`中

## 样式规范

根据原文中不同的注释类型，可以使用 `>` 或下列html代码进行注释。

```html
{% hint style="info" %}
Hello world
{% endhint %}
```

所有的脚注footnote都需使用html代码进行注释。

## 参与人员（按名称排序）

- @Administroot
- @chuchingkai
- @hsupu
- @i-bugmaker
- @imcmy
- @liuburn
- @mamh2021
- @mingmings
- @mr253727942
- @panblack
- @plutonji
- @richard-ma
- @samita2030
- @ShadowRZ
- @wuqichao
- @ysun90
- @zhaozq
- @zihengcat
- @Zjie
- @zy416548283
