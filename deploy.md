项目地址：[https://github.com/HumanAIGC-Engineering/OpenAvatarChat](https://github.com/HumanAIGC-Engineering/OpenAvatarChat)

### 拉取代码

```plain
git clone https://github.com/HumanAIGC-Engineering/OpenAvatarChat.git

cd OpenAvatarChat

```

### 更新子项目及模型

安装git lfs

```plain
sudo apt install git-lfs
git lfs install 
```

<font style="color:rgb(31, 35, 40);">项目通过git子模块方式引用三方库，运行前需要更新子模块</font>

```plain
git submodule update --init --recursive
```

### 安装UV

<font style="color:rgb(31, 35, 40);">使用uv进行进行本地环境管理。</font>

```plain
pip install uv
```

#### <font style="color:rgb(31, 35, 40);">依赖安装</font>

##### <font style="color:rgb(31, 35, 40);">安装全部依赖</font>

```plain
uv sync --all-packages
```

##### <font style="color:rgb(31, 35, 40);">仅安装所需模式的依赖</font>

```plain
uv run install.py --uv --config <配置文件的绝对路径>.yaml 
```



### 相关部署要求

#### ssl证书

<font style="color:rgb(31, 35, 40);">项目使用rtc作为视音频传输的通道，用户如果需要从localhost以外的地方连接服务的话，需要准备ssl证书以开启https，默认配置会读取ssl_certs目录下的localhost.crt和localhost.key，在scripts目录下提供了生成自签名证书的脚本。需要在项目根目录下运行脚本以使生成的证书被放到默认位置。</font>

```plain
sh scripts/create_ssl_certs.sh
```

#### <font style="color:rgb(31, 35, 40);">TURN Server</font>

<font style="color:rgb(31, 35, 40);">如果点击开始对话后，出现一直等待中的情况，可能你的部署环境存在NAT穿透方面的问题（如部署在云上机器等），需要进行数据中继。在Linux环境下，可以使用coturn来架设TURN服务。可参考以下操作在同一机器上安装、启动并配置使用coturn：</font>

```plain
$ chmod 777 scripts/setup_coturn.sh
# scripts/setup_coturn.sh
```

> 可以修改对应的端口，避免暴露过多端口

<font style="color:rgb(31, 35, 40);">修改config对应的配置文件，添加以下配置后启动服务，启动之前确保防火墙（包括云上机器安全组等策略）开放coturn所需端口。</font>

```plain
RtcClient: #若使用Lam，则此项配置为LamClient
  turn_config:
    turn_provider: "turn_server"
    urls: ["turn:127.0.0.1:3478", "turn:115.190.106.133:5349"]
    username: "username"
    credential: "password"
		  
```

### 运行api方式

tts跟llm调用百炼的api，将<font style="color:rgb(31, 35, 40);">chat_with_openai_compatible_bailian_cosyvoice.yaml文件里面对应的key进行替换</font>

```plain
LLM_Bailian: 
  moedl_name: "qwen-plus"
  system_prompt: "你是个AI对话数字人，你要用简短的对话来回答我的问题，并在合理的地方插入标点符号"
  api_url: 'https://dashscope.aliyuncs.com/compatible-mode/v1'
  api_key: 'yourapikey' # default=os.getenv("DASHSCOPE_API_KEY")
```



#### 本地运行

```plain
uv run src/demo.py --config config/chat_with_openai_compatible_bailian_cosyvoice.yaml
```

#### docker运行

对应的dockerfile可能需要注释掉最后一行的`ADD ./.env* $WORK_DIR/`

```plain
......
ADD ./resource $WORK_DIR/resource
ADD ./scripts $WORK_DIR/scripts
#ADD ./.env* $WORK_DIR/

WORKDIR $WORK_DIR
.....
```

运行

```plain
./build_and_run.sh --config config/chat_with_openai_compatible_bailian_cosyvoice.yaml
```



### 运行minicpm

#### <font style="color:rgb(31, 35, 40);">依赖模型</font>

<font style="color:rgb(31, 35, 40);">MiniCPM-o-2.6 本项目可以使用MiniCPM-o-2.6作为多模态语言模型为数字人提供对话能力，用户可以按需从</font>[<font style="color:rgb(9, 105, 218);">Huggingface</font>](https://huggingface.co/openbmb/MiniCPM-o-2_6)<font style="color:rgb(31, 35, 40);">或者</font>[<font style="color:rgb(9, 105, 218);">Modelscope</font>](https://modelscope.cn/models/OpenBMB/MiniCPM-o-2_6)<font style="color:rgb(31, 35, 40);">下载相关模型。建议将模型直接下载到 <ProjectRoot>/models/ 默认配置的模型路径指向这里，如果放置与其他位置，需要修改配置文件。scripts目录中有对应模型的下载脚本，可供在linux环境下使用，请在项目根目录下运行脚本：</font>

```plain
scripts/download_MiniCPM-o_2.6.sh
scripts/download_MiniCPM-o_2.6-int4.sh
```

部署的是int4版本，所以需要修改对应config中的文件配置





#### <font style="color:rgb(31, 35, 40);">安装专用分支的AutoGPTQ</font>

在项目目录下安装

```plain
git clone https://github.com/OpenBMB/AutoGPTQ.git && cd AutoGPTQ
git checkout minicpmo
```

检查安装对应依赖

```plain
pip install numpy gekko pandas build-essential ninja-build
```

需要安装pytorch

```plain
pip install torch==2.5.1 torchvision==0.20.1 torchaudio==2.5.1 --index-url https://download.pytorch.org/whl/cu121
```

在根目录激活对应的虚拟环境

```plain
source .venv/bin/activate
```

安装AutoGPTQ

```plain
pip install -vvv --no-build-isolation -e .
```



### 本地启动

```plain
uv run src/demo.py --config config/chat_with_minicpm.yaml
```

启动中可能会遇到

```plain
...libcusparse.so.12: undefined snvJitLinkcomplete_12_4..
```

需要添加一下环境变量

```plain
export LD_LIBRARY_PATH=/home/ai/OpenAvatarChat/.venv/lib/python3.11/site-packages/nvidia/cusparse
```



#### docker启动

需要在bulid镜像时安装autoGptq，需要修改DockerFile

```plain
FROM nvidia/cuda:12.2.2-cudnn8-devel-ubuntu22.04
LABEL authors="HumanAIGC-Engineering"

ARG CONFIG_FILE=config/chat_with_minicpm.yaml

ENV DEBIAN_FRONTEND=noninteractive
ENV TORCH_CUDA_ARCH_LIST="8.0 8.6 9.0"

# 替换为清华大学的APT源
RUN sed -i 's/archive.ubuntu.com/mirrors.tuna.tsinghua.edu.cn/g' /etc/apt/sources.list && \
    sed -i 's/security.ubuntu.com/mirrors.tuna.tsinghua.edu.cn/g' /etc/apt/sources.list

# 更新包列表并安装必要的依赖
RUN apt-get update && \
    apt-get install -y software-properties-common && \
    apt-get install -y python3.10 python3.10-dev python3-pip git && \
    apt-get install -y build-essential ninja-build

RUN update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.10 1

ARG WORK_DIR=/root/open-avatar-chat
WORKDIR $WORK_DIR

#安装核心依赖
COPY ./install.py $WORK_DIR/install.py
COPY ./pyproject.toml $WORK_DIR/pyproject.toml
COPY ./src/third_party $WORK_DIR/src/third_party
RUN pip install uv && \
    uv venv --python 3.10 && \
    uv sync --no-install-workspace

ADD ./src $WORK_DIR/src

#安装config依赖
RUN echo "Using config file: ${CONFIG_FILE}"
COPY $CONFIG_FILE /tmp/build_config.yaml
RUN uv run install.py \
    --config /tmp/build_config.yaml \
    --uv \
    --skip-core && \
    rm /tmp/build_config.yaml

RUN .venv/bin/pip install torch==2.5.1 torchvision==0.20.1 torchaudio==2.5.1 --index-url https://download.pytorch.org/whl/cu121

# 安装核心依赖后复制AutoGPTQ并安装
COPY ./AutoGPTQ $WORK_DIR/AutoGPTQ

# 安装AutoGPTQ依赖
#RUN cd AutoGPTQ && .venv/bin/pip install -vvv --no-build-isolation -e ./AutoGPTQ
# 安装AutoGPTQ（使用虚拟环境中的pip）
RUN cd AutoGPTQ && \
    git checkout minicpmo && \
    $WORK_DIR/.venv/bin/pip install --no-cache-dir -vvv --no-build-isolation -e . && \
    cd ..


ADD ./resource $WORK_DIR/resource
ADD ./scripts $WORK_DIR/scripts
#ADD ./.env* $WORK_DIR/

WORKDIR $WORK_DIR
ENTRYPOINT ["uv", "run", "src/demo.py"]
```

构建启动

```plain
./build_and_run.sh --config config/chat_with_minicpm.yaml
```

