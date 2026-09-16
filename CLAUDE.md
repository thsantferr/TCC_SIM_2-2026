# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# TCC_SIM — Contexto do Projeto

## O que é este projeto

TCC (monografia) do curso de **Ciência de Dados** da UNIVESP — Grupo TCC530-BCD-Turma 002,
orientador Cassio Silva Takarada.

- **Tema específico:** transição epidemiológica e padrões de mortalidade no Estado de São Paulo,
  usando microdados do DATASUS, entre 2000 e 2025.
- **Título provisório:** "Análise e Visualização da Transição Epidemiológica no Estado de São
  Paulo: Um Estudo dos Óbitos Registrados entre 2000 e 2025 via Ciência de Dados".
- **Problema de pesquisa:** como técnicas de Ciência de Dados podem revelar padrões, tendências e
  anomalias nas causas de óbito no Estado de São Paulo (2000–2025), evidenciando a transição
  epidemiológica e o impacto de eventos sanitários agudos.
- **Tipo de trabalho:** TCC monografia (não é continuação de PI, não há pesquisa de campo).
- Detalhes completos do briefing acadêmico estão em
  [documento_tcc/Informações preliminares para projeto TCC_Quinzena 0.docx](documento_tcc/Informações%20preliminares%20para%20projeto%20TCC_Quinzena%200.docx).

### Cronograma proposto (do documento do TCC)
1. Levantamento bibliográfico e aquisição de dados
2. Pipeline de ETL (conversão, limpeza, padronização)
3. Análise exploratória e modelagem (séries temporais, agrupamento)
4. Dashboard interativo (produto analítico)
5. Redação final e revisão (ABNT)

## Comandos comuns

**Ativar o ambiente virtual (Windows PowerShell):**
```powershell
.\.venv\Scripts\Activate.ps1
```

**Iniciar o Jupyter:**
```powershell
.\.venv\Scripts\jupyter.exe notebook
```

**Rodar scripts Python com as libs do projeto (sem ativar o venv):**
```powershell
.\.venv\Scripts\python.exe meu_script.py
```

O `python` do PATH do sistema (`C:\Python312`) **não tem** as libs do projeto — sempre usar
`.\.venv\Scripts\python.exe` explicitamente ou ativar o venv antes.

## Fonte de dados: SIM/DATASUS

Os dados são microdados de **Declarações de Óbito (DO)** do **Sistema de Informações sobre
Mortalidade (SIM)**, mantido pelo Ministério da Saúde/DATASUS. Ver [resumo.md](resumo.md) para uma
explicação completa e acessível da base, dos campos e das tabelas auxiliares.

## Estrutura do repositório

```
TCC_SIM/
├── .claude/
│   ├── agents/
│   │   ├── data_scientist.md   # Agente DS: análises, código, classificação CID, técnicas
│   │   └── code_reviewer.md    # Agente revisor de código Python/notebook
│   └── skills/                 # Skills do Claude Code (EDA, qualidade de dados, etc.)
├── arquivo/                    # Dados brutos SIM em formato .dbc (DBF comprimido) — LEGADO
│   └── DOEXT15.dbc … DOEXT25.dbc   # 11 arquivos anuais (2015–2025), não usados pelo pipeline
├── documentacao/               # Documentação oficial do SIM/DATASUS (dicionários, CID)
│   ├── Estrutura_do_SIM_2025.pdf      # Dicionário de campos ATUAL
│   ├── Estrutura_SIM_para_CD.pdf      # Dicionário versão 2019
│   ├── Estrutura_SIM_Anterior.pdf     # Dicionários 2006 e pré-2005
│   ├── INTRO.pdf                      # Histórico do SIM, legislação, qualidade dos dados
│   ├── Legislacao_PDF.pdf, Portaria.pdf
│   ├── MTAB16M.pdf                    # Lista de Tabulação para Mortalidade (agrupamento CID)
│   ├── Docs-Tabs-CID9.zip             # Tabelas auxiliares CID-9 (pré-1996)
│   └── Docs_Tabs_CID10.zip            # Tabelas auxiliares: CADMUN, CID10, TABUF, TABPAIS, TABOCUP
├── tabelas_depara/             # Tabelas de-para extraídas dos DBF (CSV ;, UTF-8)
│   ├── municipios.csv                 # CADMUN — código IBGE, nome, UF, região de saúde
│   ├── estados.csv                    # TABUF — sigla, código, nome das 27 UFs
│   ├── cid10.csv                      # CID10 — 14.198 códigos com descrição
│   ├── paises.csv                     # TABPAIS — códigos de país (naturalidade)
│   └── ocupacoes.csv                  # TABOCUP — códigos CBO com descrição
├── variavel/                   # Artefatos intermediários (não versionados — ver .gitignore)
│   ├── base_sim.pkl                   # df bruto consolidado (104 colunas, ~11,7M linhas)
│   ├── base_sim_v2.pkl                # df após remoção de colunas irrelevantes (35 colunas)
│   ├── dataframe_tratado.parquet      # df final processado (41 colunas, datas parseadas, de-para aplicados)
│   ├── municiopios.pkl                # tabela de municípios processada
│   ├── cids.pkl                       # tabela CID-10 sem coluna OPC
│   ├── path_parquet.pkl               # lista de caminhos dos .parquet baixados pelo PySUS
│   ├── anos_coletas.pkl               # lista de anos da coleta (2000–2025)
│   └── verificacao_campos.txt         # resumo de cada coluna (tipo, valores únicos, nulos)
├── documento_tcc/              # Briefing acadêmico do TCC (docx)
├── analise_inicial.ipynb       # Notebook do pipeline de coleta/consolidação (ativo)
├── dicionario_campos.md        # Dicionário dos 104 campos presentes no df consolidado
├── requiriments.txt            # Dependências Python do projeto
├── README.md                   # Visão geral do projeto
├── resumo.md                   # Explicação da base SIM em linguagem acessível
├── .env                        # Variáveis de ambiente (não versionado) — ver abaixo
├── .gitignore
└── CLAUDE.md                   # Este arquivo
```

## Variáveis de ambiente (`.env`)

Criar o arquivo `.env` na raiz com a chave da API (não versionar):

```
ANTHROPIC_API_KEY=sk-ant-...
```

## Pipeline de coleta atual (`analise_inicial.ipynb`)

O notebook **não lê os `.dbc` de `arquivo/`** — baixa os dados direto do DATASUS via `pysus.sim()`:

```python
from pysus import sim
anos_coleta = list(range(2000, 2026))
dt = sim(state='SP', year=anos_coleta)  # devolve caminhos locais de arquivos .parquet
```

Os parquets são salvos em `C:\Users\<usuário>\pysus\downloads\ducklake\sim\` (fora do repositório).

### Etapas do pipeline executadas no notebook

1. **Coleta** — `pysus.sim(state='SP', year=2000..2025)` baixa ~51 arquivos .parquet para o cache do PySUS.
2. **Consolidação** — para cada arquivo: uppercase/strip nos nomes de coluna, filtro `CODMUNRES` entre `350000`–`359999`, concatenação → `df` com **104 colunas** e ~11,7M linhas.
3. **Redução de colunas** — remove 69 colunas irrelevantes ao escopo (cartório, metadados internos, bloco materno/fetal, investigação de óbito) → `df` com **35 colunas**.
4. **Deduplicação** — `df.drop_duplicates()`.
5. **Parsing de datas** — `DTOBITO` e `DTNASC` convertidos para `datetime` (formato `ddmmaaaa`); `IDADE` recalculada como anos decimais a partir da diferença `DTOBITO - DTNASC`.
6. **Enriquecimento** — `MUNNOMEX`, `CAPITAL`, `UF` adicionados via join com `municipios`; `DESCR`, `CAT`, `SUBCAT` adicionados via join com `cid10` → **41 colunas** no parquet final.
7. **Persistência** — df processado salvo em `variavel/dataframe_tratado.parquet`; variáveis intermediárias salvas como `.pkl` em `variavel/`.

### Comportamento observado do `pysus.sim(state='SP', ...)`

O parâmetro `state='SP'` **não garante** que o arquivo baixado já venha filtrado para SP — o PySUS
devolve tipos de arquivo diferentes conforme o ano:
- `Mortalidade_Geral_AAAA.parquet` e `DOxxOPEN.parquet` (dados preliminares "abertos") — volume
  compatível com **Brasil inteiro** (ex. ~1,5 milhão de linhas/ano), não só SP.
- `DOSPAAAA.parquet` — já vem filtrado para SP (~350 mil linhas/ano, compatível com o volume real
  do estado).

Por isso o filtro manual de `CODMUNRES` no notebook é **obrigatório**, não opcional — sem ele a base
final mistura óbitos de outras UFs.

## Agentes disponíveis (`.claude/agents/`)

| Agente | Quando usar |
|---|---|
| `data-scientist` | Escrever/revisar código de análise, sugerir técnicas (séries temporais, clustering, excess mortality), consultas DuckDB/pandas, classificação de grupos CID |
| `code-reviewer` | Revisão focada em correção, boas práticas pandas, performance e reprodutibilidade do notebook |

## Pontos de atenção importantes (verificar com o(a) orientador(a) / grupo)

1. **Duplicidade em anos com dois arquivos.** Para alguns anos (ex. 2020, 2022) o `pysus.sim()`
   baixa **dois arquivos** (um tipo nacional/aberto + um `DOSP` já filtrado). Depois do filtro de
   `CODMUNRES`, os dois deveriam convergir para o mesmo subconjunto — vale checar duplicação por
   `CONTADOR`/`NUMERODO` dentro do mesmo ano.
2. **Campos não documentados.** `CODMUNCART`, `CODCART`, `NUMREGCART`, `DTREGCART`, `CRM` e
   `EXPDIFDATA` aparecem na base mas não constam nos dicionários oficiais — prováveis campos
   específicos das bases estaduais (`DOSP*`/SEADE). Ver
   [dicionario_campos.md](dicionario_campos.md#8-campos-de-cartório-e-outros-não-documentados-nos-dicionários-oficiais).
3. **Mistura de layouts de formulário.** A base cobre 2000–2025, juntando layouts do SIM atuais com
   campos de formulários antigos (pré-2010). Pares que representam o mesmo conceito com nomes
   diferentes entre vintages (`DTRECORIG`/`DTRECORIGA`, `NUDIASOBCO`/`NUDIASOBIN`,
   `FONTES`/`FONTESINF`) — decidir se serão unificados na limpeza.
4. **`arquivo/*.dbc` está desatualizado em relação ao pipeline.** Os 11 arquivos `.dbc` (2015–2025)
   não são mais lidos pelo notebook; considerar remover ou documentar por que continuam no repositório.
5. **Formato `.dbc`** (caso venham a ser usados): é DBF compactado (algoritmo PKWare/blast).
   Não dá para abrir direto com pandas — precisa de `pyreaddbc` ou da camada de baixo nível do
   `pysus` para descompactar.
6. **Viés de registro tardio.** Os dados do último ano (2025) podem estar incompletos — mencionar
   isso em qualquer análise que inclua 2025 na série temporal.

## Ambiente técnico

- Python (venv em `.venv/`) com: `pandas`, `numpy`, `pysus`, `pyreaddbc`, `dbfread`, `jupyter`,
  `duckdb`, `matplotlib`, `seaborn`, `statsmodels`, `scikit-learn`.
- **Importante:** o `python` do PATH do sistema (`C:\Python312`) não tem as libs do projeto —
  sempre usar `.\.venv\Scripts\python.exe` explicitamente ou ativar o venv antes.
- Notebook principal (ativo): `analise_inicial.ipynb` — pipeline completo de coleta/limpeza/enriquecimento.
- Datas nos campos vêm como texto `ddmmaaaa` (ex. `DTOBITO`, `DTNASC`) — já parseadas no parquet.
- Muitos campos usam `9`/`99`/`999` como "ignorado" — tratar como `NaN`, nunca como categoria válida.
- Os `.DBF` de `documentacao/Docs_Tabs_CID10.zip`/`Docs-Tabs-CID9.zip` estão em **cp850** (codepage
  de DOS) — usar esse encoding ao extrair (já feito em `tabelas_depara/`).
- `variavel/verificacao_campos.txt` (encoding cp1252) tem resumo de tipo, valores únicos, contagem
  e % de nulos para cada coluna — útil antes de escrever lógica de análise.

## Convenções de trabalho

- Idioma do projeto e da documentação: **português**.
- Preencher aqui decisões de escopo (SP vs. nacional, período final) assim que definidas com o
  orientador, para manter este arquivo como fonte de verdade do projeto.
