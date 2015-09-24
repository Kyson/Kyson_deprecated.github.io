---
layout: post
title: android中imageview scaletype属性说明
---

- CENTER/center

	按图片的原来size居中显示，当图片长/宽超过View的长/宽，则截取图片的居中部分显示

- CENTER_CROP/centerCrop

	按比例扩大图片的size居中显示，使得图片长(宽)等于或大于View的长(宽)

- CENTER_INSIDE/centerInside

	将图片的内容完整居中显示，通过按比例缩小或原来的size使得图片长/宽等于或小于View的长/宽

- FIT_CENTER/fitCenter

	把图片按比例扩大/缩小到View的宽度，居中显示

- FIT_END/fitEnd

	把图片按比例扩大/缩小到View的宽度，显示在View的下部分位置

- FIT_START/fitStart

	把图片按比例扩大/缩小到View的宽度，显示在View的上部分位置

- FIT_XY/fitXY

	把图片不按比例扩大/缩小到View的大小显示

- MATRIX/matrix

	用矩阵来绘制，动态缩小放大图片

> 说明：使用setImageResource而非setBackground
