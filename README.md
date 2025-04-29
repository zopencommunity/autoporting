# **AUTO-PORTING TOOL**
***
## **Initial Data Preparation**
The initial step involves utilizing the repositories of tools that have already been ported for the ZOs Operating System. Using the ZOs environment, we will proceed by removing all patch codes, then rebuild the tool. Subsequently, patches will be added one by one, and errors encountered during each addition will be captured for further analysis.

The dataset is structured in the following format:
```json
{
  "wrong_code": "",
  "error_category": "",
  "error": "",
  "correct_code": "",
  "patch": ""
}
```

* wrong_code: The code segment that caused the error.
* error_category: The category or type of the error.
* error: The specific error message or description encountered.
* correct_code: The modified or correct code that resolves the error.
* patch: The patch applied to the code.
  
for each and every error the above format is followed and the data is captured.
This format ensures clear tracking of errors, their categories, and the corresponding fixes.
***
## **Selecting the Appropriate Model**
* To install the Instruct Lab, ensure that **Python 3.9.20** is used, as this version is compatible for hosting the external model.
  Please go through the documentation of Instruct Lab for the installation [Instruct Lab Documentation](https://docs.instructlab.ai/)
* The selected model, **llama-3.2-1b-q4_k_m**, is a multilingual large language model developed by Meta, featuring 1 billion parameters. It is part of the LLaMA 3.2 collection, which includes models designed for efficient performance on mobile and edge devices. The 1B model is optimized for multilingual text generation and tool-calling capabilities, supporting languages such as English, German, French, Italian, Portuguese, Hindi, Spanish, and Thai. Trained on a diverse mix of publicly available online data, it is suitable for tasks like agentic retrieval and summarization. The model has a context length of up to 128,000 tokens, allowing it to handle long-context scenarios effectively.Please visit the [website](https://huggingface.co/medmekk/Llama-3.2-1B-Q4_K_M-GGUF) for further information on the model used.
***
## **Synthetic Data Generation**
### **Changing the format of qna.yaml file**
The base qna.yaml file requires five seed examples and three question-and-answer pairs under each context.
This format can be modified by adjusting the corresponding values in the schema file located at:
instructlab/schema/v3/knowledge.json.

Currently, the structure includes only one context, under which there are three question-and-answer pairs.

It is **important** to note that the number of question-and-answer pairs cannot be reduced to two or one. This is because the SDG process is designed to generate chunks of three question-and-answer pairs at a time, which are subsequently used for training.Therefore, maintaining this structure is essential for ensuring compatibility with the training pipeline. If not followed you might run into the following type of error:
```
failed to generate data with exception: 'icl_query_2'
```

***Note:*** The context must include content directly extracted from the codebase used for training. Additionally, the question-and-answer pairs must be strictly based on this context to ensure alignment with the training data.

### ***Technical Changes***
These technical specifications are provided based on the **SDG process conducted on my local device (MACBOOK M3 PRO)**.

**Modifications Made:**	
1. Context Size Adjustment:
- The `context_size` in both the **generate** and **serve** sections of the `config.yaml` file was changed **from 4096 to 100000**.
- This adjustment was made to **increase the available context window** while ensuring compatibility with my device’s **memory constraints**.

2. Reason for `100000` Instead of `131072`:
- Although the model supports a **maximum context size of 131072**, my device’s **RAM was insufficient** to handle it efficiently.
- Setting `n_ctx` to **100000** allows for **optimal performance** without exceeding memory limits. If the allocated context size **exceeds the system’s available memory**, the process may encounter a RuntimeError: llama_decode returned 1 the consider reducing the size less than 100000

3. Modifying LLaMA Configuration  

  To ensure the model utilizes a larger context window, update the configuration as follows:  

- Navigate to the **`instructlab/models/backends/llama_cpp.py`** file.  
- Modify the `n_ctx` parameter, setting its value to **100000**.  

 **Reason for This Modification**  
By default, `llama_cpp_python` is configured to use a **context size of 10016** to prevent system instability. However, this limitation can lead to errors when attempting to process larger inputs. If the `n_ctx` value remains at 10016, you may encounter the following error:  
```
failed to generate data with exception: PipelineBlockError(<class ‘instructlab.sdg.blocks.llmblock.LLMBlock’>/gen_knowledge):
Error code: 400 - {‘error’: {‘message’: “This model’s maximum context length is 10016 tokens, however you requested 12920 tokens (10872 in your prompt; 2048 for the completion).
Please reduce your prompt; or completion length.”, ‘type’: ‘invalid_request_error’, ‘param’: ‘messages’, ‘code’: ‘context_length_exceeded’}}
```
4. Ensuring Proper Batch Size for Data Generation
- The **batch size** must be set to **512** during data generation.
- Failure to do so may result in NumPy conversion errors or dimension mismatches during processing.
- If the batch size is not set to **512** or greater you may encounter the following error:
```
ValueError: could not broadcast input array from shape (128256,) into shape (0,)
```
For you reference you can view the [qna.yaml](main/qna.yaml) used.
