# CI-CD Pipeline Configuration

Este documento descreve como configurar e rodar a pipeline de CI-CD para o projeto Fake Shop.

## Pré-requisitos

1. Conta no [Docker Hub](https://hub.docker.com/)
2. Cluster Kubernetes configurado
3. Repositório no GitHub

## Configuração

### 1. Configurar Secrets no GitHub

1. Vá para a página do seu repositório no GitHub.
2. Clique em **Settings** > **Secrets and variables** > **Actions**.
3. Adicione os seguintes secrets:
   - `DOCKERHUB_USERNAME`: Seu nome de usuário do Docker Hub.
   - `DOCKERHUB_TOKEN`: Seu token de acesso do Docker Hub.
   - `K8S_KUBECONFIG`: O conteúdo do seu arquivo kubeconfig para acessar o cluster Kubernetes.

### 2. Estrutura do Projeto

Certifique-se de que seu projeto tenha a seguinte estrutura de diretórios:

### 3. Arquivo de Pipeline

O arquivo de pipeline `.github/workflows/main.yml` deve estar configurado conforme abaixo:

```yaml
name: CI-CD
on:
  push:
    branches: ["main"]
  workflow_dispatch:

jobs:
  CI:
    runs-on: ubuntu-latest
    steps:
      - name: Obtendo o codigo
        uses: actions/checkout@v4.2.2
      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      - name: Construcao e envio da imagem
        uses: docker/build-push-action@v6
        with:
          context: ./src
          push: true
          file: ./src/Dockerfile
          tags: |
            andreluiz901/fake-shop-desafio:v${{ github.run_number }}
            andreluiz901/fake-shop-desafio:latest

  CD:
    needs: [CI]
    runs-on: ubuntu-latest
    steps:
      - name: Obtendo o codigo
        uses: actions/checkout@v4.2.2

      - uses: azure/k8s-set-context@v4
        with:
          method: kubeconfig
          kubeconfig: ${{ secrets.K8S_KUBECONFIG }}

      - uses: Azure/k8s-deploy@v5
        with:
          manifests: |
            [deployment.yaml](http://_vscodecontentref_/0)
          images: |
            andreluiz901/fake-shop-desafio:v${{ github.run_number }}
```

### Executando a Pipeline

1. Faça um push para o branch main ou dispare a pipeline manualmente através da aba Actions no GitHub.

2. A pipeline irá:

- Fazer o checkout do código.
- Fazer login no Docker Hub.
- Construir e enviar a imagem Docker para o Docker Hub.
- Atualizar o deployment no Kubernetes com a nova imagem.
