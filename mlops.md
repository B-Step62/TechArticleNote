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
