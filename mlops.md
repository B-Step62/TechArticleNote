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
