---
layout: post
title: 解放在崩溃边缘的淘宝美工...
tags:
  - Python
  - Pillow
  - Pyinstaller

---

近日伴随着Z先生的交图deadline逼近，photoshop拼图拼的他整个人都在崩溃边缘。

先前他需要用photoshop做一套剪切蒙版出来，然后一张一张图往里拉，调整图形大小、旋转，最后形成一套完整的图。有没有路过的美工大大给指导一下到底要怎样才能用photoshop快速做好那些拼图啊？？哭泣T-T

总之我是不想帮他做蒙版然后拖图了，必须考虑能不能整个程序来实现。


 首先，选一个库吧。万能的Python有两个比较常用的图形库--opencv和Pillow（和PIL之间的包含和被包含关系傻傻分不清楚），但是！！！opencv在加文字的时候不能自定义字体，而Pillow可以自定义字体和大小。好了opencv被pass掉了，PIL胜出。接着是一顿查，淘宝的图都是750像素宽，那么用程序做出来的图跟photoshop做出来的在图片质量上有没有什么区别？什么72dpi和300dpi会不会影响图片质量？[知乎的这个回答解除了我的疑惑](<https://www.zhihu.com/question/26697578>)

`图片在计算机（或其他设备）里是一系列代表视觉信息的数据，它的单位是像素。因此，真正能定义图片尺寸的是分辨率。`

也就是说大家都是像素点，只有个数上的区别，点还是那个点呀（不过我忽然想到可能不同的压缩算法在计算点的特征的时候会有所差别？？唉，不管那么多了，就相信现代科学吧）。

PIL读取图片后相当于把整个图当成了一个矩阵来处理，所以在计算需要的图片总长度的时候还是挺容易实现的。

整个思路就是：把要处理的图片读进来，每个图片缩小、加水印，同时图片长度和间隔像素加和计算出画布的总长度，建一个空白画布，然后依次把处理好的图片paste上去。

写（ZHAO）一个可以实现自动缩小图片的子程序：


```
def resize_cut(image, width, height):
    top, bottom, left, right = (0, 0, 0, 0)
    #获取图像尺寸
    # h, w, _ = image.shape
    w, h = image.size
    #计算需要裁的边
    h_need = height * w / width
    w_need = width * h / height
    if h > h_need: #上下需要裁
        top = int((h - h_need)/2)
        bottom = int(h_need + (h - h_need)/2)
        left = 0
        right = w
    elif h < h_need: #左右需要裁
        top = 0
        bottom = h
        left = int((w - w_need)/2)
        right = int(w_need + (w - w_need)/2)
    else:
        right = w
        bottom = h
    # print("left, right, top, bottom: ",left,right,top,bottom)
    box = (left, top, right, bottom)
    region = image.crop(box)
    return region.resize((width, height))
```

一个在特定位置加文字水印的子程序：


```
def add_text(image, x, y):
    font = ImageFont.truetype("Copperplate Gothic Light.ttf", 37, encoding="unic")#设置字体
    draw = ImageDraw.Draw(image)
    draw.text((x, y), 'HELLO WORLD', 'white', font)  #左上角为(0,0)
    return image
```    

这里在设置字体的时候在windows下字体名按照C盘Fonts文件夹下的文件名"COPRGTL.TTF"来写可以识别，但是在MAC下写成"COPRGTL.TTF"就报了找不到字体的错误，改成"Copperplate Gothic Light.ttf"才能找到。"Copperplate Gothic Light.ttf"在windows下不能找到字体。

建一个长的空白画布和往画布里粘贴处理好的图片的子程序：

```
def init_canvas(width, height, color=(255, 255, 255)):
    canvas = np.ndarray((height, width, 3), dtype="uint8")
    canvas[:] = color
    canvas = Image.fromarray(canvas)
    return canvas

def addImg_h_w(img1, img2, top, left):
    #img1[top:top+height,left:left+width] = img2   #opencv2里的写法
    img1.paste(img2, (left,top))
    return img1
```

写好之后在MAC上试运行了一下，三十几张图得跑个几分钟，还可以接受，毕竟比拖图简单多了呀。

之后要考虑把程序改成一个方便移植的形式，毕竟不能满世界给人安装python3和各种包吧。

在windows下安装pyinstaller，

```
pyinstaller -F .\pil_resize.py
```

然后在文件夹下的dist里就可以找到封装好的.exe程序。测试下好不好用。


d=====(￣▽￣*)b 大功告成~~





愿好好写程序的~~小孩~~大人可以被这个世界温柔的对待。
