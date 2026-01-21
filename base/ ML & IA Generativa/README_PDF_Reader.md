# 📄 Belgo PDF Reader

Um leitor de PDF especializado para extrair e tabular informações de pedidos de compra da Belgo, focando especialmente na área de produtos (tabela delimitada em vermelho).

## 🚀 Funcionalidades

- ✅ **Extração automática** de informações do cabeçalho do pedido
- ✅ **Tabulação da área vermelha** - extrai tabela de produtos automaticamente
- ✅ **Exportação para Excel e CSV** com múltiplas abas
- ✅ **Relatórios detalhados** com estatísticas
- ✅ **Processamento em lote** para múltiplos PDFs
- ✅ **Validação de dados** extraídos

## 📋 Dados Extraídos

### Cabeçalho do Pedido
- Número do pedido
- Fornecedor
- Data de faturamento
- Valor total do pedido
- Endereço de entrega
- CNPJ
- Telefone

### Tabela de Produtos (Área Vermelha)
- Código do produto
- Descrição do produto
- Unidade (KG, UN, PC, etc.)
- Quantidade
- Valor unitário
- Valor total

### Estatísticas Calculadas
- Total de produtos
- Valor total calculado
- Quantidade total de itens
- Distribuição por unidades

## 🛠️ Instalação

```bash
# Instale as dependências necessárias
pip install pandas docling openpyxl
```

## 📖 Como Usar

### Uso Básico (Recomendado)

```python
from pdf_reader_belgo import extrair_dados_pedido_belgo

# Extrai dados do PDF e salva automaticamente
pdf_path = 'caminho/para/seu/pedido.pdf'
dados = extrair_dados_pedido_belgo(pdf_path)

# Arquivos gerados automaticamente:
# - pedido_produtos_extraidos.xlsx
# - pedido_produtos_extraidos.csv
```

### Uso Avançado

```python
from pdf_reader_belgo import BelgoPDFReader

# Cria o leitor
reader = BelgoPDFReader('caminho/para/pedido.pdf')

# Processa o documento
dados = reader.processar_documento()

# Salva com nome personalizado
reader.salvar_em_excel('meu_arquivo.xlsx')

# Exibe relatório completo
reader.exibir_relatorio()

# Acessa dados específicos
print(f"Pedido: {dados['cabecalho']['numero_pedido']}")
print(f"Total de produtos: {len(dados['produtos'])}")
```

## 📊 Estrutura dos Dados Extraídos

```python
dados = {
    'cabecalho': {
        'numero_pedido': '495001',
        'fornecedor': 'BELGO BEKAERT ARAMES LTDA',
        'data_faturamento': '24/02/2025',
        'valor_total_pedido': '136.768,760'
    },
    'produtos': [
        {
            'codigo': '1130901708',
            'descricao': 'PREGO 22X48 1,1/2KG BELGO',
            'unidade': 'KG',
            'quantidade': 1,
            'valor_unitario': 1560.00,
            'valor_total': 12558.00
        }
        # ... mais produtos
    ],
    'estatisticas': {
        'total_produtos': 15,
        'valor_total_calculado': 136768.76,
        'quantidade_total': 850,
        'unidades_utilizadas': {'KG': 12, 'UN': 3}
    }
}
```

## 🗂️ Arquivos Gerados

### Excel (com múltiplas abas)
- **Produtos**: Tabela completa dos produtos extraídos
- **Informacoes_Pedido**: Dados do cabeçalho
- **Estatisticas**: Resumo e métricas calculadas

### CSV
- Arquivo simples com apenas a tabela de produtos

## 🔧 Exemplos Práticos

### Executar Exemplo Completo

```bash
cd belgo_poc_match
python exemplo_uso.py
```

### Processar Múltiplos PDFs

```python
from pdf_reader_belgo import BelgoPDFReader
import pandas as pd

pdfs = ['pedido1.pdf', 'pedido2.pdf', 'pedido3.pdf']
todos_produtos = []

for pdf in pdfs:
    reader = BelgoPDFReader(pdf)
    dados = reader.processar_documento()
    if dados.get('produtos'):
        todos_produtos.extend(dados['produtos'])

# Consolida em um único arquivo
df_consolidado = pd.DataFrame(todos_produtos)
df_consolidado.to_excel('todos_pedidos_consolidados.xlsx', index=False)
```

## 🎯 Casos de Uso

1. **Controle de Compras**: Automatizar a extração de dados de pedidos
2. **Análise de Fornecedores**: Consolidar informações de múltiplos pedidos
3. **Gestão de Estoque**: Acompanhar quantidades e valores
4. **Relatórios Gerenciais**: Gerar dashboards automáticos
5. **Auditoria**: Validar informações extraídas vs. sistema

## ⚠️ Observações Importantes

- O leitor foi otimizado para o formato específico dos PDFs da Belgo
- A área delimitada em vermelho (tabela de produtos) é o foco principal
- Funciona melhor com PDFs gerados eletronicamente (não escaneados)
- Para PDFs escaneados, considere usar OCR adicional

## 🐛 Solução de Problemas

### Nenhum produto extraído?
1. Verifique se o PDF não está corrompido
2. Confirme se é o formato esperado da Belgo
3. Execute com debug para ver o texto extraído

### Dados incorretos?
1. Verifique se o padrão do PDF mudou
2. Ajuste os regex patterns se necessário
3. Valide manualmente alguns produtos

## 📞 Suporte

Para dúvidas ou melhorias, verifique:
1. O arquivo `exemplo_uso.py` para casos de uso
2. Os comentários no código fonte
3. Execute exemplos práticos para entender o funcionamento 