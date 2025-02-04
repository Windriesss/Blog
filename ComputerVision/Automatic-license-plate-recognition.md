# Automatic license plate recognition

#                                    ——车牌识别算法介绍 

汽车牌照自动识别整个处理过程分为预处理、边缘提取、车牌定位、字符分割、字符识别五大模块，其中字符识别过程主要由以下3个部分组成：

①正确地分割文字图像区域；

②正确的分离单个文字；

③正确识别单个字符。



系统设计概述

由于车辆牌照是机动车唯一的管理标识符号，在交通管理中具有不可替代的作用，因此车辆牌照识别系统应具有很高的识别正确率，对环境光照条件、拍摄位置和车辆行驶速度等因素的影响应有较大的容阈，并且要求满足实时性要求。

                             ![img](https://github.com/ForrestPi/Blog/blob/master/ComputerVision/pic/Automatic001.jpg)

该系统是计算机图像处理与字符识别技术在智能化交通管理系统中的应用，它主要由牌照图像的采集和预处理、牌照区域的定位和提取、牌照字符的分割和识别等几个部分组成。

其基本工作过程如下：

    （1）当行驶的车辆经过时，触发埋设在固定位置的传感器，系统被唤醒处于工作状态；

         一旦连接摄像头光快门的光电传感器被触发，设置在车辆前方、后方和侧面的相机同时拍摄下车辆图像；

    （2）由摄像机或CCD摄像头拍摄的含有车辆牌照的图像通视频卡输入计算机进行预处理，

         图像预处理包括图像转换、图像增强、滤波和水平较正等；

    （3）由检索模块进行牌照搜索与检测，定位并分割出包含牌照字符号码的矩形区域；

    （4）对牌照字符进行二值化并分割出单个字符，经归一化后输入字符识别系统进行识别。

总体设计方案：

车辆牌照识别整个系统主要是由车牌定位和字符识别两部分组成，其中车牌定位又可以分为图像预处理及边缘提取模块和牌照的定位及分割模块；字符识别可以分为字符分割与特征提取和单个字符识别两个模块。

为了用于牌照的分割和牌照字符的识别，原始图象应具有适当的亮度，较大的对比度和清晰可辩的牌照图象。但由于该系统的摄像部分工作于开放的户外环境，加之车辆牌照的整洁度、自然光照条件、拍摄时摄像机与牌照的矩离和角度以及车辆行驶速度等因素的影响，牌照图象可能出现模糊、歪斜和缺损等严重缺陷，因此需要对原始图象进行识别前的预处理。

牌照的定位和分割是牌照识别系统的关键技术之一，其主要目的是在经图象预处理后的原始灰度图象中确定牌照的具体位置，并将包含牌照字符的一块子图象从整个图象中分割出来，供字符识别子系统识别之用，分割的准确与否直接关系到整个牌照字符识别系统的识别率。

由于拍摄时的光照条件、牌照的整洁程度的影响，和摄像机的焦距调整、镜头的光学畸变所产生的噪声都会不同程度地造成牌照字符的边界模糊、细节不清、笔划断开或粗细不均，加上牌照上的污斑等缺陷，致使字符提取困难，进而影响字符识别的准确性。因此，需要对字符在识别之前再进行一次针对性的处理。

车牌识别的最终目的就是对车牌上的文字进行识别。

 

各部分实现流程：

一、图像采集和转换

考虑到现有牌照的字符与背景的颜色搭配一般有蓝底白字、黄底黑字、白底红字、绿底白字和黑底白字等几种，利用不同的色彩通道就可以将区域与背景明显地区分出来，例如，对蓝底白字这种最常见的牌照，采用蓝色B 通道时牌照区域为一亮的矩形，而牌照字符在区域中并不呈现。因为蓝色（255，0，0）与白色（255，255，255）在B通道中并无区分，而在G、R 通道或是灰度图象中并无此便利。同理对白底黑字的牌照可用R 通道，绿底白字的牌照可以用G 通道就可以明显呈现出牌照区域的位置，便于后续处理。

二、边缘提取

边缘是指图像局部亮度变化显著的部分，是图像风、纹理特征提取和形状特征提取等图像分析的重要基础。所以在此我们要对图像进行边缘检测。图象增强处理对图象牌照的可辩认度的改善和简化后续的牌照字符定位和分割的难度都是很有必要的。增强图象对比度度的方法有：灰度线性变换、图象平滑处理等。

2.1 灰度矫正

由于牌照图象在拍摄时受到种种条件的限制和干扰，图象的灰度值往往与实际景物不完全匹配，这将直接影响到图象的后续处理。如果造成这种影响的原因主要是由于被摄物体的远近不同，使得图象中央区域和边缘区域的灰度失衡，或是由于摄像头在扫描时各点的灵敏度有较大的差异而产生图象灰度失真，或是由于曝光不足而使得图像的灰度变化范围很窄。这时就可以采用灰度校正的方法来处理，增强灰度的变化范围、丰富灰度层次，以达到增强图象的对比度和分辨率。我们发现车辆牌照图象的灰度取值范围大多局限在r=(50,200)之间，而且总体上灰度偏低，图象较暗。根据图象处理系统的条件，最好将灰度范围展开到s=(0,255)之间。

2.2图像平滑处理

对于受噪声干扰严重的图象，由于噪声点多在频域中映射为高频分量，因此可以在通过低也可以直接在空域中用求邻域平均值的方法来通滤波器来滤除噪声，但实际中为了简化算法，削弱噪声的影响，这种方法称为图象平滑处理。

    然而，邻域平均值的平滑处理会使得图象灰度急剧变化的地方，尤其是物体边缘区域和字符轮廓等部分产生模糊作用。为了克服这种平均化引起的图象模糊现象，我们给中心点象素值与其邻域平均值的差值设置一固定的阈值，只有大于该阈值的点才能替换为邻域平均值，而差值不大于阈值时，仍保留原来的值，从而减少由于平均化引起的图象模糊。

                                           ![img](https://github.com/ForrestPi/Blog/blob/master/ComputerVision/pic/Automatic002.jpg)

    图像中车辆牌照是具有比较显著特征的一块图象区域，这此特征表现在：近似水平的矩形区域；其中字符串都是按水平方向排列的；在整体图象中的位置较为固定。正是由于牌照图象的这些特点，再经过适当的图象变换，它在整幅中可以明显地呈现出其边缘。边缘提取是较经典的算法，此处边缘的提取采用的是Roberts算子。

                 ![img](https://github.com/ForrestPi/Blog/blob/master/ComputerVision/pic/Automatic003.jpg)

分析这种情况产生的原因，归纳起来主要有以下方面:

 1、原始图像清晰度比较高，从而简化了预处理

 2、图像的平滑处理会使图像的边缘信息受到损失，图像变得模糊

 3、图像的锐化可以增强图像中物体的边缘轮廓，但同时也使一些噪声得到了增强

 

三、牌照的定位和分割

牌照的定位和分割是牌照识别系统的关键技术之一，其主要目的是在经图象预处理后的原始灰度图象中确定牌照的具体位置，并将包含牌照字符的一块子图象从整个图象中分割出来，供字符识别子系统识别之用，分割的准确与否直接关系到整个牌照字符识别系统的识别率。由于牌照图象在原始图象中是很有特征的一个子区域，确切说是水平度较高的横向近似的长方形，它在原始图象中的相对位置比较集中，而且其灰度值与周边区域有明显的不同，因而在其边缘形成了灰度突变的边界，这样就便于通过边缘检测来对图象进行分割。

    3.1牌照区域定位

    ​牌照图象经过了以上的处理后，牌照区域已经十分明显，而且其边缘得到了勾勒和加强。此时可进一步确定牌照在整幅图象中的准确位置。这里选用的是数学形态学的方法，其基本思想是用具有一定形态的机构元素去量度和提取图像中的对应形状以达到对图像分析和识别的目的。数学形态学的应用可以简化图像数据，保持它们基本的形态特征，并除去不相干的结构。

  3.2牌照区域分割    ​

对车牌的分割可以有很多种方法，本程序是利用车牌的彩色信息的彩色分割方法。根据车牌底色等有关的先验知识，采用彩色像素点统计的方法分割出合理的车牌区域，确定车牌底色蓝色RGB对应的各自灰度范围，然后行方向统计在此颜色范围内的像素点数量，设定合理的阈值，确定车牌在行方向的合理区域。然后，在分割出的行区域内，统计列方向蓝色像素点的数量，最终确定完整的车牌区域。

                            ![img](https://github.com/ForrestPi/Blog/blob/master/ComputerVision/pic/Automatic004.jpg)

3.3统一处理    ​

经过上述方法分割出来的车牌图像中存在目标物体、背景还有噪声，要想从图像中直接提取出目标物体，最常用的方法就是设定一个阈值T，用T将图像的数据分成两部分：大于T的像素群和小于T的像素群，即对图像二值化。均值滤波是典型的线性滤波算法，它是指在图像上对目标像素给一个模板，该模板包括了其周围的临近像素。再用模板中的全体像素的平均值来代替原来像素值。

                                  ![img](https://github.com/ForrestPi/Blog/blob/master/ComputerVision/pic/Automatic005.jpg)

四、字符分割和比对

4.1字符分割

在汽车牌照自动识别过程中，字符分割有承前启后的作用。它在前期牌照定位的基础上进行字符的分割，然后再利用分割的结果进行字符识别。字符识别的算法很多，因为车牌字符间间隔较大，不会出现字符粘连情况，所以此处采用的方法为寻找连续有文字的块，若长度大于某阈值，则认为该块有两个字符组成，需要分割。

                                 ![img](https://github.com/ForrestPi/Blog/blob/master/ComputerVision/pic/Automatic006.jpg)

4.2字符归一化

一般分割出来的字符要进行进一步的处理，以满足下一步字符识别的需要。但是对于车牌的识别，并不需要太多的处理就已经可以达到正确识别的目的。在此只进行了归一化处理，然后进行后期处理。

                                     ![img](https://github.com/ForrestPi/Blog/blob/master/ComputerVision/pic/Automatic007.jpg)

4.3字符识别

字符的识别目前用于车牌字符识别(OCR)中的算法主要有基于模板匹配的OCR算法以及基于人工神经网络的OCR算法。基于模板匹配的OCR的基本过程是:首先对待识别字符进行二值化并将其尺寸大小缩放为字符数据库中模板的大小，然后与所有的模板进行匹配，最后选最佳匹配作为结果。用人工神经网络进行字符识别主要有两种方法:一种方法是先对待识别字符进行特征提取，然后用所获得的特征来训练神经网络分类器。识别效果与字符特征的提取有关，而字符特征提取往往比较耗时。因此，字符特征的提取就成为研究的关键。另一种方法则充分利用神经网络的特点，直接把待处理图像输入网络，由网络自动实现特征提取直至识别。

模板匹配的主要特点是实现简单，当字符较规整时对字符图像的缺损、污迹干扰适应力强且识别率相当高。综合模板匹配的这些优点我们将其用为车牌字符识别的主要方法。

模板匹配是图象识别方法中最具代表性的基本方法之一，它是将从待识别的图象或图象区域f(i,j)中提取的若干特征量与模板T(i,j)相应的特征量逐个进行比较，计算它们之间规格化的互相关量，其中互相关量最大的一个就表示期间相似程度最高，可将图象归于相应的类。也可以计算图象与模板特征量之间的距离，用最小距离法判定所属类。然而，通常情况下用于匹配的图象各自的成像条件存在差异，产生较大的噪声干扰，或图象经预处理和规格化处理后，使得图象的灰度或像素点的位置发生改变。在实际设计模板的时候，是根据各区域形状固有的特点，突出各类似区域之间的差别，并将容易由处理过程引起的噪声和位移等因素都考虑进去，按照一些基于图象不变特性所设计的特征量来构建模板，就可以避免上述问题。

此处采用相减的方法来求得字符与模板中哪一个字符最相似，然后找到相似度最大的输出。汽车牌照的字符一般有七个，大部分车牌第一位是汉字，通常代表车辆所属省份，或是军种、警别等有特定含义的字符简称；紧接其后的为字母与数字。车牌字符识别与一般文字识别在于它的字符数有限，汉字共约50多个，大写英文字母26个，数字10个。所以建立字符模板库也极为方便。为了实验方便，结合本次设计所选汽车牌照的特点，只建立了4个数字26个字母与10个数字的模板。其他模板设计的方法与此相同。

首先取字符模板，接着依次取待识别字符与模板进行匹配，将其与模板字符相减，得到的0越多那么就越匹配。把每一幅相减后的图的0值个数保存，然后找数值最大的，即为识别出来的结果。

                                              ![img](https://github.com/ForrestPi/Blog/blob/master/ComputerVision/pic/Automatic008.jpg)