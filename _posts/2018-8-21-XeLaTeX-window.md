---
layout: post
title: windows下一个奇怪的XeLaTeX报错!
tags:
  - LaTeX
  - XeLaTeX
---

前几天使用XelaTex生成pdf时遇到一个报错：

    ThisCenterWallPaper Cannot determine size of graphic in .png (no BoundingBox)
    
源代码是这样的：

````
\newpage
\thispagestyle{empty} 
\mbox{}
\ThisCenterWallPaper{1}{xxxxxxxx_xxx_1.png}
````

说它奇怪是因为其他很多图片也是这种方式插入的，都没有问题，单单这个图片报错。

搜索了一圈儿，以为是图片大小的问题，发现其他图片也有比这图片大的。

唉，为啥这个图片报错，它看上去只是比别的png名字长一点而已。等等！难道是名字太长的锅？

我试着把它的名字改短了。

然后...就不报错了。

成吉思汗~

是以为记。
