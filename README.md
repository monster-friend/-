# [AI训练营]PaddleSeg实现语义分割Baseline

手把手教你基于PaddleSeg实现语义分割任务

------


# 一、作业任务

> 本次任务将基于PaddleSeg展开语义分割任务的学习与实践，baseline会提供PaddleSeg套件的基本使用，相关细节如有遗漏，可参考[10分钟上手PaddleSeg
](https://aistudio.baidu.com/aistudio/projectdetail/1672610?channelType=0&channel=0)

1. 选择提供的**五个数据集**中的一个数据集作为本次作业的训练数据，并**根据baseline跑通项目**

2. **可视化1-3张**预测图片与预测结果（本次提供的数据集没有验证与测试集，可将训练数据直接进行预测，展示训练数据的预测结果即可）


**加分项:**

3. 尝试**划分验证集**，再进行训练

4. 选择**其他网络**，调整训练参数进行更好的尝试


**PS**:PaddleSeg相关参考项目:

- [常规赛：PALM病理性近视病灶检测与分割基线方案](https://aistudio.baidu.com/aistudio/projectdetail/1941312)

- [PaddleSeg 2.0动态图：车道线图像分割任务简介](https://aistudio.baidu.com/aistudio/projectdetail/1752986?channelType=0&channel=0)

------

------

# 二、数据集说明

------

本项目使用的数据集是:[AI训练营]语义分割数据集合集，包含马分割，眼底血管分割，车道线分割，场景分割以及人像分割。

该数据集已加载到本环境中，位于:

**data/data103787/segDataset.zip**



```python
# unzip: 解压指令
# -o: 表示解压后进行输出
# -q: 表示静默模式，即不输出解压过程的日志
# -d: 解压到指定目录下，如果该文件夹不存在，会自动创建
!unzip -oq data/data103787/segDataset.zip -d segDataset
```

解压完成后，会在左侧文件目录多出一个**segDataset**的文件夹，该文件夹下有**5**个子文件夹：

- **horse -- 马分割数据**<二分类任务>

![](https://ai-studio-static-online.cdn.bcebos.com/2b12a7fab9ee409587a2aec332a70ba2bce0fcc4a10345a4aa38941db65e8d02)

- **fundusVessels -- 眼底血管分割数据**

> 灰度图，每个像素值对应相应的类别 -- 因此label不宜观察，但符合套件训练需要

![](https://ai-studio-static-online.cdn.bcebos.com/b515662defe548bdaa517b879722059ad53b5d87dd82441c8c4611124f6fdad0)

- **laneline -- 车道线分割数据**

![](https://ai-studio-static-online.cdn.bcebos.com/2aeccfe514e24cf98459df7c36421cddf78d9ddfc2cf41ffa4aafc10b13c8802)

- **facade -- 场景分割数据**

![](https://ai-studio-static-online.cdn.bcebos.com/57752d86fc5c4a10a3e4b91ae05a3e38d57d174419be4afeba22eb75b699112c)

- **cocome -- 人像分割数据**

> label非直接的图片，为json格式的标注文件，有需要的小伙伴可以看一看PaddleSeg的[PaddleSeg实战——人像分割](https://aistudio.baidu.com/aistudio/projectdetail/2177440?channelType=0&channel=0)


```python
# tree: 查看文件夹树状结构
# -L: 表示遍历深度
!tree segDataset -L 2
```

    segDataset
    ├── cocome
    │   ├── Annotations
    │   └── Images
    ├── facade
    │   ├── Annotations
    │   └── Images
    ├── FundusVessels
    │   ├── Annotations
    │   └── Images
    ├── horse
    │   ├── Annotations
    │   └── Images
    └── laneline
        ├── Annotations
        └── Images
    
    15 directories, 0 files


> 查看数据label的像素分布，可从中得知分割任务的类别数： 脚本位于: **show_segDataset_label_cls_id.py**

> 关于人像分割数据分析，这里不做提示，可参考[PaddleSeg实战——人像分割](https://aistudio.baidu.com/aistudio/projectdetail/2177440?channelType=0&channel=0)


```python
# 查看label中像素分类情况
!python show_segDataset_label_cls_id.py
```

    100%|████████████████████████████████████████| 328/328 [00:00<00:00, 873.61it/s]
    horse-cls_id:  [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 211, 212, 213, 214, 216, 217, 218, 219, 220, 221, 222, 223, 224, 225, 226, 227, 228, 229, 230, 231, 232, 233, 234, 235, 236, 237, 238, 239, 240, 241, 242, 243, 244, 245, 246, 247, 248, 249, 250, 251, 252, 253, 254, 255]
    horse为90分类
    horse实际应转换为2分类(将非0像素转换为像素值为1)
    
    
    100%|████████████████████████████████████████| 845/845 [00:04<00:00, 196.27it/s]
    facade-cls_id:  [0, 1, 2, 3, 4, 5, 6, 7, 8]
    facade为9分类
    
    
    100%|████████████████████████████████████████| 200/200 [00:01<00:00, 180.58it/s]
    fundusvessels-cls_id:  [0, 1]
    fundusvessels为2分类
    
    
    100%|███████████████████████████████████████| 4878/4878 [01:29<00:00, 54.48it/s]
    laneline-cls_id:  [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19]
    laneline为20分类


# 三、数据预处理

> 这里就以horse数据作为示例

-----

- 首先，通过上边的像素值分析以及horse本身的标签表现，我们确定horse数据集为二分类任务

- 然而，实际label中，却包含多个像素值，因此需要将horse中的所有label进行一个预处理

- 预处理内容为: 0值不变，非0值变为1，然后再保存label

- **并且保存文件格式为png，单通道图片为Label图片，最好保存为png——否则可能出现异常像素**

**对应horse的预处理脚本，位于:**

parse_horse_label.py


```python
!python parse_horse_label.py
```

    100%|████████████████████████████████████████| 328/328 [00:00<00:00, 394.77it/s]
    [0, 1]
    100%|███████████████████████████████████████| 328/328 [00:00<00:00, 1127.10it/s]
    horse-cls_id:  [0, 1]
    horse为2分类


- 预处理完成后，配置训练的索引文件txt，方便后边套件读取数据

> txt创建脚本位于: **horse_create_train_list.py**

> 同时，生成的txt位于: **segDataset/horse/train_list.txt**


```python
# 创建训练的数据索引txt
# 格式如下
# line1: train_img1.jpg train_label1.png
# line2: train_img2.jpg train_label2.png
!python horse_create_train_list.py
```

    100%|██████████████████████████████████████| 328/328 [00:00<00:00, 16427.82it/s]


# 四、使用套件开始训练

- 1.解压套件: 已挂载到本项目, 位于:**data/data102250/PaddleSeg-release-2.1.zip**


```python
# 解压套件
!unzip -oq data/data102250/PaddleSeg-release-2.1.zip
# 通过mv指令实现文件夹重命名
!mv PaddleSeg-release-2.1 PaddleSeg
```

- 2.选择模型，baseline选择**bisenet**, 位于: **PaddleSeg/configs/quick_start/bisenet_optic_disc_512x512_1k.yml**

- 3.配置模型文件

> 首先修改训练数据集加载的dataset类型:

![](https://ai-studio-static-online.cdn.bcebos.com/2f5363d71034490290f720ea8bb0d6873d7df2712d4b4e84ae41b0378aed8b89)

> 然后配置训练数据集如下:

![](https://ai-studio-static-online.cdn.bcebos.com/29547856db4b4bfc80aa3732e143f2788589f9316c694f369c9bd1da44b815dc)

> 类似的，配置验证数据集: -- **注意修改train_path为val_path**

![](https://ai-studio-static-online.cdn.bcebos.com/09713aaaed6b4611a525d25aae67e4f0538224f7ac0241eb941d97892bf6c4c1)

<font color="red" size=4>其它模型可能需要到: PaddleSeg/configs/$_base_$  中的数据集文件进行配置，但修改参数与bisenet中的数据集修改参数相同 </font>

![](https://ai-studio-static-online.cdn.bcebos.com/b154dcbf15e14f43aa13455c0ceeaaebe0489c9a09dd439f9d32e8b0a31355ec)


- 4.开始训练

使用PaddleSeg的train.py，传入模型文件即可开启训练


```python
!python PaddleSeg/train.py\
--config PaddleSeg/configs/quick_start/bisenet_optic_disc_512x512_1k.yml\
--batch_size 4\
--iters 2000\
--learning_rate 0.01\
--save_interval 200\
--save_dir PaddleSeg/output\
--seed 2021\
--log_iters 20\
--do_eval\
--use_vdl

# --batch_size 4\  # 批大小
# --iters 2000\    # 迭代次数 -- 根据数据大小，批大小估计迭代次数
# --learning_rate 0.01\ # 学习率
# --save_interval 200\ # 保存周期 -- 迭代次数计算周期
# --save_dir PaddleSeg/output\ # 输出路径
# --seed 2021\ # 训练中使用到的随机数种子
# --log_iters 20\ # 日志频率 -- 迭代次数计算周期
# --do_eval\ # 边训练边验证
# --use_vdl # 使用vdl可视化记录
# 用于断训==即中断后继续上一次状态进行训练
# --resume_model model_dir
```

    /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages/paddle/fluid/layers/utils.py:26: DeprecationWarning: `np.int` is a deprecated alias for the builtin `int`. To silence this warning, use `int` by itself. Doing this will not modify any behavior and is safe. When replacing `np.int`, you may wish to use e.g. `np.int64` or `np.int32` to specify the precision. If you wish to review your current use, check the release note link for additional information.
    Deprecated in NumPy 1.20; for more details and guidance: https://numpy.org/devdocs/release/1.20.0-notes.html#deprecations
      def convert_to_list(value, n, name, dtype=np.int):
    2021-08-15 14:22:33 [INFO]	
    ------------Environment Information-------------
    platform: Linux-4.4.0-150-generic-x86_64-with-debian-stretch-sid
    Python: 3.7.4 (default, Aug 13 2019, 20:35:49) [GCC 7.3.0]
    Paddle compiled with cuda: True
    NVCC: Cuda compilation tools, release 10.1, V10.1.243
    cudnn: 7.6
    GPUs used: 1
    CUDA_VISIBLE_DEVICES: None
    GPU: ['GPU 0: Tesla V100-SXM2-32GB']
    GCC: gcc (Ubuntu 7.5.0-3ubuntu1~16.04) 7.5.0
    PaddlePaddle: 2.0.2
    OpenCV: 4.1.1
    ------------------------------------------------
    2021-08-15 14:22:33 [INFO]	
    ---------------Config Information---------------
    batch_size: 4
    iters: 2000
    loss:
      coef:
      - 1
      - 1
      - 1
      - 1
      - 1
      types:
      - ignore_index: 255
        type: CrossEntropyLoss
    lr_scheduler:
      end_lr: 0
      learning_rate: 0.01
      power: 0.9
      type: PolynomialDecay
    model:
      pretrained: null
      type: BiSeNetV2
    optimizer:
      momentum: 0.9
      type: sgd
      weight_decay: 4.0e-05
    train_dataset:
      dataset_root: segDataset/horse
      mode: train
      num_classes: 2
      train_path: segDataset/horse/train_list.txt
      transforms:
      - target_size:
        - 512
        - 512
        type: Resize
      - type: RandomHorizontalFlip
      - type: Normalize
      type: Dataset
    val_dataset:
      dataset_root: data/optic_disc_seg
      mode: val
      transforms:
      - target_size:
        - 512
        - 512
        type: Resize
      - type: Normalize
      type: OpticDiscSeg
    ------------------------------------------------
    W0815 14:22:33.183596  3065 device_context.cc:362] Please NOTE: device: 0, GPU Compute Capability: 7.0, Driver API Version: 10.1, Runtime API Version: 10.1
    W0815 14:22:33.183640  3065 device_context.cc:372] device: 0, cuDNN Version: 7.6.
    /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages/paddle/fluid/dataloader/dataloader_iter.py:89: DeprecationWarning: `np.bool` is a deprecated alias for the builtin `bool`. To silence this warning, use `bool` by itself. Doing this will not modify any behavior and is safe. If you specifically wanted the numpy scalar type, use `np.bool_` here.
    Deprecated in NumPy 1.20; for more details and guidance: https://numpy.org/devdocs/release/1.20.0-notes.html#deprecations
      if isinstance(slot[0], (np.ndarray, np.bool, numbers.Number)):
    2021-08-15 14:22:40 [INFO]	[TRAIN] epoch: 1, iter: 20/2000, loss: 3.0075, lr: 0.009914, batch_cost: 0.1111, reader_cost: 0.01205, ips: 36.0173 samples/sec | ETA 00:03:39
    2021-08-15 14:22:42 [INFO]	[TRAIN] epoch: 1, iter: 40/2000, loss: 2.3644, lr: 0.009824, batch_cost: 0.0934, reader_cost: 0.00007, ips: 42.8129 samples/sec | ETA 00:03:03
    2021-08-15 14:22:44 [INFO]	[TRAIN] epoch: 1, iter: 60/2000, loss: 2.2829, lr: 0.009734, batch_cost: 0.0922, reader_cost: 0.00007, ips: 43.4015 samples/sec | ETA 00:02:58
    2021-08-15 14:22:46 [INFO]	[TRAIN] epoch: 1, iter: 80/2000, loss: 2.1542, lr: 0.009644, batch_cost: 0.0918, reader_cost: 0.00007, ips: 43.5666 samples/sec | ETA 00:02:56
    2021-08-15 14:22:48 [INFO]	[TRAIN] epoch: 2, iter: 100/2000, loss: 1.9857, lr: 0.009553, batch_cost: 0.0980, reader_cost: 0.00431, ips: 40.8273 samples/sec | ETA 00:03:06
    2021-08-15 14:22:49 [INFO]	[TRAIN] epoch: 2, iter: 120/2000, loss: 1.9929, lr: 0.009463, batch_cost: 0.0932, reader_cost: 0.00007, ips: 42.9087 samples/sec | ETA 00:02:55
    2021-08-15 14:22:51 [INFO]	[TRAIN] epoch: 2, iter: 140/2000, loss: 1.9276, lr: 0.009372, batch_cost: 0.0918, reader_cost: 0.00007, ips: 43.5591 samples/sec | ETA 00:02:50
    2021-08-15 14:22:53 [INFO]	[TRAIN] epoch: 2, iter: 160/2000, loss: 1.8901, lr: 0.009282, batch_cost: 0.0921, reader_cost: 0.00007, ips: 43.4208 samples/sec | ETA 00:02:49
    2021-08-15 14:22:55 [INFO]	[TRAIN] epoch: 3, iter: 180/2000, loss: 1.7530, lr: 0.009191, batch_cost: 0.0964, reader_cost: 0.00347, ips: 41.4772 samples/sec | ETA 00:02:55
    2021-08-15 14:22:57 [INFO]	[TRAIN] epoch: 3, iter: 200/2000, loss: 1.7375, lr: 0.009100, batch_cost: 0.0942, reader_cost: 0.00007, ips: 42.4707 samples/sec | ETA 00:02:49
    2021-08-15 14:22:57 [INFO]	Start evaluating (total_samples: 76, total_iters: 76)...
    /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages/paddle/fluid/dygraph/math_op_patch.py:238: UserWarning: The dtype of left and right variables are not the same, left dtype is VarType.INT32, but right dtype is VarType.BOOL, the right dtype will convert to VarType.INT32
      format(lhs_dtype, rhs_dtype, lhs_dtype))
    /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages/paddle/fluid/dygraph/math_op_patch.py:238: UserWarning: The dtype of left and right variables are not the same, left dtype is VarType.INT64, but right dtype is VarType.BOOL, the right dtype will convert to VarType.INT64
      format(lhs_dtype, rhs_dtype, lhs_dtype))
    76/76 [==============================] - 2s 26ms/step - batch_cost: 0.0261 - reader cost: 3.4729e-
    2021-08-15 14:22:59 [INFO]	[EVAL] #Images: 76 mIoU: 0.3027 Acc: 0.5736 Kappa: 0.0421 
    2021-08-15 14:22:59 [INFO]	[EVAL] Class IoU: 
    [0.566  0.0394]
    2021-08-15 14:22:59 [INFO]	[EVAL] Class Acc: 
    [0.9985 0.0395]
    2021-08-15 14:22:59 [INFO]	[EVAL] The model with the best validation mIoU (0.3027) was saved at iter 200.
    2021-08-15 14:23:01 [INFO]	[TRAIN] epoch: 3, iter: 220/2000, loss: 1.8460, lr: 0.009009, batch_cost: 0.0934, reader_cost: 0.00008, ips: 42.8166 samples/sec | ETA 00:02:46
    2021-08-15 14:23:03 [INFO]	[TRAIN] epoch: 3, iter: 240/2000, loss: 1.8050, lr: 0.008918, batch_cost: 0.0921, reader_cost: 0.00007, ips: 43.4340 samples/sec | ETA 00:02:42
    2021-08-15 14:23:05 [INFO]	[TRAIN] epoch: 4, iter: 260/2000, loss: 1.7326, lr: 0.008827, batch_cost: 0.0965, reader_cost: 0.00382, ips: 41.4593 samples/sec | ETA 00:02:47
    2021-08-15 14:23:07 [INFO]	[TRAIN] epoch: 4, iter: 280/2000, loss: 1.7003, lr: 0.008735, batch_cost: 0.0939, reader_cost: 0.00007, ips: 42.5925 samples/sec | ETA 00:02:41
    2021-08-15 14:23:09 [INFO]	[TRAIN] epoch: 4, iter: 300/2000, loss: 1.7414, lr: 0.008644, batch_cost: 0.0932, reader_cost: 0.00007, ips: 42.9363 samples/sec | ETA 00:02:38
    2021-08-15 14:23:11 [INFO]	[TRAIN] epoch: 4, iter: 320/2000, loss: 1.6094, lr: 0.008552, batch_cost: 0.0933, reader_cost: 0.00007, ips: 42.8554 samples/sec | ETA 00:02:36
    2021-08-15 14:23:12 [INFO]	[TRAIN] epoch: 5, iter: 340/2000, loss: 1.5780, lr: 0.008461, batch_cost: 0.0964, reader_cost: 0.00345, ips: 41.4923 samples/sec | ETA 00:02:40
    2021-08-15 14:23:14 [INFO]	[TRAIN] epoch: 5, iter: 360/2000, loss: 1.5669, lr: 0.008369, batch_cost: 0.0938, reader_cost: 0.00007, ips: 42.6432 samples/sec | ETA 00:02:33
    2021-08-15 14:23:16 [INFO]	[TRAIN] epoch: 5, iter: 380/2000, loss: 1.6444, lr: 0.008277, batch_cost: 0.0928, reader_cost: 0.00007, ips: 43.0983 samples/sec | ETA 00:02:30
    2021-08-15 14:23:18 [INFO]	[TRAIN] epoch: 5, iter: 400/2000, loss: 1.6620, lr: 0.008185, batch_cost: 0.0926, reader_cost: 0.00007, ips: 43.1767 samples/sec | ETA 00:02:28
    2021-08-15 14:23:18 [INFO]	Start evaluating (total_samples: 76, total_iters: 76)...
    76/76 [==============================] - 2s 25ms/step - batch_cost: 0.0254 - reader cost: 2.9794e-0
    2021-08-15 14:23:20 [INFO]	[EVAL] #Images: 76 mIoU: 0.2642 Acc: 0.5073 Kappa: 0.0199 
    2021-08-15 14:23:20 [INFO]	[EVAL] Class IoU: 
    [0.5002 0.0281]
    2021-08-15 14:23:20 [INFO]	[EVAL] Class Acc: 
    [0.9917 0.0283]
    2021-08-15 14:23:20 [INFO]	[EVAL] The model with the best validation mIoU (0.3027) was saved at iter 200.
    2021-08-15 14:23:22 [INFO]	[TRAIN] epoch: 6, iter: 420/2000, loss: 1.6128, lr: 0.008093, batch_cost: 0.0978, reader_cost: 0.00405, ips: 40.8862 samples/sec | ETA 00:02:34
    2021-08-15 14:23:24 [INFO]	[TRAIN] epoch: 6, iter: 440/2000, loss: 1.6083, lr: 0.008001, batch_cost: 0.0945, reader_cost: 0.00011, ips: 42.3471 samples/sec | ETA 00:02:27
    2021-08-15 14:23:26 [INFO]	[TRAIN] epoch: 6, iter: 460/2000, loss: 1.6901, lr: 0.007909, batch_cost: 0.0929, reader_cost: 0.00008, ips: 43.0674 samples/sec | ETA 00:02:23
    2021-08-15 14:23:28 [INFO]	[TRAIN] epoch: 6, iter: 480/2000, loss: 1.4864, lr: 0.007816, batch_cost: 0.0928, reader_cost: 0.00008, ips: 43.1039 samples/sec | ETA 00:02:21
    2021-08-15 14:23:30 [INFO]	[TRAIN] epoch: 7, iter: 500/2000, loss: 1.5544, lr: 0.007724, batch_cost: 0.0967, reader_cost: 0.00395, ips: 41.3706 samples/sec | ETA 00:02:25
    2021-08-15 14:23:32 [INFO]	[TRAIN] epoch: 7, iter: 520/2000, loss: 1.5045, lr: 0.007631, batch_cost: 0.0946, reader_cost: 0.00007, ips: 42.3043 samples/sec | ETA 00:02:19
    2021-08-15 14:23:33 [INFO]	[TRAIN] epoch: 7, iter: 540/2000, loss: 1.3530, lr: 0.007538, batch_cost: 0.0926, reader_cost: 0.00007, ips: 43.2097 samples/sec | ETA 00:02:15
    2021-08-15 14:23:35 [INFO]	[TRAIN] epoch: 7, iter: 560/2000, loss: 1.5993, lr: 0.007445, batch_cost: 0.0919, reader_cost: 0.00007, ips: 43.5194 samples/sec | ETA 00:02:12
    2021-08-15 14:23:37 [INFO]	[TRAIN] epoch: 8, iter: 580/2000, loss: 1.4581, lr: 0.007352, batch_cost: 0.0968, reader_cost: 0.00411, ips: 41.3214 samples/sec | ETA 00:02:17
    2021-08-15 14:23:39 [INFO]	[TRAIN] epoch: 8, iter: 600/2000, loss: 1.5573, lr: 0.007259, batch_cost: 0.0935, reader_cost: 0.00007, ips: 42.7657 samples/sec | ETA 00:02:10
    2021-08-15 14:23:39 [INFO]	Start evaluating (total_samples: 76, total_iters: 76)...
    76/76 [==============================] - 2s 25ms/step - batch_cost: 0.0254 - reader cost: 3.0213e-
    2021-08-15 14:23:41 [INFO]	[EVAL] #Images: 76 mIoU: 0.5240 Acc: 0.9496 Kappa: 0.1579 
    2021-08-15 14:23:41 [INFO]	[EVAL] Class IoU: 
    [0.9493 0.0987]
    2021-08-15 14:23:41 [INFO]	[EVAL] Class Acc: 
    [0.9866 0.1281]
    2021-08-15 14:23:41 [INFO]	[EVAL] The model with the best validation mIoU (0.5240) was saved at iter 600.
    2021-08-15 14:23:43 [INFO]	[TRAIN] epoch: 8, iter: 620/2000, loss: 1.5926, lr: 0.007166, batch_cost: 0.0926, reader_cost: 0.00008, ips: 43.1843 samples/sec | ETA 00:02:07
    2021-08-15 14:23:45 [INFO]	[TRAIN] epoch: 8, iter: 640/2000, loss: 1.4762, lr: 0.007072, batch_cost: 0.0923, reader_cost: 0.00007, ips: 43.3215 samples/sec | ETA 00:02:05
    2021-08-15 14:23:47 [INFO]	[TRAIN] epoch: 9, iter: 660/2000, loss: 1.5754, lr: 0.006978, batch_cost: 0.0982, reader_cost: 0.00514, ips: 40.7220 samples/sec | ETA 00:02:11
    2021-08-15 14:23:49 [INFO]	[TRAIN] epoch: 9, iter: 680/2000, loss: 1.4131, lr: 0.006885, batch_cost: 0.0944, reader_cost: 0.00008, ips: 42.3646 samples/sec | ETA 00:02:04
    2021-08-15 14:23:51 [INFO]	[TRAIN] epoch: 9, iter: 700/2000, loss: 1.5264, lr: 0.006791, batch_cost: 0.0926, reader_cost: 0.00007, ips: 43.2133 samples/sec | ETA 00:02:00
    2021-08-15 14:23:53 [INFO]	[TRAIN] epoch: 9, iter: 720/2000, loss: 1.5231, lr: 0.006697, batch_cost: 0.0920, reader_cost: 0.00007, ips: 43.4622 samples/sec | ETA 00:01:57
    2021-08-15 14:23:55 [INFO]	[TRAIN] epoch: 10, iter: 740/2000, loss: 1.3910, lr: 0.006603, batch_cost: 0.0956, reader_cost: 0.00437, ips: 41.8354 samples/sec | ETA 00:02:00
    2021-08-15 14:23:56 [INFO]	[TRAIN] epoch: 10, iter: 760/2000, loss: 1.3882, lr: 0.006508, batch_cost: 0.0943, reader_cost: 0.00007, ips: 42.3964 samples/sec | ETA 00:01:56
    2021-08-15 14:23:58 [INFO]	[TRAIN] epoch: 10, iter: 780/2000, loss: 1.4280, lr: 0.006414, batch_cost: 0.0928, reader_cost: 0.00007, ips: 43.0940 samples/sec | ETA 00:01:53
    2021-08-15 14:24:00 [INFO]	[TRAIN] epoch: 10, iter: 800/2000, loss: 1.4971, lr: 0.006319, batch_cost: 0.0928, reader_cost: 0.00007, ips: 43.0831 samples/sec | ETA 00:01:51
    2021-08-15 14:24:00 [INFO]	Start evaluating (total_samples: 76, total_iters: 76)...
    76/76 [==============================] - 2s 26ms/step - batch_cost: 0.0260 - reader cost: 3.1172e-
    2021-08-15 14:24:02 [INFO]	[EVAL] #Images: 76 mIoU: 0.4525 Acc: 0.8677 Kappa: 0.0425 
    2021-08-15 14:24:02 [INFO]	[EVAL] Class IoU: 
    [0.867 0.038]
    2021-08-15 14:24:02 [INFO]	[EVAL] Class Acc: 
    [0.985 0.042]
    2021-08-15 14:24:02 [INFO]	[EVAL] The model with the best validation mIoU (0.5240) was saved at iter 600.
    2021-08-15 14:24:04 [INFO]	[TRAIN] epoch: 10, iter: 820/2000, loss: 1.5384, lr: 0.006224, batch_cost: 0.0914, reader_cost: 0.00007, ips: 43.7620 samples/sec | ETA 00:01:47
    2021-08-15 14:24:06 [INFO]	[TRAIN] epoch: 11, iter: 840/2000, loss: 1.6333, lr: 0.006129, batch_cost: 0.0986, reader_cost: 0.00339, ips: 40.5486 samples/sec | ETA 00:01:54
    2021-08-15 14:24:08 [INFO]	[TRAIN] epoch: 11, iter: 860/2000, loss: 1.4332, lr: 0.006034, batch_cost: 0.0932, reader_cost: 0.00007, ips: 42.8956 samples/sec | ETA 00:01:46
    2021-08-15 14:24:10 [INFO]	[TRAIN] epoch: 11, iter: 880/2000, loss: 1.4018, lr: 0.005939, batch_cost: 0.0939, reader_cost: 0.00007, ips: 42.6127 samples/sec | ETA 00:01:45
    2021-08-15 14:24:12 [INFO]	[TRAIN] epoch: 11, iter: 900/2000, loss: 1.3624, lr: 0.005844, batch_cost: 0.0928, reader_cost: 0.00007, ips: 43.1087 samples/sec | ETA 00:01:42
    2021-08-15 14:24:14 [INFO]	[TRAIN] epoch: 12, iter: 920/2000, loss: 1.3956, lr: 0.005748, batch_cost: 0.0998, reader_cost: 0.00417, ips: 40.0791 samples/sec | ETA 00:01:47
    2021-08-15 14:24:16 [INFO]	[TRAIN] epoch: 12, iter: 940/2000, loss: 1.3608, lr: 0.005652, batch_cost: 0.0930, reader_cost: 0.00007, ips: 43.0135 samples/sec | ETA 00:01:38
    2021-08-15 14:24:18 [INFO]	[TRAIN] epoch: 12, iter: 960/2000, loss: 1.3518, lr: 0.005556, batch_cost: 0.0931, reader_cost: 0.00007, ips: 42.9606 samples/sec | ETA 00:01:36
    2021-08-15 14:24:19 [INFO]	[TRAIN] epoch: 12, iter: 980/2000, loss: 1.4034, lr: 0.005460, batch_cost: 0.0930, reader_cost: 0.00007, ips: 43.0154 samples/sec | ETA 00:01:34
    2021-08-15 14:24:21 [INFO]	[TRAIN] epoch: 13, iter: 1000/2000, loss: 1.3908, lr: 0.005364, batch_cost: 0.0981, reader_cost: 0.00413, ips: 40.7756 samples/sec | ETA 00:01:38
    2021-08-15 14:24:21 [INFO]	Start evaluating (total_samples: 76, total_iters: 76)...
    76/76 [==============================] - 2s 26ms/step - batch_cost: 0.0255 - reader cost: 3.0768e-0
    2021-08-15 14:24:23 [INFO]	[EVAL] #Images: 76 mIoU: 0.3683 Acc: 0.6967 Kappa: 0.0518 
    2021-08-15 14:24:23 [INFO]	[EVAL] Class IoU: 
    [0.6923 0.0442]
    2021-08-15 14:24:23 [INFO]	[EVAL] Class Acc: 
    [0.9937 0.0448]
    2021-08-15 14:24:24 [INFO]	[EVAL] The model with the best validation mIoU (0.5240) was saved at iter 600.
    2021-08-15 14:24:25 [INFO]	[TRAIN] epoch: 13, iter: 1020/2000, loss: 1.3855, lr: 0.005267, batch_cost: 0.0924, reader_cost: 0.00007, ips: 43.2770 samples/sec | ETA 00:01:30
    2021-08-15 14:24:27 [INFO]	[TRAIN] epoch: 13, iter: 1040/2000, loss: 1.3651, lr: 0.005170, batch_cost: 0.0932, reader_cost: 0.00007, ips: 42.9062 samples/sec | ETA 00:01:29
    2021-08-15 14:24:29 [INFO]	[TRAIN] epoch: 13, iter: 1060/2000, loss: 1.3928, lr: 0.005073, batch_cost: 0.0933, reader_cost: 0.00007, ips: 42.8552 samples/sec | ETA 00:01:27
    2021-08-15 14:24:31 [INFO]	[TRAIN] epoch: 14, iter: 1080/2000, loss: 1.4002, lr: 0.004976, batch_cost: 0.0975, reader_cost: 0.00357, ips: 41.0051 samples/sec | ETA 00:01:29
    2021-08-15 14:24:33 [INFO]	[TRAIN] epoch: 14, iter: 1100/2000, loss: 1.4900, lr: 0.004879, batch_cost: 0.0948, reader_cost: 0.00028, ips: 42.2078 samples/sec | ETA 00:01:25
    2021-08-15 14:24:35 [INFO]	[TRAIN] epoch: 14, iter: 1120/2000, loss: 1.3908, lr: 0.004781, batch_cost: 0.0931, reader_cost: 0.00007, ips: 42.9637 samples/sec | ETA 00:01:21
    2021-08-15 14:24:37 [INFO]	[TRAIN] epoch: 14, iter: 1140/2000, loss: 1.2982, lr: 0.004684, batch_cost: 0.0933, reader_cost: 0.00007, ips: 42.8949 samples/sec | ETA 00:01:20
    2021-08-15 14:24:39 [INFO]	[TRAIN] epoch: 15, iter: 1160/2000, loss: 1.3729, lr: 0.004586, batch_cost: 0.0971, reader_cost: 0.00381, ips: 41.1868 samples/sec | ETA 00:01:21
    2021-08-15 14:24:41 [INFO]	[TRAIN] epoch: 15, iter: 1180/2000, loss: 1.3370, lr: 0.004487, batch_cost: 0.0947, reader_cost: 0.00008, ips: 42.2311 samples/sec | ETA 00:01:17
    2021-08-15 14:24:42 [INFO]	[TRAIN] epoch: 15, iter: 1200/2000, loss: 1.3899, lr: 0.004389, batch_cost: 0.0926, reader_cost: 0.00007, ips: 43.1737 samples/sec | ETA 00:01:14
    2021-08-15 14:24:42 [INFO]	Start evaluating (total_samples: 76, total_iters: 76)...
    76/76 [==============================] - 2s 26ms/step - batch_cost: 0.0256 - reader cost: 3.0058e-
    2021-08-15 14:24:44 [INFO]	[EVAL] #Images: 76 mIoU: 0.4730 Acc: 0.8932 Kappa: 0.0726 
    2021-08-15 14:24:44 [INFO]	[EVAL] Class IoU: 
    [0.8925 0.0534]
    2021-08-15 14:24:44 [INFO]	[EVAL] Class Acc: 
    [0.9863 0.06  ]
    2021-08-15 14:24:45 [INFO]	[EVAL] The model with the best validation mIoU (0.5240) was saved at iter 600.
    2021-08-15 14:24:46 [INFO]	[TRAIN] epoch: 15, iter: 1220/2000, loss: 1.3806, lr: 0.004290, batch_cost: 0.0935, reader_cost: 0.00008, ips: 42.8030 samples/sec | ETA 00:01:12
    2021-08-15 14:24:48 [INFO]	[TRAIN] epoch: 16, iter: 1240/2000, loss: 1.3737, lr: 0.004191, batch_cost: 0.0977, reader_cost: 0.00453, ips: 40.9277 samples/sec | ETA 00:01:14
    2021-08-15 14:24:50 [INFO]	[TRAIN] epoch: 16, iter: 1260/2000, loss: 1.2770, lr: 0.004092, batch_cost: 0.0948, reader_cost: 0.00007, ips: 42.2008 samples/sec | ETA 00:01:10
    2021-08-15 14:24:52 [INFO]	[TRAIN] epoch: 16, iter: 1280/2000, loss: 1.3850, lr: 0.003992, batch_cost: 0.0924, reader_cost: 0.00007, ips: 43.2953 samples/sec | ETA 00:01:06
    2021-08-15 14:24:54 [INFO]	[TRAIN] epoch: 16, iter: 1300/2000, loss: 1.3245, lr: 0.003892, batch_cost: 0.0925, reader_cost: 0.00007, ips: 43.2290 samples/sec | ETA 00:01:04
    2021-08-15 14:24:56 [INFO]	[TRAIN] epoch: 17, iter: 1320/2000, loss: 1.3422, lr: 0.003792, batch_cost: 0.0961, reader_cost: 0.00382, ips: 41.6049 samples/sec | ETA 00:01:05
    2021-08-15 14:24:58 [INFO]	[TRAIN] epoch: 17, iter: 1340/2000, loss: 1.3811, lr: 0.003692, batch_cost: 0.0950, reader_cost: 0.00027, ips: 42.1251 samples/sec | ETA 00:01:02
    2021-08-15 14:25:00 [INFO]	[TRAIN] epoch: 17, iter: 1360/2000, loss: 1.3189, lr: 0.003591, batch_cost: 0.0924, reader_cost: 0.00007, ips: 43.2759 samples/sec | ETA 00:00:59
    2021-08-15 14:25:02 [INFO]	[TRAIN] epoch: 17, iter: 1380/2000, loss: 1.2513, lr: 0.003490, batch_cost: 0.0926, reader_cost: 0.00007, ips: 43.1992 samples/sec | ETA 00:00:57
    2021-08-15 14:25:04 [INFO]	[TRAIN] epoch: 18, iter: 1400/2000, loss: 1.2613, lr: 0.003389, batch_cost: 0.0973, reader_cost: 0.00463, ips: 41.1087 samples/sec | ETA 00:00:58
    2021-08-15 14:25:04 [INFO]	Start evaluating (total_samples: 76, total_iters: 76)...
    76/76 [==============================] - 2s 27ms/step - batch_cost: 0.0264 - reader cost: 3.1380e-0
    2021-08-15 14:25:06 [INFO]	[EVAL] #Images: 76 mIoU: 0.4843 Acc: 0.9107 Kappa: 0.0828 
    2021-08-15 14:25:06 [INFO]	[EVAL] Class IoU: 
    [0.9102 0.0584]
    2021-08-15 14:25:06 [INFO]	[EVAL] Class Acc: 
    [0.986  0.0675]
    2021-08-15 14:25:06 [INFO]	[EVAL] The model with the best validation mIoU (0.5240) was saved at iter 600.
    2021-08-15 14:25:08 [INFO]	[TRAIN] epoch: 18, iter: 1420/2000, loss: 1.3319, lr: 0.003287, batch_cost: 0.0931, reader_cost: 0.00008, ips: 42.9574 samples/sec | ETA 00:00:54
    2021-08-15 14:25:10 [INFO]	[TRAIN] epoch: 18, iter: 1440/2000, loss: 1.2639, lr: 0.003185, batch_cost: 0.0926, reader_cost: 0.00007, ips: 43.1901 samples/sec | ETA 00:00:51
    2021-08-15 14:25:11 [INFO]	[TRAIN] epoch: 18, iter: 1460/2000, loss: 1.3500, lr: 0.003083, batch_cost: 0.0929, reader_cost: 0.00007, ips: 43.0353 samples/sec | ETA 00:00:50
    2021-08-15 14:25:13 [INFO]	[TRAIN] epoch: 19, iter: 1480/2000, loss: 1.2412, lr: 0.002980, batch_cost: 0.0962, reader_cost: 0.00374, ips: 41.5910 samples/sec | ETA 00:00:50
    2021-08-15 14:25:15 [INFO]	[TRAIN] epoch: 19, iter: 1500/2000, loss: 1.3816, lr: 0.002877, batch_cost: 0.0957, reader_cost: 0.00007, ips: 41.8166 samples/sec | ETA 00:00:47
    2021-08-15 14:25:17 [INFO]	[TRAIN] epoch: 19, iter: 1520/2000, loss: 1.2165, lr: 0.002773, batch_cost: 0.0934, reader_cost: 0.00007, ips: 42.8356 samples/sec | ETA 00:00:44
    2021-08-15 14:25:19 [INFO]	[TRAIN] epoch: 19, iter: 1540/2000, loss: 1.3516, lr: 0.002669, batch_cost: 0.0934, reader_cost: 0.00007, ips: 42.8316 samples/sec | ETA 00:00:42
    2021-08-15 14:25:21 [INFO]	[TRAIN] epoch: 20, iter: 1560/2000, loss: 1.2660, lr: 0.002565, batch_cost: 0.0963, reader_cost: 0.00406, ips: 41.5420 samples/sec | ETA 00:00:42
    2021-08-15 14:25:23 [INFO]	[TRAIN] epoch: 20, iter: 1580/2000, loss: 1.2821, lr: 0.002460, batch_cost: 0.0938, reader_cost: 0.00007, ips: 42.6222 samples/sec | ETA 00:00:39
    2021-08-15 14:25:25 [INFO]	[TRAIN] epoch: 20, iter: 1600/2000, loss: 1.3484, lr: 0.002355, batch_cost: 0.0927, reader_cost: 0.00008, ips: 43.1356 samples/sec | ETA 00:00:37
    2021-08-15 14:25:25 [INFO]	Start evaluating (total_samples: 76, total_iters: 76)...
    76/76 [==============================] - 2s 24ms/step - batch_cost: 0.0244 - reader cost: 2.9002e-0
    2021-08-15 14:25:27 [INFO]	[EVAL] #Images: 76 mIoU: 0.5350 Acc: 0.9666 Kappa: 0.1707 
    2021-08-15 14:25:27 [INFO]	[EVAL] Class IoU: 
    [0.9664 0.1035]
    2021-08-15 14:25:27 [INFO]	[EVAL] Class Acc: 
    [0.9852 0.1693]
    2021-08-15 14:25:27 [INFO]	[EVAL] The model with the best validation mIoU (0.5350) was saved at iter 1600.
    2021-08-15 14:25:29 [INFO]	[TRAIN] epoch: 20, iter: 1620/2000, loss: 1.2305, lr: 0.002249, batch_cost: 0.0923, reader_cost: 0.00009, ips: 43.3275 samples/sec | ETA 00:00:35
    2021-08-15 14:25:31 [INFO]	[TRAIN] epoch: 20, iter: 1640/2000, loss: 1.3287, lr: 0.002142, batch_cost: 0.0914, reader_cost: 0.00007, ips: 43.7524 samples/sec | ETA 00:00:32
    2021-08-15 14:25:32 [INFO]	[TRAIN] epoch: 21, iter: 1660/2000, loss: 1.2836, lr: 0.002035, batch_cost: 0.0991, reader_cost: 0.00393, ips: 40.3649 samples/sec | ETA 00:00:33
    2021-08-15 14:25:34 [INFO]	[TRAIN] epoch: 21, iter: 1680/2000, loss: 1.3107, lr: 0.001927, batch_cost: 0.0930, reader_cost: 0.00007, ips: 43.0010 samples/sec | ETA 00:00:29
    2021-08-15 14:25:36 [INFO]	[TRAIN] epoch: 21, iter: 1700/2000, loss: 1.2511, lr: 0.001819, batch_cost: 0.0928, reader_cost: 0.00007, ips: 43.1022 samples/sec | ETA 00:00:27
    2021-08-15 14:25:38 [INFO]	[TRAIN] epoch: 21, iter: 1720/2000, loss: 1.2889, lr: 0.001710, batch_cost: 0.0924, reader_cost: 0.00007, ips: 43.3077 samples/sec | ETA 00:00:25
    2021-08-15 14:25:40 [INFO]	[TRAIN] epoch: 22, iter: 1740/2000, loss: 1.3148, lr: 0.001600, batch_cost: 0.0990, reader_cost: 0.00432, ips: 40.4203 samples/sec | ETA 00:00:25
    2021-08-15 14:25:42 [INFO]	[TRAIN] epoch: 22, iter: 1760/2000, loss: 1.2526, lr: 0.001489, batch_cost: 0.0944, reader_cost: 0.00008, ips: 42.3649 samples/sec | ETA 00:00:22
    2021-08-15 14:25:44 [INFO]	[TRAIN] epoch: 22, iter: 1780/2000, loss: 1.1961, lr: 0.001377, batch_cost: 0.0933, reader_cost: 0.00007, ips: 42.8676 samples/sec | ETA 00:00:20
    2021-08-15 14:25:46 [INFO]	[TRAIN] epoch: 22, iter: 1800/2000, loss: 1.3171, lr: 0.001265, batch_cost: 0.0933, reader_cost: 0.00007, ips: 42.8638 samples/sec | ETA 00:00:18
    2021-08-15 14:25:46 [INFO]	Start evaluating (total_samples: 76, total_iters: 76)...
    76/76 [==============================] - 2s 25ms/step - batch_cost: 0.0253 - reader cost: 3.4588e-
    2021-08-15 14:25:48 [INFO]	[EVAL] #Images: 76 mIoU: 0.5036 Acc: 0.9422 Kappa: 0.0987 
    2021-08-15 14:25:48 [INFO]	[EVAL] Class IoU: 
    [0.942  0.0653]
    2021-08-15 14:25:48 [INFO]	[EVAL] Class Acc: 
    [0.985 0.085]
    2021-08-15 14:25:48 [INFO]	[EVAL] The model with the best validation mIoU (0.5350) was saved at iter 1600.
    2021-08-15 14:25:50 [INFO]	[TRAIN] epoch: 23, iter: 1820/2000, loss: 1.2431, lr: 0.001151, batch_cost: 0.0985, reader_cost: 0.00451, ips: 40.5931 samples/sec | ETA 00:00:17
    2021-08-15 14:25:52 [INFO]	[TRAIN] epoch: 23, iter: 1840/2000, loss: 1.2270, lr: 0.001036, batch_cost: 0.0933, reader_cost: 0.00007, ips: 42.8774 samples/sec | ETA 00:00:14
    2021-08-15 14:25:54 [INFO]	[TRAIN] epoch: 23, iter: 1860/2000, loss: 1.2790, lr: 0.000919, batch_cost: 0.0927, reader_cost: 0.00007, ips: 43.1372 samples/sec | ETA 00:00:12
    2021-08-15 14:25:55 [INFO]	[TRAIN] epoch: 23, iter: 1880/2000, loss: 1.3235, lr: 0.000801, batch_cost: 0.0928, reader_cost: 0.00007, ips: 43.0882 samples/sec | ETA 00:00:11
    2021-08-15 14:25:57 [INFO]	[TRAIN] epoch: 24, iter: 1900/2000, loss: 1.2420, lr: 0.000681, batch_cost: 0.0976, reader_cost: 0.00395, ips: 40.9713 samples/sec | ETA 00:00:09
    2021-08-15 14:25:59 [INFO]	[TRAIN] epoch: 24, iter: 1920/2000, loss: 1.2636, lr: 0.000558, batch_cost: 0.0937, reader_cost: 0.00007, ips: 42.6881 samples/sec | ETA 00:00:07
    2021-08-15 14:26:01 [INFO]	[TRAIN] epoch: 24, iter: 1940/2000, loss: 1.2589, lr: 0.000432, batch_cost: 0.0925, reader_cost: 0.00007, ips: 43.2407 samples/sec | ETA 00:00:05
    2021-08-15 14:26:03 [INFO]	[TRAIN] epoch: 24, iter: 1960/2000, loss: 1.2392, lr: 0.000302, batch_cost: 0.0924, reader_cost: 0.00007, ips: 43.2695 samples/sec | ETA 00:00:03
    2021-08-15 14:26:05 [INFO]	[TRAIN] epoch: 25, iter: 1980/2000, loss: 1.1866, lr: 0.000166, batch_cost: 0.0975, reader_cost: 0.00403, ips: 41.0445 samples/sec | ETA 00:00:01
    2021-08-15 14:26:07 [INFO]	[TRAIN] epoch: 25, iter: 2000/2000, loss: 1.2386, lr: 0.000011, batch_cost: 0.0940, reader_cost: 0.00008, ips: 42.5441 samples/sec | ETA 00:00:00
    2021-08-15 14:26:07 [INFO]	Start evaluating (total_samples: 76, total_iters: 76)...
    76/76 [==============================] - 2s 26ms/step - batch_cost: 0.0256 - reader cost: 2.9707e-
    2021-08-15 14:26:09 [INFO]	[EVAL] #Images: 76 mIoU: 0.5110 Acc: 0.9553 Kappa: 0.1041 
    2021-08-15 14:26:09 [INFO]	[EVAL] Class IoU: 
    [0.9552 0.0668]
    2021-08-15 14:26:09 [INFO]	[EVAL] Class Acc: 
    [0.9843 0.0977]
    2021-08-15 14:26:09 [INFO]	[EVAL] The model with the best validation mIoU (0.5350) was saved at iter 1600.
    <class 'paddle.nn.layer.conv.Conv2D'>'s flops has been counted
    Customize Function has been applied to <class 'paddle.nn.layer.norm.SyncBatchNorm'>
    Cannot find suitable count function for <class 'paddle.nn.layer.pooling.MaxPool2D'>. Treat it as zero FLOPs.
    <class 'paddle.nn.layer.pooling.AdaptiveAvgPool2D'>'s flops has been counted
    <class 'paddle.nn.layer.pooling.AvgPool2D'>'s flops has been counted
    Cannot find suitable count function for <class 'paddle.nn.layer.activation.Sigmoid'>. Treat it as zero FLOPs.
    <class 'paddle.nn.layer.common.Dropout'>'s flops has been counted
    /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages/paddle/tensor/creation.py:143: DeprecationWarning: `np.object` is a deprecated alias for the builtin `object`. To silence this warning, use `object` by itself. Doing this will not modify any behavior and is safe. 
    Deprecated in NumPy 1.20; for more details and guidance: https://numpy.org/devdocs/release/1.20.0-notes.html#deprecations
      if data.dtype == np.object:
    /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages/paddle/fluid/dygraph/math_op_patch.py:238: UserWarning: The dtype of left and right variables are not the same, left dtype is VarType.FP32, but right dtype is VarType.INT32, the right dtype will convert to VarType.FP32
      format(lhs_dtype, rhs_dtype, lhs_dtype))
    Total Flops: 8061050880     Total Params: 2328346



```python
# 单独进行评估 -- 上边do_eval就是这个工作
!python PaddleSeg/val.py\
--config PaddleSeg/configs/quick_start/bisenet_optic_disc_512x512_1k.yml\
--model_path PaddleSeg/output/best_model/model.pdparams
# model_path： 模型参数路径
```

    /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages/paddle/fluid/layers/utils.py:26: DeprecationWarning: `np.int` is a deprecated alias for the builtin `int`. To silence this warning, use `int` by itself. Doing this will not modify any behavior and is safe. When replacing `np.int`, you may wish to use e.g. `np.int64` or `np.int32` to specify the precision. If you wish to review your current use, check the release note link for additional information.
    Deprecated in NumPy 1.20; for more details and guidance: https://numpy.org/devdocs/release/1.20.0-notes.html#deprecations
      def convert_to_list(value, n, name, dtype=np.int):
    2021-08-15 14:26:37 [INFO]	
    ---------------Config Information---------------
    batch_size: 4
    iters: 1000
    loss:
      coef:
      - 1
      - 1
      - 1
      - 1
      - 1
      types:
      - type: CrossEntropyLoss
    lr_scheduler:
      end_lr: 0
      learning_rate: 0.01
      power: 0.9
      type: PolynomialDecay
    model:
      pretrained: null
      type: BiSeNetV2
    optimizer:
      momentum: 0.9
      type: sgd
      weight_decay: 4.0e-05
    train_dataset:
      dataset_root: segDataset/horse
      mode: train
      num_classes: 2
      train_path: segDataset/horse/train_list.txt
      transforms:
      - target_size:
        - 512
        - 512
        type: Resize
      - type: RandomHorizontalFlip
      - type: Normalize
      type: Dataset
    val_dataset:
      dataset_root: data/optic_disc_seg
      mode: val
      transforms:
      - target_size:
        - 512
        - 512
        type: Resize
      - type: Normalize
      type: OpticDiscSeg
    ------------------------------------------------
    W0815 14:26:37.848217  3419 device_context.cc:362] Please NOTE: device: 0, GPU Compute Capability: 7.0, Driver API Version: 10.1, Runtime API Version: 10.1
    W0815 14:26:37.848265  3419 device_context.cc:372] device: 0, cuDNN Version: 7.6.
    2021-08-15 14:26:42 [INFO]	Loading pretrained model from PaddleSeg/output/best_model/model.pdparams
    2021-08-15 14:26:42 [INFO]	There are 356/356 variables loaded into BiSeNetV2.
    2021-08-15 14:26:42 [INFO]	Loaded trained params of model successfully
    2021-08-15 14:26:42 [INFO]	Start evaluating (total_samples: 76, total_iters: 76)...
    /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages/paddle/fluid/dataloader/dataloader_iter.py:89: DeprecationWarning: `np.bool` is a deprecated alias for the builtin `bool`. To silence this warning, use `bool` by itself. Doing this will not modify any behavior and is safe. If you specifically wanted the numpy scalar type, use `np.bool_` here.
    Deprecated in NumPy 1.20; for more details and guidance: https://numpy.org/devdocs/release/1.20.0-notes.html#deprecations
      if isinstance(slot[0], (np.ndarray, np.bool, numbers.Number)):
    /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages/paddle/fluid/dygraph/math_op_patch.py:238: UserWarning: The dtype of left and right variables are not the same, left dtype is VarType.INT32, but right dtype is VarType.BOOL, the right dtype will convert to VarType.INT32
      format(lhs_dtype, rhs_dtype, lhs_dtype))
    /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages/paddle/fluid/dygraph/math_op_patch.py:238: UserWarning: The dtype of left and right variables are not the same, left dtype is VarType.INT64, but right dtype is VarType.BOOL, the right dtype will convert to VarType.INT64
      format(lhs_dtype, rhs_dtype, lhs_dtype))
    76/76 [==============================] - 2s 26ms/step - batch_cost: 0.0260 - reader cost: 0.002
    2021-08-15 14:26:44 [INFO]	[EVAL] #Images: 76 mIoU: 0.5350 Acc: 0.9666 Kappa: 0.1707 
    2021-08-15 14:26:44 [INFO]	[EVAL] Class IoU: 
    [0.9664 0.1035]
    2021-08-15 14:26:44 [INFO]	[EVAL] Class Acc: 
    [0.9852 0.1693]


- 5.开始预测


```python
# 进行预测
!python PaddleSeg/predict.py\
--config PaddleSeg/configs/quick_start/bisenet_optic_disc_512x512_1k.yml\
--model_path PaddleSeg/output/best_model/model.pdparams\
--image_path segDataset/horse/Images\
--save_dir PaddleSeg/output/horse
# image_path: 预测图片路径/文件夹 -- 这里直接对训练数据进行预测，得到预测结果
# save_dir： 保存预测结果的路径 -- 保存的预测结果为图片
```

    /opt/conda/envs/python35-paddle120-env/lib/python3.7/site-packages/paddle/fluid/layers/utils.py:26: DeprecationWarning: `np.int` is a deprecated alias for the builtin `int`. To silence this warning, use `int` by itself. Doing this will not modify any behavior and is safe. When replacing `np.int`, you may wish to use e.g. `np.int64` or `np.int32` to specify the precision. If you wish to review your current use, check the release note link for additional information.
    Deprecated in NumPy 1.20; for more details and guidance: https://numpy.org/devdocs/release/1.20.0-notes.html#deprecations
      def convert_to_list(value, n, name, dtype=np.int):
    2021-08-15 14:27:04 [INFO]	
    ---------------Config Information---------------
    batch_size: 4
    iters: 1000
    loss:
      coef:
      - 1
      - 1
      - 1
      - 1
      - 1
      types:
      - type: CrossEntropyLoss
    lr_scheduler:
      end_lr: 0
      learning_rate: 0.01
      power: 0.9
      type: PolynomialDecay
    model:
      pretrained: null
      type: BiSeNetV2
    optimizer:
      momentum: 0.9
      type: sgd
      weight_decay: 4.0e-05
    train_dataset:
      dataset_root: segDataset/horse
      mode: train
      num_classes: 2
      train_path: segDataset/horse/train_list.txt
      transforms:
      - target_size:
        - 512
        - 512
        type: Resize
      - type: RandomHorizontalFlip
      - type: Normalize
      type: Dataset
    val_dataset:
      dataset_root: data/optic_disc_seg
      mode: val
      transforms:
      - target_size:
        - 512
        - 512
        type: Resize
      - type: Normalize
      type: OpticDiscSeg
    ------------------------------------------------
    W0815 14:27:04.356029  3513 device_context.cc:362] Please NOTE: device: 0, GPU Compute Capability: 7.0, Driver API Version: 10.1, Runtime API Version: 10.1
    W0815 14:27:04.356073  3513 device_context.cc:372] device: 0, cuDNN Version: 7.6.
    2021-08-15 14:27:09 [INFO]	Number of predict images = 328
    2021-08-15 14:27:09 [INFO]	Loading pretrained model from PaddleSeg/output/best_model/model.pdparams
    2021-08-15 14:27:09 [INFO]	There are 356/356 variables loaded into BiSeNetV2.
    2021-08-15 14:27:09 [INFO]	Start to predict...
    328/328 [==============================] - 16s 49ms/ste


# 五、可视化预测结果

通过PaddleSeg预测输出的结果为图片，对应位于:PaddleSeg/output/horse

其中包含两种结果：

- 一种为掩膜图像，即叠加预测结果与原始结果的图像 -- 位于: **PaddleSeg/output/horse/added_prediction**
- 另一种为预测结果的伪彩色图像，即预测的结果图像 -- 位于: **PaddleSeg/output/horse/pseudo_color_prediction**


```python
# 查看预测结果文件夹分布
!tree PaddleSeg/output/horse -L 1
```

    PaddleSeg/output/horse
    ├── added_prediction
    └── pseudo_color_prediction
    
    2 directories, 0 files


分别展示两个文件夹中的预测结果(下载每个预测结果文件夹中的一两张图片到本地，然后上传notebook)


上传说明:

![](https://ai-studio-static-online.cdn.bcebos.com/29cb48e1263a4ea49557a8564f289be1690e1b23dab7412388420f9c244f366c)

> 以下为展示结果

<font color='red' size=5> ---数据集 horse 的预测结果展示--- </font>

<font color='black' size=5> 掩膜图像： </font>

![](https://ai-studio-static-online.cdn.bcebos.com/45497756d58f4ba0b2f589327456e3d509073b93e60b43f5a6ac8399e4ec8eae)

![](https://ai-studio-static-online.cdn.bcebos.com/143571a5256e4a009aad0896cebf5096d8ac575a633e4d9e9ae482da6165e5e3)

<font color='black' size=5> 伪彩色图像： </font>

![](https://ai-studio-static-online.cdn.bcebos.com/52eb446af4504e908d39a50efa83ed4512968a9c3ea24ee0a3994b1fccb2eea9)

![](https://ai-studio-static-online.cdn.bcebos.com/b1935fcce3f243d9a6d0ad6300479a781c6ef790ef4c40c58a9585519eb107b7)


<font color='red' size=5>特别声明，使用horse数据集进行提交时，预测结果展示不允许使用horse242.jpg和horse242.png的预测结果，否则将可能认定为未进行本baseline作业的训练、预测过程 </font>

# 六、提交作业流程

1. 生成项目版本

![](https://ai-studio-static-online.cdn.bcebos.com/1c19ac6cfb314353b5377421c74bc5d777dcb5724fad47c1a096df758198f625)

2. (有能力的同学可以多捣鼓一下)根据需要可将notebook转为markdown，自行提交到github上

![](https://ai-studio-static-online.cdn.bcebos.com/0363ab3eb0da4242844cc8b918d38588bb17af73c2e14ecd92831b89db8e1c46)

3. (一定要先生成项目版本哦)公开项目

![](https://ai-studio-static-online.cdn.bcebos.com/8a6a2352f11c4f3e967bdd061f881efc5106827c55c445eabb060b882bf6c827)

# 七、寄语

<font size=4>

最后祝愿大家都能完成本次作业，圆满结业，拿到属于自己独一无二的结业证书——这一次训练营，将是你们AI之路成长的开始！

希望未来的你们能够坚持在自己热爱的方向上继续自己的梦想！

同时也期待你们能够在社区中创造更多有创意有价值基于飞桨系列的AI项目，发扬开源精神！

<br>

最后，再次祝愿大家都能顺利结业！
 </font>
