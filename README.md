## 本文档提供了从可视化工具的镜像构建、镜像部署、到工具的使用方法指南,涵盖了从数据处理到信号碱基序列可视化的整个流程.

- 目前cytools(碱基序列、movetable)、uncalled4(ul-tag)、signal-analyzer(信号碱基序列对齐可视化)分步依序独立串行运行.

---

## 📌 前置条件

- 已安装 Docker 环境

---

## 🚀 镜像创建与部署(如果已有镜像可跳过)

- 要求当前目录包含 `Dockerfile` 和 `build_container.sh` 脚本
- cyclone自研 cytools、minimap2、pyslow5、uncalled4、signal-analyzer、工具包

### 1. 构建基础镜像

进入包含 `Dockerfile` 和 `build_container.sh` 的目录，执行以下命令构建初始镜像：

```bash
nohup docker build -t signal_uncalled4:0.5 . &
```
提示：使用 nohup 可避免终端断开导致构建中断。构建过程可能耗时较长，请耐心等待。


### 2. 创建并启动容器

使用脚本启动容器，并映射指定端口（替换 `<port>` 为实际端口号）：

```bash
bash build_container.sh <port>
```
该脚本将启动一个运行中的容器实例，供后续软件安装使用。


### 3. 进入容器并安装依赖
进入容器后，依次执行以下操作：修改为实际对应的路径

#### 3.1 安装 cytools
```bash
cd /data/dengjunyi/env/softs
bash cytools_1.1.11.2_20250711.sh
```

#### 3.2 安装 Uncalled4
```bash
cd /data/dengjunyi/code/uncalled4
pip install -e .
```

#### 3.3 安装 Cyclone 开发版 pyslow5 (pyccf5)
```bash
cd /data/dengjunyi/code/uncalled4/pyccf5-main
python setup.py install
```

#### 3.4 安装 Signal-Analyzer
```sh
conda env create -f environment.yml -p ./env
conda activate ./env

# 从源码安装
cd external/cyccf && python setup.py install && cd ../../
pip install -ve .

# 从wheel文件安装
pip install dist/pyslow5-1.2.0b0-cp312-cp312-linux_x86_64.whl
pip install dist/signal_analyzer-0.2.1-py3-none-any.whl
```


### 4. 提交容器为新镜像
完成所有安装后，退出容器，使用其容器 ID 提交为新的镜像版本：
```bash
docker commit bb7778d19378 signal_uncalled4:0.6
```
⚠️ 请将 bb7778d19378 替换为实际的容器 ID， signal_uncalled4:0.6改为实际目标版本。


### 5. 导出镜像（可选）
为便于迁移或备份，可将新镜像导出为 tar 包：
```bash
docker save -o signal_uncalled4_6.tar signal_uncalled4:0.6
```
⚠️ 请依据4修改signal_uncalled4:0.6，并将signal_uncalled4_6 替换为实际的目标输出镜像。




## 🚀 启动容器

使用脚本启动容器，并映射指定端口（替换 `<port>` 为实际端口号）：

```bash
bash build_container.sh <port>
```
该脚本将启动一个运行中的容器实例，为工具的运行环境。




## 🔧 Uncalled4 工具的使用
- 输入要求已有经过cytools处理产出的bam文件
###  参数修改
依据实际情况修改align.toml文件中的配置参数:
```bash
[pore_model]: 
    #提供模型路径it10.model.npz
    name = "/data/dengjunyi/code/uncalled4/example/it10.model.npz"  
[tracks]: 
    #提供参考序列路径
    ref = "/data/zhangjiayuan/Data/reference/HG002_Ash1/Ash1_v2.2.fasta"
[tracks.io]
    #并行运行任务数
    processes = 10
    run_id = "250F302971012"

    #包含moveable的使用cytools产出的bam文件路径
    cytools_bam = "/data/dengjunyi/data/input/HG002_WY_6/fastq/250F302971012/summary/250F302971012.bam"
    fastq_dir = "/data/dengjunyi/data/input/HG002_WY_6/fastq/250F302971012/all/bases"

    #最终uncalled4产出结果输出路径
    bam_out = "/data/dengjunyi/data/output/uncalled4/hg002/250F302953011/250F302953011_align_docker_image_cyir2.bam"
```

### 命令执行
#更换指定align.toml路径即可
```bash
uncalled4 align -C /data/dengjunyi/code/uncalled4/example/align.toml -p 1 --bam-f5c
```




## 📊 signal-analyzer 工具的使用
- 输入要求已有经过Uncalled4处理产出的bam文件
###  使用 signal-to-reference 比对信息作图，对目标区域进行可视化如 ad36f57dd29d43c6_1.fwd:1030-1070 , sample-size为采样个数
```bash
REGION="ad36f57dd29d43c6_1.fwd:1030-1070"
signal-analyzer plot-pileup \
    --bam-path $OUTPUT_PATH/movetable/realign.bam \
    --ccf5-dir $OUTPUT_PATH/sample_signal \
    --reference-path $REFERENCE_PATH \
    --region $REGION \
    --sample-size 6 \
    --output-dir $OUTPUT_DIR/pileup_plots
```


