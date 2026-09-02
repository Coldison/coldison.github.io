---
title: 本地部署fauxpilot
date: 2022-08-19 15:38:56
tags:
- python
- copilot
categories:
- 教程
- Docker使用
---
由于copilot决定要收费，而且代码上传到对面的服务器也存在安全上的疑虑。小伙伴给我推了[fauxpilot](https://github.com/moyix/fauxpilot)，正好组里面显卡闲置，所以自己本地部署了一下。由于使用的是shell脚本，因此可能需要注意下不同系统的编码问题，直接在linux上git clone或者解压。

### 包和依赖

+ 安装Docker。
+ 安装docker compose >= 1.28。Docker Compose是一个用来定义和运行复杂应用的Docker工具。一个使用Docker容器的应用，通常由多个容器组成。使用Docker Compose不再需要使用shell脚本来启动容器。 Compose 通过一个yml配置文件来管理多个Docker容器，在配置文件中，所有的容器通过services来定义，然后使用docker compose脚本来启动，停止和重启应用，和应用中的服务以及所有依赖服务的容器，非常适合组合使用多个容器进行开发的场景。
+ NVIDIA算力 >= 7.0的GPU，根据显存选择合适的模型。
+ 安装nvidia-docker，这是nvidia弄出的链接显卡的容器技术。
+ curl和zstd用于下载与解压模型，建议提前检查有没有安装。

### 部署

参见README。

1. 运行 ``bash setup.sh``，我习惯使用bash命令。按照提示，输入命令，该脚本会下载对应的模型并且配置config.env文件，主要是一些参数。以下是我的示例。

   ```config
   MODEL=codegen-2B-mono
   NUM_GPUS=1
   MODEL_DIR=/data/ds/fauxpilot-main/models
   ```

   如果要重新部署，需要先删除config.env文件，否则脚本还是会执行原本的配置。
2. 运行 ``bash launch.sh``。读取对应的参数并且执行 ``docker compose``命令。

   ```yml
    version: '3.3'
    services:
        triton:
            image: moyix/triton_with_ft:22.06
            command: bash -c "CUDA_VISIBLE_DEVICES=${GPUS} mpirun -n 1 --allow-run-as-root /opt/tritonserver/bin/tritonserver --model-repository=/model"
            shm_size: '2gb'
            volumes:
            - ${MODEL_DIR}:/model
            ports:
            - "8000:8000"
            - "8001:8001"
            - "8002:8002"
            deploy:
            resources:
                reservations:
                devices:
                    - driver: nvidia
                    count: all
                    capabilities: [gpu]
        copilot_proxy:
            image: moyix/copilot_proxy:latest
            command: python3 -m flask run --host=0.0.0.0 --port=5000
            ports:
            - "5000:5000"
   ```

   以上就是docker_compose.yml文件，可以看到主要运行了两个服务，一个推理服务，一个是flask网络应用。使用 ``docker ps``可以查询是否正在运行。
   ![docker进程](https://link.jscdn.cn/1drv/aHR0cHM6Ly8xZHJ2Lm1zL3UvcyFBa3A3T0FGTWxmRUtoNWdMN1N0N3VtdkZGVFo4cFE_ZT03VDlYaDI.png)

   ```bash
   $ ./launch.sh 
   [+] Running 2/0
   ⠿ Container fauxpilot-triton-1         Created                                                                                                                                                                                                                                                                                             0.0s
   ⠿ Container fauxpilot-copilot_proxy-1  Created                                                                                                                                                                                                                                                                                             0.0s
   Attaching to fauxpilot-copilot_proxy-1, fauxpilot-triton-1
   fauxpilot-triton-1         | 
   fauxpilot-triton-1         | =============================
   fauxpilot-triton-1         | == Triton Inference Server ==
   fauxpilot-triton-1         | =============================
   fauxpilot-triton-1         | 
   fauxpilot-triton-1         | NVIDIA Release 22.06 (build 39726160)
   fauxpilot-triton-1         | Triton Server Version 2.23.0
   fauxpilot-triton-1         | 
   fauxpilot-triton-1         | Copyright (c) 2018-2022, NVIDIA CORPORATION & AFFILIATES.  All rights reserved.
   fauxpilot-triton-1         | 
   fauxpilot-triton-1         | Various files include modifications (c) NVIDIA CORPORATION & AFFILIATES.  All rights reserved.
   fauxpilot-triton-1         | 
   fauxpilot-triton-1         | This container image and its contents are governed by the NVIDIA Deep Learning Container License.
   fauxpilot-triton-1         | By pulling and using the container, you accept the terms and conditions of this license:
   fauxpilot-triton-1         | https://developer.nvidia.com/ngc/nvidia-deep-learning-container-license
   fauxpilot-copilot_proxy-1  | WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
   fauxpilot-copilot_proxy-1  |  * Debug mode: off
   fauxpilot-copilot_proxy-1  |  * Running on all addresses (0.0.0.0)
   fauxpilot-copilot_proxy-1  |    WARNING: This is a development server. Do not use it in a production deployment.
   fauxpilot-copilot_proxy-1  |  * Running on http://127.0.0.1:5000
   fauxpilot-copilot_proxy-1  |  * Running on http://172.18.0.3:5000 (Press CTRL+C to quit)
   fauxpilot-triton-1         | 
   fauxpilot-triton-1         | ERROR: This container was built for NVIDIA Driver Release 515.48 or later, but
   fauxpilot-triton-1         |        version  was detected and compatibility mode is UNAVAILABLE.
   fauxpilot-triton-1         | 
   fauxpilot-triton-1         |        [[]]
   fauxpilot-triton-1         | 
   fauxpilot-triton-1         | I0803 01:51:02.690042 93 pinned_memory_manager.cc:240] Pinned memory pool is created at '0x7f6104000000' with size 268435456
   fauxpilot-triton-1         | I0803 01:51:02.690461 93 cuda_memory_manager.cc:105] CUDA memory pool is created on device 0 with size 67108864
   fauxpilot-triton-1         | I0803 01:51:02.692434 93 model_repository_manager.cc:1191] loading: fastertransformer:1
   fauxpilot-triton-1         | I0803 01:51:02.936798 93 libfastertransformer.cc:1226] TRITONBACKEND_Initialize: fastertransformer
   fauxpilot-triton-1         | I0803 01:51:02.936818 93 libfastertransformer.cc:1236] Triton TRITONBACKEND API version: 1.10
   fauxpilot-triton-1         | I0803 01:51:02.936821 93 libfastertransformer.cc:1242] 'fastertransformer' TRITONBACKEND API version: 1.10
   fauxpilot-triton-1         | I0803 01:51:02.936850 93 libfastertransformer.cc:1274] TRITONBACKEND_ModelInitialize: fastertransformer (version 1)
   fauxpilot-triton-1         | W0803 01:51:02.937855 93 libfastertransformer.cc:149] model configuration:
   fauxpilot-triton-1         | {
   [... lots more output trimmed ...]
   fauxpilot-triton-1         | I0803 01:51:04.711929 93 libfastertransformer.cc:321] After Loading Model:
   fauxpilot-triton-1         | I0803 01:51:04.712427 93 libfastertransformer.cc:537] Model instance is created on GPU NVIDIA RTX A6000
   fauxpilot-triton-1         | I0803 01:51:04.712694 93 model_repository_manager.cc:1345] successfully loaded 'fastertransformer' version 1
   fauxpilot-triton-1         | I0803 01:51:04.712841 93 server.cc:556] 
   fauxpilot-triton-1         | +------------------+------+
   fauxpilot-triton-1         | | Repository Agent | Path |
   fauxpilot-triton-1         | +------------------+------+
   fauxpilot-triton-1         | +------------------+------+
   fauxpilot-triton-1         | 
   fauxpilot-triton-1         | I0803 01:51:04.712916 93 server.cc:583] 
   fauxpilot-triton-1         | +-------------------+-----------------------------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------+
   fauxpilot-triton-1         | | Backend           | Path                                                                        | Config                                                                                                                                                         |
   fauxpilot-triton-1         | +-------------------+-----------------------------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------+
   fauxpilot-triton-1         | | fastertransformer | /opt/tritonserver/backends/fastertransformer/libtriton_fastertransformer.so | {"cmdline":{"auto-complete-config":"false","min-compute-capability":"6.000000","backend-directory":"/opt/tritonserver/backends","default-max-batch-size":"4"}} |
   fauxpilot-triton-1         | +-------------------+-----------------------------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------+
   fauxpilot-triton-1         | 
   fauxpilot-triton-1         | I0803 01:51:04.712959 93 server.cc:626] 
   fauxpilot-triton-1         | +-------------------+---------+--------+
   fauxpilot-triton-1         | | Model             | Version | Status |
   fauxpilot-triton-1         | +-------------------+---------+--------+
   fauxpilot-triton-1         | | fastertransformer | 1       | READY  |
   fauxpilot-triton-1         | +-------------------+---------+--------+
   fauxpilot-triton-1         | 
   fauxpilot-triton-1         | I0803 01:51:04.738989 93 metrics.cc:650] Collecting metrics for GPU 0: NVIDIA RTX A6000
   fauxpilot-triton-1         | I0803 01:51:04.739373 93 tritonserver.cc:2159] 
   fauxpilot-triton-1         | +----------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
   fauxpilot-triton-1         | | Option                           | Value                                                                                                                                                                                        |
   fauxpilot-triton-1         | +----------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
   fauxpilot-triton-1         | | server_id                        | triton                                                                                                                                                                                       |
   fauxpilot-triton-1         | | server_version                   | 2.23.0                                                                                                                                                                                       |
   fauxpilot-triton-1         | | server_extensions                | classification sequence model_repository model_repository(unload_dependents) schedule_policy model_configuration system_shared_memory cuda_shared_memory binary_tensor_data statistics trace |
   fauxpilot-triton-1         | | model_repository_path[0]         | /model                                                                                                                                                                                       |
   fauxpilot-triton-1         | | model_control_mode               | MODE_NONE                                                                                                                                                                                    |
   fauxpilot-triton-1         | | strict_model_config              | 1                                                                                                                                                                                            |
   fauxpilot-triton-1         | | rate_limit                       | OFF                                                                                                                                                                                          |
   fauxpilot-triton-1         | | pinned_memory_pool_byte_size     | 268435456                                                                                                                                                                                    |
   fauxpilot-triton-1         | | cuda_memory_pool_byte_size{0}    | 67108864                                                                                                                                                                                     |
   fauxpilot-triton-1         | | response_cache_byte_size         | 0                                                                                                                                                                                            |
   fauxpilot-triton-1         | | min_supported_compute_capability | 6.0                                                                                                                                                                                          |
   fauxpilot-triton-1         | | strict_readiness                 | 1                                                                                                                                                                                            |
   fauxpilot-triton-1         | | exit_timeout                     | 30                                                                                                                                                                                           |
   fauxpilot-triton-1         | +----------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
   fauxpilot-triton-1         | 
   fauxpilot-triton-1         | I0803 01:51:04.740423 93 grpc_server.cc:4587] Started GRPCInferenceService at 0.0.0.0:8001
   fauxpilot-triton-1         | I0803 01:51:04.740608 93 http_server.cc:3303] Started HTTPService at 0.0.0.0:8000
   fauxpilot-triton-1         | I0803 01:51:04.781561 93 http_server.cc:178] Started Metrics Service at 0.0.0.0:8002
   ```

   运行成功后最后会有服务开启的提示。
3. 创建交互API

这里需要安装openai的包。

```ipython
$ ipython
Python 3.8.10 (default, Mar 15 2022, 12:22:08) 
Type 'copyright', 'credits' or 'license' for more information
IPython 8.2.0 -- An enhanced Interactive Python. Type '?' for help.

In [1]: import openai

In [2]: openai.api_key = 'dummy'

In [3]: openai.api_base = 'http://127.0.0.1:5000/v1'

In [4]: result = openai.Completion.create(engine='codegen', prompt='def hello', max_tokens=16, temperature=0.1, stop=["\n\n"])

In [5]: result
Out[5]: 
<OpenAIObject text_completion id=cmpl-6hqu8Rcaq25078IHNJNVooU4xLY6w at 0x7f602c3d2f40> JSON: {
  "choices": [
    {
      "finish_reason": "stop",
      "index": 0,
      "logprobs": null,
      "text": "() {\n    return \"Hello, World!\";\n}"
    }
  ],
  "created": 1659492191,
  "id": "cmpl-6hqu8Rcaq25078IHNJNVooU4xLY6w",
  "model": "codegen",
  "object": "text_completion",
  "usage": {
    "completion_tokens": 15,
    "prompt_tokens": 2,
    "total_tokens": 17
  }
}

```

4. 配置copilot插件
   修改vscode配置文件，在全局或者项目中打开 ``setting.json``，编辑如下内容即可：

```json
    "github.copilot.advanced": {
        "debug.overrideEngine": "codegen",
        "debug.testOverrideProxyUrl": "http://localhost:5000",
        "debug.overrideProxyUrl": "http://localhost:5000"
    }
```

localhost改成具体的ip地址即可在内网使用。
