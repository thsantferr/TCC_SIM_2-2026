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

## Fonte de dados: SIM/DATASUS

Os dados são microdados de **Declarações de Óbito (DO)** do **Sistema de Informações sobre
Mortalidade (SIM)**, mantido pelo Ministério da Saúde/DATASUS. Ver [resumo.md](resumo.md) para uma
explicação completa e acessível da base, dos campos e das tabelas auxiliares.

## Estrutura do repositório

```
TCC_SIM/
├── arquivo/                  # Dados brutos SIM em formato .dbc (DBF comprimido) — fonte LEGADA
│   └── DOEXT15.dbc … DOEXT25.dbc   # 11 arquivos anuais (2015 a 2025), não usados pelo pipeline atual
├── documentacao/              # Documentação oficial do SIM/DATASUS (dicionários de dados, CID)
│   ├── Estrutura_do_SIM_2025.pdf      # Dicionário de campos ATUAL (layout federal recente)
│   ├── Estrutura_SIM_para_CD.pdf      # Dicionário de campos versão 2019
│   ├── Estrutura_SIM_Anterior.pdf     # Dicionários de campos 2006 e pré-2005 (formato antigo)
│   ├── INTRO.pdf                      # Histórico do SIM, legislação, qualidade dos dados
│   ├── Legislacao_PDF.pdf, Portaria.pdf  # Base legal do SIM/SINASC
│   ├── MTAB16M.pdf                    # Lista de Tabulação para Mortalidade (agrupamento de CID)
│   ├── Docs-Tabs-CID9.zip             # Tabelas auxiliares para dados antigos em CID-9 (pré-1996)
│   └── Docs_Tabs_CID10.zip            # Tabelas auxiliares atuais: CADMUN, CID10, TABUF, TABPAIS, TABOCUP
├── tabelas_depara/             # Tabelas de-para extraídas dos DBF de documentacao/ (CSV ;, UTF-8)
│   ├── municipios.csv                 # CADMUN — código IBGE, nome, UF, região de saúde etc.
│   ├── estados.csv                    # TABUF — sigla, código, nome das 27 UFs
│   ├── cid10.csv                      # CID10 — 14.198 códigos CID-10 com descrição
│   ├── paises.csv                     # TABPAIS — códigos de país (naturalidade)
│   └── ocupacoes.csv                  # TABOCUP — códigos CBO com descrição
├── documento_tcc/              # Briefing acadêmico do TCC (docx)
├── analise_inicial.ipynb        # Notebook do pipeline de coleta/consolidação (ativo)
├── verificacao_exemplos.csv     # Amostra de linhas do df consolidado, gerada pelo notebook
├── dicionario_campos.md         # Dicionário dos 104 campos presentes no df consolidado do notebook
├── CLAUDE.md                    # Este arquivo
└── resumo.md                    # Explicação da base SIM em linguagem acessível
```

`documentacao.zip` na raiz é o zip original de onde a pasta `documentacao/` foi extraída — pode ser
removido depois de confirmar que o conteúdo extraído está completo.

## Pipeline de coleta atual (`analise_inicial.ipynb`)

O notebook **não lê os `.dbc` de `arquivo/`** — baixa os dados direto do DATASUS via `pysus.sim()`:

```python
from pysus import sim
anos_coleta = list(range(2000, 2026))
dt = sim(state='SP', year=anos_coleta)  # devolve caminhos locais de arquivos .parquet
```

Depois, para cada arquivo baixado: lê com `pd.read_parquet`, uppercase/strip nos nomes de coluna
(unifica variações tipo `contador`/`CONTADOR`), filtra `CODMUNRES` entre `350000` e `359999`
(município de residência em SP) e concatena tudo num único `df` — hoje com **104 colunas** para
2000–2025. Ver [dicionario_campos.md](dicionario_campos.md) para a explicação de cada campo.

### Comportamento observado do `pysus.sim(state='SP', ...)`

O parâmetro `state='SP'` **não garante** que o arquivo baixado já venha filtrado para SP — o PySUS
devolve tipos de arquivo diferentes conforme o ano:
- `Mortalidade_Geral_AAAA.parquet` e `DOxxOPEN.parquet` (dados preliminares "abertos") — volume
  compatível com **Brasil inteiro** (ex. ~1,5 milhão de linhas/ano), não só SP.
- `DOSPAAAA.parquet` — já vem filtrado para SP (~350 mil linhas/ano, compatível com o volume real
  do estado).

Por isso o filtro manual de `CODMUNRES` no notebook é **obrigatório**, não opcional — sem ele a base
final mistura óbitos de outras UFs (foi exatamente isso que gerou a primeira versão de
`verificacao_exemplos.csv`, com municípios de MG, AL, PE, RJ, CE, BA e GO).

## Pontos de atenção importantes (verificar com o(a) orientador(a) / grupo)

1. **Duplicidade em anos com dois arquivos.** Para alguns anos (ex. 2020, 2022) o `pysus.sim()`
   baixa **dois arquivos** (um tipo nacional/aberto + um `DOSP` já filtrado). Depois do filtro de
   `CODMUNRES`, os dois deveriam convergir para o mesmo subconjunto de óbitos de SP — vale checar se
   não há duplicação (mesmo óbito contado duas vezes) antes de seguir para a limpeza, por exemplo
   deduplicando por `CONTADOR`/`NUMERODO` dentro do mesmo ano.
2. **Campos não documentados.** `CODMUNCART`, `CODCART`, `NUMREGCART`, `DTREGCART`, `CRM` e
   `EXPDIFDATA` aparecem na base mas não constam em nenhum dos três dicionários oficiais em
   `documentacao/` — prováveis campos específicos das bases estaduais (`DOSP*`/SEADE). Ver
   inferências em [dicionario_campos.md](dicionario_campos.md#8-campos-de-cartório-e-outros-não-documentados-nos-dicionários-oficiais).
3. **Mistura de layouts de formulário.** Como a coleta agora cobre 2000–2025, a base junta o layout
   atual do SIM com campos de formulários antigos (pré-2010), incluindo pares que parecem ser o
   mesmo conceito com nome diferente entre vintages (`DTRECORIG`/`DTRECORIGA`,
   `NUDIASOBCO`/`NUDIASOBIN`, `FONTES`/`FONTESINF`) — decidir se serão unificados na limpeza.
4. **`arquivo/*.dbc` está desatualizado em relação ao pipeline.** Os 11 arquivos `.dbc` (2015–2025)
   não são mais lidos pelo notebook; se não houver motivo para mantê-los (ex. comparação/auditoria
   contra o download do PySUS), considerar remover ou documentar por que continuam no repositório.
5. **Formato `.dbc`** (caso ainda venham a ser usados): é DBF compactado (algoritmo PKWare/blast).
   Não dá para abrir direto com pandas — precisa de `pyreaddbc` ou da camada de baixo nível do
   `pysus` para descompactar.

## Ambiente técnico

- Python (venv em `.venv/`) já com: `pandas`, `numpy`, `pysus`, `pyreaddbc`, `dbfread`, `jupyter`.
- **Importante:** o `python` do PATH do sistema (`C:\Python312`) é diferente do venv do projeto —
  para rodar scripts com essas libs, usar `./.venv/Scripts/python.exe` explicitamente.
- Notebook principal (ativo): `analise_inicial.ipynb` — coleta via `pysus.sim()`, ver seção acima.
- Datas nos campos vêm como texto `ddmmaaaa` (ex. `DTOBITO`, `DTNASC`) — exigem parsing manual.
- Muitos campos usam `9`/`99`/`999` como "ignorado" — tratar como missing na limpeza, não como
  categoria numérica válida.
- Os `.DBF` de `documentacao/Docs_Tabs_CID10.zip`/`Docs-Tabs-CID9.zip` estão em **cp850** (codepage
  de DOS), não UTF-8/latin1 — usar esse encoding ao extrair (já feito em `tabelas_depara/`).

## Convenções de trabalho

- Idioma do projeto e da documentação: **português**.
- Preencher aqui decisões de escopo (SP vs. nacional, período final) assim que definidas com o
  orientador, para manter este arquivo como fonte de verdade do projeto.
