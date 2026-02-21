# Deploy para Google Cloud Run

## 🌐 Serviço em Produção

**URL**: https://agente-contabil-55837972640.us-central1.run.app

```bash
curl https://agente-contabil-55837972640.us-central1.run.app/health
curl -X POST https://agente-contabil-55837972640.us-central1.run.app/chat \
  -H "Content-Type: application/json" -d '{"message": "O que é ISS?"}'
```

## 🚀 Deploy

**Pré-requisitos:**
```bash
brew install --cask google-cloud-sdk
gcloud auth login
gcloud config set project ctbz-ia-assessoria-poc
gcloud auth configure-docker us-central1-docker.pkg.dev
```

**Deploy completo:**
```bash
docker build --platform linux/amd64 -t agente-contabil:amd64 .

# Tag e Push
docker tag agente-contabil:amd64 us-central1-docker.pkg.dev/ctbz-ia-assessoria-poc/cloud-run-source-deploy/agente-contabil:amd64
docker push us-central1-docker.pkg.dev/ctbz-ia-assessoria-poc/cloud-run-source-deploy/agente-contabil:amd64

# Deploy
gcloud run deploy agente-contabil \
  --image us-central1-docker.pkg.dev/ctbz-ia-assessoria-poc/cloud-run-source-deploy/agente-contabil:amd64 \
  --region us-central1 \
  --allow-unauthenticated \
  --set-env-vars OPENAI_API_KEY=sua_chave
```

## 🔄 Atualizar

```bash
docker build --platform linux/amd64 -t agente-contabil:amd64 .
docker tag agente-contabil:amd64 us-central1-docker.pkg.dev/ctbz-ia-assessoria-poc/cloud-run-source-deploy/agente-contabil:latest
docker push us-central1-docker.pkg.dev/ctbz-ia-assessoria-poc/cloud-run-source-deploy/agente-contabil:latest
gcloud run deploy agente-contabil --image us-central1-docker.pkg.dev/ctbz-ia-assessoria-poc/cloud-run-source-deploy/agente-contabil:latest --region us-central1
```

## 🧪 Teste Local

```bash
docker build --platform linux/amd64 -t agente-contabil:test .
docker run -d -p 8080:8080 --env-file src/.env --name agente-test agente-contabil:test
curl http://localhost:8080/health
docker stop agente-test && docker rm agente-test
```

## ⚙️ Configurações

```bash
# Env vars
gcloud run services update agente-contabil --region us-central1 --set-env-vars OPENAI_API_KEY=nova_chave

# Recursos
gcloud run deploy agente-contabil --region us-central1 --memory 2Gi --cpu 2 --timeout 300

# Logs
gcloud run services logs tail agente-contabil --region us-central1
```

## 🗑️ Derrubar Serviço

```bash
# Deletar serviço Cloud Run
gcloud run services delete agente-contabil --region us-central1

# Deletar imagem do Artifact Registry (opcional)
gcloud artifacts docker images delete \
  us-central1-docker.pkg.dev/ctbz-ia-assessoria-poc/cloud-run-source-deploy/agente-contabil:amd64
```
