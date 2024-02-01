> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [segmentfault.com](https://segmentfault.com/a/1190000044489274)

> 此文档是《OCR 文字识别实战教程 - 零基础，SpringBoot 结合 PaddleOCR》实现车牌识别、文本识别、身份证识别 地址：[链接]，教程对应的部分文档，全部文档共 89 页...

### 基本介绍

**此文档是**《[OCR 文字识别实战教程 - 零基础](https://www.bilibili.com/video/BV1RK411b7DU/)，SpringBoot 结合 PaddleOCR》实现车牌识别、文本识别、身份证识别 地址：[https://www.bilibili.com/video/BV1RK411b7DU/?vd_source=b4307343204f5c0271966f7fe276f0eb](https://www.bilibili.com/video/BV1RK411b7DU/?vd_source=b4307343204f5c0271966f7fe276f0eb)，**教程对应的部分文档，全部文档共 89 页，剩余文档 (Springboo 服务开发、Docker 镜像打包部署、手机端部分) 和完整源代码 请关注 B 站视频**

#### 效果演示

![](https://segmentfault.com/img/bVdaPQY)  
上面的图片为 Android APP 的界面，从左边开始，分别为主界面、拍照界面、识别及结果界面。

#### 架构说明

OCR 实战项目整体架构图如下：  
![](https://segmentfault.com/img/remote/1460000044489276)

*   APP/WEB / 小程序为 OCR 识别接口调用端，调用 OCR 接口，实现 OCR 功能。本项目我们只实现 Android APP 开发。
*   Nginx 反向代理和负载均衡功能，通过 Nginx 实现对外网暴露接口，对内负载均衡 SpringBoot 实现的 OCR 服务。
*   OCR 服务通过 Springboot 实现，主要功能是提供具体的 OCR 接口实现，其流程是调用内部 PaddleOCR 服务，解析和处理返回结果，最终返回结果给接口调用者。为了稳定性和安全性，添加了熔断限流、Token 认证功能。为了方便部署，会以 Docker 形式部署该服务。
*   PaddleOCR 是 OCR 识别的具体实现，会提供一个 OCR 识别接口，供内部调用。由于不同的部署方式（普通部署和 paddleocr serving 方式部署），PaddleOCR 在普通部署方式下，无法利用 CPU 多核（Servering 方式不存在该问题），因此会在同一个服务器部署多个实例，解决 CPU 利用率差以提升性能。为了方便 PaddleOCR 部署，会以 Docker 形式部署。后边会讲解普通方式部署和 Servering 方式部署，如何构建 docker 镜像及部署流程。
    
    #### 主要技术栈
    
*   开发语言：java、python（不需要 python 基础）
*   springboot 实现业务接口
*   python flask 实现识别接口
*   Sentinel 限流熔断
*   JWT Token 认证
*   PaddlePaddle
*   PaddleOCR
*   Nginx 反向代理和负载均衡
*   Docker 镜像制作及部署服务
*   Android 原生开发

本课程我们将借助 PaddleOCR 和 PP-OPCRv4/3 实现文本识别、车牌识别、身份证识别。**本课程不涉及算法、模型训练等知识**，使用 PaddleOCR 提供的训练好的模型，没有晦涩难懂的技术，小白也能轻松入手。

**文档中的代码会和实际源代码有细微差别，以源代码为准（对实际学习不会有任何影响）**

### PaddleOCR 介绍

PaddleOCR 是一款由百度开发的 OCR（光学字符识别）工具库。它旨在为开发者提供一套丰富、领先、且实用的 OCR 工具，以帮助他们训练出更好的模型并应用于实际场景。  
PaddleOCR 具有以下特点：

1.  **超轻量模型：PaddleOCR 采用了轻量级模型，以便在移动设备和嵌入式设备上运行。**
2.  通用识别大模型：除了轻量级模型外，PaddleOCR 还提供了通用识别大模型，以适应更多的应用场景。
3.  算法丰富且开源：PaddleOCR 集成了多种与 OCR 相关的前沿算法，并进行了开源，以便更多的开发者可以共享和使用。
4.  支持自定义训练：**开发者可以根据自己的需求，使用 PaddleOCR 提供的工具和框架自定义训练模型**。
5.  支持 C++ 预测、**端侧部署、服务部署**：PaddleOCR 不仅支持 C++ 预测，还支持在端侧和服务上进行部署，具有很好的灵活性和可扩展性。
6.  行业特色模型：PaddleOCR 开发了具有行业特色的模型 PP-OCR 和 PP-Structure，并打通了数据生产、模型训练、压缩、预测部署的全流程。

总的来说，PaddleOCR 是一款功能强大、实用便捷的 OCR 工具库，它提供了一系列前沿的算法和自定义训练的支持，旨在帮助开发者更好地应用 OCR 技术于各种实际场景中。  
github:[https://github.com/PaddlePaddle/PaddleOCR](https://link.segmentfault.com/?enc=7F66RqZZ8i51ocyR06jvHQ%3D%3D.dj12sn0yFEgj4ZvBYn%2BVg4dSiFDsg3yF2reDJHU5N4FO%2FNalhPos74Z%2BCjpCDoJt)

#### PaddleOCR 应用场景

表单识别、票据识别、电表识别、车牌识别、身份证 & 银行卡、手写体识别、化验单识别 等等

#### PP-OCRv4 模型

PP-OCRv4 提供一套通用的 OCR 识别模型，可以识别多语言的文字，在速度和精度上都达到了比较好的效果。**不指定模型版本，会默认下载最新的模型（PP-OCRv4）。**  
![](https://segmentfault.com/img/remote/1460000044489277)  
![](https://segmentfault.com/img/remote/1460000044489278)  
具体参考 [https://github.com/PaddlePaddle/PaddleOCR/blob/release/2.7/doc/doc_ch/models_list.md](https://link.segmentfault.com/?enc=f1rlJQ9W270WB%2BggcjfVYQ%3D%3D.CkZ27pJTfgyk0z9oYiiqKZM8ew5%2Fy2Nu0u0yC2MKjICD48l2wBXZd%2B3O4JNHtIr3gH42GqHDZeh%2FSIWWeY9WREmDxAZgZHkVuSDiFeGykl4Ew4V%2FionqfGIa6DdHPf7a)  
![](https://segmentfault.com/img/remote/1460000044489279)  
使用时，我们只需要下载推理模型即可。  
下载模型后，解压放到对应目录即可，windows 为 C:\Users \ 用户. paddleocr\whl ，  
linux 为用户目录下. paddleocr\whl

### PaddleOCR 环境搭建

#### windows 下环境搭建

##### 安装 python

下载安装包：[https://www.python.org/ftp/python/3.8.10/python-3.8.10-amd64.exe](https://link.segmentfault.com/?enc=NKLrWaPgEUs2ey4VkCaMGQ%3D%3D.yPWjoF1OoM1ML3k7tMzctkRaCcax2zYwZegpe4vZWeHmw5fyaXq2%2FOBJqDqCaF2md6rLjfKwDtGjXxjCDRvUwhIqf1oE0H0nbmBjsXj9WJM%3D)  
**1. 双击安装包进行安装**  
![](https://segmentfault.com/img/remote/1460000044489280)  
**2. 勾上 Install launcher for all users 和 Add Python 3.8 to PATH**  
![](https://segmentfault.com/img/remote/1460000044489281)  
**3. 可以选择默认安装，也可以选择自定义安装。选择自定义安装如下：**  
![](https://segmentfault.com/img/remote/1460000044489282)  
保持默认，点击 next  
![](https://segmentfault.com/img/remote/1460000044489283)  
如上图，选择 Install for all users. 最后 点击 Install 安装。  
**4. 验证**  
在命令行中输入如下：  
python -V  
输出 Python 3.8.10 安装成功

##### 安装 PaddlePaddle

打开：[https://www.paddlepaddle.org.cn/](https://link.segmentfault.com/?enc=%2Fp9NYA9F79hedwkKmyu%2BwA%3D%3D.DxtS5Ln410Vp0ka8xxqJWep6%2FfvDEAHk4eXhGnnNwv6MhVet2K1LpZLSecsiO7Wa) ，选择 pip 方式，安装比较简单方便。 选择如下（**CPU 版本**），复制安装命令，安装：  
![](https://segmentfault.com/img/remote/1460000044489284)

```
python -m pip install paddlepaddle==2.5.1 -i https://pypi.tuna.tsinghua.edu.cn/simple

```

**验证安装**  
安装完成后，打开命令行，您可以使用输入 python 或 python3 进入 python 解释器，输入 import paddle ，再输入 paddle.utils.run_check()  
如果出现 PaddlePaddle is installed successfully!，说明您已成功安装。  
![](https://segmentfault.com/img/remote/1460000044489285)

##### 安装 PaddleOCR

参考 [https://github.com/PaddlePaddle/PaddleOCR/blob/release/2.6/doc/doc_ch/quickstart.md](https://link.segmentfault.com/?enc=h2sLJ0tS3%2Fykt1q6NluBHQ%3D%3D.4dr17XIxL7pK9VhTd5box8%2FX71GTdTcLUgck8YsXeCDofsRJTKU0zwWsqtJtcKLALNnCc0lf1hKHGek8wh%2FKOccLyWN9vM4FVgjSCZdLRSZb%2FjmRMU7ENA0a16oPHh3m)  
执行如下命令：

```
pip install "paddleocr>=2.6.0" -i https://mirror.baidu.com/pypi/simple

```

如果安装时提示如下：  
![](https://segmentfault.com/img/remote/1460000044489286)  
根据提示我们需要把下面的路径加入到环境变量，否则可能 paddocr 命令无法执行  
C:\Users \ 用户 \ AppData\Local\Programs\Python\Python38\Scripts

**验证**  
1. 输入 pip list ，看是否有 paddleocr 模块  
2. **使用 paddocr 命令，进行 ocr 识别**。./imgs/11.jpg 替换成某个图片路径，windows 下替换成 windos 风格的路径，例如”C:\ 文档资料 \ ocr\1.png“

```
#--use_angle_cls true设置使用方向分类器识别180度旋转文字，--use_gpu false设置不使用GPU 
paddleocr --image_dir ./imgs/11.jpg --use_angle_cls true --use_gpu false

```

例如输出如下：

![](https://segmentfault.com/img/remote/1460000044489287)
---------------------------------------------------------

#### Linux 环境搭建（centos7）

centos 下安装相对 windows 来说，会复杂一些，会遇到一些问题，不过大家不用担心，本文档后面会提供基于 docker 的环境构建和部署，会简单很多。这里只是让大家从 0 开始体验一 centos 下搭建 paddleocr 环境。

##### python 安装

**1. 安装依赖包**

```
yum install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gcc make mysql-devel gcc-devel python-devel -y

```

**2. 再执行安装 libffi-devel，不安装会导致 pip 安装失败**

```
yum install libffi-devel wget -y

```

缺少这一步，会出现 ModuleNotFound：No module named ‘_ctypes’  
**3. 下载 python3.8 包，并解压**

```
wget https://www.python.org/ftp/python/3.8.1/Python-3.8.1.tgz

```

**4. 安装 python3.8**

```
tar -zxvf Python-3.8.1.tgz
cd Python-3.8.1/
./configure ; make && make install

```

**5. 配置环境变量**  
输入 "python3.8 -V", 打印 3.8.1 说明 3.8.1 安装成功  
输入 "python3 -V", 看 python3 对应的版本；  
例如阿里云服务器会默认按照 python2.7 (对应 python -V) 和 python3.6.x (对应 python3 -V)

```
mv /usr/bin/python /usr/bin/python.bak
ln -s /usr/local/bin/python3.8 /usr/bin/python
mv /usr/bin/pip /usr/bin/pip.bak
ln -s /usr/local/bin/pip3.8 /usr/bin/pip

```

**6. 验证**  
输入 "python -V" , 输出 3.8.1 环境变量配置成功  
输入 "pip -V", 输出 19.2.3 ，配置成功  
**7. 配置 yum**  
上面操作完成后，输入 yum 会报错如下，是因为 yum 依赖 python2.7，现在版本改成 3.8.1 了，因此会出错

```
yum update File “/usr/bin/yum”, line 30 except KeyboardInterrupt, e: ^ SyntaxError: invalid syntax

```

**编辑文件 / usr/libexec/urlgrabber-ext-down**

```
vim /usr/libexec/urlgrabber-ext-down

```

改成和下图一样：  
![](https://segmentfault.com/img/remote/1460000044489288)  
编辑 /usr/bin/yum , 和上一步同一样的修改  
**8. 安装依赖**

```
sudo yum install libxml2-devel libxslt-devel -y
sudo pip install lxml
sudo pip install requests

```

##### 安装 PaddlePaddle

参考：[https://www.paddlepaddle.org.cn/documentation/docs/zh/install/pip/linux-pip.html](https://link.segmentfault.com/?enc=XcuUlFG%2BkQPO28Hu4Gj1vQ%3D%3D.rwnlMzdn7xKufvbPALi6wnzxu8hGnLx%2FtveUxo4d%2BnYfq8GKJQUMovBDdL8eusFo0Kl5oX7uIDXDT0W8GSJVMunOlEY%2BSscc9bOFxuAvXtabpPWVH%2Fy873Dkj5d2xsCp)  
**1. 确认 pip 版本，要求 pip 版本为 20.2.2 或更高版本，如果版本低那么更新**

```
pip -V

```

如果 pip 版本低于 20.2.2

```
pip install --upgrade pip

```

**2. 需要确认 Python 和 pip 是 64bit，并且处理器架构是 x86_64（或称作 x64、Intel 64、AMD64）架构。**  
下面的第一行输出的是”64bit”，第二行输出的是”x86_64”、”x64” 或”AMD64” 即可：

```
python -c "import platform;print(platform.architecture()[0]);print(platform.machine())"

```

**3. 安装 paddlepaddle**

```
python -m pip install paddlepaddle==2.5.1 -i https://mirror.baidu.com/pypi/simple

```

**4. 验证 paddlepaddle 安装**  
安装完成后您可以使用命令 python 进入 python 解释器，输入 import paddle ，再输入 paddle.utils.run_check()  
如果出现 PaddlePaddle is installed successfully!，说明您已成功安装。  
**问题处理**  
验证是否成功安装时可能会出现如下错误：  
![](https://segmentfault.com/img/remote/1460000044489289)  
说明缺少 GLIBCXX_3.4.20 依赖。  
解决方法参考：[https://www.jianshu.com/p/050b2b777b9d](https://link.segmentfault.com/?enc=1qd3admQGNm0954gYNS4Nw%3D%3D.lsuGfMGId3wZ%2B9lieOlXoJia5e32kTnzZeGrdhxu96iVqnDFaKr%2FI%2FZ3gMZHLkqN) ，具体执行步骤如下：

```
1. 查看系统版本
strings /usr/lib64/libstdc++.so.6 | grep  ，输出如下：
GLIBCXX
GLIBCXX_3.4
GLIBCXX_3.4.1
GLIBCXX_3.4.2
GLIBCXX_3.4.3
GLIBCXX_3.4.4
GLIBCXX_3.4.5
GLIBCXX_3.4.6
GLIBCXX_3.4.7
GLIBCXX_3.4.8
GLIBCXX_3.4.9
GLIBCXX_3.4.10
GLIBCXX_3.4.11
GLIBCXX_3.4.12
GLIBCXX_3.4.13
GLIBCXX_3.4.14
GLIBCXX_3.4.15
GLIBCXX_3.4.16
GLIBCXX_3.4.17
GLIBCXX_3.4.18
GLIBCXX_3.4.19
GLIBCXX_DEBUG_MESSAGE_LENGTH

发现少了GLIBCXX_3.4.20，解决方法是升级libstdc++.

2. sudo yum provides libstdc++.so.6 ，输出如下：
Loaded plugins: fastestmirror, langpacks
Determining fastest mirrors
libstdc++-4.8.5-39.el7.i686 : GNU Standard C++ Library
Repo        : base
Matched from:
Provides    : libstdc++.so.6

3. cd /usr/local/lib64
# 下载最新版本的libstdc.so_.6.0.26
sudo wget http://www.vuln.cn/wp-content/uploads/2019/08/libstdc.so_.6.0.26.zip
unzip libstdc.so_.6.0.26.zip
# 将下载的最新版本拷贝到 /usr/lib64
cp libstdc++.so.6.0.26 /usr/lib64
cd  /usr/lib64
# 查看 /usr/lib64下libstdc++.so.6链接的版本
ls -l | grep libstdc++ ，输出如下：
libstdc++.so.6 ->libstdc++.so.6.0.19
# 删除/usr/lib64原来的软连接libstdc++.so.6，删除之前先备份一份
sudo rm libstdc++.so.6
# 链接新的版本
sudo ln -s libstdc++.so.6.0.26 libstdc++.so.6
# 查看新版本，成功
strings /usr/lib64/libstdc++.so.6 | grep GLIBCXX ，输出如下：
...
GLIBCXX_3.4.18
GLIBCXX_3.4.19
GLIBCXX_3.4.20
GLIBCXX_3.4.21
GLIBCXX_3.4.22
GLIBCXX_3.4.23
GLIBCXX_3.4.24
GLIBCXX_3.4.25
GLIBCXX_3.4.26
GLIBCXX_DEBUG_MESSAGE_LENGTH
...

```

##### 安装 PaddleOCR

同 windows 下步骤一样，这里不再赘述  
**错误处理，执行 paddleocr 命令时可能会有以下错误：**  
错误 1：缺少 libGL ，“ImportError: libGL.so.1: cannot open shared object file: No such file or directory”，执行如下指令解决：

```
yum install mesa-libGL.x86_64

```

错误 2：urllib3 版本高，依赖的 openssl 版本低，  
ImportError: urllib3 v2.0 only supports OpenSSL 1.1.1+, currently the 'ssl' module is compiled with 'OpenSSL 1.0.2k-fips 26 Jan 2017'. See: [https://github.com/urllib3/urllib3/issues/2168](https://link.segmentfault.com/?enc=QWN3EDCeFHekJbCd39m4ug%3D%3D.qvSDDCa%2Bc0CSryiPEDkIQ1mv3FuXPAxUxDxHygkhijFGAzq0LHbIBqiK%2FPng1BHY)  
解决：降低 urllibs3 的版本，执行如下指令：

```
pip install urllib3==1.26.16 -i https://pypi.tuna.tsinghua.edu.cn/simple

```

### PaddleOCR 命令讲解

前面在验证 PaddleOCR 安装是否成功时已经简单使用过命令了，下面我们详细的讲解一下 paddleocr 指令。查看具体使用方法：

```
paddleocr --help

```

1. 检测 + 方向分类器 + 识别全流程（一般都是用这个）：

```
paddleocr --image_dir ./imgs/11.jpg --use_angle_cls true --use_gpu false --ocr_version PP-OCRv3

```

**--image_dir** 识别图片的路径  
**--use_angle_cls true** 设置使用方向分类器识别 180 度旋转文字  
**--use_gpu false** 设置不使用 GPU  
**--ocr_version PP-OCRv3** 制定模型 ， PaddleOCR 默认会下载使用最新的模型，当前是 PP-OCRv3, 这里只是告诉大家这个参数怎么用  
输出结果是结果是一个 list，每个 item 包含了文本框（坐标），文字和识别置信度：

```
[[[28.0, 37.0], [302.0, 39.0], [302.0, 72.0], [27.0, 70.0]], ('纯臻营养护发素', 0.9658738374710083)]
......

```

文本框分别表示 左上、右上、右下、左下 顺时针方向矩形框的四个角像素坐标

2. 单独使用检测：设置 **--rec 为 false**

```
paddleocr --image_dir ./imgs/11.jpg --rec false

```

结果是一个 list，每个 item 只包含文本框：

```
[[27.0, 459.0], [136.0, 459.0], [136.0, 479.0], [27.0, 479.0]]
[[28.0, 429.0], [372.0, 429.0], [372.0, 445.0], [28.0, 445.0]]
......

```

3. 单独使用识别：设置 **--det 为 false**

```
paddleocr --image_dir ./imgs_words/ch/word_1.jpg --det false

```

结果是一个 list，每个 item 只包含识别结果和识别置信度

```
['韩国小馆', 0.994467]

```

4. 指定语言 **--lang**（默认也能识别英文，制定语言效果会更好）

```
paddleocr --image_dir ./imgs_en/254.jpg --lang=en

```

5.paddleocr 也支持输入 pdf 文件，并且可以通过指定参数 **page_num** 来控制推理前面几页，默认为 0，表示推理所有页

```
paddleocr --image_dir ./xxx.pdf --use_angle_cls true --use_gpu false --page_num 2

```

### Python 脚本中使用 PaddleOCR

*   检测 + 方向分类器 + 识别全流程，只需要下面三行代码
    
    ```
    #导入依赖
    from paddleocr import PaddleOCR
    #创建PaddleOCR对象，只需要在初始化时执行一次该语句
    ocr = PaddleOCR(use_angle_cls=True, det=False, use_gpu=False)
    #识别图片返回结果，cls=True 表示识别旋转180度的文字，如果没有文字旋转180度，那么
    #可以cls=False，这样会提升性能，旋转90度和270度也能够识别
    result = ocr.ocr(imgPath, cls=True)
    
    ```
    
    ### 识别服务开发
    
    ![](https://segmentfault.com/img/remote/1460000044489290)
    
    #### PaddleOCR 内部接口开发
    
    ##### 开发工具
    
    开发工具为 PyCharm 社区版
    
    ##### 代码开发
    
    新建一个项目，将下面的代码，拷贝到 main.py(PyCharm 默认会新建一个)  
    ![](https://segmentfault.com/img/remote/1460000044489291)  
    下面这段 paddle_ocr_web.py python 代码主要是实现：通过 flask 创建 web 容器，并通过 /ocr 接口调用 paddleocr 进行图片识别，并通过 json 格式返回识别结果。
    
    ```
    import json
    import logging
    from paddleocr import PaddleOCR
    from flask import Flask, request, jsonify
    
    # 设置日志输出等级
    logging.basicConfig(level=logging.DEBUG)
    
    # 只需要初始化加载一次
    ocr = PaddleOCR(use_angle_cls=True)
    
    # 所有的Flask都必须创建程序实例，程序实例是Flask的对象，一般情况下用如下方法实例化
    app = Flask(__name__)
    
    # https://cloud.tencent.com/developer/article/1539199 flask request常用方法
    # @app.route('/ocr', methods=["GET"])
    # def paddleOcr():
    #     params = request.args
    #     # 获取参数
    #     imgPath = params.get("imgPath", 0)
    #     logging.info("paddle ocr img %s", imgPath)
    #     result = ocr.ocr(imgPath, cls=True)
    #     return jsonify(data=result), 200
    
    @app.route('/ocr', methods=["POST"])
    def paddleOcr():
      # 获取传入参数
      data = json.loads(request.data)
      # 获取传入的图片路径
      imgPath = data["imgPath"]
      logging.info("paddle ocr2 imgPath %s", imgPath)
      #进行ocr识别
      # 如果不需要检测坐标，那么可以设置 det=False,result = ocr.ocr(imgPath, cls=True, det=False)
      result = ocr.ocr(imgPath, cls=True)
      #识别结果通过json格式返回
      return jsonify(data=result), 200
    
    
    #  程序实例用run方法启动flask集成的web服务器
    if __name__ == '__main__':
      # 可以返回中文字符,否则汉字会以Unicode编码形式返回
      app.config['JSON_AS_ASCII'] = False
      # 接收所有IP的请求,debug=True 代码修改，web容器会重启
      app.run(host='0.0.0.0', debug=True, port=8888)
    
    ```
    
*   import 表示导入整个 module(模块)，一个. py 文件就是一个 module
*   from A import B 表示导入 A 模块中的 B（可以为方法、类）
*   flask 是 python 实现的 web 框架，类似 Tomcat
    
    ```
           ![](https://cdn.nlark.com/yuque/0/2023/png/13011155/1696664523936-11347d46-d464-4f1d-8c04-7b49f64fc454.png#averageHue=%23f6eacf&clientId=u88108e0d-ebc3-4&from=paste&height=324&id=u59905347&originHeight=357&originWidth=693&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=stroke&taskId=uf2d39be9-7b2a-41d0-a6b7-da5877bd086&title=&width=629)
     1.如果是json格式的请求数据，则是采用request.data来获取请求体的字符串。
     2.如果是form表单的请求体，那么则可以使用request.form来获取参数。
     3.如果是url参数，例如：url?param1=xx¶m2=xx，那么则可以使用request.args来获取参数。
     4.如果需要区分GET\POST请求方法，则可以使用request.method来进行判断区分
     5.参考：[https://cloud.tencent.com/developer/article/1539199](https://cloud.tencent.com/developer/article/1539199)，解释的很清楚
     6.@app.route 声明一个接口，指定请求方法和U请求路径
    
    
    ```
    
*   作为 Python 的内置变量，****name**** 它是每个 Python 模块必备的属性，但它的值取决于你是如何执行这段代码。通过 ****name**** 变量，可以判断出这时代码是被直接运行，还是被导入到其他程序中去了。当直接执行一段脚本的时候，这段脚本的 ****name**** 变量等于 **'__main__'**，当这段脚本被导入其他程序的时候，****name**** 变量等于脚本本身的名字。
    
    ##### 测试
    
    运行 main.py, 在项目下创建 test 文件夹，并拷贝一张图片到该文件下  
    ![](https://segmentfault.com/img/remote/1460000044489292)  
    使用 postman 请求，如下图：  
    ![](https://segmentfault.com/img/remote/1460000044489293)  
    返回结果：
    
    ```
    {
    "data": [
      [
        [
          [
            [
              20.0,
              35.0
            ],
            [
              190.0,
              35.0
            ],
            [
              190.0,
              61.0
            ],
            [
              20.0,
              61.0
            ]
          ],
          [
            "Docker部署",
            0.9223976731300354
          ]
        ]
      ]
    ]
    }
    
    ```
    
    **此文档是**《[OCR 文字识别实战教程 - 零基础](https://www.bilibili.com/video/BV1RK411b7DU/)，SpringBoot 结合 PaddleOCR》 地址：[https://www.bilibili.com/video/BV1RK411b7DU/?vd_source=b4307343204f5c0271966f7fe276f0eb](https://www.bilibili.com/video/BV1RK411b7DU/?vd_source=b4307343204f5c0271966f7fe276f0eb)，**教程对应的部分文档，全部文档共 89 页，剩余文档和完整源代码 请关注 B 站视频**