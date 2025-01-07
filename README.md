# Step-by-Step Mastery: Enhancing Soft Constraint Following Ability of Large Language Models
[![Github](https://img.shields.io/static/v1?logo=github&style=flat&color=pink&label=github&message=happy12348/FollowSoftConstraints)]([https://github.com/YJiangcm/FollowBench](https://github.com/meowpass/FollowComplexInstruction))
[![HuggingFace](https://img.shields.io/badge/%F0%9F%A4%97-huggingface-yellow)](https://huggingface.co/datasets/Abbey4799/Complex-Instructions-DPO)

Official implementation of the paper "Step-by-Step Mastery: Enhancing Soft Constraint Following Ability of Large Language Models". 

We systematically study **how to enhance the ability of LLMs to follow soft constraints**, addressing the following research questions:
- ***How to construct* multi-constraint instruction following dataset**
  - To enable the model to learn how to follow each constraint, we increase only one constraint at a time, enabling the model progressively learn to follow each constraint during the training process.
- ***How to obtain* high-quality outputs?**
  - We introduce Judger to reorder the outputs based on the extent of constraint following to ensure the quality of outputs.
- ***How to effectively utilize* the data obtained through Judger reording?**
  - We we develop a training paradigm based on curriculum learning to enhance the training process.
  - We conduct extensive experiments to prove the effectiveness of our methods in terms of *overall performance and generalization abilities*.



![image](https://github.com/meowpass/FCS/assets/56729976/debacf40-1858-402b-b94a-700e7b7ad20b)

## 🔥Updates
* 2024/6/18: We posted the second version of our [paper](https://arxiv.org/pdf/2404.15846)
* 2024/4/24: We posted the first version of our paper.
* 2024/4/22: We released the data and code of FCS

## ⚙️How to Use the Code

### Install Dependencies

```
conda create -n fcs python=3.10.9
conda activate fcs
conda install pytorch==1.13.1 torchvision==0.14.1 torchaudio==0.13.1 pytorch-cuda=11.7 -c pytorch -c nvidia
pip install -r requirements.txt
```

### Obtain Complex Instructions

#### Complex Instruction Synthesis

To obtain complex instructions: 
- First, we collect seed instructions from three widely used instruction-tuning datasets. 
- Then, we rewrite the instructions to incorporate multiple constraints. 
Here, you can complete the whole procedure by running the script `gen_inst.sh`:

```shell
python ../get_data/gen_inst.py \
    --seed_path=../get_data/data/seed_data.jsonl  \
    --data_path=../get_data/data/data.jsonl\
    --api_key=YOUR_API_KEY_TO_ACESS_GPT4\
```

An example of complex instrutcion is shown as below:
![image](https://github.com/meowpass/FollowComplexInstruction/assets/56729976/cd6810af-d472-42e7-afff-43b83e30dc42)
Here are 3 different constraints in the instructions.

#### Get Model Outputs

You need to do inference with your model to get the responses to the complex instructions. Here, we provide a script to do inference for LLaMA via the script `do_inference.sh`:

```shell
CUDA_VISIBLE_DEVICES=YOUR_CUDA_DEVICES python ../get_data/do_inference.py \
    --data_path=../get_data/data/data.jsonl \
    --res_path=../get_data/data/res_llama2.jsonl \
    --model_path=PATH_TO_YOUR_MODEL\
    --lora_path=PATH_TO_YOUR_LORA_WEIGHT IF YOU USE LORA \
```



#### Teacher Correction

We propose a discrimination-based approach for obtaining the output, shown to be more effective than directly generating output with advanced LLMs. 

First, we utilize the test scripts from [IFEval](https://github.com/google-research/google-research/tree/master/instruction_following_eval) to identify the constraints the model failed to follow since the constraints are objective and automatically verifiable. Simply run the script `check.sh`:

```shell
python ../get_data/check.py \
    --input_data=../get_data/data/data.jsonl \
    --input_response_data=../get_data/data/res_llama2.jsonl \
    --output_dir=../get_data/data/ \
    --output_file_name=checked_res_llama2
```

Then, we adopt advanced LLMs (teacher model) GPT-3.5-turbo to correct the failed constraints one by one. You can correct the response to simultaneously get data for IFT and DPO with the script `correct.sh`:

```shell
python ../get_data/correct.py \
    --res_path=../get_data/data/res_llama2.jsonl  \
    --ift_data_path=../dpo_train/data/ift_train.jsonl \
    --dpo_data_path=../dpo_train/data/dpo_train.jsonl \
    --api_key=YOUR_API_KEY_TO_ACESS_GPT4\
```


### Contrastive Method (Go for DPO Training)
The slight changes in the instruction (i.e. `json` to `xml`) can cause substantial output differences. Hence, negative samples failing to meet certain constraints, also offer valuable supervision signals. we leverage the positive and negative samples through reinforcement learning fine-tuning.

Here, we provide a revised implementation for an advanced DPO in `dpo_train`. You can set your model_path and data_path in `dpo_train/dpo_train.py`. Then, you can train the model with the script `train_dpo.sh`:

```shell
CUDA_VISIBLE_DEVICES=YOUR_CUDA_DEVICES accelerate launch \
    --config_file ../dpo_train/deepspeed_zero1.yaml dpo_train.py \
    --output_dir=PATH_TO_SAVE_MODEL \
```
## Citation
```
@misc{he2024complex,
      title={From Complex to Simple: Enhancing Multi-Constraint Complex Instruction Following Ability of Large Language Models}, 
      author={Qianyu He and Jie Zeng and Qianxi He and Jiaqing Liang and Yanghua Xiao},
      year={2024},
      eprint={2404.15846},
      archivePrefix={arXiv},
      primaryClass={cs.CL}
}
```
