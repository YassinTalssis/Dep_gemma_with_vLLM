# 🚀 Gemma 27B (IT INT4 AWQ) Deployment with vLLM

This repository contains instructions to deploy the **Gemma 27B INT4 AWQ model** on an HPC environment using **vLLM** and access it locally via SSH tunneling.


## 1️⃣ Setup Conda Environment

Create and activate the environment:

```bash
conda create --name gemma27b_vllm python=3.10
conda activate gemma27b_vllm

```

---

### 2️⃣ Prepare the Model Directory
```bash
mkdir models
cd models
```
Download the model:
```bash
huggingface-cli download gaunernst/gemma27b-it-int4-awq --local-dir ./gemma27b-it-int4-awq
```

---

### 3️⃣ Create the PBS Job Script
Save the following as `script.sh`:

```bash
#!/bin/bash
#PBS -N gemma_vllm
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

# Navigate to model directory
cd /home/skiredj.abderrahman/models

# Start vLLM server
python -m vllm.entrypoints.openai.api_server     --model /home/skiredj.abderrahman/models/gemma27b-it-int4-awq   --tensor-parallel-size 1     --port 8000
```

---

### 4️⃣ Submit the Job
```bash
qsub script.sh
```

Check if it’s running:
```bash
qstat
```

---

### 5️⃣ Access the Model from Your Local Machine
Create an SSH tunnel to forward port `8000` from the GPU node (`gpu06` in this example) to your local machine:
```bash
ssh -L 8000:gpu06:8000 skiredj.abderrahman@172.30.30.11
```
*(Replace `gpu06` with the node assigned to your job.)*

---

## 🔍 Verifying the Deployment

### ✅ Inside the Cluster (direct access to the node)
Open new CMD(command line)
Run:
```bash
curl http://localhost:8000/v1/models
```
Expected output (example):
```json
{"object":"list","data":[{"id":"/home/skiredj.abderrahman/models/gemma3-12b-awq","object":"model"}]}
```
Test a prompt:
```bash
curl http://localhost:8000/v1/completions -H "Content-Type: application/json" -d '{"model":"/home/skiredj.abderrahman/models/gemma3-12b-awq","prompt":"Hello, how are you?","max_tokens":50}'
```

---

### 🌐 Outside the Cluster (via SSH tunnel)
On your local machine (after tunneling):
```bash
curl.exe http://localhost:8000/v1/models
```
Expected output is the same as inside the cluster.

Test a prompt:
```bash
curl.exe http://localhost:8000/v1/completions ^
-H "Content-Type: application/json" ^
-d "{\"model\":\"/home/skiredj.abderrahman/models/gemma3-12b-awq\",\"prompt\":\"Hello, how are you?\",\"max_tokens\":50}"
```


