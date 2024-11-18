# 🧪 Evalchemy

> *A framework for gold standard language model evaluations*

![alt text](https://github.com/mlfoundations/evalchemy/blob/main/image.png)

Evalchemy is a unified and easy-to-use toolkit for evaluating language models, focussing on post-trained models. Evalchemy is developed by the [DataComp community](https://datacomp.ai) and [Bespoke Labs](https://bespokelabs.ai)  and builds on the [LM-Eval-Harness](https://github.com/EleutherAI/lm-evaluation-harness) to provide a unified, easy-to-use platform for language model evaluation. Evalchemy integrates multiple existing benchmarks, such as RepoBench, AlpacaEval, and ZeroEval. We've streamlined the process by:

### Key Features

- **Unified Installation**: One-step setup for all benchmarks, eliminating dependency conflicts
- **Parallel Evaluation**:
  - Data-Parallel: Distribute evaluations across multiple GPUs for faster results
  - Model-Parallel: Handle large models that don't fit on a single GPU
- **Simplified Usage**: Run any benchmark with a consistent command-line interface
- **Results Management**: 
  - Local results tracking with standardized output format
  - Optional database integration for systematic tracking
  - Leaderboard submission capability (requires database setup)

## ⚡ Quick Start

### Installation

We suggest using conda ([installation instructions](https://docs.anaconda.com/miniconda/#quick-command-line-install)). 

```bash
# Create and activate conda environment
conda create --name evalchemy python=3.10
conda activate evalchemy      

# Install dependencies
pip install -e ".[eval]"
pip install -e eval/chat_benchmarks/alpaca_eval

# Log into HuggingFace for datasets and models.
huggingface-cli login
```

## 📚 Available Tasks

### Built-in Benchmarks
- All tasks from [LM Evaluation Harness](https://github.com/EleutherAI/lm-evaluation-harness)
- Custom instruction-based tasks (found in `eval/chat_benchmarks/`):
  - **MTBench**: [Multi-turn dialogue evaluation benchmark](https://github.com/mtbench101/mt-bench-101)
  - **WildBench**: [Real-world task evaluation](https://github.com/allenai/WildBench)
  - **RepoBench**: [Code understanding and repository-level tasks](https://github.com/Leolty/repobench)
  - **MixEval**: [Comprehensive evaluation across domains](https://github.com/Psycoy/MixEval)
  - **IFEval**: [Instruction following capability evaluation](https://github.com/google-research/google-research/tree/master/instruction_following_eval)
  - **AlpacaEval**: [Instruction following evaluation](https://github.com/tatsu-lab/alpaca_eval)
  - **HumanEval**: [Code generation and problem solving](https://github.com/openai/human-eval)
  - **ZeroEval**: [Logical reasoning and problem solving](https://github.com/WildEval/ZeroEval)
  - **MBPP**: [Python programming benchmark](https://github.com/google-research/google-research/tree/master/mbpp)
  - **Arena-Hard-Auto** (Coming soon): [Automatic evaluation tool for instruction-tuned LLMs](https://github.com/lmarena/arena-hard-auto)
  - **SWE-Bench** (Coming soon): [Evaluating large language models on real-world software issues](https://github.com/princeton-nlp/SWE-bench)
  - **SafetyBench** (Coming soon): [Evaluating the safety of LLMs](https://github.com/thu-coai/SafetyBench)
  - **Berkeley Function Calling Leaderboard** (Coming soon): [Evaluating ability of LLMs to use APIs](https://gorilla.cs.berkeley.edu/blogs/13_bfcl_v3_multi_turn.html)


### Basic Usage

Make sure your `OPENAI_API_KEY` is set in your environment before running evaluations.

```bash
python -m eval.eval \
    --model hf \
    --tasks HumanEval,mmlu \
    --model_args "pretrained=mistralai/Mistral-7B-Instruct-v0.3" \
    --batch_size 2 \
    --output_path logs
```

The results will be written out in `output_path`. If you have `jq` [installed](https://jqlang.github.io/jq/download/), you can view the results easily after evaluation. Example: `jq '.results' logs/Qwen__Qwen2.5-7B-Instruct/results_2024-11-17T17-12-28.668908.json`

**Args**: 

- `--model`: Which model type or provider is evaluated (example: hf)
- `--tasks`: Comma-separated list of tasks to be evaluated.
- `--model_args`: Model path and parameters. Comma-separated list of parameters passed to the model constructor. Accepts a string of the format `"arg1=val1,arg2=val2,..."`. You can find the list supported arguments [here](https://github.com/EleutherAI/lm-evaluation-harness/blob/365fcda9b85bbb6e0572d91976b8daf409164500/lm_eval/models/huggingface.py#L66).
- `--batch_size`: Batch size for inference
- `--output_path`: Directory to save evaluation results

Example running multiple benchmarks:
```bash
python -m eval.eval \
    --model hf \
    --tasks MTBench,WildBench,alpaca_eval \
    --model_args "pretrained=mistralai/Mistral-7B-Instruct-v0.3" \
    --batch_size 2 \
    --output_path logs
```

We add several more command examples in [`eval/examples`](https://github.com/mlfoundations/Evalchemy/tree/main/eval/examples) to help you start using Evalchemy. 

## 🔧 Advanced Usage

### Support for different models

Through LM-Eval-Harness, we support all HuggingFace models and are currently adding support for all LM-Eval-Harness models, such as OpenAI and VLLM. For more information on such models, please check out the [models page](https://github.com/EleutherAI/lm-evaluation-harness/tree/main/lm_eval/models).

To choose a model, simply set 'pretrained=<name of hf model>' where the model name can either be a HuggingFace model name or a path to a local model. 

#### Coming Soon
- Support for [vLLM models](https://vllm.ai/)
- Support for [OpenAI](https://openai.com/)

### Multi-GPU Evaluation

For faster evaluation using data parallelism (recommended):

```bash
accelerate launch --num-processes <num-gpus> --num-machines <num-nodes> \
    --multi-gpu -m eval.eval \
    --model hf \
    --tasks MTBench,alpaca_eval \
    --model_args 'pretrained=mistralai/Mistral-7B-Instruct-v0.3' \
    --batch_size 2 \
    --output_path logs
```

### Large Model Evaluation

For models that don't fit on a single GPU, use model parallelism:

```bash
python -m eval.eval \
    --model hf \
    --tasks MTBench,alpaca_eval \
    --model_args 'pretrained=mistralai/Mistral-7B-Instruct-v0.3,parallelize=True' \
    --batch_size 2 \
    --output_path logs
```

> **💡 Note**: While "auto" batch size is supported, we recommend manually tuning the batch size for optimal performance. The optimal batch size depends on the model size, GPU memory, and the specific benchmark. We used a maximum of 32 and a minimum of 4 (for RepoBench) to evaluate Llama-3-8B-Instruct on 8xH100 GPUs.

### Output Log Structure

Our generated logs include critical information about each evaluation to help inform your experiments. We highlight important items in our generated logs. 

- Model Configuration
  - `model`: Model framework used
  - `model_args`: Model arguments for the model framework
  - `batch_size`: Size of processing batches
  - `device`: Computing device specification
  - `annotator_model`: Model used for annotation ("gpt-4o-mini-2024-07-18")
- Seed Configuration
  - `random_seed`: General random seed
  - `numpy_seed`: NumPy-specific seed
  - `torch_seed`: PyTorch-specific seed
  - `fewshot_seed`: Seed for few-shot examples
- Model Details
  - `model_num_parameters`: Number of model parameters
  - `model_dtype`: Model data type
  - `model_revision`: Model version
  - `model_sha`: Model commit hash

- Version Control
  - `git_hash`: Repository commit hash
  - `date`: Unix timestamp of evaluation
  - `transformers_version`: Hugging Face Transformers version
- Tokenizer Configuration
  - `tokenizer_pad_token`: Padding token details
  - `tokenizer_eos_token`: End of sequence token
  - `tokenizer_bos_token`: Beginning of sequence token
  - `eot_token_id`: End of text token ID
  - `max_length`: Maximum sequence length
- Model Settings
  - `model_source`: Model source platform
  - `model_name`: Full model identifier
  - `model_name_sanitized`: Sanitized model name for file system usage
  - `chat_template`: Conversation template
  - `chat_template_sha`: Template hash
- Timing Information
  - `start_time`: Evaluation start timestamp
  - `end_time`: Evaluation end timestamp
  - `total_evaluation_time_seconds`: Total duration
- Hardware Environment
  - PyTorch version and build configuration
  - Operating system details
  - GPU configuration
  - CPU specifications
  - CUDA and driver versions
  - Relevant library versions

### Customizing Evaluation

#### 🤖 Change Annotator Model

As part of Evalchemy, we want to make swapping in different Language Model Judges for standard benchmarks easy. Currently, we support two judge settings. The first is the default setting, where we use a benchmark's default judge. To activate this, you can either do nothing or pass in
```bash
--annotator_model auto
```
In addition to the default assignments, we support using gpt-4o-mini-2024-07-18 as a judge:

```bash
--annotator_model gpt-4o-mini-2024-07-18
```

We are planning on adding support for different judges in the future!

### ⏱️ Runtime and Cost Analysis

Evalchemy makes running common benchmarks simple, fast, and versatile! We list the speeds and costs for each benchmark we achieve with Evalchemy for Meta-Llama-3-8B-Instruct on 8xH100 GPUs.

| Benchmark | Runtime (8xH100) | Batch Size | Total Tokens | Default Judge Cost ($) | GPT-4o-mini Judge Cost ($) | Notes |
|-----------|------------------|------------|--------------|----------------|-------------------|--------|
| MTBench | 14:00 | 32 | ~196K | 6.40 | 0.05 | |
| WildBench | 38:00 | 32 | ~2.2M | 30.00 | 0.43 | Using GPT-4-mini judge |
| RepoBench | 46:00 | 4 | - | - | - | Lower batch size due to memory |
| MixEval | 13:00 | 32 | ~4-6M | 3.36 | 0.76 | Varies by judge model |
| AlpacaEval | 16:00 | 32 | ~936K | 9.40 | 0.14 | |
| HumanEval | 4:00 | 32 | - | - | - | No API costs |
| IFEval | 1:30 | 32 | - | - | - | No API costs |
| ZeroEval | 1:44:00 | 32 | - | - | - | Longest runtime |
| MBPP | 6:00 | 32 | - | - | - | No API costs |
| MMLU | 7:00 | 32 | - | - | - | No API costs |
| ARC | 4:00 | 32 | - | - | - | No API costs |
| DROP | 20:00 | 32 | - | - | - | No API costs |

**Notes:**
- Runtimes measured using 8x H100 GPUs with Meta-Llama-3-8B-Instruct model
- Batch sizes optimized for memory and speed
- API costs vary based on judge model choice

**Cost-Saving Tips:**
- Use gpt-4o-mini-2024-07-18 judge when possible for significant cost savings
- Adjust batch size based on available memory
- Consider using data-parallel evaluation for faster results

### 🔐 Special Access Requirements

#### ZeroEval Access
To run ZeroEval benchmarks, you need to:

1. Request access to the [ZebraLogicBench-private dataset](https://huggingface.co/datasets/allenai/ZebraLogicBench-private) on Hugging Face
2. Accept the terms and conditions
3. Log in to your Hugging Face account when running evaluations

## 🛠️ Implementing Custom Evaluations

To add a new evaluation system:

1. Create a new directory under `eval/chat_benchmarks/`
2. Implement `eval_instruct.py` with two required functions:
   - `eval_instruct(model)`: Takes an LM Eval Model, returns results dict
   - `evaluate(results)`: Takes results dictionary, returns evaluation metrics

### Adding External Evaluation Repositories

Use git subtree to manage external evaluation code:

```bash
# Add external repository
git subtree add --prefix=eval/chat_benchmarks/new_eval https://github.com/original/repo.git main --squash

# Pull updates
git subtree pull --prefix=eval/chat_benchmarks/new_eval https://github.com/original/repo.git main --squash

# Push contributions back
git subtree push --prefix=eval/chat_benchmarks/new_eval https://github.com/original/repo.git contribution-branch
```

### 🔍 Debug Mode

To run evaluations in debug mode, add the `--debug` flag:

```bash
python -m eval.eval \
    --model hf \
    --tasks MTBench \
    --model_args "pretrained=mistralai/Mistral-7B-Instruct-v0.3" \
    --batch_size 2 \
    --output_path logs \
    --debug
```

This is particularly useful when testing new evaluation implementations, debugging model configurations, verifying dataset access, and testing database connectivity.

### 🚀 Performance Tips

1. Utilize batch processing for faster evaluation:
```python
all_instances.append(
    Instance(
        "generate_until",
        example,
        (
            inputs,
            {
                "max_new_tokens": 1024,
                "do_sample": False,
            },
        ),
        idx,
    )
)

outputs = self.compute(model, all_instances)
```

2. Use the LM-eval logger for consistent logging across evaluations

### 🔧 Troubleshooting
Evalchemy has been tested on CUDA 12.4. If you run into issues like this: `undefined symbol: __nvJitLinkComplete_12_4, version libnvJitLink.so.12`, try updating your CUDA version:
```bash
wget https://developer.download.nvidia.com/compute/cuda/repos/debian11/x86_64/cuda-keyring_1.1-1_all.deb
sudo dpkg -i cuda-keyring_1.1-1_all.deb
sudo add-apt-repository contrib
sudo apt-get update
sudo apt-get -y install cuda-toolkit-12-4
```

## 🏆 Leaderboard Integration
To track experiments and evaluations, we support logging results to a PostgreSQL database. Details on the entry schemas and database setup can be found in the [database](./database/) directory.