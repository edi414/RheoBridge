# Automação Uniplus - Ajuste de Estoque

Automação desktop para processar ajustes de estoque no sistema Uniplus a partir de planilha CSV. Utiliza reconhecimento de imagem (pyautogui) e Win32 API para interação confiável com o sistema.

## 📁 Estrutura do Projeto

```
automation-desktop-uniplus/
├── bot.py                      # Script principal da automação
├── config.py                   # Configurações
├── requirements.txt            # Dependências
├── data/
│   └── ajustes_estoque.csv    # Planilha com ajustes (sku, quantidade)
├── resources/
│   └── images/                 # Imagens para busca visual
│       ├── estoque_button.png        # Menu "Estoque" (NECESSÁRIO)
│       └── ajuste_estoque_button.png # Item "Ajuste de estoque" (NECESSÁRIO)
├── logs/
│   └── bot.log                # Log de execução
└── utils/
    ├── logger.py              # Utilitário de logging
    └── automation.py          # Funções utilitárias de automação (Win32 API)
```

## 🚀 Instalação

1. Instalar dependências:
```bash
pip install -r requirements.txt
```

2. **IMPORTANTE**: Capturar as imagens necessárias no sistema Uniplus:
   - `resources/images/estoque_button.png` - Screenshot do menu "Estoque" na barra superior
   - `resources/images/ajuste_estoque_button.png` - Screenshot do item "Ajuste de estoque" no dropdown

## 📝 Formato do CSV

O arquivo `data/ajustes_estoque.csv` deve ter o formato:

```csv
sku,quantidade
12345,10
67890,-5
```

- **sku**: Código do produto
- **quantidade**: Quantidade do ajuste (positivo = entrada, negativo = saída)

## ▶️ Execução

**Execução Normal:**
```bash
python bot.py
```

**Execução como Administrador (Recomendado no Windows):**
Execute o PowerShell como Administrador e rode:
```powershell
python bot.py
```

> **Nota:** Executar como administrador garante que os comandos Win32 API funcionem corretamente e evita problemas de permissão.

## 🔧 O que a automação faz

1. Abre o Uniplus
2. Navega: Estoque → Ajuste de estoque (usando reconhecimento de imagem)
3. Para cada produto no CSV:
   - F2 para abrir novo ajuste + aguarda 5s
   - Tab para navegar
   - Seleciona tipo (entrada/saída conforme sinal) com setas para baixo
   - Tab
   - Preenche "motivo" + Enter + Tab
   - Shift+Tab (3x) para voltar
   - Preenche "geral" + Enter + Tab
   - Tab (2x) para pular campos
   - Preenche SKU + Enter
   - Tab para quantidade
   - Preenche quantidade (valor absoluto) + Enter (2x)
   - F10 para finalizar + aguarda 5s
   - Enter para confirmar
   - ESC para fechar modal
   - Repete para o próximo item

## ⚙️ Configurações

Ajuste no arquivo `config.py`:
- Caminho do Uniplus
- Tempo de espera entre ações
- Quantidade de teclas para navegação

## 📋 Checklist de Preparação

- [ ] Instalar dependências (`pip install -r requirements.txt`)
- [ ] Criar diretório `resources/images/`
- [ ] Capturar imagem `estoque_button.png`
- [ ] Capturar imagem `ajuste_estoque_button.png`
- [ ] Preparar CSV com ajustes em `data/ajustes_estoque.csv`
- [ ] Testar com 1-2 produtos primeiro
- [ ] Verificar que o Uniplus está configurado corretamente no `config.py`

## 🖼️ Como Capturar as Imagens

1. Abra o Uniplus
2. Para **estoque_button.png**: 
   - Tire um screenshot da palavra "Estoque" na barra de menu superior (com destaque laranja se possível)
   - Salve como `resources/images/estoque_button.png`

3. Para **ajuste_estoque_button.png**:
   - Clique no menu Estoque para abrir o dropdown
   - Tire um screenshot do item "Ajuste de estoque" no dropdown
   - Salve como `resources/images/ajuste_estoque_button.png`

**Dica**: Use a ferramenta de captura do Windows (Win + Shift + S) e recorte apenas a região necessária. Quanto mais específica a captura, maior a precisão do reconhecimento.

## 📊 Logs

Os logs são salvos em `logs/bot.log` e também exibidos no console durante a execução. Os logs incluem:
- Informações sobre o processamento de cada item
- Coordenadas das imagens encontradas
- Erros e avisos durante a execução
- Status de conclusão de cada ajuste

## 🔧 Tecnologias Utilizadas

- **pyautogui**: Reconhecimento de imagem na tela
- **Win32 API**: Interação de baixo nível com Windows (cliques e teclas)
- **pandas**: Processamento do arquivo CSV
- **OpenCV**: Backend do pyautogui para matching de imagens

## 📝 Observações

- O script funciona corretamente mesmo quando executado como administrador (diretório de trabalho é ajustado automaticamente)
- As imagens são encontradas usando matching com confiança configurável (padrão 0.9 para menu, 0.8 para dropdown)
- O script processa todos os itens do CSV sequencialmente
- Cada ajuste é processado independentemente (F2 abre novo ajuste, ESC fecha após cada um)
