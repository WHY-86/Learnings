# AV2T-Gen 学习路线

本指南围绕航拍可见光到热红外生成，按「直觉 → 代码 → 论文 → 对照阅读」学习扩散模型、文本条件和对象掩码。

## 建议路线

1. **先看整体图解**：[The Illustrated Stable Diffusion](https://jalammar.github.io/illustrated-stable-diffusion/)。重点看文本编码、U-Net 多次去噪和 VAE 解码，画出预测数据流。
2. **理解扩散代码**：[The Annotated Diffusion Model](https://huggingface.co/blog/annotated-diffusion)。先看 What is a diffusion model 与 forward diffusion；再搜索 `q_sample`、`p_losses`、`p_sample`，对照训练和采样。此教程是通用 DDPM，不等同于 AV2T-Gen 的损失实现。
3. **读 DDPM 原文**：[Denoising Diffusion Probabilistic Models](https://arxiv.org/abs/2006.11239)。看第 2、3 节和 Algorithm 1、2；识别干净数据、噪声、时间步和网络输出。
4. **补 VAE**：[What is a variational autoencoder?](https://jaan.io/what-is-variational-autoencoder-vae-tutorial/) 和 [An Introduction to VAEs](https://arxiv.org/abs/1906.02691)。重点是编码器、均值与方差、重参数化、解码器。
5. **补 U-Net**：[U-Net 原论文](https://arxiv.org/abs/1505.04597)。先看 Figure 1 和第 2 节，理解下采样、上采样、跳跃连接；扩散 U-Net 还可能包含时间嵌入与注意力。
6. **理解 CLIP**：[Learning Transferable Visual Models From Natural Language Supervision](https://arxiv.org/abs/2103.00020)。看 Figure 1 和第 2.3 节，关注图文对比学习与文本编码。
7. **连接潜空间与条件**：[Latent Diffusion Models](https://arxiv.org/abs/2112.10752)。重点看第 3.1–3.3 节：图像压缩、潜空间扩散、交叉注意力条件。
8. **看图像加文本的训练**：[InstructPix2Pix 论文](https://arxiv.org/abs/2211.09800) 和 [Diffusers 训练说明及脚本](https://github.com/huggingface/diffusers/blob/main/docs/source/en/training/instructpix2pix.md)。沿 `tokenizer`、`text_encoder`、`vae.encode`、`add_noise`、`timesteps`、`unet`、`loss` 跟踪数据。它是参考实现，不是 AV2T-Gen 源码。

## 对象条件支线

| 问题 | 材料 | 阅读重点 |
|---|---|---|
| 图中有什么？ | [RAM](https://arxiv.org/abs/2306.03514) | 模型架构图和图像标签识别 |
| 文本对应哪里？ | [Grounding DINO](https://arxiv.org/abs/2303.05499) | 整体结构和文本引导定位 |
| 对象覆盖哪些像素？ | [SAM](https://arxiv.org/abs/2304.02643) | 图像编码器、提示编码器、掩码解码器 |

用「标签是什么 → 框在哪里 → 掩码覆盖哪些像素」串起来。车辆运动状态需要序列信息；它是热外观线索，不能当作直接测量的温度。

## 回到热红外生成

- [F-ViTA 论文](https://openaccess.thecvf.com/content/WACV2026/papers/Paranjape_F-ViTA_Foundation_Model_Guided_Visible_to_Infrared_Translation_WACV_2026_paper.pdf)：看方法图和对象标签、掩码对生成的引导。
- [AV2T-Gen 论文](https://media.eventhosts.cc/Conferences/ECCV2026/pdfs/3703.pdf)：沿训练及预测两条路径标注 RGB、热图、环境、车辆状态、标签和掩码。
- [作者仓库](https://github.com/NPU-CVPG/AV2T)：查看作者当前发布的代码与资料。

## 阅读时记录的问题

- [ ] VAE 为什么要输出分布参数？冻结解码器后，图像损失的梯度如何传回生成网络？
- [ ] 前向加噪加入什么噪声？训练与预测时的时间步分别如何使用？
- [ ] CLIP 文本向量和图像特征如何经交叉注意力交互？
- [ ] 如何将同一辆车的运动标签与掩码对应？车辆状态为什么不能等同于真实温度？
- [ ] RGB 和热图各自在哪个阶段作为输入？预测时是否需要真实热图？
- [ ] 潜空间、边缘、感知损失分别比较什么？具体计算以原论文和代码为准。

## 提示与证据界限

环境 prompt 可由拍摄时间、气温、湿度等字段按模板生成；经 tokenizer 和文本编码器后成为模型条件。对象语义与掩码可共同形成局部条件。具体张量尺寸、融合方式、采样器和损失实现，应以论文及作者发布的代码为准；公式中的括号本身不足以确定融合操作。生成图像的视觉质量也不能直接证明绝对温度测量准确。

本仓库整理的是学习链接与阅读指引，论文版权归原作者所有。

