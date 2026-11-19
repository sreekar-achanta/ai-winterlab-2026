# AI WinterLab 2026

This repository contains scripts for AI WinterLab 2026.

## Participant handout

Run these steps from your laptop unless a step says to run commands inside the container.

### 1. Sign in and deploy your assigned Ollama app

Use Azure CLI in a Bash or Zsh terminal. Replace the placeholders with the values supplied by the instructor, and only deploy when your wave starts. The app uses the existing GPU profile and exposes Ollama only inside the Container Apps environment.

Keep this terminal open for step 2 so the `RG` and `APP` variables remain available.

```bash
az login --tenant '<TENANT_ID>'
az account set --subscription '<SUBSCRIPTION_ID>'

RG=rg-ollama-workshop
ENV=aca-ollama-workshop
APP=ollama-<your-name>
IMAGE=ollama/ollama
CPU=8
MEMORY=56Gi

az containerapp create \
  --resource-group "$RG" \
  --name "$APP" \
  --environment "$ENV" \
  --workload-profile-name gpu-t4 \
  --image "$IMAGE" \
  --cpu "$CPU" \
  --memory "$MEMORY" \
  --ingress internal \
  --target-port 11434 \
  --min-replicas 1 \
  --max-replicas 1 \
  --env-vars OLLAMA_HOST=0.0.0.0:11434 \
             OLLAMA_CONTEXT_LENGTH=2048 \
             OLLAMA_NUM_PARALLEL=1
```

### 2. Load a model, create a variant, and verify GPU use

Open a shell in your new app. If Azure says it can’t find a replica immediately after deployment, wait until the replica is running before trying again.

```bash
az containerapp exec -g "$RG" -n "$APP" --command /bin/sh
```

Run the following commands **inside the container**. Replace `user01` with your assigned ID. The pull downloads the base model; create registers your variant with a different system prompt. It doesn’t retrain the model.

```bash
MODEL=workshop-<your-name>

ollama pull llama3.2:3b

cat > /tmp/Modelfile <<'EOF'
FROM llama3.2:3b
PARAMETER num_ctx 2048
SYSTEM """You are a platform engineering tutor. Explain concepts in plain language and give one practical example."""
EOF

ollama create "$MODEL" -f /tmp/Modelfile
ollama run "$MODEL" "Explain Docker Compose in one sentence."
ollama ps
```

Check the `PROCESSOR` column in `ollama ps`. If it says `100% GPU`, you’ve verified GPU inference. Then open the shared Open WebUI link and select the model name announced by the instructor. If the instructor connected your backend, select your own `workshop-userXX` model.

### 3. Hand back your GPU slot

At the end of the wave, exit the shell and tell the instructor you’re done. Closing the terminal does not stop the GPU app. The instructor should stop the finished apps in Azure and verify that quota usage drops before starting the next wave. With `minReplicas: 1`, idle GPU replicas still bill
