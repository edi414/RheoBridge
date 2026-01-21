# web-automation-pje

Automação do sistema PJe (Processo Judicial Eletrônico) para extração estruturada de informações de processos judiciais.

## 🎯 O que faz

Este projeto automatiza dois fluxos principais:

### Fluxo 1: Extração de Dados do PJe

1. **Autenticação automática** - Login com perfil do Chrome ou credenciais (com suporte a 2FA)
2. **Busca de processos** - Navegação até formulário e preenchimento do número do processo (a partir de CSV)
3. **Download de documentos** - Download automático do PDF dos autos do processo
4. **Extração de dados** - Extração estruturada de informações do processo:
   - Dados gerais (classe judicial, assunto, jurisdição, autuação, etc.)
   - Polos ativo e passivo (usando LLM para parsing inteligente)
   - Códigos CDA encontrados no PDF (com páginas onde foram encontrados)
   - Dados estruturados adicionais
5. **Geração de JSON** - Exporta todas as informações em formato JSON validado em `output/`

### Fluxo 2: Registro no Legal One

1. **Leitura de JSONs** - Lê todos os arquivos JSON gerados do diretório `output/`
2. **Preenchimento automático** - Preenche formulário do Legal One com dados extraídos
3. **Submissão** - Submete o formulário e registra o processo
4. **Gestão de arquivos** - Move arquivos processados para `output/processed/`

## 📦 Instalação

```bash
pip install -r requirements.txt
uvx browser-use install
cp .env.example .env
```

Edite o arquivo `.env` com suas credenciais (OPENAI_API_KEY, PJE_CPF, PJE_SENHA, CHROME_PROFILE_EMAIL).

## 🚀 Como usar

### 1. Extração de Dados do PJe

Crie o arquivo `processos.csv` na raiz do projeto com os números dos processos:
```csv
numero_processo
0008453-46.2014.4.05.8300
0813690-37.2021.8.14.0301
```

Execute:
```bash
python main.py
```

O script processa cada processo do CSV e gera arquivos JSON em `output/`.

---

### 2. Registro no Legal One (`register_process.py`)

Este fluxo lê os JSONs gerados e registra os processos no sistema Legal One.

#### Execução

Execute o script de registro:
```bash
python register_process.py
```

O script irá:
1. Ler todos os arquivos JSON do diretório `output/`
2. Para cada JSON:
   - Extrair dados do formulário
   - Navegar para o formulário de criação do Legal One
   - Preencher todos os campos (ver mapeamento abaixo)
   - Submeter o formulário
   - Mover o arquivo JSON para `output/processed/` após sucesso
3. Exibir resumo final (sucessos/falhas)

**Nota:** 
- Arquivos processados com sucesso são movidos para `output/processed/`
- Arquivos com falha permanecem em `output/` para reprocessamento
- O script processa todos os JSONs encontrados em `output/` (exceto os já em `processed/`)

---

## 📋 Mapeamento de Campos (Register Process)

A tabela abaixo mostra a origem de cada campo preenchido no Legal One:

| Campo no Legal One | Fonte no JSON | Observações |
|-------------------|---------------|-------------|
| **Número CNJ** | `numero_processo` (do nome do arquivo) | Extraído do nome do arquivo JSON |
| **Status** | Valor fixo: `"Ativo"` | Sempre o mesmo valor |
| **Pasta** | `None` | Campo não mapeado (TODO) |
| **Cliente principal** | `polos.passivo[0].nome` | Primeiro polo passivo, com limpeza de sufixos (ex: "- EM RECUPERACAO JUDICIAL") |
| **Posição do cliente principal** | `polos.passivo[0].tipo` | Tipo do primeiro polo passivo (ex: "EXECUTADO", "EXEQUENTE") |
| **Contrário principal** | `polos.ativo[0].nome` | Primeiro polo ativo, com limpeza de sufixos |
| **Responsável principal** | Valor fixo: `"MÁRCIO FAM GONDIM"` | Sempre o mesmo valor |
| **Posição Responsável principal** | Valor fixo: `"Responsável"` | Sempre o mesmo valor |
| **Ação** | `classe_judicial` | Ex: "EXECUÇÃO FISCAL (1116)" |
| **Natureza** | Preenchido automaticamente | Campo lido após preencher "Ação" |
| **Fase** | `natureza` (valor lido automaticamente) | Usa o valor lido do campo "Natureza" |
| **Título** | `classe_judicial` (parte textual) | Extrai apenas a parte textual (ex: "EXECUÇÃO FISCAL") |
| **Valores** (Valor da causa) | `valor_da_causa` | Ex: "R$ 580.124,90" (formatado no padrão brasileiro) |
| **Outros envolvidos** | Valores fixos | Nome: "Rafael", Situação: "Responsável" |
| **Previsão e resultado** | Valores fixos | Contingência: "Passiva", Probabilidade: "Perda", Probabilidade lookup: "Provável", Risco: "Alto" |
| **Observações** | `cda_codes[].code` | Lista de códigos CDA (um por linha) |

## 📁 Estrutura do Projeto

```
web-automation-pje/
├── main.py                # Script principal: processa múltiplos processos do CSV
├── pje_automation.py      # Lógica de processamento de um único processo (PJe)
├── register_process.py    # Script: registra processos no Legal One a partir dos JSONs
├── utils/                 # Módulos auxiliares
│   ├── auth.py           # Autenticação (login, 2FA)
│   ├── chrome_profile.py  # Gerenciamento de perfil Chrome
│   ├── pje_browser.py    # Configuração do browser
│   ├── pje_navigation.py # Navegação no PJe
│   ├── pje_form.py       # Preenchimento de formulários
│   ├── pje_search.py     # Busca e resultados
│   ├── pje_download.py   # Download de documentos
│   ├── pje_pdf_reader.py # Leitura de PDFs e extração CDA
│   ├── pje_process_info.py # Extração de informações do processo
│   ├── legal_one_utils.py # Utilitários para Legal One
│   ├── legal_one/        # Módulos específicos do Legal One
│   │   ├── navigation.py # Navegação no Legal One
│   │   ├── cnj_number.py # Preenchimento Número CNJ
│   │   ├── status.py     # Preenchimento Status
│   │   ├── cliente_principal.py # Preenchimento Cliente principal
│   │   ├── valores.py    # Preenchimento Valores
│   │   ├── outros_envolvidos.py # Preenchimento Outros envolvidos
│   │   ├── previsao_resultado.py # Preenchimento Previsão e resultado
│   │   ├── observacoes.py # Preenchimento Observações
│   │   └── submit.py     # Submissão do formulário
│   └── logger.py         # Sistema de logging
├── output/                # JSONs gerados (processados pelo register_process.py)
│   └── processed/        # JSONs já processados e registrados
├── .env                   # Variáveis de ambiente (não commitado)
├── .env.example           # Template de variáveis
└── requirements.txt      # Dependências Python
```

## 📊 Formato do JSON Gerado

O JSON gerado contém:

```json
{
  "classe_judicial": "...",
  "assunto": "...",
  "jurisdicao": "...",
  "autuacao": "...",
  "ultima_distribuicao": "...",
  "valor_da_causa": "...",
  "segredo_de_justica": "...",
  "justica_gratuita": "...",
  "tutela_liminar": "...",
  "prioridade": "...",
  "orgao_julgador": "...",
  "cargo_judicial": "...",
  "competencia": "...",
  "polos": {
    "ativo": [
      {
        "nome": "...",
        "cnpj": "...",
        "tipo": "EXEQUENTE"
      }
    ],
    "passivo": [...]
  },
  "cda_codes": [
    {
      "code": "FGPE201200153",
      "pages": [1, 5, 10]
    }
  ],
  "dados_estruturados": [...],
  "total_campos": 13,
  "total_polos_ativo": 2,
  "total_polos_passivo": 1,
  "total_cda_codes": 3
}
```

## 🔄 Fluxo Completo

O processo completo funciona em duas etapas:

1. **Extração (PJe → JSON)**
   - `main.py` lê `processos.csv`
   - Para cada processo, extrai dados do PJe
   - Salva JSON em `output/processo_<numero>_<timestamp>.json`

2. **Registro (JSON → Legal One)**
   - `register_process.py` lê todos os JSONs de `output/`
   - Para cada JSON, preenche formulário no Legal One
   - Após sucesso, move JSON para `output/processed/`

## ⚙️ Funcionalidades Avançadas

- **Perfil persistente do Chrome**: Mantém sessão ativa, evitando login repetido
- **Fallback de autenticação**: Se não detectar login, tenta login automático
- **Extração inteligente de polos**: Usa LLM para parsear informações complexas
- **Validação rigorosa**: JSON gerado é validado antes de salvar
- **Logging detalhado**: Sistema de logs para debug e monitoramento
- **Tratamento de erros**: Validações em cada etapa do processo
- **Gestão de arquivos processados**: Move arquivos após registro bem-sucedido
