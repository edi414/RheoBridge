# 🤖 Belgo POC - Sistema de Matching Semântico de Produtos

Sistema híbrido de matching semântico desenvolvido para identificar e associar produtos de pedidos da Belgo ao catálogo de materiais utilizando tecnologias avançadas de NLP e Machine Learning.

## 📋 Visão Geral

Este projeto implementa um sistema completo de extração e matching semântico que:
- **Extrai produtos** de PDFs de pedidos de compra da Belgo
- **Faz matching inteligente** com o catálogo de materiais usando embeddings semânticos
- **Compara algoritmos** diferentes para otimizar precisão e performance
- **Gera relatórios** com resultados detalhados e métricas de acurácia

## 🛠️ Tecnologias Utilizadas

### Processamento de Documentos
- **Docling**: Extração avançada de texto de PDFs
- **Pandas**: Manipulação e análise de dados estruturados

### Machine Learning & NLP
- **OpenAI Embeddings (text-embedding-3-small)**: Geração de embeddings semânticos executados em ambiente local para garantir controle e prevenção de vazamento de dados sensíveis
- **Sentence Transformers**: Modelos multilíngues para embeddings (alternativa local)
- **TF-IDF Vectorizer**: Análise de similaridade textual baseada em frequência de termos
- **scikit-learn**: KNN (K-Nearest Neighbors) para busca eficiente no espaço vetorial
- **PyTorch**: Framework para modelos de deep learning

### Otimização e Performance
- **NumPy**: Operações numéricas eficientes
- **ThreadPoolExecutor**: Processamento paralelo de embeddings
- **tqdm**: Barras de progresso para monitoramento

## 🏗️ Arquitetura do Sistema

O sistema é composto por três módulos principais:

### 1. **PDF Reader** (`pdf_reader_belgo.py`)
Extração de dados estruturados de PDFs de pedidos:
- **Reconhecimento de padrões** com regex para identificar campos
- **Extração de produtos** da tabela principal
- **Exportação** para Excel e CSV

### 2. **Match Maker** (`match_maker.py` & `match_maker_optimized.py`)
Sistema híbrido de matching semântico:

#### **Pipeline de Processamento:**

```
1. Preparação dos Dados
   ├── Normalização de texto (lowercase, remoção de caracteres especiais)
   ├── Limpeza de dados (remoção de "REF:", "BELGO", etc.)
   └── Unificação de encoding (NFKD -> ASCII)

2. Geração de Embeddings
   ├── OpenAI Embeddings (text-embedding-3-small) - Ambiente local
   ├── TF-IDF Vectorization (ngram_range: 1-3, max_features: 5000-10000)
   └── Cache de embeddings para otimização (arquivos .pkl)

3. Busca com KNN
   ├── Construção do índice com NearestNeighbors (métrica: cosine)
   ├── Busca dos top-K candidatos (K=10, 20, 30, 50)
   └── Cálculo de distâncias semânticas e textuais

4. Re-ranking Inteligente (Otimizado)
   ├── Recalcula scores híbridos apenas nos top-K candidatos
   ├── Combina similaridade semântica (80-85%) + textual (15-20%)
   └── Seleciona o melhor match final

5. Score Híbrido Final
   └── weighted_score = (semantic_weight × semantic_sim) + (string_weight × string_sim)
```

### 3. **Teste e Otimização** (`test_optimizations.py`)
Comparação sistemática de algoritmos:
- Testa 4 configurações diferentes
- Gera métricas de acurácia e tempo de execução
- Salva resultados comparativos em Excel

## 🔄 Como Funciona o Processo

### Fluxo Completo:

```
┌─────────────────┐
│   PDF Pedido    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ PDF Reader      │ ← Extrai produtos usando Docling + Regex
│ (Extração)      │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Limpeza de Dados│ ← Remove "REF:", normaliza texto
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Embeddings      │ ← OpenAI (local) + TF-IDF
│ (Vectorização)  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ KNN Search      │ ← Busca top-K candidatos
│ (Busca)         │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Re-ranking      │ ← Recalcula scores híbridos
│ (Otimização)    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Melhor Match    │ ← Produto correspondente + Score
└─────────────────┘
```

### Algoritmos Testados

O sistema compara **4 estratégias diferentes**:

1. **Baseline (K=10)**
   - 10 candidatos mais próximos
   - Peso: 70% semântico + 30% textual
   - Sem re-ranking

2. **Otimizado K=20 + Re-ranking**
   - 20 candidatos, depois re-ranking
   - Peso: 80% semântico + 20% textual
   - Re-ranking dos top-20

3. **Otimizado K=50 + Re-ranking**
   - 50 candidatos, depois re-ranking
   - Peso: 85% semântico + 15% textual
   - Re-ranking dos top-50

4. **Semântico Puro (100%)**
   - 30 candidatos
   - Peso: 100% semântico + 0% textual
   - Apenas significado semântico

**Resultado dos Testes**: A configuração otimizada (K=50, 85% semântica + 15% textual) demonstrou melhor performance, confirmando que análise semântica profunda supera matching textual simples, especialmente para produtos com descrições variadas.

## 📁 Estrutura de Arquivos

```
belgo_poc_match/
├── pdf_reader_belgo.py           # Leitor de PDFs e extração de dados
├── match_maker.py                # Matcher básico (baseline)
├── match_maker_optimized.py      # Matcher otimizado com re-ranking
├── test_optimizations.py         # Script principal de testes e comparação
├── test_belgo.py                 # Testes unitários básicos
├── compare_methods.py            # Comparação de métodos (ST vs OpenAI)
├── requirements.txt              # Dependências do projeto
├── .env                          # Variáveis de ambiente (OPENAI_API_KEY)
│
├── Materiais Fábricas.xlsx       # Catálogo de materiais da Belgo
├── catalog_with_*.pkl            # Embeddings em cache (otimização)
│
└── Resultados/
    ├── resultado_matching_completo.csv  # CSV final com matches
    ├── optimization_comparison.xlsx     # Comparação de algoritmos
    └── matching_results_openai.xlsx     # Resultados detalhados
```

## 🚀 Como Executar

### 1. Instalação de Dependências

```bash
pip install -r requirements.txt
```

### 2. Configuração do Ambiente

Crie um arquivo `.env` na raiz do projeto:

```env
OPENAI_API_KEY=sua_api_key_aqui
```

### 3. Preparação dos Dados

- Coloque o arquivo `Materiais Fábricas.xlsx` no diretório
- Prepare os arquivos `.pkl` com produtos extraídos dos PDFs em `ocr_extract/`

### 4. Executar Testes e Comparação

```bash
python test_optimizations.py
```

Este script irá:
- ✅ Carregar produtos do arquivo `.pkl`
- ✅ Gerar/carregar embeddings (com cache)
- ✅ Testar 4 algoritmos diferentes
- ✅ Gerar CSV com resultados completos
- ✅ Criar comparação de performance em Excel

## 📊 Saídas do Sistema

### Arquivos Gerados:

1. **`resultado_matching_completo.csv`**
   - Produtos do pedido
   - Material correspondente no catálogo
   - Score de matching (0-1)
   - Descrições do catálogo

2. **`optimization_comparison.xlsx`**
   - Tabela comparativa dos 4 algoritmos
   - Métricas: acurácia, tempo de execução
   - Ranking de performance

3. **`catalog_with_openai_embeddings.pkl`**
   - Cache de embeddings (evita recálculo)
   - Reutilizável em execuções futuras

## 🔐 Segurança de Dados

- **OpenAI Embeddings executados localmente**: Garante controle total sobre dados sensíveis
- **Cache de embeddings**: Reduz chamadas à API e exposição de dados
- **Variáveis de ambiente**: API keys protegidas via `.env`

## 📈 Métricas e Performance

O sistema calcula:
- **Acurácia**: Taxa de matches corretos (threshold > 0.7)
- **Tempo de execução**: Performance de cada algoritmo
- **Scores de matching**: Similaridade semântica e textual combinadas

## 🎯 Tecnologias-Chave

### Embeddings Semânticos
- **OpenAI text-embedding-3-small**: Captura significado profundo dos produtos
- **Execução local**: Segurança e controle de dados
- **1536 dimensões**: Representação rica do contexto semântico

### TF-IDF
- **N-grams (1-3)**: Captura frases e termos compostos
- **Até 10.000 features**: Vocabulário extenso
- **Filtros inteligentes**: Remove termos muito raros/comuns

### KNN Otimizado
- **Métrica cosseno**: Mede similaridade de direção (melhor para embeddings)
- **Busca eficiente**: Índices pré-construídos
- **Paralelização**: Múltiplos cores para acelerar busca

### Re-ranking
- **Híbrido**: Combina semântica + textual
- **Top-K**: Analisa apenas candidatos promissores
- **Pesos ajustáveis**: Balanceamento customizável
