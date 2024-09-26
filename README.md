# Archon: An Architecture Search Framework for Inference-Time Techniques
Inference-time techniques allow us to bolster the strengths of existing LMs by utilizing multiple sample calls and multiple LMs to increase system performance for a given task.
Archon provides a modular framework for combining different inference-time techniques and LMs with just a JSON config file.

![Archon Overview Diagram](readme_assets/archon_itas_diagram.svg)


## Table of Contents
- [Installation](#installation)
    - [Python Package](#python-package)
    - [Running Locally](#running-locally)
- [Archon Overview](#archon-overview)
- [QuickStart](#QuickStart)
- [gen_answers.py and Benchmarks](#gen_answerspy-and-benchmarks)
    - [Add Your Own Benchmark](#add-your-own-benchmark)
- [Key Handling](#key-handling)

### Running Locally

Alternatively, you can use Archon directly from this repository. This is the most up to date and offers the most flexibility working with our codebase

```shell
git submodule init
git submodule update
```
  
We recommend using our environment that can be initialized with:
```
conda env create -f archon_env.yml
conda activate archon_env
pip install -r requirements.txt
````

We recommend you work within the `archon/` directory.
Archon can be instantiated via the Archon class in `archon.py`. Get started by cloning the repository or creating your own starter file that imports our Archon class as shown below: 
```python
from archon import Archon
```
## Archon Overview

At the moment, we provide support for these inference times techniques: `generator`, `fuser`, `critic`, `ranker`, `verifier`, `unit_test_generator`, `unit_test_evaluator`. Their prompts can be found [here](archon/components/prompts.py). As well as the ability to [add your own components](#tutorialsexamples). 

We also provide these API Access points that can be used in Archon configurations: `Together_API`, `OpenAI_API`, `Anthropic_API`, `Groq_API`, `Google_API`, `tgi`, `Bedrock_API`. You can also [add your own generators/endpoints](#tutorialsexamples).

## QuickStart

Archon works by taking in a config file in JSON format that specifies the architecture you want to run and its available parameters. 
Say I want to ask a compound GPT 4o system a question and output a singular response. We want to sample gpt-4o 10 times, rank the top 5 responses, and then fuse for a final response. We can create a config that looks like this:
```
archon_gpt_config = {
    "name": "archon-gpt-multi-model",
    "layers": [
        [
            {
                "type": "generator",
                "model": "gpt-4o",
                "model_type": "OpenAI_API",
                "top_k": 1,
                "temperature": 0.7,
                "max_tokens": 2048,
                "samples": 10
            }
        ],
        [
            {
                "type": "ranker",
                "model": "gpt-4o",
                "model_type": "OpenAI_API",
                "top_k": 5,
                "temperature": 0.7,
                "max_tokens": 2048,
            }
        ],
        [
            {
                "type": "fuser",
                "model": "gpt-4o",
                "model_type": "OpenAI_API",
                "temperature": 0.7,
                "max_tokens": 2048,
                "samples": 1
            }
        ]
    ]
}
```
To generate a response:
```python
# archon_config can also be stored and read from a file. Then passed to Archon()
archon = Archon(archon_gpt_config)

testing_instruction = [{"role": "user", "content": "How do I make a cake?"}]

response = archon.generate(testing_instruction)

print(response)
```
A full local example can be seen in our [quickstart.py](archon/quickstart.py) file.

## gen_answers.py and Benchmarks

We provide the script [gen_answers.py](archon/gen_answers.py) so users can generate answers for supported benchmarks. We recommend you open and understand the file and its arguments.

We provide seperate READMEs for their respective usage and evaluation as seen with [MT Bench](archon/mt_bench/README.md), [Arena-Hard-Auto](archon/arena_hard_auto/README.md), [AlpacaEval](archon/alpaca_eval/README.md), [MixEval](archon/mix_eval/README.md), [MATH](archon/math/README.md), [CodeContests](archon/code_contests/README.md), [GSM8K](archon/gsm8k/README.md), and more. A complete list and their implmentation can be found at [archon/benchmarks.py](archon/benchmarks.py).

Here is an example on how to use gen_answers.py for running your config against ArenaHardAuto:
```python
# run gen_answers out of archon/ directory
python3 gen_answers.py --benchmark arena_hard_auto --config configs/<your-config-file>.json  --parallel 16
```
This will run the model structure specified in your config file against the question set specified under the ArenaHardAuto class and output the responses in `.jsonl` format to the `outputs/arena_hard_auto/model_answer/` folder.

#### Add Your Own Benchmark
To add your benchmark, you must edit the benchmarks.py file and add your benchmark class. The 'Benchmark' class can be used as a base class for interfacing between gen_answers.py and your benchmark. Lastly, make sure to add your evaluation to 'BENCHMARK_CLASSES' so it can be used as an argument in gen_answers.py

## Key Handling

For Archon to use your API keys, you can pass a path to a JSON file that holds your keys. An example of the expected file format is seen with [api_keys.json](api_keys.json). For example. you would initialize Archon with:
```python
archon = Archon(config, "path_to_keys/file.json")
```
We have a built-in system that swaps through your keys if you hit a rate limit. We found this helpful when working with APIs.

Alternatively, if no keys are provided, our system will look for environment variables corresponding to the keys in `api_keys.json`.
