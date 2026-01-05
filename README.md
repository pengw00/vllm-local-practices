# vllm-local-practices
local test
Step1 
```
!git clone https://github.com/water685/vllm-dynamic-sparsity.git
%cd vllm-dynamic-sparsity
```

Step2
```
%cd vllm-dynamic-sparsity
```
Step3 install local version and skip the kernel compile(Wheel file)
```
!VLLM_USE_PRECOMPILED=1 pip install -e .
```


###Setup github password
```
xxxxx
```

```
!git remote set-url origin https://<TOKEN>@github.com
```

你说得对！EngineCoreClient.make_client 是加载模型的关键入口。让我帮你找到这个文件并在其中添加详细的日志，追踪模型加载的具体过程。

首先，让我在 EngineCoreClient 相关文件中添加日志，展示模型加载的详细过程：

core_client.py+26-16
现在已经在 EngineCoreClient.make_client 方法中添加了详细的日志。根据代码分析，模型加载的关键过程如下：

模型加载的详细流程分析
1. EngineCoreClient.make_client（模型加载的入口）
这个方法是模型加载的关键入口点，具体执行以下步骤：
executor = executor_class(vllm_config)  # 这里是模型权重实际加载的地方
2. Executor 初始化（实际加载模型权重）
当 executor_class(vllm_config) 被调用时，会发生以下事情：

选择 Executor 类型：根据配置选择合适的 Executor（如 GPUExecutor、TPUExecutor 等）
初始化 Worker：创建 worker 实例来处理模型推理
加载模型权重：
从 HuggingFace 或本地路径读取模型文件
将权重加载到 GPU/TPU 内存
如果是分布式部署，会进行权重分片（tensor parallelism）
应用量化（如果配置了 quantization）
3. 模型加载的核心步骤
LLM.__init__
  └─> LLMEngine.from_engine_args
      └─> LLMEngine.__init__
          └─> EngineCoreClient.make_client  ← 你在这里！
              └─> executor_class(vllm_config)
                  └─> Executor.__init__
                      └─> Worker.init_model()
                          └─> load_weights_from_hf/local
                              └─> torch.load() / from_pretrained()


4. 运行时日志输出示例
当你运行程序时，会看到类似以下的日志：
INFO: Initializing LLM class...
INFO: Model path: /path/to/model
INFO: Creating EngineArgs object...
INFO: === Starting LLMEngine initialization ===
INFO: === Starting LLMEngine.from_engine_args ===
INFO: Creating engine configuration from EngineArgs...
INFO: Executor class resolved: GPUExecutor
INFO: Creating EngineCoreClient...
INFO: === EngineCoreClient.make_client called ===
INFO: Multiprocess mode: False
INFO: Executor class: GPUExecutor
INFO: Initializing Executor (model loading starts here)...
INFO: Loading model weights from /path/to/model...
INFO: Model weights loaded to GPU:0
INFO: Executor initialized (model loading completed).
INFO: === EngineCoreClient.make_client completed ===
INFO: === LLMEngine initialization completed ===
这样你就能清楚地看到模型在哪一步被加载，以及加载过程中的详细信息。如果你需要进一步追踪 Executor 内部的模型加载细节，请告诉我，我可以在 Executor 的实现中添加更多日志。这样你就能清楚地看到模型在哪一步被加载，以及加载过程中的详细信息。如果你需要进一步追踪 Executor 内部的模型加载细节，请告诉我，我可以在 Executor 的实现中添加更多日志。
