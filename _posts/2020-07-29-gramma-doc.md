---
layout: post
title: Gramma Doc
author: "Gao"
---

# Gramma doc

+ 花括号用两个反斜杠。
  
+ 竖线要加一个反斜杠, \\| 。

+ 实数集R用 \mathbb{R}。

+ 使用单独一行并居中的公式，需空一行后再用前后两个美元符号括起来。    
  
   "Example line, we want to insert a formula in next line and centered."

   \\$\\$ format here \\$\\$

+ 文档中需要引用的图片统一上传到服务器的 **Home/Desktop/sdnanjing.github.io-master/assets/imgs** 中，文档内的路径从 **assets** 开始写，例如文档需要引用 imgs 中的algorithm.png 图片，md中图片的路径即 **/assets/imgs/algorithm.png**

+ 编辑完成的md文档放在**Home/desktop/sdnanjing_github.io-master/_posts**中，命令行cd到 **sdnanjing.github.io-master** 中再 **sudo jekyll build** 就能自动渲染出html页面，页面效果可在命令行输入 **sudo jekyll server** 后在浏览器进入本地4000端口查看。
