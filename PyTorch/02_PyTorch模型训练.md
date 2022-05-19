## PyTorch模型训练

### 1. 自定义损失函数

PyTorch在``torch.nn``模块为我们提供了许多常用的损失函数，比如：MSELoss，L1Loss，BCELoss… 但是随着深度学习的发展，出现了越来越多的非官方提供的Loss，比如DiceLoss，HuberLoss，SobolevLoss… 这些Loss Function专门针对一些非通用的模型，PyTorch不能将他们全部添加到库中去，因此这些损失函数的实现则需要我们通过自定义损失函数来实现。

#### 1.1 以函数方式定义

通过输出值和目标值进行计算，返回损失值

```python
import torch

def my_loss(output, target):
    loss = torch.mean((output - target)**2)
    return loss
```

#### 1.2 以类方式定义(更常用)

一般地，``Loss``函数部分继承自``_loss``, 部分继承自``_WeightedLoss``

而``_WeightedLoss``继承自``_loss``，``_loss``继承自 ``nn.Module``

所以，我们自定义的损失函数类就需要继承自``nn.Module``类

下面以分割领域常见的``Dice Loss``损失函数举例：
$$
DSC = \frac{2|X∩Y|}{|X|+|Y|}
$$

```python
import torch.nn as nn
from torch.autograd.grad_mode import F


class DiceLoss(nn.Module):
    def __init__(self, weight=None, size_average=True):
        super(DiceLoss, self).__init__()

    def forward(self, inputs, targets, smooth=1):
        inputs = F.sigmoid(inputs)
        inputs = inputs.view(-1)
        targets = targets.view(-1)
        intersection = (inputs * targets).sum()
        dice = (2. * intersection + smooth) / (inputs.sum() + targets.sum() + smooth)
        return 1 - dice
```



### 2. 动态调整学习率

学习率的选择: 学习速率设置过小，会极大降低收敛速度，增加训练时间；学习率太大，可能导致参数在最优解两侧来回振荡。

但是当我们选定了一个合适的学习率后，经过许多轮的训练后，可能会出现准确率震荡或loss不再下降等情况，说明当前学习率已不能满足模型调优的需求。此时我们就可以通过一个适当的学习率衰减策略来改善这种现象，提高我们的精度。

虽然PyTorch官方给我们提供了许多的API，但是在实验中也有可能碰到需要我们自己定义学习率调整策略的情况，而我们的方法是自定义函数adjust_learning_rate来改变param_group中lr的值。

#### 2.1 官方API

[各种 Scheduler 学习率曲线可视化](https://zhuanlan.zhihu.com/p/352821601)

```python
#  使用函数（Lambda）来直接返回新的学习率，可操作性最高，其他 Scheduler 均可以由这个学习率来实现
lr_scheduler.LambdaLR  # LambdaLR更新学习率方式是 lr = lr*lr_lambda
# 使用函数（lambda）返回一个衰减因子，当前学习率乘以这个衰减因子来获得新的学习率。
lr_scheduler.MultiplicativeLR  
# 在每个固定步长 step_size 后按固定比例（gamma）衰减学习率
lr_scheduler.StepLR  
# 以用户设置的 milestone 为节点，经过每个 milestone 后都以一定比例（gamma）衰减学习率
lr_scheduler.MultiStepLR  
# 以固定比例（gamma）直接衰减学习率
lr_scheduler.ExponentialLR  
# 余弦退火，这个 Scheduler 在各种比赛中经常会用到
lr_scheduler.CosineAnnealingLR
# 基于训练过程中的某些测量值对学习率进行动态的下降
lr_scheduler.ReduceLROnPlateau
# 学习率在两个边界之间以固定频率变化，该 Scheduler 的 step 函数应该在每个 iteration（而不是 epoch）中被调用
lr_scheduler.CyclicLR
# 先从一个较低的学习率开始，逐渐达到一个峰值，然后再从峰值下降到一个比初始学习率更小的值
lr_scheduler.OneCycleLR
# 跟余弦退火类似，只是在学习率上升时使用热启动 在各种比赛中也经常会用到
lr_scheduler.CosineAnnealingWarmRestarts
```

#### 2.2 自定义scheduler

通过自定义函数`adjust_learning_rate`来改变`param_group`中`lr`的值来实现

```python
def adjust_learning_rate(optimizer, epoch):
    lr = args.lr * (0.1 ** (epoch // 30))
    for param_group in optimizer.param_groups:
        param_group['lr'] = lr
```

调用过程如下(伪代码)：

```python
def adjust_learning_rate(optimizer, ...):
    ...


optimizer = torch.optim.SGD(model.parameters(), lr=args.lr, momentum=0.9)
for epoch in range(10):
    train(...)
    validate(...)
    adjust_learning_rate(optimizer, epoch)
```

### 3. 模型微调

迁移学习(transfer learning)，就是将从源数据集学到的知识迁移到目标数据集上。

例如，虽然ImageNet数据集的图像大多跟椅子无关，但在该数据集上训练的模型可以抽取较通用的图像特征，从而能够帮助识别边缘、纹理、形状和物体组成等。

迁移学习的一大应用场景是模型微调（finetune）。简单来说，就是我们先找到一个同类的别人训练好的模型，把别人现成的训练好了的模型拿过来，换成自己的数据，通过训练调整一下参数。 

在PyTorch中提供了许多预训练好的网络模型（VGG，ResNet系列，mobilenet系列…），这些模型都是PyTorch官方在相应的大型数据集训练好的。

#### 3.1 概念

通过对已经训练好的模型进行参数调整，使其用于新数据集训练

#### 3.2 流程

1. 拿到源模型，可以下载或者自行在源数据集训练
2. 将源模型输出层外所有的结构和参数复制到目标模型
3. 重构目标模型的输出层，并随机初始化改成的对应参数
4. 使用目标数据集训练目标模型

#### 3.3 图示

<img src="D:/NoteBook_md/images/Snipaste_2022-03-19_21-12-16.png" style="zoom:67%;" />
