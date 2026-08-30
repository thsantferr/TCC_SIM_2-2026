# TCC_SIM

TCC (monografia) do curso de **Ciência de Dados** — UNIVESP, Grupo TCC530-BCD-Turma 002,
orientador Cassio Silva Takarada.

## Sobre o projeto

- **Tema:** transição epidemiológica e padrões de mortalidade no Estado de São Paulo, usando
  microdados do DATASUS (SIM — Sistema de Informações sobre Mortalidade), entre 2000 e 2025.
- **Título provisório:** "Análise e Visualização da Transição Epidemiológica no Estado de São
  Paulo: Um Estudo dos Óbitos Registrados entre 2000 e 2025 via Ciência de Dados".
- **Problema de pesquisa:** como técnicas de Ciência de Dados podem revelar padrões, tendências e
  anomalias nas causas de óbito no Estado de São Paulo (2000–2025), evidenciando a transição
  epidemiológica e o impacto de eventos sanitários agudos.
- Não é continuação de um Projeto Integrador nem envolve pesquisa de campo — é uma análise
  documental/estatística de dados públicos já existentes.
- Briefing acadêmico completo em
  [documento_tcc/Informações preliminares para projeto TCC_Quinzena 0.docx](documento_tcc/Informações%20preliminares%20para%20projeto%20TCC_Quinzena%200.docx).

### Cronograma

1. Levantamento bibliográfico e aquisição de dados
2. Pipeline de ETL (conversão, limpeza, padronização)
3. Análise exploratória e modelagem (séries temporais, agrupamento)
4. Dashboard interativo (produto analítico)
5. Redação final e revisão (ABNT)

## Estrutura do repositório

| Caminho | O que é |
|---|---|
| [analise_inicial.ipynb](analise_inicial.ipynb) | Notebook ativo — coleta os dados (via `pysus`), filtra para SP e consolida numa base única |
| `documentacao/` | Dicionários oficiais do SIM/DATASUS, CID-10, legislação (PDFs e tabelas auxiliares em `.zip`) |
| `tabelas_depara/` | Tabelas de-para prontas em CSV (municípios, estados, CID-10, países, ocupações) |
| `documento_tcc/` | Briefing acadêmico do TCC (docx) |
| [resumo.md](resumo.md) | O que é o SIM e como a Declaração de Óbito é estruturada, em linguagem acessível |
| [dicionario_campos.md](dicionario_campos.md) | O que significa cada um dos campos que aparecem na base consolidada |
| [CLAUDE.md](CLAUDE.md) | Contexto técnico detalhado do pipeline (para quem for mexer no código, humano ou IA) |

## Como começar

1. **Ambiente**: já existe um venv em `.venv/` com `pandas`, `numpy`, `pysus`, `pyreaddbc`, `dbfread`
   e `jupyter` instalados. Ative-o ou aponte o kernel do Jupyter para
   `.venv/Scripts/python.exe` (Windows) — o `python` do PATH do sistema é outro e não tem essas libs.
2. **Rodar o notebook**: abra [analise_inicial.ipynb](analise_inicial.ipynb) e execute as células em
   ordem. Ele baixa os dados de 2000 a 2025 direto do DATASUS (pode demorar/gerar bastante tráfego de
   rede na primeira vez) e monta o `df` consolidado, já filtrado para SP.
3. **Entender os dados antes de tratar**: leia [resumo.md](resumo.md) (o que é o SIM) e
   [dicionario_campos.md](dicionario_campos.md) (o que é cada campo) antes de escrever qualquer
   lógica de limpeza — vários campos só fazem sentido para subconjuntos específicos de óbitos
   (fetal/infantil, óbito investigado etc.), e boa parte dos nulos é esperada, não é erro.
4. **Traduzir códigos**: use as tabelas em `tabelas_depara/` para transformar `CODMUNRES` em nome de
   município, `CAUSABAS` em descrição da doença (CID-10), `OCUP` em ocupação etc.

## O que precisamos entender/decidir antes de avançar

- **Duplicidade em anos com dois arquivos.** Para alguns anos (ex. 2020, 2022) o download traz dois
  arquivos diferentes para o mesmo ano; depois do filtro por SP eles deveriam representar o mesmo
  conjunto de óbitos — precisa confirmar que não há duplicação antes de calcular qualquer estatística
  (ver nota em [CLAUDE.md](CLAUDE.md#pontos-de-atenção-importantes-verificar-com-oa-orientadora--grupo)).
- **Campos sem documentação oficial.** `CODMUNCART`, `CODCART`, `NUMREGCART`, `DTREGCART`, `CRM` e
  `EXPDIFDATA` aparecem na base mas não constam nos dicionários oficiais do SIM — provavelmente
  específicos das bases estaduais de SP (SEADE). Ver detalhes em
  [dicionario_campos.md](dicionario_campos.md).
- **Mistura de formulários ao longo do tempo.** Cobrindo 2000–2025, a base junta o layout atual do
  SIM com campos de formulários mais antigos (pré-2010) — alguns conceitos existem em duas colunas
  com nomes diferentes conforme o ano (ex. `DTRECORIG`/`DTRECORIGA`). Decidir se serão unificados.
- **Definição da causa básica para "transição epidemiológica".** O objetivo do TCC exige agrupar
  `CAUSABAS` em categorias amplas (infecciosas × crônico-degenerativas × causas externas), não
  analisar o código CID-10 isolado — usar `tabelas_depara/cid10.csv`/`documentacao/MTAB16M.pdf` como
  base para essa classificação.
- **O que fazer com `arquivo/*.dbc`.** Não é mais lido pelo pipeline atual — decidir se mantém como
  fonte alternativa/auditoria ou se remove do repositório.
- **Cinco referências bibliográficas** já estão definidas no documento do TCC (Saúde Brasil
  2017/2018/2020-2021, artigo sobre transição da morbimortalidade e artigo sobre qualidade do
  SIM/SINASC) — conferir se ainda cobrem o recorte final da pesquisa.

## Convenções

- Idioma do projeto e da documentação: **português**.
- Decisões de escopo definidas com o orientador devem ser registradas em [CLAUDE.md](CLAUDE.md) para
  manter aquele arquivo como fonte de verdade técnica do projeto.
