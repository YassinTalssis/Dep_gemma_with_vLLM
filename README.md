🚀 Deploying Gemma 27B (IT INT4 AWQ) with vLLM
🛠️ Setup Instructions
1️⃣ Create and Activate a Conda Environment

Note: vLLM supports Python 3.9 – 3.11. Python 3.10 is recommended.

conda create --name gemma27b_vllm python=3.10
conda activate gemma27b_vllm


Install required dependencies:

pip install huggingface_hub vllm

2️⃣ Prepare the Model Directory
mkdir -p ~/models
cd ~/models


Download the model:

huggingface-cli download gaunernst/gemma-27b-it-int4-awq --local-dir ./gemma27b-it-int4-awq

3️⃣ Create the PBS Job Script

Save the following as gemma27b_vllm.sh:

#!/bin/bash
#PBS -N gemma27b_vllm
#PBS -l select=1:ncpus=4:mem=180gb:ngpus=1
#PBS -l walltime=24:00:00
#PBS -q gpu_1d
#PBS -o logs/output.log
#PBS -e logs/error.log

set -euo pipefail
mkdir -p logs

# ---- Load Anaconda module ----
module use /app/common/modules
module load anaconda3-2024.10

# ---- Init Conda ----
eval "$("$(command -v conda)" shell.bash hook)"
conda activate gemma27b_vllm

# ---- Navigate to model directory ----
cd /home/skiredj.abderrahman/models

# ---- Start vLLM server ----
python -m vllm.entrypoints.openai.api_server \
  --model /home/skiredj.abderrahman/models/gemma27b-it-int4-awq \
  --tensor-parallel-size 1 \
  --host 0.0.0.0 \
  --port 9998

4️⃣ Submit the Job
qsub gemma27b_vllm.sh


Check if it’s running:

qstat -u skiredj.abderrahman

5️⃣ Access the Model from Your Local Machine

Once the job is running, create an SSH tunnel to forward the server port to your local machine:

ssh -L 9998:<compute_node>:9998 skiredj.abderrahman@172.30.30.11


Replace <compute_node> with the node assigned to your job (check with qstat -f <jobid>).

🔍 Verifying the Deployment
✅ Inside the HPC Node

List available models:

curl http://localhost:9998/v1/models


Expected output:

{"object":"list","data":[{"id":"/home/skiredj.abderrahman/models/gemma27b-it-int4-awq","object":"model"}]}


Test a prompt:

curl http://localhost:9998/v1/completions -H "Content-Type: application/json" -d "{\"model\":\"/home/skiredj.abderrahman/models/gemma27b-it-int4-awq\",\"prompt\":\"Hello, how are you?\",\"max_tokens\":50}"

🌐 From Your Local Machine (via SSH Tunnel)

List models:

curl.exe http://localhost:9998/v1/models


Test a prompt:

curl.exe http://localhost:9998/v1/completions ^
-H "Content-Type: application/json" ^
-d "{\"model\":\"/home/skiredj.abderrahman/models/gemma27b-it-int4-awq\",\"prompt\":\"He
