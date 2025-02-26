# ICLR 2025: Streamlining Redundant Layers to Compress Large Language Models

#### Open Source Models:
We have released two open source models, [Llama-2-4.7B](https://huggingface.co/XiaodongChen/Llama-2-4.7B) and [Llama-3.1-5.4B](https://huggingface.co/XiaodongChen/Llama-3.1-5.4B) on Hugging Face.
Below are the evaluation results on lm-eval:
|                | arc_c | arc_e | boolq | hellaswag | openbookqa | rte  | winogrande | Avg  |
|----------------|-------|-------|-------|-----------|------------|------|------------|------|
| Llama-3.1-8B   | 50.4  | 80.3  | 81.2  | 60.2      | 34.8       | 67.9 | 73.0       | 64.0 |
| Llama-3.1-5.4B | 43.3  | 74.8  | 80.7  | 55.8      | 32.2       | 69.3 | 73.0       | 61.3 |
|----------------|-------|-------|-------|-----------|------------|------|------------|------|
| Llama-2-7b   | 43.3  | 76.4  | 77.7  | 57.2      | 31.4       | 62.8 | 69.1       | 59.7 |
| Llama-2-4.7b | 34.0  | 64.6  | 74.7  | 49.8      | 27.4       | 61.7 | 66.4       | 54.1 |

#### Supported LLMs:
-  [Llama-3 Hugging Face]
-  [Llama-2 Hugging Face]
-  [OPT Hugging Face]

## Requirements
To install requirements:

```setup
pip install -r requirements.txt
```

## Pruning 
Our code focuses on using Transformer layers as the lightweight network. The parameter weights of the first pruned layer are inherited for training, as this approach produces better results compared to using FFN or SwiGLU.

To train the lightweight network using MSE loss, execute:
```
python mseloss_entry.py
```
This training process will be executed on a single GPU. By default, Llama3.1-8B will be pruned and 8 layers will be removed from the model. All the pre-trained models and the dataset will be automatically downloaded, so you do not need to manually download the resource. When running it for the first time, it will require some time to download the model and the dataset. Please ensure that there is sufficient memory available, as all hidden states will be stored in memory. If memory is insufficient, you may modify the code to store the hidden states on the disk or utilize LLM loss for training.

To train the lightweight network using LLM loss under the Accelerate and DeepSpeed frameworks, execute:
```
CUDA_VISIBLE_DEVICES=0,1,2,3 accelerate launch --config_file 4gpu.yaml llmloss_entry.py
```
This training process will be executed on 4 GPUs. Compared to mse loss, this will require more GPU memory.

We recommend using MSE Loss for training when GPU resources are limited, but when GPU resources are sufficient, using LLM loss will yield better results.

### Arguments
You can change all the Arguments in the LLM_Streamline/args.py file.
- ``model_name``: Path to pretrained model or model identifier from huggingface.co/models
- ``layer_intervals``: Number of layers to prune.
- ``cosine_num_data``: Amount of data used to calculate cosine similarity.
- ``train_num_data``: Amount of data used to train the lightweight model.
- ``batch_size``: Batch size for training.
- ``gradient_accumulation_step``: Number of gradient accumulation steps during training. The effective batch size is the product of gradient_accumulation_step and batch_size.
- ``epoches``: Number of training epochs.
- ``lr``: Learning rate for training.
- ``min_lr``: Minimum learning rate during training.

## Calculate Stability
To calculate stability, execute:
```
python calculate_stability.py arg1 arg2
```
Here, arg1 refers to the model's evaluation predictions before pruning, and arg2 refers to the predictions after pruning. Both predictions are generated by OpenCompass.

For example:

arg1: "./opencompass/outputs/default/20241121_220629/predictions/llama-3-70b-hf"

arg2: "./opencompass/outputs/default/20241123_220629/predictions/llama-3-70b-hf"
