# PaddleClas_Using_2

AiStudio:[全流程文档 第二篇](https://aistudio.baidu.com/aistudio/projectdetail/1585531)

# 再[PaddleClas第一篇](https://aistudio.baidu.com/aistudio/projectdetail/1133588)的基础上，来实现PaddleClas的服务功能

#### **PaddleClas全流程文档 第二篇，主要展示如何部署PaddleClas的Hub Serving服务，服务化部署。上一篇（[配置使用PaddleClas](https://aistudio.baidu.com/aistudio/projectdetail/1133588)），下一篇（[部署三DLL调用/EXE](https://aistudio.baidu.com/aistudio/projectdetail/1184186)）**
------------------------------------------------------------------------------------------------------------

# 基于PaddleHub Serving的服务部署

#### **hubserving服务部署配置服务包`clas`下包含3个必选文件，目录如下：**
```
deploy/hubserving/clas/
  └─  __init__.py    空文件，必选
  └─  config.json    配置文件，可选，使用配置启动服务时作为参数传入
  └─  module.py      主模块，必选，包含服务的完整逻辑
  └─  params.py      参数文件，必选，包含模型路径、前后处理参数等参数
```


<img src="https://ai-studio-static-online.cdn.bcebos.com/036ae99e840149deb8d600d6737c4c404b4fbf04446b48ca9dc6439167ffd4ec" width = "800" height = "400" align=center />

<br/>

## 快速启动服务

### 打开anaconda，激活paddle环境（已在[环境搭建](https://aistudio.baidu.com/aistudio/projectdetail/1147447)中建立）


<img src="https://ai-studio-static-online.cdn.bcebos.com/f2ece916d81042adae1e40cda6e72e74fe807bf7c8314c338063af46579d6112" width = "800" height = "400" align=center />

<br/>


### 1. 准备环境

```
# 安装paddlehub2.0版本
pip3 install paddlehub==2.0.0b1 --upgrade -i https://pypi.tuna.tsinghua.edu.cn/simple
```

<img src="https://ai-studio-static-online.cdn.bcebos.com/43268a51cfd443ae8996201139e51b638418c9d6a1cb429f95243a8dc5748b88" width = "800" height = "400" align=center />

<br/>


### 2. 下载推理模型
安装服务模块前，需要准备推理模型并放到正确路径，默认模型路径为：
```
分类推理模型结构文件：./inference/cls_infer.pdmodel
分类推理模型权重文件：./inference/cls_infer.pdiparams
```  

**模型路径可在deploy/hubserving/clas/`params.py`中查看和修改。**

<br/>

<img src="https://ai-studio-static-online.cdn.bcebos.com/7ebca8613c3642a099d8a5712dc6473f7c18504a934943f7ac98d4a80ca3d316" width = "800" />

<br/>
 
- 可以使用基于ImageNet-1k数据集的预训练模型，模型列表及下载地址详见[模型库概览](https://github.com/PaddlePaddle/PaddleClas/blob/release/2.0/docs/zh_CN/models/models_intro.md)
- 也可以替换成自己训练转换好的模型。
- 这里我替换成了之前训练的模型

<br/>
 
<img src="https://ai-studio-static-online.cdn.bcebos.com/eed72cec70e3468bad2883ee07b5a11d3cb0e35d008a4c9f847e38e730ce2524" width = "800" height = "400" align=center />

<br/>
 
### 3. 安装服务模块

* 进入PaddleClas目录 安装示例如下：
```
# 进入目录
cd /d E:\PaddleClas-release-2.0
# 安装服务模块：  
hub install deploy\hubserving\clas\
```

<img src="https://ai-studio-static-online.cdn.bcebos.com/3ff132a2292a4b919e7bbb667d29f426476385cbf45844c39d1a8871932d860e" width = "800" height = "400" align=center />

<br/>

- **如果出现 拒绝访问 错误，可以重新使用管理员身份打开Anaconda，再依次操作即可**

### 4. 启动服务
#### 方式1. 命令行命令启动（仅支持CPU）
**启动命令：**  
```shell
$ hub serving start --modules Module1==Version1 \
                    --port XXXX \
                    --use_multiprocess \
                    --workers \
```  

**参数：**  

|参数|用途|  
|-|-|  
|--modules/-m| [**必选**] PaddleHub Serving预安装模型，以多个Module==Version键值对的形式列出<br>*`当不指定Version时，默认选择最新版本`*|  
|--port/-p| [**可选**] 服务端口，默认为8866|  
|--use_multiprocess| [**可选**] 是否启用并发方式，默认为单进程方式，推荐多核CPU机器使用此方式<br>*`Windows操作系统只支持单进程方式`*|
|--workers| [**可选**] 在并发方式下指定的并发任务数，默认为`2*cpu_count-1`，其中`cpu_count`为CPU核数|  

如按默认参数启动服务：  ```hub serving start -m clas_system```  

这样就完成了一个服务化API的部署，使用默认端口号8866。

#### 方式2. 配置文件启动（支持CPU、GPU）
**启动命令：**  
```hub serving start -c config.json```  

其中`config.json`在目录deploy/hubserving/clas/内，格式如下：
```json
{
    "modules_info": {
        "clas_system": {
            "init_args": {
                "version": "1.0.0",
                "use_gpu": true,
                "enable_mkldnn": false
            },
            "predict_args": {
            }
        }
    },
    "port": 8866,
    "use_multiprocess": false,
    "workers": 2
}
```

- `init_args`中的可配参数与`module.py`中的`_initialize`函数接口一致。其中，
  - 当`use_gpu`为`true`时，表示使用GPU启动服务。
  - 当`enable_mkldnn`为`true`时，表示使用MKL-DNN加速。
- `predict_args`中的可配参数与`module.py`中的`predict`函数接口一致。

**注意:**  
- 使用配置文件启动服务时，其他参数会被忽略。
- 如果使用GPU预测(即，`use_gpu`置为`true`)，则需要在启动服务之前，设置CUDA_VISIBLE_DEVICES环境变量，如：```set CUDA_VISIBLE_DEVICES=0```，否则不用设置。
- **`use_gpu`不可与`use_multiprocess`同时为`true`**。
- **`use_gpu`与`enable_mkldnn`同时为`true`时，将忽略`enable_mkldnn`，而使用GPU**。

如，使用GPU 0号卡启动串联服务(Windows 下不支持多卡一般使用0号卡，如果有问题可以尝试禁用其他显卡只保留一张使用)：  
```shell
set CUDA_VISIBLE_DEVICES=0
hub serving start -c deploy/hubserving/clas/config.json
```  

<img src="https://ai-studio-static-online.cdn.bcebos.com/16217b09329b476e9b7c9ac18b1ed377473664660a604be9b36047553615dafe" width = "800" height = "400" align=center />

<br/>

- **启动服务后窗口最好不要关闭，因为不用的时候可以再窗口随时取消，放在后台容易忘记，浪费gpu资源**

## 发送预测请求
#### **配置好服务端，可使用以下命令发送预测请求，获取预测结果:**

```python tools/test_hubserving.py server_url image_path```  

#### **需要给脚本传递2个参数**：  
- **server_url**：服务地址，格式为  
`http://[ip_address]:[port]/predict/[module_name]`  
- **image_path**：测试图像路径，可以是单张图片路径，也可以是图像集合目录路径
- **top_k**：[**可选**] 返回前 `top_k` 个 `score` ，默认为 `1`。

#### **访问示例：**  
```python tools/test_hubserving.py http://127.0.0.1:8866/predict/clas_system ./dataset/cat_12/cat_12_train/0aSixIFj9X73z41LMDUQ6ZykwnBA5YJW.jpg```

- **打开新命令窗口，输入以上指令：**

<img src="https://ai-studio-static-online.cdn.bcebos.com/5cb5504453034aea82e3228985e838aac25ba4d4bfab47eea3bf891013cd456d" width = "800" height = "400" align=center />

<br/>

### 返回结果格式说明
#### **返回结果为列表（list），包含top-k个分类结果，以及对应的得分，还有此图片预测耗时，具体如下：**
```
list: 返回结果
└─ list: 第一张图片结果
   └─ list: 前k个分类结果，依score递减排序
   └─ list: 前k个分类结果对应的score，依score递减排序
   └─ float: 该图分类耗时，单位秒
```

**说明：** 如果需要增加、删除、修改返回字段，可在相应模块的`module.py`文件中进行修改，完整流程参考下一节自定义修改服务模块。

## 自定义修改服务模块
#### **如果需要修改服务逻辑，你一般需要操作以下步骤：**  

- 1、 停止服务  
```hub serving stop --port/-p XXXX```  

- 2、 到相应的`module.py`和`params.py`等文件中根据实际需求修改代码。  
  例如，例如需要替换部署服务所用模型，则需要到`params.py`中修改模型路径参数`cfg.model_file`和`cfg.params_file`。

  修改并安装（`hub install deploy/hubserving/clas/`）完成后，在进行部署前，可通过`python deploy/hubserving/clas/test.py`测试已安装服务模块。

- 3、 卸载旧服务包  
```hub uninstall clas_system```  

- 4、 安装修改后的新服务包  
```hub install deploy/hubserving/clas/```  

- 5、重新启动服务  
```hub serving start -m clas_system```  

# 更多内容请查看[PaddleClas官网](https://github.com/PaddlePaddle/PaddleClas)

## 更多PaddleClas部署

[部署三（生成exe文件部署）](https://aistudio.baidu.com/aistudio/projectdetail/1184186)

[部署四（Python/C#调用DLL部署，可传参）](https://aistudio.baidu.com/aistudio/projectdetail/1589564)

### **如果在操作过程中有任何问题欢迎在评论区提出，或者在[PaddleClas-Github](https://github.com/PaddlePaddle/PaddleClas)提issue**
