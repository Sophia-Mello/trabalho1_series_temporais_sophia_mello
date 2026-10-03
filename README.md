# Trabalho Prático I — Séries Temporais e Aprendizagem Supervisionada II

Análise exploratória, estacionariedade e seleção de atributos (FAC/FACP) do **índice de volume de vendas no varejo brasileiro** (PMC/IBGE, mensal, jan/2000 a jul/2026, n = 319).

**Disciplina:** Séries Temporais e Aprendizagem Supervisionada II  
**Professor:** Gildo Leonel  
**Autoria:** Sophia Cristine Soares Mello

## Arquivos

| Arquivo | Conteúdo |
|---|---|
| `trabalho1_series_temporais.ipynb` | Notebook executado (Opção A): código, gráficos, tabelas e explicações das 5 etapas, mais um bônus de modelagem |
| `data/pmc_varejo_volume_ibge.csv` | Cópia dos dados usada no notebook (baixada em 02/10/2026) |
| `requirements.txt` | Bibliotecas necessárias |

## Fonte dos dados

IBGE, Pesquisa Mensal de Comércio: [Tabela SIDRA 8880](https://sidra.ibge.gov.br/tabela/8880), variável 7169 (número-índice, 2022 = 100), índice de volume de vendas no comércio varejista, Brasil, sem ajuste sazonal.

O notebook lê o CSV local por padrão, para que os números do texto sejam reproduzíveis. Para baixar a versão mais recente pela API do IBGE, use `carregar_pmc(atualizar=True)`.

## Como reproduzir

```bash
python -m venv .venv
# Windows: .venv\Scripts\activate    |    Linux/macOS: source .venv/bin/activate
pip install -r requirements.txt
jupyter notebook trabalho1_series_temporais.ipynb
```

No Jupyter, execute *Kernel → Restart & Run All*. Testado com Python 3.14, numpy 2.5, pandas 3.0, matplotlib 3.11 e statsmodels 0.15.

## Principais resultados

- **Decomposição:** a sazonalidade é multiplicativa (dezembro ≈ 30% acima da tendência, fevereiro ≈ 11% abaixo), o que justifica usar `log(y)`. O padrão sazonal também mudou ao longo do tempo: dezembro perdeu peso para novembro, mudança compatível com a Black Friday.
- **Janelas deslizantes:** `create_sliding_window` com p = 12 gera X com formato (307, 12) e y com (307,), confirmando n − p amostras.
- **Estacionariedade:** a série original não é estacionária (ADF p = 0,683). A 1ª diferença passa no ADF, mas a FAC revela uma raiz unitária sazonal. A série `∇∇₁₂ log(y)` é estacionária (p < 0,0001).
- **FAC/FACP:** 11 dos 36 lags são significativos na FACP (1, 2, 4, 5, 11, 12, 13, 19, 23, 25 e 36).
- **Bônus:** a regressão (OLS) com os lags da FACP teve o menor erro no teste (MAPE de 1,44%). A janela completa de 36 lags sobreajustou e ficou em 2,06%.
