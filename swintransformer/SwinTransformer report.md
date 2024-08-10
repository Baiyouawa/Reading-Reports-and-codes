## swinTransformer report

### 论文网址：https://arxiv.org/pdf/2103.14030

### 视频网址：【Swin Transformer论文精读【论文精读】】https://www.bilibili.com/video/BV13L4y1475U?vd_source=88664659bdda4409e78f614f5f213ce8

### （一）：论文精读：

#### 1.标题：使用移动窗口层级式的VIT----Swintransformer

**解读：Swintransformer就是让VIT也能像CNN一样可以分成几个block，能形成层级式的特征提取，具有多尺度【1】的概念。**

【1】：**多尺度特征：**多尺寸特征（Multi-Scale Features）是计算机视觉和深度学习中常用的概念，尤其在处理图像和视频时非常重要。它指的是在不同尺度（分辨率）上提取的特征，以捕捉图像中的不同层次的细节和信息。以下是对多尺寸特征的详细讲解：

1. **概念理解**：
   - **多尺度**：指的是在不同的分辨率或尺度上观察图像。例如，原始图像、下采样后的图像以及进一步下采样后的图像等。
   - **特征**：指的是从图像中提取出的能够代表图像内容的信息，例如边缘、纹理、颜色等。
2. **为什么需要多尺寸特征**：
   - **捕捉细节**：在高分辨率下，可以捕捉到图像的细节信息，比如边缘和纹理。
   - **捕捉全局信息**：在低分辨率下，可以捕捉到图像的整体结构和全局信息。
   - **提高鲁棒性**：结合多尺度的特征，可以提高模型对各种尺度变化（如物体的大小、位置）的鲁棒性。
3. **实现方法**：
   - **金字塔结构**：将图像逐层下采样，形成一系列不同分辨率的图像金字塔。每一层对应一个尺度。
   - **卷积神经网络（CNN）**：在深度学习中，通过多层卷积和池化操作自然形成了多尺度特征。例如，早期层提取的是低级特征（如边缘），而后期层提取的是高级特征（如对象部件）。
   - **特征融合**：将不同尺度下提取的特征进行融合，形成多尺度特征表示。这可以通过跳跃连接（skip connections）或特征金字塔网络（Feature Pyramid Network, FPN）来实现。
4. **应用**：
   - **目标检测**：多尺度特征可以帮助检测不同大小的目标。例如，FPN就是一种典型的使用多尺度特征来进行目标检测的方法。
   - **语义分割**：多尺度特征可以帮助精确地分割图像中的不同区域，尤其是边界部分。
   - **图像分类**：多尺度特征可以提供更多的信息，有助于提高分类准确率。

------

#### 2.摘要：

本文提出了一种新的视觉Transformer，称为Swin Transformer，它能够作为计算机视觉的通用主干网络。将Transformer从语言适配到视觉领域面临的挑战，源自于这两个领域之间的差异，例如**视觉实体的尺度变化大**以及**图像像素的高分辨率**相较于文本中的词汇。为了应对这些差异，我们提出了一种分层Transformer，**其表示通过Shifted Windows计算**。Shifted Windows机制通过**将自注意力计算限制在不重叠的局部窗口**中，**提高了效率，同时也允许跨窗口连接**。这种分层架构具有在不同尺度上建模的灵活性，并且相对于图像大小具有**线性计算复杂度**。这些Swin Transformer的特点使其能够兼容广泛的视觉任务，包括图像分类（在ImageNet-1K上达到了87.3的top-1准确率）和密集预测任务，如目标检测（在COCO test-dev上达到了58.7的box AP和51.1的mask AP）和语义分割（在ADE20K val上达到了53.5的mIoU）。它的性能大幅度超过了之前的最先进水平，在COCO上提升了+2.7的box AP和+2.6的mask AP，在ADE20K上提升了+3.2的mIoU，展示了基于Transformer的模型作为视觉主干网络的潜力。分层设计和Shifted Windows方法对所有**MLP架构也同样有益**。代码和模型已在[此处](https://github.com/microsoft/Swin-Transformer)公开提供。https://github.com/microsoft/Swin-Transformer

------

#### 3.引言

计算机视觉中的建模长期以来一直由**卷积神经网络**（CNN）主导。从AlexNet [39] 及其在ImageNet图像分类挑战中的革命性表现开始，CNN架构 通过更大规模[30, 76]、更多的连接[34]和更复杂的卷积形式[70, 18, 84]，不断进化，变得越来越强大。随着CNN作为各种视觉任务的主干网络，这些架构进步带来了性能的提升，大大推动了整个领域的发展。

另一方面，自然语言处理（NLP）领域的网络架构演变走了一条不同的道路，目前主流的架构是Transformer [64]。设计用于序列建模和转换任务，Transformer因其使用注意力机制来建模数据中的长距离依赖而闻名。其在语言领域的巨大成功引导研究人员探讨其在计算机视觉中的适应性，最近在某些任务上展现了有希望的结果，特别是图像分类[20]和联合视觉-语言建模[47]。

![](F:\科研文档\Transformer\Bert\swin1.png)

在本文中，我们**旨在扩展Transformer的适用性**，使其能够像在NLP中那样，作为计算机视觉的通用主干网络。我们观察到，将其在语言领域的高性能转移到视觉领域面临着显著挑战，这可以通过两种模式的差异来解释。其中一个差异涉及尺度。与语言Transformer中作为基本处理单元的词元不同，视觉元素在尺度上可能有很大差异，这在目标检测[42, 53, 54]等任务中尤为重要。在现有的基于Transformer的模型中[64, 20]，词元都是固定尺度的，这种属性不适用于这些视觉应用。另一个差异是图像像素的分辨率远高于文本段落中的词汇。有许多视觉任务需要像素级别的密集预测，例如语义分割，对于高分辨率图像，**Transformer的自注意力计算复杂度为图像大小的平方**，这将是不可行的。为了解决这些问题，我们提出了一种通用的Transformer主干网络，称为**Swin Transformer**，它构建了分层特征图，并且具有**相对于图像大小的线性计算复杂度**。正如图1(a)所示，Swin Transformer通过**从小尺寸的图像块（灰色框出）开始，逐渐在更深的Transformer层中合并相邻块，构建了分层表示**。通过这些分层特征图，Swin Transformer模型可以方便地利用先进的密集预测技术，如特征金字塔网络（FPN）[42]或U-Net[51]。线性计算复杂度是通过在划分图像的**非重叠窗口内局部计算自注意力（红色框出）实现的**。每个窗口中的块数是固定的，因此复杂度变为图像大小的线性。与之前的基于Transformer的架构[20]相比，这些优点使Swin Transformer适合作为各种视觉任务的通用主干网络。

![](F:\科研文档\Transformer\Bert\屏幕截图2024.08.071.png)

Swin Transformer的一个关键设计元素是其在**连续自注意力层之间的窗口分区的移位**，如图2所示。移位的窗口将前一层的窗口连接起来，提供了显著增强建模能力的连接（见表4）。这种策略在实际延迟方面也很有效：一个窗口内的所有查询块共享相同的关键集，这有助于硬件中的内存访问。相比之下，早期的基于滑动窗口的自注意力方法在通用硬件上由于不同查询像素的不同关键集而表现出较低的延迟。我们的实验表明，提出的移位窗口方法在建模能力上与滑动窗口方法相似（见表5和表6），但延迟却低得多。移位窗口方法对所有MLP架构也有益。

提出的Swin Transformer在图像分类、目标检测和语义分割的识别任务上表现出色。它在这三项任务上显著优于ViT / DeiT [20, 63]和ResNe(X)t模型[30, 70]，且具有类似的延迟。在COCO test-dev集上，其58.7的box AP和51.1的mask AP超过了之前的最先进结果（Copy-paste [26]不含外部数据）2.7的box AP和DetectoRS [46] 2.6的mask AP。在ADE20K语义分割任务上，它在验证集上获得了53.5的mIoU，比之前的最先进结果（SETR [81]）提高了3.2的mIoU。在ImageNet-1K图像分类上，它还达到了87.3%的top-1准确率。

我们相信，跨计算机视觉和自然语言处理的统一架构可以为两个领域带来好处，因为它将**促进视觉和文本信号的联合建模**，并且这两个领域的建模知识可以更深层次地共享。我们希望Swin Transformer在各种视觉问题上的强大表现可以在社区中进一步推动这一信念，并鼓励视觉和语言信号的统一建模。

**解读：其实这里的swin就是规避了全局注意力计算，改为对小窗口进行注意力计算。局部计算注意力：利用了CNN的局部性先验知识，就是说同一个物体的不同部位，或者说语义相近的不同物体，还是大概率会出现在相邻的地方，所以即使在小窗口的范围内计算自注意力也是够用的。另外一个挑战是如何生成多尺寸特征：回想卷积神经网络，就是具有池化操作，池化操作能够增大每一个卷积核能看到的感受野，从而使得每一次池化过程中能够抓住物体的不同尺寸。类似的，这里提出了Patch merging，把相邻的小patch合并成一个大patch**

#### 4.结论：

本文介绍了Swin Transformer，这是一种新的视觉Transformer，能够生成分层特征表示，并且相对于输入图像大小具有线性计算复杂度。Swin Transformer在COCO目标检测和ADE20K语义分割任务中达到了最先进的性能，显著超越了之前的最佳方法。我们希望Swin Transformer在各种视觉问题上的强大表现能够推动视觉和语言信号的统一建模。作为Swin Transformer的关键元素，基于移位窗口的自注意力在视觉问题上被证明是有效且高效的，我们期待进一步研究其在自然语言处理中的应用。

#### 5.相关工作：

CNN及其变体在计算机视觉中一直作为标准的网络模型。虽然CNN已经存在了几十年，但直到AlexNet的引入，CNN才开始迅速发展并成为主流。自那以后，提出了更深、更有效的卷积神经网络架构，进一步推动了计算机视觉领域的深度学习浪潮，例如VGG、GoogleNet、ResNet、DenseNet、HRNet和EfficientNet。除了这些架构进展外，还在改进单个卷积层方面进行了大量工作，例如深度卷积和可变形卷积。尽管CNN及其变体仍然是计算机视觉应用的主要骨干架构，我们强调Transformer类架构在视觉和语言统一建模中的强大潜力。我们的工作在多个基本视觉识别任务上表现出色，我们希望这将有助于模型的转型。

基于自注意力的骨干架构 受NLP领域中自注意力层和Transformer架构成功的启发，一些工作采用自注意力层来替换流行的ResNet中的部分或全部空间卷积层。在这些工作中，自注意力在每个像素的局部窗口内计算，以加速优化，并且在精度/FLOPs权衡上比对等的ResNet架构稍有改善。然而，它们昂贵的内存访问导致实际延迟显著大于卷积网络。我们提出在连续层之间移动窗口，而不是使用滑动窗口，这允许在通用硬件中更高效的实现。

自注意力/Transformers补充CNN 另一类工作是用自注意力层或Transformers增强标准的CNN架构。自注意力层可以通过提供编码远距离依赖或异质交互的能力来补充骨干网络或头网络。最近，Transformer中的编码器-解码器设计已应用于目标检测和实例分割任务。我们的工作探索了Transformer在基本视觉特征提取中的适应性，与这些工作互补。

基于Transformer的视觉骨干 与我们的工作最相关的是Vision Transformer（ViT）及其后续工作。ViT的开创性工作直接在中等大小的不重叠图像块上应用Transformer架构进行图像分类。相比卷积网络，它在图像分类上达到了令人印象深刻的速度-精度权衡。虽然ViT需要大规模训练数据集（如JFT-300M）才能表现良好，但DeiT引入了几种训练策略，使得ViT在较小的数据集ImageNet-1K上也能有效。ViT在图像分类上的结果令人鼓舞，但其架构不适合作为高分辨率图像或密集视觉任务的通用骨干网络，因为其特征图分辨率低，并且复杂度随图像大小呈二次方增加。有一些工作通过直接上采样或反卷积将ViT模型应用于目标检测和语义分割等密集视觉任务，但性能相对较低。与我们的工作同时进行的是一些修改ViT架构以更好地进行图像分类的工作。我们发现我们的Swin Transformer架构在图像分类上实现了最好的速度-精度权衡，尽管我们的工作重点是通用性能，而不仅仅是分类。另一个同时进行的工作探索了在Transformers上构建多分辨率特征图的类似思路。其复杂度仍然是图像大小的二次方，而我们的复杂度是线性的，并且在局部操作，这在建模视觉信号的高相关性方面已被证明是有益的。我们的方法既高效又有效，在COCO目标检测和ADE20K语义分割上实现了最先进的准确性。

#### 6.方法：

<img src="F:\科研文档\Transformer\swin\swin.png"  />

**解读：图片首先打成Patch，224×224×3的维度，经过4×4的patch，图片尺寸变为56×56×48（4×4×3RGB通道），接下来进行Embedding，设超参数为C（96）（经过全连接层将48扩大到96），变为56×56×96；前面的拉直变为3136序列长度，96是token维度；3136序列太长了，所以引入了窗口计算。每个窗口有7×7=49个patch（默认参数）； 经过第一个部分的Block后，得到56×56×96，随后进行（Patch Merging）过程以便获取多尺寸的信息，类似于卷积神经网络中池化的操作。将邻近的小patch合成一个大Patch，下采样两倍，所以要隔一个点选一个，形成后为了将通道数仅仅扩大一倍（2），所以做一个1×1C方向上卷积，将通道数从4变为2，经过多层之后，进行一次全局池化的过程（分类的话） **

##### 3.1. 总体架构

图3展示了Swin Transformer架构的概述，具体以其小型版本（Swin-T）为例。它首先通过一个补丁分割模块将输入的RGB图像分割为不重叠的补丁块，类似于ViT。每个补丁块被视为一个“标记”，其特征设置为原始像素RGB值的串联。在我们的实现中，使用的补丁大小为4×4，因此每个补丁的特征维度为4×4×3=48。一个线性嵌入层应用于这些原始值特征，将其**投影到任意维度（记作C**）。

几个具有修改后的自注意力计算的Transformer块（Swin Transformer块）应用于这些补丁标记。这些Transformer块保持标记的数量（H/4×W/4），并与线性嵌入层一起被称为“阶段1”。

为了生成分层表示，随着网络的加深，标记的数量通过补丁合并层减少。第一个补丁合并层将每组2×2相邻补丁的特征串联起来，并对4C维的串联特征应用线性层。这将标记的数量减少了一个2×2=4的倍数（分辨率下降2倍），输出维度设置为2C。之后应用Swin Transformer块进行特征转换，分辨率保持在H/8×W/8。这第一个补丁合并和特征转换块被称为“阶段2”。该过程重复两次，分别为“阶段3”和“阶段4”，输出分辨率分别为H/16×W/16和H/32×W/32。

这些阶段共同生成了一个分层表示，具有与**典型卷积网络（例如VGG和ResNet）相同的特征图分辨率**。因此，所提出的架构可以方便地替代现有方法中的骨干网络，用于各种视觉任务。

##### 3.2. 基于移位窗口的自注意力

标准Transformer架构及其图像分类的改编版本都进行全局自注意力计算，其中计算了一个标记与所有其他标记之间的关系。这种**全局计算导致了与标记数量相关的二次复杂度**，使其**不适合需要大量标记进行密集预测**或**高分辨率图像**表示的许多视觉问题。

##### 在非重叠窗口中的自注意力
为了高效建模，我们提出在局部窗口内计算自注意力。窗口被安排为均匀地划分图像，不重叠。假设每个窗口包含M×M个补丁，则在h×w个补丁的图像上，全球MSA模块和基于窗口的模块的计算复杂度分别为：

$$
\[ \Omega(MSA) = 4hwC^2 + 2(hw)^2C \]\\
\[ \Omega(W-MSA) = 4hwC^2 + 2M^2hwC \]
$$
其中前者与补丁数量hw呈二次关系，而后者在M固定时（默认设为7）是线性的。对于大的hw，全局自注意力计算通常是不可负担的，而基于窗口的自注意力是可扩展的。

##### 在连续块中移位窗口分区
基于窗口的自注意力模块**缺乏跨窗口的连接**，这限制了其建模能力。为了在保持非重叠窗口高效计算的同时引入跨窗口连接，我们提出了一种移位窗口分区方法，在连续的Swin Transformer块中交替使用两种分区配置。

如图2所示，第一个模块使用从左上角像素开始的常规窗口分区策略，并将8×8特征图均匀分为2×2的4×4（M=4）窗口。然后，下一个模块采用从前一层分区移位的窗口配置，通过将窗口从常规分区窗口移位(bM/2c, bM/2c)像素。

通过移位窗口分区方法，连续的Swin Transformer块计算如下：

$$
\[ \hat{z}_l = W-MSA(LN(z_{l-1})) + z_{l-1} \]\\
\[ z_l = MLP(LN(\hat{z}_l)) + \hat{z}_l \]\\
\[ \hat{z}_{l+1} = SW-MSA(LN(z_l)) + z_l \]\\
\[ z_{l+1} = MLP(LN(\hat{z}_{l+1})) + \hat{z}_{l+1} \]
$$
其中 \(\hat{z}_l\) 和 \(z_l\) 分别表示块l的(S)W-MSA模块和MLP模块的输出特征。W-MSA和SW-MSA分别表示使用常规和移位窗口分区配置的基于窗口的多头自注意力。

移位窗口分区方法在前一层的相邻非重叠窗口之间引入了连接，事实证明在图像分类、目标检测和语义分割中是有效的，如表4所示。

##### 移位配置的高效批处理计算
移位窗口分区的一个问题是它会导致更多的窗口，从 \( \lceil h/M \rceil \times \lceil w/M \rceil \) 到 \( (\lceil h/M \rceil + 1) \times (\lceil w/M \rceil + 1) \) 的移位配置，有些窗口将小于M×M。一个简单的解决方案是将较小的窗口填充到M×M大小，并在计算注意力时屏蔽填充值。当常规分区中的窗口数量较少时，例如2×2，这种简单解决方案的计算量增加是相当大的（2×2 → 3×3，增加了2.25倍）。

在此，我们提出了一种更高效的批处理计算方法，通过向左上方循环移位，如图4所示。此移位后，批处理窗口可能由特征图中不相邻的几个子窗口组成，因此采用屏蔽机制限制自注意力计算在每个子窗口内。

通过循环移位，批处理窗口的数量保持与常规窗口分区相同，因此也很高效。此方法的低延迟在表5中得以显示。

##### 移动窗口计算问题：

**在进行移动窗口过后，会出现每个窗口的Patch个数不同的情况，这也就导致我们不能转化为一个batch进行计算，如果是补充0（在外围）又会增加计算的复杂度，所以我们采取掩码的方式（拼图）**

#### ![image-20240809133040351](C:\Users\10946\AppData\Roaming\Typora\typora-user-images\image-20240809133040351.png)

**我们采用一种循环移位，分割填补。有的窗口应该进行注意力计算，但同样的，对于有的窗口它是填充了带有其它部分的。不应该进行自注意力计算。所以采取掩码方式。 **