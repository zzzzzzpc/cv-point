# UW-Madison Tract Image Segmentation（铜）

比赛地址：https://www.kaggle.com/competitions/uw-madison-gi-tract-image-segmentation

类别：医学图像，CT，多器官分割

名次：89/1548

### Our Method

基本思路：

使用3D-Unet模型，baseline参考：https://www.kaggle.com/code/yiheng/3d-solution-with-monai-infer

提升思路：

微调模型，先用大学习率2e-4训练1000个epoch，然后再使用小学习率1e-6训练1000个epoch，之后使用TTA训练200epoch，TTA分别在第2，第3，以及第（2，3）维度上进行flip的操作（原来的维度为（batchsize，深度，长，宽），在2，3维度上进行flip相当于对角转一下）。

![image-20220724195026005](D:\OneDrive\Documents\cv-point\imgs\kaggle-uw1.png)

交叉验证5个模型，但是最后实际没有提交全部的5个模型（5个模型的效果不是很好，无论是LB还是CV，或许和数据分布有关系？）

### 一些失败的尝试

