## [Deploying Large Language Models in Production: LLMOps with MLflow](https://www.analyticsvidhya.com/blog/2023/05/deploying-large-language-models-in-production-llmops-with-mlflow/)
- Challanges in LLM are (1) compute resource management (2) model performance (3) model versioning (4) infrastructure.
- MLflow can track models in various library, kinda nice it has unified abostracted interface
  - Huggingface: `mlflow.transformers.log_model(transformers_model=hf_pipeline, ...)`
  - OpenAI (custom python wrapper for API) `mlflow.pyfunc.log_model(python_model=custom_wrapper, ...)`
  - LangChain: `mlflow.langchain.log_model(lc_model=llm_chain, ...)`
- Also can define inference as spark UDF, fancy
```
english_to_french_udf = mlflow.pyfunc.spark_udf(
    spark=spark,
    model_uri="models:/english-to-french-chain-gpt-3.5-turbo-1/1",
    result_type="string"
)
```
## [深層モデルの高速化](https://speakerdeck.com/joisino/shen-ceng-moderunogao-su-hua?slide=25) (Faster inference of deep learning model)
- **Quantitization** - downcasting float into integer like int32, usually with scaling factor e.g. `W = 0.1 x W_int`. Main reasons for why it makes inference faster. 
  1. reduce RAM usage per sample so increase throughput.
  2. [CPU] SIMD - e.g. a single register has 256 ~512 bit, so it can do 32-64 int8 operation at once.
  3. [GPU] Tensor Core - similar to SIMD but for matrix operation.
  - int8 or FP16 (low precision float) is usually good start to get speed w/o major perf regression.
- **Distillation** - Train smaller model with large model's supervision. 
- **Model pruning** - usually done by cutting weight with small magnitude. Better to do finetuning after pruning. One caveat is that [some work](https://arxiv.org/abs/1810.05270) denotes training pruned model from scratch performs better or same.
- **Low rank approximation** - for attention model, approximation to Gram matrix is also used.
- FLOPs is often used for evaluating those  methods, but it's not an accurate indicator for modern hardware architecture. Should rather **measure  actual inference time**. 

## [7 Frameworks for Serving LLMs](https://betterprogramming.pub/frameworks-for-serving-llms-60b7f7b23407)
![](https://miro.medium.com/v2/resize:fit:2000/format:webp/1*Yym4eiSJn7fOQSXnAt-KYg.png)
- Speed: [vLLM](https://github.com/vllm-project/vllm) > [CTranslate2](https://github.com/OpenNMT/CTranslate2) > [DeepSpeed-MII](https://github.com/microsoft/DeepSpeed-MII) >>> Text generation inference >>> OpenLLM >= Ray Serve >= MLC LLM
- **vLLM**
  - [Pros] The fastest option thanks to its original optimization called PagedAttention. It stores keys and values in non-contiguous *block*, which allows more flexible and efficient memory mapping. The idea was inspired by memory virtualization and paging in OS, with analogy of blocks = pages, tokens = bytes, and sequences = processes.
  - [Cons] Since it needs architecture level optimization, adding custom models or adaptors e.g. LoRA are challenging. Also, no weight quantization. (tho both are on [roadmap](https://github.com/vllm-project/vllm/issues/244))
- **Text generation ingerence (by huggingface)**
  - [Pros] Native support for huggingface models. Allows granular control for inference e.g. precision adjustment, quantization, etc.
  - [Cons] Written in Rust (if unfamiliar). Incomplete documentation.
- **CTranslates**
  - [Pros] Efficient in both CPU and GPU. Multiple CPU arthicetcure support. Prompt caching (system prompt).
  - [Cons] Need to run REST server manually. No adaptor support.
- **DeepSpeed-MII (by Microsoft)**
  - [Pros] Built-in load balancing over replicas. Native Azure integration.
  - [Cons] Still in beta. Limited number of suppreted models e.g. LLaMA2.
- **OpenLLM**
  - [Pros] Adaptor support. Multiple runtime support such as Pytorch, Tensorflow, Flax. LangChain integration.
  - [Cons] No batch inference. Lack of built-in distributed inference.
- **Ray Serve**
  - [Pros] String monitoring. Autoscale support. Dynamic batching. Stable.
  - [Cons] No built-in LLM optimization. Steep learning curve.
- **MLC LLM**
  - [Pros] Platform-native runtimes. Suitable for native App - iOs or Anddroid. Memory optimization.
  - [Cons] Limited functionaliry, complicated installations.
- Ray Serve for a stable pipeline and flexible deployment.
- MLC LLM for edge computing.
