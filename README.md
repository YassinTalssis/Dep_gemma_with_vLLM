# 🚀 Deploying Gemma 3 12B with vLLM

## 🛠️ Setup Instructions

###  Create and activate a conda environment

conda create --name gemma_vllm python=3.10 //vLLM suport python version 3.9-3.11, so version 3.10 is safe

conda activate gemma_vllm

pip install huggingface_hub
### create a directory named models
mkdir models
cd models
### download the model
huggingface-cli download gaunernst/gemma-3-12b-it-int4-awq --local-dir ./gemma3-12b-awq

### Create a bash script named script.sh
#!/bin/bash
#PBS -N test
#PBS -l select=1:ncpus=4:mem=180gb:ngpus=1
#PBS -q gpu_1d
#PBS -o output.log
#PBS -e error.log

# Load Anaconda module
module use /app/common/modules
module load anaconda3-2024.10

# Init Conda
source /app/common/anaconda3-2024.10/etc/profile.d/conda.sh
conda activate gemma_vllm

# Move to your model directory
cd /home/skiredj.abderrahman/models

# Start vLLM server
python -m vllm.entrypoints.openai.api_server \
    --model /home/skiredj.abderrahman/models/gpt-oss-20b \
    --tensor-parallel-size 1 \
    --port 8000



