## PyTorch模型定义

### 1. PyTorch基础知识

- torch.nn 模块里提供的一个**模型构造类Module**
  (nn.Module)，是所有神经网络模块的基类，我们可以继承它来定义我们想要的模型
- PyTorch模型定义应包括两个主要部分：(1) 各个部分的初始化（init）；(2) 数据流向定义（forward）

基于nn.Module，我们可以通过Sequential，ModuleList和ModuleDict三种方式定义PyTorch模型。

### 2. 通过Sequential定义PyTorch模型

当模型的前向计算为简单串联各个层的计算时， Sequential 类可以通过更加简单的方式定义模型。它可以接收一个子模块的有序字典(OrderedDict) 或者一系列子模块作为参数来逐一添加 Module 的实例，⽽模型的前向计算就是将这些实例按添加的顺序逐⼀计算。

我们结合Sequential和定义方式加以理解：

```python
class MySequential(nn.Module):
    from collections import OrderedDict
    def __init__(self, *args):
        super(MySequential, self).__init__()
        if len(args) == 1 and isinstance(args[0], OrderedDict): # 如果传入的是一个OrderedDict
            for key, module in args[0].items():
                self.add_module(key, module)  # add_module方法会将module添加进self._modules(一个OrderedDict)
        else:  # 传入的是一些Module
            for idx, module in enumerate(args):
                self.add_module(str(idx), module)
    def forward(self, input):
        # self._modules返回一个 OrderedDict，保证会按照成员添加时的顺序遍历成
        for module in self._modules.values():
            input = module(input)
        return input
```

使用Sequential来定义模型。只需要将模型的层按序排列起来即可，根据层名的不同，排列的时候有两种方式：

#### （1）直接排列

```python
net = nn.Sequential(
        nn.Linear(784, 256),
        nn.ReLU(),
        nn.Linear(256, 10), 
        )
print(net)
```

```python
Sequential(
  (0): Linear(in_features=784, out_features=256, bias=True)
  (1): ReLU()
  (2): Linear(in_features=256, out_features=10, bias=True)
)
```

#### （2）使用OrderedDict：

```python
import collections
import torch.nn as nn
net2 = nn.Sequential(collections.OrderedDict([
          ('fc1', nn.Linear(784, 256)),
          ('relu1', nn.ReLU()),
          ('fc2', nn.Linear(256, 10))
          ]))
print(net2)
```

```python
Sequential(
  (fc1): Linear(in_features=784, out_features=256, bias=True)
  (relu1): ReLU()
  (fc2): Linear(in_features=256, out_features=10, bias=True)
)
```

可以看到，使用Sequential定义模型的好处在于简单、易读，同时使用Sequential定义的模型不需要再写forward，因为顺序已经定义好了。但使用Sequential也会使得模型定义丧失灵活性，比如需要在模型中间加入一个外部输入时就不适合用Sequential的方式实现。使用时需根据实际需求加以选择。

### 3. 通过ModuleList定义PyTorch模型

ModuleList 接收一个子模块（或层，需属于nn.Module类）的列表作为输入，然后也可以类似List那样进行append和extend操作。同时，子模块或层的参数也会自动添加到网络中来。

```python
net = nn.ModuleList([nn.Linear(784, 256), nn.ReLU()])
net.append(nn.Linear(256, 10)) # # 类似List的append操作
print(net[-1])  # 类似List的索引访问
print()
print(net)
```

```python
Linear(in_features=256, out_features=10, bias=True)

ModuleList(
  (0): Linear(in_features=784, out_features=256, bias=True)
  (1): ReLU()
  (2): Linear(in_features=256, out_features=10, bias=True)
)
```

要特别注意的是，nn.ModuleList 并没有定义一个网络，它只是将不同的模块储存在一起。ModuleList中元素的先后顺序并不代表其在网络中的真实位置顺序，需要经过forward函数指定各个层的先后顺序后才算完成了模型的定义。具体实现时用for循环即可完成：

```python
class model(nn.Module):
    def __init__(self, modulelist):
        self.modulelist = modulelist

    def forward(self, x):
        for layer in self.modulelist:
            x = layer(x)
            return x
```

### 4. 通过ModuleDict定义PyTorch模型

ModuleDict和ModuleList的作用类似，只是ModuleDict能够更方便地为神经网络的层添加名称。

```python
net = nn.ModuleDict({
    'linear': nn.Linear(784, 256),
    'act': nn.ReLU(),
})
net['output'] = nn.Linear(256, 10) # 添加
print(net['linear']) # 访问
print()
print(net.output)
print()
print(net)
```

```python
Linear(in_features=784, out_features=256, bias=True)

Linear(in_features=256, out_features=10, bias=True)

ModuleDict(
  (linear): Linear(in_features=784, out_features=256, bias=True)
  (act): ReLU()
  (output): Linear(in_features=256, out_features=10, bias=True)
)
```

### 5. 三种方法的比较与适用场景

Sequential适用于快速验证结果，因为已经明确了要用哪些层，直接写一下就好了，不需要同时写__init__和forward；

ModuleList和ModuleDict在某个完全相同的层需要重复出现多次时，非常方便实现，可以”一行顶多行“；

当我们需要之前层的信息的时候，比如 ResNets 中的 残差计算，当前层的结果需要和之前层中的结果进行融合，一般使用 ModuleList/ModuleDict 比较方便。

### 6. 利用模型块快速搭建复杂网络

#### (1) U-Net简介

U-Net是分割 (Segmentation) 模型的杰作，在以医学影像为代表的诸多领域有着广泛的应用。U-Net模型结构如下图所示，通过残差连接结构解决了模型学习中的退化问题，使得神经网络的深度能够不断扩展。

![在这里插入图片描述](images/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5L6N5Lyf,size_20,color_FFFFFF,t_70,g_se,x_16-16474365102392.png)

U-Net模型具有非常好的对称性。模型从上到下分为若干层，每层由左侧和右侧两个模型块组成，每侧的模型块与其上下模型块之间有连接；同时位于同一层左右两侧的模型块之间也有连接，称为“Skip-connection”。此外还有输入和输出处理等其他组成部分。由于模型的形状非常像英文字母的“U”，因此被命名为“U-Net”。

组成U-Net的模型块主要有如下几个部分：

1. 每个子块内部的两次卷积（Double Convolution）
2. 左侧模型块之间的下采样连接，即最大池化（Max pooling）
3. 右侧模型块之间的上采样连接（Up sampling）
4. 输出层的处理

除模型块外，还有模型块之间的横向连接，输入和U-Net底部的连接等计算，这些单独的操作可以通过forward函数来实现。

#### (2) U-Net模型块实现

在使用PyTorch实现U-Net模型时，我们不必把每一层按序排列显式写出，这样太麻烦且不宜读，一种比较好的方法是先定义好模型块，再定义模型块之间的连接顺序和计算方式。就好比装配零件一样，我们先装配好一些基础的部件，之后再用这些可以复用的部件得到整个装配体。

根据功能我们将基础模块命名为：DoubleConv, Down, Up, OutConv。下面给出U-Net中模型块的PyTorch 实现：

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class DoubleConv(nn.Module):
    """(convolution => [BN] => ReLU) * 2"""

    def __init__(self, in_channels, out_channels, mid_channels=None):
        super().__init__()
        if not mid_channels:
            mid_channels = out_channels
        self.double_conv = nn.Sequential(
            nn.Conv2d(in_channels, mid_channels, kernel_size=3, padding=1, bias=False),
            nn.BatchNorm2d(mid_channels),
            nn.ReLU(inplace=True),
            nn.Conv2d(mid_channels, out_channels, kernel_size=3, padding=1, bias=False),
            nn.BatchNorm2d(out_channels),
            nn.ReLU(inplace=True)
        )

    def forward(self, x):
        return self.double_conv(x)

class Down(nn.Module):
    """Downscaling with maxpool then double conv"""

    def __init__(self, in_channels, out_channels):
        super().__init__()
        self.maxpool_conv = nn.Sequential(
            nn.MaxPool2d(2),
            DoubleConv(in_channels, out_channels)
        )

    def forward(self, x):
        return self.maxpool_conv(x)

class Up(nn.Module):
    """Upscaling then double conv"""

    def __init__(self, in_channels, out_channels, bilinear=True):
        super().__init__()

        # if bilinear, use the normal convolutions to reduce the number of channels
        if bilinear:
            self.up = nn.Upsample(scale_factor=2, mode='bilinear', align_corners=True)
            self.conv = DoubleConv(in_channels, out_channels, in_channels // 2)
        else:
            self.up = nn.ConvTranspose2d(in_channels, in_channels // 2, kernel_size=2, stride=2)
            self.conv = DoubleConv(in_channels, out_channels)

    def forward(self, x1, x2):
        x1 = self.up(x1)
        # input is CHW
        diffY = x2.size()[2] - x1.size()[2]
        diffX = x2.size()[3] - x1.size()[3]

        x1 = F.pad(x1, [diffX // 2, diffX - diffX // 2,
                        diffY // 2, diffY - diffY // 2])
        # if you have padding issues, see
        # https://github.com/HaiyongJiang/U-Net-Pytorch-Unstructured-Buggy/commit/0e854509c2cea854e247a9c615f175f76fbb2e3a
        # https://github.com/xiaopeng-liao/Pytorch-UNet/commit/8ebac70e633bac59fc22bb5195e513d5832fb3bd
        x = torch.cat([x2, x1], dim=1)
        return self.conv(x)

class OutConv(nn.Module):
    def __init__(self, in_channels, out_channels):
        super(OutConv, self).__init__()
        self.conv = nn.Conv2d(in_channels, out_channels, kernel_size=1)

    def forward(self, x):
        return self.conv(x)
```

#### (3) 利用模型块组装U-Net

使用写好的模型块，可以非常方便地组装U-Net模型。可以看到，通过模型块的方式实现了代码复用，整个模型结构定义所需的代码总行数明显减少，代码可读性也得到了提升。

```python
class UNet(nn.Module):
    def __init__(self, n_channels, n_classes, bilinear=True):
        super(UNet, self).__init__()
        self.n_channels = n_channels
        self.n_classes = n_classes
        self.bilinear = bilinear

        self.inc = DoubleConv(n_channels, 64)
        self.down1 = Down(64, 128)
        self.down2 = Down(128, 256)
        self.down3 = Down(256, 512)
        factor = 2 if bilinear else 1
        self.down4 = Down(512, 1024 // factor)
        self.up1 = Up(1024, 512 // factor, bilinear)
        self.up2 = Up(512, 256 // factor, bilinear)
        self.up3 = Up(256, 128 // factor, bilinear)
        self.up4 = Up(128, 64, bilinear)
        self.outc = OutConv(64, n_classes)

    def forward(self, x):
        x1 = self.inc(x)
        x2 = self.down1(x1)
        x3 = self.down2(x2)
        x4 = self.down3(x3)
        x5 = self.down4(x4)
        x = self.up1(x5, x4)
        x = self.up2(x, x3)
        x = self.up3(x, x2)
        x = self.up4(x, x1)
        logits = self.outc(x)
        return logits
```

### 7. 修改模型层

以pytorch官方视觉库torchvision预定义好的模型ResNet50为例，探索如何修改模型的某一层或者某几层。我们先看看模型的定义是怎样的：

```python
import torchvision.models as models
net = models.resnet50()
print(net)  # 太多
```

假设我们要用这个resnet模型去做一个10分类的问题，就应该修改模型的fc层，将其输出节点数替换为10。另外，我们觉得一层全连接层可能太少了，想再加一层。可以做如下修改：

```python
from collections import OrderedDict
classifier = nn.Sequential(OrderedDict([('fc1', nn.Linear(2048, 128)),
                          ('relu1', nn.ReLU()), 
                          ('dropout1',nn.Dropout(0.5)),
                          ('fc2', nn.Linear(128, 10)),
                          ('output', nn.Softmax(dim=1))
                          ]))
    
net.fc = classifier
```

这里的操作相当于将模型（net）最后名称为“fc”的层替换成了名称为“classifier”的结构，该结构是我们自己定义的。这里使用了Sequential+OrderedDict的模型定义方式。至此，我们就完成了模型的修改，现在的模型就可以去做10分类任务了。

### 8. 添加外部输入

有时候在模型训练中，除了已有模型的输入之外，还需要输入额外的信息。比如在CNN网络中，我们除了输入图像，还需要同时输入图像对应的其他信息，这时候就需要在已有的CNN网络中添加额外的输入变量。基本思路是：将原模型添加输入位置前的部分作为一个整体，同时在forward中定义好原模型不变的部分、添加的输入和后续层之间的连接关系，从而完成模型的修改。

我们以torchvision的resnet50模型为基础，任务还是10分类任务。不同点在于，我们希望利用已有的模型结构，在倒数第二层增加一个额外的输入变量add_variable来辅助预测。具体实现如下：

```python
class Model(nn.Module):
    def __init__(self, net):
        super(Model, self).__init__()
        self.net = net
        self.relu = nn.ReLU()
        self.dropout = nn.Dropout(0.5)
        self.fc_add = nn.Linear(1001, 10, bias=True)
        self.output = nn.Softmax(dim=1)
        
    def forward(self, x, add_variable):
        x = self.net(x)
        x = torch.cat((self.dropout(self.relu(x)), add_variable.unsqueeze(1)),1)
        x = self.fc_add(x)
        x = self.output(x)
        return x
```

这里的实现要点是通过torch.cat实现了tensor的拼接。torchvision中的resnet50输出是一个1000维的tensor，我们通过修改forward函数（配套定义一些层），先将1000维的tensor通过激活函数层和dropout层，再和外部输入变量"add_variable"拼接，最后通过全连接层映射到指定的输出维度10。

另外这里对外部输入变量"add_variable"进行unsqueeze操作是为了和net输出的tensor保持维度一致，常用于add_variable是单一数值 (scalar) 的情况，此时add_variable的维度是 (batch_size, )，需要在第二维补充维数1，从而可以和tensor进行torch.cat操作。

之后对我们修改好的模型结构进行实例化，就可以使用了：

```python
import torchvision.models as models
net = models.resnet50()
model = Model(net).cuda()
```

训练中在输入数据的时候要给两个inputs：

```python
outputs = model(inputs, add_var)
```

### 9. 添加额外输出

有时候在模型训练中，除了模型最后的输出外，我们需要输出模型某一中间层的结果，以施加额外的监督，获得更好的中间层结果。基本的思路是修改模型定义中forward函数的return变量。

我们依然以resnet50做10分类任务为例，在已经定义好的模型结构上，同时输出1000维的倒数第二层和10维的最后一层结果。具体实现如下：

```python
class Model(nn.Module):
    def __init__(self, net):
        super(Model, self).__init__()
        self.net = net
        self.relu = nn.ReLU()
        self.dropout = nn.Dropout(0.5)
        self.fc1 = nn.Linear(1000, 10, bias=True)
        self.output = nn.Softmax(dim=1)
        
    def forward(self, x):
        x1000 = self.net(x)
        x10 = self.dropout(self.relu(x1000))
        x10 = self.fc1(x10)
        x10 = self.output(x10)
        return x10, x1000
```

之后对我们修改好的模型结构进行实例化，就可以使用了：

```python
import torchvision.models as models
net = models.resnet50()
model = Model(net).cuda()
```

训练中在输入数据后会有两个outputs：

```python
out10, out1000 = model(inputs)
```

### 10. Pytorch模型存储

一个PyTorch模型主要包含两个部分：模型结构和权重。其中模型是继承nn.Module的类，权重的数据结构是一个字典（key是层名，value是权重向量）。存储也由此分为两种形式：存储整个模型（包括结构和权重）和只存储模型权重。

```python
from torchvision import models
model = models.resnet152(pretrained=True)

# 保存整个模型
torch.save(model, save_dir)
# 保存模型权重
torch.save(model.state_dict, save_dir)
```

对于PyTorch而言，pt, pth和pkl三种数据格式均支持模型权重和整个模型的存储，因此使用上没有差别。

### 11. 单卡保存+单卡加载

在使用os.envision命令指定使用的GPU后，即可进行模型保存和读取操作。注意这里即便保存和读取时使用的GPU不同也无妨。

```python
import os
import torch
from torchvision import models

os.environ['CUDA_VISIBLE_DEVICES'] = '0'   #这里替换成希望使用的GPU编号
model = models.resnet152(pretrained=True)
model.cuda()

# 保存+读取整个模型
torch.save(model, save_dir)
loaded_model = torch.load(save_dir)
loaded_model.cuda()

# 保存+读取模型权重
torch.save(model.state_dict(), save_dir)
loaded_dict = torch.load(save_dir)
loaded_model = models.resnet152()   #注意这里需要对模型结构有定义
loaded_model.state_dict = loaded_dict
loaded_model.cuda()
```

### 参考

组队学习之深入浅出Pytorch
https://github.com/datawhalechina/thorough-pytorch
https://zhuanlan.zhihu.com/p/64990232
https://github.com/milesial/Pytorch-UNet