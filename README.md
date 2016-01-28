# 《Advanced Bash-Scripting Guide》 in Chinese

《高级Bash脚本编程指南》Revision 10中文版

## 翻译周期的说明
因译者自身原因，更新频率不固定，尽量保证每周1-2节。

## 原著及早期翻译作品
### 原著
- 原著链接：http://tldp.org/LDP/abs/html/
- 原作：Mendel Cooper
- 原著版本：Revision 10, 10 Mar 2014

### 译著
- 早期译著连接：http://www.linuxsir.org/bbs/thread256887.html
- 译者：杨春敏 黄毅
- 译著版本：Revision 3.7, 23 Oct 2005
- 最新 Revision 10 由 Linux Story 社区的 imcmy 同学发起并组织翻译。
- Linux Story 通告地址 ：http://www.linuxstory.org/asdvanced-bash-scripting-guide-in-chinese/ 

## 翻译作品
翻译的部分都放置在[GitBook](https://www.gitbook.com/book/imcmy/advanced-bash-scripting-guide-in-chinese/details)上，欢迎阅读！

## 翻译进度

- 第一部分 初见Shell[@imcmy]
	- 1\. 为什么使用shell编程[@imcmy]
	- 2\. Sha-Bang（#!）一起出发[@imcmy]
- 第二部分 Shell基础[@imcmy]
	- 3\. 特殊字符[@imcmy]
	- 4\. 变量与参数[@imcmy]
	- 5\. 引用[@mr253727942]
	- 6\. 退出与退出状态[@samita2030]
	- 7\. 测试[@imcmy]
	- 8\. Operations and Related Topics[@samita2030]
- 第三部分 Shell进阶[@imcmy]
	- 9\. Another Look at Variables[@Ninestd]
	- 10\. 变量处理[@imcmy]
	- 11\. 循环与分支[@imcmy]
	- 12\. 命令替换[@imcmy]
	- 13\. 算术扩展[@imcmy]
	- 14\. 休息时间[@imcmy]
- 第四部分. 命令[@zhaozq]
	- 15\. 内建命令[@zhaozq]
	- 16\. 外部过滤器，程序与命令[@zhaozq]
	- 17\. 系统与高级命令[@zhaozq]
- 第五章. Advanced Topics
	- 18\. 正则表达式(@bootingman)
	- 19\. Here Documents(@mingmings)
	- 20\. I/O Redirection(@mingmings)
	- 21\. Subshells(@mingmings)
	- 22\. Restricted Shells
	- 23\. Process Substitution
	- 24\. Functions(@zy416548283)
	- 25\. Aliases(@mingmings)
	- 26\. List Constructs
	- 27\. Arrays(@zy416548283)
	- 28\. Indirect References
	- 29\. /dev and /proc
	- 30\. Network Programming
	- 31\. Of Zeros and Nulls
	- 32\. Debugging
	- 33\. Options
	- 34\. Gotchas
	- 35\. Scripting With Style[@chuchingkai]
	- 36\. Miscellany
	- 37\. Bash, versions 2, 3, and 4
	- 38\. Endnotes
		- 38.1 Author's Note
		- 38.2 About the Author
		- 38.3 Where to Go For Help
		- 38.4 Tools Used to Produce This Book
		- 38.5 Credits
		- 38.6 Disclaimer
- Bibliography
- Appendix
	- A\. Contributed Scripts
	- B\. Reference Cards
	- C\. A Sed and Awk Micro-Primer
		- C.1 Sed
		- C.2 Awk
	- D\. Parsing and Managing Pathnames
	- E\. Exit Codes With Special Meanings
	- F\. A Detailed Introduction to I/O and I/O Redirection
	- G\. Command-Line Options
		- G.1 Standard Command-Line Options
		- G.2 Bash Command-Line Options
	- H\. Important Files
	- I\. Important System Directories
	- J\. An Introduction to Programmable Completion
	- K\. Localization
	- L\. History Commands
	- M\. Sample .bashrc and .bash_profile Files
	- N\. Converting DOS Batch Files to Shell Scripts
	- O\. Exercises
		- O.1 Analyzing Scripts
		- O.2 Writing Scripts
	- P\. Revision History
	- Q\. Download and Mirror Sites
	- R\. To Do List
	- S\. Copyright
	- T\. ASCII Table
- Index
- List of Tables
- List of Examples

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
4. 术语尽量保证和已翻译的一致，也可以查询[微软术语搜索](http://www.microsoft.com/Language/zh-cn/Search.aspx)或[Linux中国术语词典](https://github.com/LCTT/TranslateProject/blob/master/Dict.md)
5. 你可以将你认为是术语的词汇加入术语表`TERM.md`中

## 关于版权
根据原著作者的要求，翻译成果属于公有领域(CC0)，翻译参与人员及原著作者Mendel Cooper享有署名权
