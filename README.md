# 总览

本文主要记录修改vllm相关代码，来适配Pascal框架，涉及到以下内容

 - [x] vllm 0.14.0 修改及build
 - [x] pytorch 2.9.1 修改及build
 - [x] triton 3.6.0 修改及build

本文 patch 思路来自于大佬的项目：[vllm-pascal](https://github.com/ampir-nn/vllm-pascal)，同时适配了新版本的部分语法
# 正文

## 背景

我在本地通过P40搭建了一套算力系统，但是vllm部署时，由于P40系统架构较老，导致模型部署时，会遇到很多架构不适配的错误

网上找到了针对于 vllm 0.10.2、vllm 0.11.0 这两个版本的魔改： [vllm-pascal](https://github.com/ampir-nn/vllm-pascal)，但是这两个版本在部署`AutoGLM-Phone-9B` 等最新模型时，遇到了api不适配的问题，所以自己按照大佬的patch，重新build了最新的 vllm
## 使用方案

我重新build后的GitHub仓库：https://github.com/shu1qin9/vllm-0.14.0-pascal

![image.png](https://lsq-markdown.oss-cn-hangzhou.aliyuncs.com/wiki20251226171434.png)


下载最新的三个 `whl` 文件，然后创建最新的 python env 环境，就可以开始对应的工作，相关shell如下

```shell
# python env 创建
python3 -m venv pyenv-P40
source pyenv-p40/bin/activate

# vllm安装
pip install vllm-0.14.0rc1.dev122+g42826bbcc.d20251226.cu128-cp312-cp312-linux_x86_64.whl
```

注意，安装打过补丁的vllm后，要先卸载triton、torch，vllm 会自动安装这两个包，按照下面命令执行即可

```shell
# triton、torch 安装
pip uninstall torch triton -y

pip install triton-3.6.0+git9e449252-cp312-cp312-linux_x86_64.whl
pip install torch-2.9.1a0+gitd38164a-cp312-cp312-linux_x86_64.whl
```

这里在安装 triton 补丁时，会出现依赖版本的Error，无需关注，所有的安装完成后，`pip list`，可以看到以下内容

只要 list 中对应的三个库，是我们补丁过的即可

```shell
sniffio                           1.3.1
sse-starlette                     3.0.4
starlette                         0.50.0
supervisor                        4.3.0
sympy                             1.14.0
tabulate                          0.9.0
tiktoken                          0.12.0
tokenizers                        0.22.1
torch                             2.9.1a0+gitd38164a
torchaudio                        2.9.1
torchvision                       0.24.1
tqdm                              4.67.1
transformers                      4.57.3
triton                            3.6.0+git9e449252
typer                             0.21.0
typing_extensions                 4.15.0
typing-inspection                 0.4.2
urllib3                           2.6.2
uvicorn                           0.40.0
uvloop                            0.22.1
vllm                              0.14.0rc1.dev122+g42826bbcc.d20251226.cu128
```

![image.png](https://lsq-markdown.oss-cn-hangzhou.aliyuncs.com/wiki20251226171940.png)

接下来 python vllm 启动大模型即可
