# UW-Madison Tract Image Segmentation（铜）

比赛地址：https://www.kaggle.com/competitions/uw-madison-gi-tract-image-segmentation

类别：医学图像，CT，多器官分割

名次：89/1548

### Our Method

基本思路：

使用3D-Unet模型，baseline参考：https://www.kaggle.com/code/yiheng/3d-solution-with-monai-infer

提升思路：

微调模型，学习率余弦退火，先用大学习率2e-4训练1000个epoch（使用bce+dice），然后再使用小学习率1e-6训练1000个epoch（只用dice），之后使用TTA训练200epoch，TTA分别在第2，第3，以及第（2，3）维度上进行flip的操作（原来的维度为（batchsize，深度，长，宽），在2，3维度上进行flip相当于对角转一下）。

![image-20220724195026005](..\imgs\kaggle-uw1.png)

交叉验证5个模型，但是最后实际没有提交全部的5个模型（5个模型的效果不是很好，无论是LB还是CV，或许和数据分布有关系？）

数据增强：形变，擦除，flip（水平，垂直），仿射变换。

后处理：手工设定范围，去除不存在label的结果，提高了精度。

![image-20220724202028191](..\imgs\kaggle-uw2.png)

### 一些失败的尝试

2.5D模型，效果上（5channel，stride=1）>（3channel，stride=2）。但是2.5D效果不如3D，训练epoch为25个（可能是2.5D的epoch不够，训练时间太久1天多训练完5个fold）。

2.5D使用不同的backbone，效果不如effnet。2.5D尝试Attention-unet（时间太久，放弃了），unet++（有提升，但还是不如3DUnet）。

数据增强：光照，mixup，效果变低了。

图片的尺寸，不同的padding。

### 别人的方法

#### 2st

先预测哪些slice有label，然后2.5D和3D分别预测那些有标记的slice，最后ensemble。

https://www.kaggle.com/competitions/uw-madison-gi-tract-image-segmentation/discussion/337400

### 3st

纯2.5D

detetor检测main area，然后再其中进行2.5D的分割。

https://www.kaggle.com/competitions/uw-madison-gi-tract-image-segmentation/discussion/337468

#### 1st

![img](https://raw.githubusercontent.com/CarnoZhao/Kaggle-UWMGIT/kaggle_tractseg/data/tract/pipeline.png)

不同之处在于先分割，然后将后面的没有label的slice挑选出来。最后2.5D和3D融合。

https://www.kaggle.com/competitions/uw-madison-gi-tract-image-segmentation/discussion/337197

### 总结

还是要先判断哪些slice有标记，然后分割，多模型ensemble。