# SageMaker Batch Transform com Go & LightGBM (sa-east-1)

Este repositório documenta a implementação de um container de inferência genérico de alta performance utilizando a linguagem **Go** e a biblioteca **Leaves** para processamento de modelos LightGBM no Amazon SageMaker.

## 🚀 Objetivo
Substituir containers de inferência tradicionais (Python) por uma solução nativa em Go para reduzir custos e aumentar a vazão (throughput) na região **sa-east-1 (São Paulo)**.

## 📊 Comparativo Executivo: Python vs. Go

| Métrica | Python (Standard) | Go (Leaves) | Impacto |
| :--- | :--- | :--- | :--- |
| **Linguagem** | Interpretada (Lenta) | Compilada Nativa (Rápida) | Performance Bruta |
| **Paralelismo** | Multi-processo (GIL) | Goroutines (Threads Leves) | 5x mais vazão |
| **Memória RAM** | ~800MB+ Idle | ~20MB Idle | Downsizing de Instância |
| **Instância** | ml.m5.xlarge ($0.33/h) | ml.c5.large ($0.17/h) | **-48% no custo/h** |
| **Tamanho Imagem**| > 1GB | ~40MB | Startup Instantâneo |

## 🛠️ Arquitetura do Container Genérico
O container foi desenhado para ser **multitenant**, carregando o modelo dinamicamente via variáveis de ambiente passadas pelo SageMaker Job.

### Hierarquia de Pastas (S3)
`s3://{BUCKET_MODELOS}/{PROJETO_ID}/{EXPERIMENTO_ID}/model.txt`

### Variáveis de Ambiente Suportadas
- `MODEL_S3_BUCKET`: Nome do bucket S3.
- `MODEL_S3_KEY`: Caminho completo do arquivo `.txt`.
- `PREDICTION_THRESHOLD`: Limiar para classificação binária (opcional).

## 💻 Implementação Técnica

### 1. Dockerfile (Multi-stage Build)
Otimizado para gerar um binário estático sem dependências externas.
*(Ver seção de arquivos no repositório)*

### 2. Go Handler (Performance)
- **Parser CSV:** Uso de `reader.ReuseRecord = true` para minimizar alocações de memória.
- **S3 Downloader:** Uso do `manager.NewDownloader` da AWS SDK v2 para download paralelo de modelos.
- **Inferência:** Biblioteca `leaves` (Go puro) para carregar modelos LightGBM de forma thread-safe.

## 📉 Plano de Validação de Precisão
Para garantir que a tradução de Python para Go (Leaves) não altere as predições:
1. Gerar predições de referência no ambiente de treino (Python).
2. Processar o mesmo dataset no container Go.
3. Comparar via script de similaridade (Tolerância sugerida: `1e-5`).

## 💰 Estimativa de Custo em sa-east-1
Ao reduzir o tempo de execução e utilizar instâncias da família **C5**, a economia projetada é de **55% a 70%** em comparação ao processamento padrão em Python.

---
*Estudo técnico desenvolvido para otimização de custos e performance em infraestrutura de ML na AWS.*
