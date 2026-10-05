# Análise do Estado Atual do Projeto — TCC_SIM

**Data:** 2026-09-16  
**Gerado por:** agente data-scientist (Claude Code)

---

## (a) O que o notebook já faz

O pipeline de ETL (`analise_inicial.ipynb`) está essencialmente concluído. Etapas realizadas:

### Coleta e seleção de arquivos (células 1–25)
- 51 arquivos `.parquet` baixados do PySUS para os anos 2000–2025.
- Lógica de priorização: quando existe um arquivo `DOSP<ano>` (estadual SP), os arquivos nacionais `Mortalidade_Geral_*` e `DO**OPEN` do mesmo ano são descartados.
- 26 arquivos efetivamente carregados: um `DOSP` por ano de 2000 a 2024, mais `Mortalidade_Geral_2025`.

### Consolidação e filtro geográfico (células 27–28)
- Filtro por `CODMUNRES >= 350000 AND < 360000`, aplicado após fatiamento para 6 dígitos (resolve problema de 7 dígitos identificado nos anos anteriores).
- Resultado: **7.450.875 linhas** e **103 colunas**.

### Redução de escopo e limpeza (células 60–75)
- 69 colunas removidas (cartório, bloco materno/fetal, investigação de óbito) → **35 colunas**.
- Sentinelas de "ignorado" (`9`/`99`/`999`) substituídos por `NaN` nos campos categóricos.
- `DTOBITO` e `DTNASC` convertidos para `datetime`; `IDADE` recalculada como anos decimais.

### Enriquecimento via de-para (células 77–78)
- Merge com `municipios.csv` → adiciona `MUNNOMEX`, `CAPITAL`, `UF`.
- Merge com `cid10.csv` → adiciona `DESCR`, `CAT`, `SUBCAT`.
- Parquet final: `variavel/dataframe_tratado.parquet` com **41 colunas** e **7.450.453 linhas** (422 duplicatas removidas).

### Visualizações iniciais (células 91–94)
Quatro figuras já produzidas:
- **Figura 1:** Série temporal de óbitos totais 2000–2025, com bandas de H1N1 e COVID-19.
- **Figura 2:** Composição proporcional por grupo de causa (stacked bar por ano).
- **Figura 3:** Excesso de mortalidade mensal 2020–2022 em relação ao baseline 2015–2019.
- **Figura 4:** Painel duplo — idade mediana ao óbito e proporção por sexo ao longo do tempo.

---

## (b) Problemas e Gaps Identificados

| # | Problema | Impacto |
|---|---|---|
| 1 | **26.974 óbitos sem CID mapeado** — códigos como `J09` (gripe H1N1), `K85x` (pancreatite), `C80x` (neoplasia sítio desconhecido) não estão na tabela `cid10.csv` | **Alto** — afeta análises por causa |
| 2 | **Séries temporais sem normalização por população** — SP cresceu demograficamente, distorcendo comparações entre anos | **Alto** — pergunta central do TCC |
| 3 | **Classificação de grupos CID na fig. 2 muito grosseira** (apenas 7 categorias) e não usa a função `grupo_cid()` canônica | **Alto** — transição epidemiológica |
| 4 | **`ESC2010` com ~51% de NaN** — campo ausente nos arquivos anteriores a 2010; sem estratégia declarada para análises de escolaridade | **Médio** — análises de desigualdade |
| 5 | **`RACACOR` não mapeado para rótulos** (Branca, Preta, Parda, Amarela, Indígena) e sem análise iniciada | **Médio** — dimensão de desigualdade |
| 6 | **2025 anomalamente alto (354.163 óbitos)** — superior a 2024 e próximo ao pico de 2022; viés de registro tardio não quantificado | **Médio** — truncar série em 2024 nas tendências |
| 7 | **Excess mortality sem intervalo de confiança** — figura atual exibe apenas valor pontual, sem respaldo estatístico | **Baixo** — melhoria para monografia |
| 8 | **Sentinelas tratados antes da deduplicação** — ordem lógica invertida (deveria ser `drop_duplicates → replace_sentinelas`) | **Baixo** — risco prático pequeno |
| 9 | **Notebook mistura ETL com análise** — sem estrutura analítica dedicada às perguntas de pesquisa do TCC | **Estrutural** |

### Detalhes adicionais

**Problema 1 — Códigos CID sem mapeamento:**  
Os principais afetados são `K859`, `K850`, `K851`, `J09`, `K852`, `I489`, `C809`. O `J09` (influenza pandêmica) é especialmente relevante pois seria esperado em 2009 (H1N1). Esses registros aparecem como `NaN` em `DESCR/CAT/SUBCAT`.

**Problema 6 — Viés de 2025:**  
354.163 óbitos em 2025 é valor implausível epidemiologicamente para dado consolidado — provavelmente reflexo de registros ainda em consolidação no DATASUS. A nota "*2025 possivelmente incompleto" está na figura, mas não há quantificação do viés.

---

## (c) Próximos Passos Recomendados

### Prioridade 1 — Corrigir os 26.974 óbitos sem CID mapeado
Derivar `GRUPO_CAUSA` diretamente de `CAUSABAS` com a função `grupo_cid()`, que opera sobre o código CID sem depender do join com `cid10.csv`. Isso resolve o problema completamente para agrupamentos e mantém todos os registros na análise.

```python
import pandas as pd, duckdb

def grupo_cid(codigo):
    if pd.isna(codigo):
        return 'Ignorado'
    c = str(codigo).strip().upper()
    if c.startswith(('A', 'B')):             return 'Infecciosas e Parasitárias'
    if c.startswith('C') or c.startswith('D0') or c[:2] in ('D1','D2','D3','D4'):
                                              return 'Neoplasias'
    if c.startswith(('E',)):                  return 'Endócrinas e Metabólicas'
    if c.startswith('F'):                     return 'Transtornos Mentais'
    if c.startswith('G'):                     return 'Sistema Nervoso'
    if c.startswith('I'):                     return 'Cardiovasculares'
    if c.startswith('J'):                     return 'Respiratórias'
    if c.startswith('K'):                     return 'Digestivas'
    if c.startswith(('V','W','X','Y')):       return 'Causas Externas'
    if c == 'R99' or c.startswith('R'):       return 'Causas Mal Definidas'
    return 'Demais'

df = pd.read_parquet('variavel/dataframe_tratado.parquet',
                     columns=['DTOBITO','CAUSABAS','SEXO','IDADE','RACACOR','ESC2010','MUNNOMEX','UF'])
df['GRUPO_CAUSA'] = df['CAUSABAS'].apply(grupo_cid)
```

### Prioridade 2 — Criar `analise_exploratoria.ipynb`
Separar ETL de análise. O novo notebook deve carregar diretamente do `.parquet` via DuckDB ou pandas com seleção de colunas. Estrutura recomendada:

1. Verificação de cobertura temporal (óbitos por ano; confirmar anomalia de 2025)
2. Série temporal normalizada (taxa por 100k — requer dados populacionais)
3. Transição epidemiológica: composição de `GRUPO_CAUSA` por ano
4. Anomalias: excess mortality COVID-19 e H1N1 com intervalo de confiança
5. Perfis demográficos: sexo, faixa etária, raça/cor, escolaridade

### Prioridade 3 — Obter dados populacionais do IBGE
Sem a população por ano de SP, é impossível calcular taxas e comparar períodos.

- **Fonte:** Estimativas de População IBGE — série 2000–2024 por UF
- **Salvar em:** `variavel/populacao_sp_ibge.csv`
- **Cálculo:** `taxa_bruta = (obitos_ano / pop_ano) * 100_000`

### Prioridade 4 — Análise da Transição Epidemiológica (pergunta central)
Com `GRUPO_CAUSA` derivado e normalização por população:

- Heatmap de proporção de óbitos por grupo × ano (evidencia o deslocamento de infecciosas para crônico-degenerativas)
- Séries de taxa por 100k para os 4 grupos principais (Cardiovasculares, Neoplasias, Infecciosas e Parasitárias, Causas Externas) com linha de tendência (regressão linear ou LOESS)
- Teste de quebra estrutural na série de Infecciosas em 2020 — `statsmodels` ou teste de Chow

### Prioridade 5 — Excess Mortality com Intervalo de Confiança
Baseline ±2 desvios-padrão da média 2015–2019 por mês. Transformar a visualização em resultado com respaldo estatístico: quantos meses de 2020–2022 ficaram acima do limite superior.

### Prioridade 6 — Análise por Raça/Cor e Escolaridade (desigualdades)
- Mapear `RACACOR`: `{1: 'Branca', 2: 'Preta', 3: 'Amarela', 4: 'Parda', 5: 'Indígena'}`
- Calcular idade mediana ao óbito por raça/cor — indicador direto de desigualdade em expectativa de vida
- Para escolaridade: usar `ESC2010` apenas pós-2010, ou criar variável harmonizada combinando `ESC` e `ESC2010`

### Prioridade 7 — Truncar 2025 nas Análises de Tendência
Todas as análises de tendência de longo prazo devem usar **2000–2024**. O ano de 2025 pode ser mencionado separadamente com a ressalva de dado preliminar.

### Prioridade 8 — Padronizar nomenclatura das figuras
Criar subpasta `variavel/figuras/` e adotar nomes descritivos:
- `fig01_serie_temporal_obitos.png`
- `fig02_transicao_grupos_cid.png`
- `fig03_excess_mortality_covid.png`
- `fig04_perfil_demografico.png`

---

## Resumo Executivo

O ETL está pronto e funcional. O parquet final tem todos os 26 anos da série (2000–2025) com cobertura contínua e sem lacunas visíveis no volume por ano.

**Gap principal:** ausência de dados populacionais externos (para taxas por 100k) e de um notebook analítico estruturado pelas perguntas de pesquisa.

**O projeto está em condições de avançar diretamente para as análises.**
