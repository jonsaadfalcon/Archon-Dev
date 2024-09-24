# Archon: An Architecture Search Framework for Inference-Time Techniques
Inference-time techniques allow us to bolster the strengths of existing LMs by utilizing multiple sample calls and multiple LMs to increase system performance for a given task.
Archon provides a modular framework for combining different inference-time techniques and LMs with just a JSON config file. Check out our [Quick Start](#quick-start) guide to get started.
![Archon Overview Diagram](readme_assets/archon_itas_diagram.svg)


## Table of Contents

- [Installation](#installation)
    - [Python Package](#python-package)
    - [Running Locally](#running-locally)
- [Archon Overview](#archon-overview)
- [Quick Start](#quick-start)
- [Tutorials/Examples](#tutorialsexamples)
- [Benchmarks](#benchmarks)
    - [Add Your Own Benchmark](#add-your-own-benchmark)
- [Key Handling](#key-handling)
- [Resources](#resources)

## Installation 

### Python Package
Archon is publicly available for use at https://pypi.org/project/archon-ai/.
```
pip install archon-ai
```
### Running Locally

Alternatively, you can use Archon directly from this repository. This is the most up to date and offers the most flexibility working with our codebase

```shell
git clone https://github.com/ScalingIntelligence/Archon.git
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
Archon can be instantiated via the [Archon](https://github.com/ScalingIntelligence/Archon/blob/main/archon/archon.py#L176) class in `archon.py`. Get started by cloning the repository and navigating to our [quickstart.py](https://github.com/ScalingIntelligence/Archon/blob/main/archon/quickstart.py) or creating your own starter file that imports our Archon class as shown below: 
```python
from archon import Archon
```
## Archon Overview
Our Archon codebase provides users with the ability to run Archon configurations. We highly recommend reading our research paper [here](link_to_paper).

At the moment, we provide support for these inference times techniques: `generator`, `fuser`, `critic`, `ranker`, `verifier`, `unit_test_generator`, `unit_test_evaluator`. Their prompts can be found [here](archon/components/prompts.py). As well as the ability to [add your own components](#tutorialsexamples). 

We also provide these API Access points that can be used in Archon configurations: `Together_API`, `OpenAI_API`, `Anthropic_API`, `Groq_API`, `Google_API`, `tgi`, `Bedrock_API`. You can also [add your own generators/endpoints](#tutorialsexamples).

## Quick Start
This colab notebook demonstrates how to use the Archon python package.

<a target="_blank" href="https://colab.research.google.com/drive/1DH-vsm6aLxoaIPqs9xFYOBj_n6NFKEPm?usp=sharing">
  <img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/>
</a>
 <br> <br>

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

## Tutorials/Examples 
To fully take advantage of what Archon has to offer, we recommend you take a look at our tutorials/examples.

|  **Tutorial** | **Run in Colab** | **Description** |
| ------------- |  -------------  |  -------------  | 
| [**Quick Start**](archon/quickstart.py) | [<img align="center" src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab" />](https://colab.research.google.com/drive/1Y9qg4DDQs8RK-VXSEj9jZaK-fAUfTbFM?usp=sharing)  |  Introduces Archon and how to create and use a compound Archon configuration of gpt-4o models. |
|   [**Single Model Query**]() | [<img align="center" src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab" />](https://colab.research.google.com/drive/1Y9qg4DDQs8RK-VXSEj9jZaK-fAUfTbFM?usp=sharing) | A single gpt-4o model call. |
|   [**Custom Components**](https://colab.research.google.com/drive/11h-uL2D7oFiSu6HdQIHlf_xDyRDOy56m?usp=sharing) | [<img align="center" src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab" />](https://colab.research.google.com/drive/11h-uL2D7oFiSu6HdQIHlf_xDyRDOy56m?usp=sharing) | We offer support on adding your own components apart from the supported ones (generation, fusion, ranking, etc). The colab notebook will show you how to create a component, add it to your configs, and add it to Archon with add_component.|
|   [**Custom Generator/Endpoint**](https://colab.research.google.com/drive/1D1Mecl1jJNRY875ThLGJZupRRQC53lk3?usp=sharing) | [<img align="center" src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab" />](https://colab.research.google.com/drive/1D1Mecl1jJNRY875ThLGJZupRRQC53lk3?usp=sharing) | We offer support to add your own generator if you need a custom access point or need more than the ones provided (Together, OpenAI, etc). The colab notebook will show you how to add a custom endpoint. Here you will see how to create a generator function, add it to your configs, and add it to Archon with add_generator.|
|   [**Improving GPT 4o using Archon**](https://colab.research.google.com/drive/14ohSRBD9mDympZk0MO0MumWnjA4tCk1e#scrollTo=ZrxjlqILrWla) | N/A | An Archon configuration which uses multiple gpt-4o calls to produce higher quality result compared to a single call of gpt-4o|
| [**Open-sourced Archon to outperform GPT 4o**](https://colab.research.google.com/drive/1mhXNc6xfR6CxrHv_tF2xcr5xzKmnA9nD#scrollTo=ZrxjlqILrWla) | N/A | An Archon configuration which uses only open sourced LLMs that would produce higher quality results compared to a single call of gpt-4o|
|[**Advanced Multi-Model Setup**](https://colab.research.google.com/drive/1LBFIi_IRXvQ6m_z0Fjjer8XdyZPdUN0b?usp=sharing) | [<img align="center" src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab" />](https://colab.research.google.com/drive/1LBFIi_IRXvQ6m_z0Fjjer8XdyZPdUN0b?usp=sharing) | Archon config that showcases multiple layers of generations, critics, ranking, and fusing.|


## Benchmarks
Once you have created a config, you can leverage pre-existing benchmark frameworks to assess accuracy. We provide classes for [AlpacaEval](https://github.com/tatsu-lab/alpaca_eval), [ArenaHardAuto](https://github.com/lm-sys/arena-hard-auto), and [MT Bench](https://huggingface.co/spaces/lmsys/mt-bench). The classes are under ```archon/benchmarks.py``` where you can modify the input question files under the class ```load_dataset()``` call.
Here is an example command for running your config against ArenaHardAuto:
```
python3 archon/gen_answers.py --benchmark arena_hard_auto --config archon/configs/<your-config-file>.json --output_dir outputs --parallel 16
```
This will run the model structure specified in your config file against the question set specified under the ArenaHardAuto class and output the responses in `.jsonl` format under an `outputs` folder. 

#### Add Your Own Benchmark
To add your benchmark, you must edit the benchmarks.py file and add your benchmark class. The 'Benchmark' class can be used as a base class for interfacing between gen_answers.py and your benchmark. Lastly, make sure to add your evaluation to 'BENCHMARK_CLASSES' so it can be used as an argument in gen_answers.py

## Key Handling

You can add your keys to `api_keys.json`. We have a built in system that swaps through your keys if you hit a rate limit. Alternatively, if no keys are provided, our system will look for environment variables corresponding to `api_keys.json`.

If using the pip package, you can provide a path to your api keys to Archon when initialzing with

```python
archon = Archon(config, "path_to_keys/file.json")
```

Otherwise, it will look for environment variables.

## Citation
```bibtex
@article{placeholder,
      title={Archon: An Architecture Search Framework for Inference-Time Techniques},
      author={placeholder},
      year={2024},
      eprint={placeholder},
      archivePrefix={arXiv}
}
```

