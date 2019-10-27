---
title: ubuntu 下postman管标位置错位
date: 2019-10-27 22:33:03
tags postman:
---

系统环境: ubuntu 18.04
#### 出现的问题
postman 在ubuntu 中字体显示不正常,光标选中位置实际是错位的.在github中也有类似问题的lssues[lssues][https://github.com/postmanlabs/postman-app-support/issues/2985]
![](/img/postman.png)
#### 解决办法
修改postman的默认字体
- 找到postman 的安装路径
- 打开文件 /postman的安装目的/Postman/app/resources/app/js/requester.css
- 搜索.editor.ace_editor
```
.editor.ace_editor {
 1148     font: 12px "Monaco", "Ubuntu Mono", "Menlo", "Consolas", "source-code-pro", "Cousine", monospace, monospace; }
```
可以看到现在默认使用的字体是 "Monaco" 只要修改为适合系统的字体即可比如"Ubuntu Mono"
.editor.ace_editor {
 1148     font: 12px "Ubuntu Mono", "Monaco", "Menlo", "Consolas", "source-code-pro", "Cousine", monospace, monospace; }
*最后重启postman 大功告成*


