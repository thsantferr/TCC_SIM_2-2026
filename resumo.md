# Resumo da Base SIM/DATASUS

Este documento explica, em linguagem acessível, o que é a base de dados usada no TCC, como ela é
estruturada e o que cada arquivo/tabela representa. Serve como referência rápida para quem for
mexer nos dados sem precisar reler os PDFs originais em `documentacao/`.

## 1. O que é o SIM

O **Sistema de Informações sobre Mortalidade (SIM)** é o sistema oficial do Ministério da Saúde
que registra **todos os óbitos ocorridos no Brasil**, a partir da **Declaração de Óbito (DO)** — o
documento que todo cartório exige para autorizar um sepultamento.

- Existe desde 1976 e roda de forma padronizada em todo o país.
- Cada linha da base = uma DO = um óbito.
- As causas de morte são codificadas segundo a **CID** (Classificação Internacional de Doenças) —
  hoje a 10ª revisão (CID-10); antes de 1996 usava-se a CID-9.
- Cobertura não é 100%: há **sub-registro** (óbitos que nunca chegam a ser declarados) e uma fração
  de causas **"mal definidas"**, mais comuns em regiões com pior infraestrutura de registro civil.
  Historicamente Sul e Sudeste têm a melhor cobertura e menor sub-registro do país.

## 2. Os arquivos de dados deste projeto

Pasta `arquivo/`: 11 arquivos `.dbc` (DBF comprimido), um por ano, de **2015 a 2025**
(`DOEXT15.dbc` … `DOEXT25.dbc`). Cada arquivo contém um registro por óbito daquele ano.

> (1) a amostra de verificação sugere que os dados cobrem **o Brasil todo**, não só SP; 
> (2) faltam os anos 2000–2014 previstos no escopo do TCC.

`.dbc` não é um formato comum — é um `.dbf` (dBASE) comprimido com um algoritmo específico do
DATASUS. Para abrir em Python, usa-se `pysus` ou `pyreaddbc` (já instalados no ambiente virtual do
projeto); não dá para ler direto com `pandas.read_csv` ou similar.

## 3. Como a Declaração de Óbito é organizada

A DO em papel é dividida em blocos, que se refletem nos campos da base:

| Bloco | Conteúdo |
|---|---|
| **Identificação** | tipo de óbito (fetal/não fetal), data/hora, naturalidade, data de nascimento, idade, sexo, raça/cor, estado civil, escolaridade, ocupação |
| **Residência** | município de residência do falecido |
| **Ocorrência** | local (hospital, domicílio, via pública...), estabelecimento de saúde, município de ocorrência |
| **Fetal / menor de 1 ano** | dados da mãe (idade, escolaridade, ocupação, nº de filhos), gestação, tipo de parto, peso ao nascer — só preenchido para óbitos fetais ou de crianças com menos de 1 ano |
| **Condições e causas da morte** | assistência médica recebida, necropsia, e as **causas de morte** propriamente ditas (linhas A/B/C/D do atestado + causa básica) |
| **Causas externas** | preenchido só quando a morte foi violenta/não natural: acidente, suicídio, homicídio, acidente de trabalho |
| **Campos de investigação/sistema** | datas de cadastro, recebimento, investigação de óbito materno/infantil, versão do sistema, indicadores de qualidade do preenchimento |

### Campos-chave para a análise epidemiológica

- **`CAUSABAS`** — causa básica do óbito (o principal campo para estudar padrões de mortalidade),
  já codificada em CID-10 (ex.: `I10` = hipertensão, `J189` = pneumonia).
- **`DTOBITO`** — data do óbito (texto `ddmmaaaa`) → base para séries temporais.
- **`IDADE`** — codificada em 2 partes: 1º dígito = unidade (1=minuto, 2=hora, 3=mês, 4=ano,
  5=mais de 100 anos), 2 dígitos seguintes = quantidade. Precisa de conversão para "idade em anos".
- **`SEXO`**, **`RACACOR`**, **`ESTCIV`**, **`ESC2010`** — perfil demográfico.
- **`CODMUNRES`** / **`CODMUNOCOR`** — município de residência / de ocorrência (código IBGE de 6-7
  dígitos; os 2 primeiros dígitos identificam a UF — SP = `35`).
- **`LOCOCOR`** — local da morte (hospital, domicílio, via pública etc.).
- Campos com **`9`, `99`, `999`** geralmente significam **"ignorado"** — devem ser tratados como
  dado faltante, não como categoria válida.

O dicionário de campos completo (nome, tipo, tamanho, valores válidos e regras de preenchimento)
está em `documentacao/Estrutura_do_SIM_2025.pdf` — essa é a versão que bate com os dados extraídos
(confirmado comparando com `verificacao_exemplos.csv`). As versões `Estrutura_SIM_para_CD.pdf`
(2019) e `Estrutura_SIM_Anterior.pdf` (2006 e pré-2005) só são relevantes se o projeto vier a usar
dados anteriores a 2015 — o layout de campos mudou ao longo do tempo (por exemplo, `SEXO` era
`M`/`F`/`I` no formato antigo e passou a ser `1`/`2`/`9`).

## 4. Tabelas auxiliares (para "traduzir" os códigos)

Estão em `documentacao/Docs_Tabs_CID10.zip` (tabelas atuais, formato CID-10) e
`Docs-Tabs-CID9.zip` (equivalentes antigas, só relevantes para dados pré-1996):

| Tabela | O que faz |
|---|---|
| `CID10.DBF` | 14.198 códigos CID-10 com descrição — traduz `CAUSABAS` para o nome da doença |
| `CIDCAP10.DBF` | Os 21 capítulos da CID-10 (grandes grupos de causas, ex. "Doenças do aparelho circulatório") |
| `CADMUN.DBF` | Cadastro de municípios do Brasil (5.652 municípios): nome, UF, região de saúde, lat/long/altitude — usar para traduzir `CODMUNRES`/`CODMUNOCOR` |
| `TABUF.DBF` | As 27 UFs: sigla, código, nome |
| `TABPAIS.DBF` | Códigos de país (para naturalidade de estrangeiros) |
| `TABOCUP.DBF` | Códigos CBO (ocupação) e descrição |

`MTAB16M.pdf` é a **Lista de Tabulação para Mortalidade** — um agrupamento padronizado dos milhares
de códigos CID-10 em categorias resumidas, pensado para comparabilidade estatística (equivalente
moderno da antiga "Lista Brasileira CID-BR" citada em `INTRO.pdf`). Útil se a análise for por
grandes grupos de causa em vez de código CID individual.

## 5. Contexto legal e histórico (se precisar para a fundamentação teórica)

- `documentacao/INTRO.pdf`: histórico do SIM (criado em 1975/76), papel do Centro Brasileiro de
  Classificação de Doenças (CBCD/USP), fluxo de coleta (cartório → secretaria estadual → DATASUS),
  e discussão sobre qualidade/sub-registro dos dados — bom material para a seção de metodologia.
- `documentacao/Legislacao_PDF.pdf` e `documentacao/Portaria.pdf`: base legal do SIM/SINASC (ex.:
  Lei 6.015/1973 sobre registros civis; Portaria 938/2002 sobre incentivo ao registro civil de
  nascimento em maternidades).

## 6. Amostra de conferência

`verificacao_exemplos.csv` (na raiz do projeto) tem 10 óbitos de exemplo, um por coluna, com todos
os ~94 campos — útil para conferir visualmente se a extração/leitura dos `.dbc` está trazendo os
valores esperados antes de rodar o pipeline completo.
