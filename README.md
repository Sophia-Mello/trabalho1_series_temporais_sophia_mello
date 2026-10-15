# Séries Temporais e Aprendizagem Supervisionada II — Trabalhos Práticos

Análise do **índice de volume de vendas no varejo brasileiro** (PMC/IBGE, mensal, jan/2000 a jul/2026, n = 319), a mesma série nos dois trabalhos práticos da disciplina.

**Professor:** Gildo Leonel  
**Autoria:** Sophia Cristine Soares Mello

## Arquivos

| Arquivo | Conteúdo |
|---|---|
| `trabalho1_series_temporais.ipynb` | Trabalho I: análise exploratória, decomposição, janelas deslizantes, ADF e FAC/FACP, mais um bônus de modelagem |
| `trabalho2/trabalho2_box_jenkins_arima.ipynb` | Trabalho II: metodologia Box-Jenkins, busca em grade ARIMA(p, d, q) e seleção por AIC/BIC |
| `data/pmc_varejo_volume_ibge.csv` | Cópia dos dados usada nos dois notebooks (baixada em 02/10/2026) |
| `requirements.txt` | Bibliotecas necessárias |

Os dois notebooks estão executados, com as saídas (gráficos e tabelas) salvas.

## Fonte dos dados

IBGE, Pesquisa Mensal de Comércio: [Tabela SIDRA 8880](https://sidra.ibge.gov.br/tabela/8880), variável 7169 (número-índice, 2022 = 100), índice de volume de vendas no comércio varejista, Brasil, sem ajuste sazonal.

Os notebooks leem o CSV local, para que os números do texto sejam reproduzíveis. No Trabalho I, `carregar_pmc(atualizar=True)` baixa a versão mais recente pela API do IBGE.

## Como reproduzir

```bash
python -m venv .venv
# Windows: .venv\Scripts\activate    |    Linux/macOS: source .venv/bin/activate
pip install -r requirements.txt
jupyter notebook
```

Abra o notebook desejado e execute *Kernel → Restart & Run All*. O notebook do Trabalho II lê os dados em `../data/`, então deve ser aberto a partir da pasta `trabalho2/`, que é o comportamento padrão do Jupyter. Testado com Python 3.14, numpy 2.5, pandas 3.0, matplotlib 3.11 e statsmodels 0.15.

## Principais resultados

### Trabalho I

- **Decomposição:** a sazonalidade é multiplicativa (dezembro ≈ 30% acima da tendência, fevereiro ≈ 11% abaixo), o que justifica usar `log(y)`. O padrão sazonal também mudou ao longo do tempo: dezembro perdeu peso para novembro, mudança compatível com a Black Friday.
- **Janelas deslizantes:** `create_sliding_window` com p = 12 gera X com formato (307, 12) e y com (307,), confirmando n − p amostras.
- **Estacionariedade:** a série original não é estacionária (ADF p = 0,683). A 1ª diferença passa no ADF, mas a FAC revela uma raiz unitária sazonal. A série `∇∇₁₂ log(y)` é estacionária (p < 0,0001).
- **FAC/FACP:** 11 dos 36 lags são significativos na FACP (1, 2, 4, 5, 11, 12, 13, 19, 23, 25 e 36).
- **Bônus:** a regressão (OLS) com os lags da FACP teve o menor erro no teste (MAPE de 1,44%). A janela completa de 36 lags sobreajustou e ficou em 2,06%.

### Trabalho II

- **Identificação:** d = 1, com a diferença sazonal mantida como passo fixo, como no Trabalho I. Candidatos lidos na FAC/FACP: ARIMA(0,1,1), (2,1,0), (1,1,1) e (4,1,0).
- **Estimação:** 25 modelos (p e q de 0 a 4) ajustados por máxima verossimilhança no treino (jan/2000 a jul/2023). Os últimos 36 meses ficaram reservados como teste.
- **Seleção:** o AIC escolheu o ARIMA(3,1,4) e o BIC escolheu o ARIMA(1,1,1). O campeão é o (1,1,1): mais parcimonioso, com todos os coeficientes significativos.
- **Uma volta extra no ciclo:** os resíduos do campeão ainda mostram autocorrelação no lag 12. Com um MA sazonal, o SARIMA(0,1,1)(0,1,1)₁₂ melhora o AIC em 68 pontos usando o mesmo número de parâmetros.
