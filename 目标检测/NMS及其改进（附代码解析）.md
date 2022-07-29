# NMS及其改进（附代码解析）

### NMS(非极大值抑制)

在目标检测任务当中，我们可能同时在一定范围内的区域中预测出了多个框，但是实际上，我们需要的可能只有一个框，那么如果去掉这些多余的框呢？NMS算法的目的便在于此。

具体可以看下面这个例子：

![face box](https://images2017.cnblogs.com/blog/606386/201708/606386-20170826152837558-1289161833.png)

左边是我们网络预测的原始框，可以看到有很多的多余的框，例如在Yolo算法当中，每一个小格子BoundingBox都要负责预测一个框，那么可能相邻的格子预测的框就会叠加在一起，我们希望得到的最终效果是如右图，只有一个框框出我们需要的目标区域。

NMS的过程用一句话就可以说清楚：

对于Bounding Box的列表B及其对应的置信度S,采用下面的计算方式，选择具有最大score的检测框M，将其从B集合中移除并加入到最终的检测结果D中。通常将B中剩余检测框中与M的IoU大于阈值Nt的框从B中移除，重复这个过程，直到B为空。

在下面这个例子当中，我们有两个目标，分别要识别出两个奥特曼的人脸：

1. 6个带置信率的region proposals，我们先预设一个IOU的阈值为0.7
2. 按照置信率的大小对6个框排序，结果分别为0.94, 0.91, 0.90, 0.83, 0.79. 0.77.
3. 设定置信率为0.94的region proposals为一个物体框
4. 在剩下5个region proposals中进行循环遍历，去掉与0.94物体框IOU大于0.7的
5. 重复2~4的步骤，直到没有region proposals为止
6. 每次获取到的最大置信率的region proposals就是我们筛选出来的目标

![](https://files.mdnice.com/user/6935/d16d78d9-7595-44bf-895c-4d88902c7c2c.jpg)

```python
# 假设输入
import numpy as np

def NMS(dets, thresh):
    x1 = dets[:, 0]
    y1 = dets[:, 1]
    x2 = dets[:, 2]
    y2 = dets[:, 3]
    scores = dets[:, 4]
    
    areas = (x2 - x1 + 1) * (y2 - y1 + 1)
    # 从大到小排列，取-1
    order = scores.argsort()[::-1]
    
    keep = []
    while order.size() > 0:
        i = order[0]
        xx1 = np.maximum(x1[i], x1[order[1:]])
        yy1 = np.maximum(y1[i], y1[order[1:]])
        xx2 = np.maximum(x2[i], x2[order[1:]])
        yy2 = np.minimum(y2[i], y2[order[1:]])
        
        # 计算相交的面积，不重叠时候面积为0
        w = np.maximum(0.0, xx2 - xx1 + 1)
        h = np.maximum(0.0, yy2 - yy1 + 1)
        inter = w * h
        
        # 计算IOU面积
        ovr = inter / (areas[i] + areas[order[1:]] - inter)
        
        # 保留IoU小于阈值的box，注意这里返回一个tuple，取0
        inds = np.where(ovr <= thresh)[0]
        
        # 替换掉原来的order，注意这里加了1，因为ovr数组长度比原来少一个
        order = order[inds + 1]
```

### SoftNMS

... 未完待续