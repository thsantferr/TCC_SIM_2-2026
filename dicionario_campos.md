# Dicionário de Campos — Base Consolidada (2000–2025, SP)

Este documento explica **cada um dos 104 campos** que aparecem na base depois do pipeline atual do
[analise_inicial.ipynb](analise_inicial.ipynb): download via `pysus.sim(state='SP', year=2000..2025)`,
nomes de coluna padronizados em maiúsculas, filtro por `CODMUNRES` entre `350000` e `359999` (SP) e
concatenação de todos os arquivos anuais/variantes.

Como o notebook agora baixa também anos antigos (2000–2014), a base final mistura o **layout atual**
do SIM (documentado em `documentacao/Estrutura_do_SIM_2025.pdf`) com campos de **formulários
antigos** (`documentacao/Estrutura_SIM_para_CD.pdf`, versão 2019, e
`documentacao/Estrutura_SIM_Anterior.pdf`, formato 2006/pré-2005) e alguns campos específicos das
bases estaduais de SP (prefixo `DOSP*`, de origem SEADE) que **não aparecem em nenhum dos três
dicionários oficiais** — esses estão marcados como "não documentado" abaixo, com uma inferência
razoável a partir do nome.

> Nomenclatura: os nomes abaixo já estão em maiúsculas, pois o notebook faz
> `df.columns = [c.upper().strip() for c in df.columns]` para unificar variações como
> `contador`/`CONTADOR` vindas de arquivos diferentes.

## 1. Identificação do óbito e do falecido

| Campo | Descrição | Observações |
|---|---|---|
| `CONTADOR` | Identificador sequencial interno do registro na base | Antes da uniformização de maiúsculas aparecia como `contador` em alguns arquivos e `CONTADOR` em outros — hoje é uma coluna só |
| `ORIGEM` | Origem do registro: 1-Oracle, 2-Banco estadual via FTP, 3-Banco SEADE, 9-Ignorado | Útil para saber de qual pipeline/fonte veio cada linha |
| `TIPOBITO` | Tipo do óbito: 1-Fetal, 2-Não fetal | Campo obrigatório |
| `DTOBITO` | Data do óbito, texto `ddmmaaaa` | Base de qualquer série temporal — precisa de parsing |
| `HORAOBITO` | Hora do óbito, formato 24h (`hhmm`) | |
| `NATURAL` | Naturalidade (país e UF de nascimento), código de 3 dígitos | Se estrangeiro, código de país (ver `tabelas_depara/paises.csv`) |
| `CODMUNNATU` | Código do município de naturalidade (7 dígitos) | Traduzir com `tabelas_depara/municipios.csv` |
| `DTNASC` | Data de nascimento, texto `ddmmaaaa` | Em óbito fetal deve ser igual a `DTOBITO` |
| `IDADE` | Idade composta: 1º dígito = unidade (1=min, 2=hora, 3=mês, 4=ano, 5=>100 anos), 2 dígitos seguintes = quantidade | Precisa de conversão para "idade em anos" |
| `SEXO` | Sexo: 1/M-masculino, 2/F-feminino, 9/I/0-ignorado | Códigos variam entre formulário novo (1/2/9) e antigo (M/F/I) |
| `RACACOR` | Raça/cor: 1-Branca, 2-Preta, 3-Amarela, 4-Parda, 5-Indígena | |
| `ESTCIV` | Situação conjugal: 1-Solteiro, 2-Casado, 3-Viúvo, 4-Separado/divorciado, 5-União estável, 9-Ignorado | |
| `ESC` | Escolaridade em anos de estudo (formulário antigo): 1-Nenhuma, 2-1 a 3, 3-4 a 7, 4-8 a 11, 5-12+, 9-Ignorado | Coexiste com `ESC2010`; usar o que estiver preenchido conforme o ano |
| `ESC2010` | Escolaridade por nível (formulário 2010+): 0-Sem escolaridade, 1-Fund. I, 2-Fund. II, 3-Médio, 4-Superior incompleto, 5-Superior completo, 9-Ignorado | |
| `SERIESCFAL` | Última série escolar concluída pelo falecido (1 a 8) | |
| `ESCFALAGR1` | Escolaridade do falecido agregada (formulário 2010+, categorias 00–12) | Versão mais detalhada de `ESC2010` |
| `OCUP` | Ocupação habitual do falecido, código CBO 2002 | Traduzir com `tabelas_depara/ocupacoes.csv`; só preenchido a partir de ~5 anos de idade |

## 2. Residência e ocorrência

| Campo | Descrição | Observações |
|---|---|---|
| `CODMUNRES` | Código do município de residência (6–7 dígitos) | Campo usado no notebook para filtrar SP (`350000`–`359999`) |
| `CODBAIRES` | Código do bairro de residência | Só existe em formulários antigos (pré-2010); não é usado nos layouts atuais |
| `LOCOCOR` | Local de ocorrência do óbito: 1-Hospital, 2-Outro estab. saúde, 3-Domicílio, 4-Via pública, 5-Outros, 6-Aldeia indígena, 9-Ignorado | |
| `CODESTAB` | Código do estabelecimento de saúde (CNES) | Obrigatório quando `LOCOCOR` é 1 ou 2 |
| `ESTABDESCR` | Nome/descrição textual do estabelecimento de saúde | Aparece nas bases estaduais (`DOSP*`); ausente no layout federal atual |
| `CODMUNOCOR` | Código do município de ocorrência do óbito | |
| `CODBAIOCOR` | Código do bairro de ocorrência | Só em formulários antigos (mesma observação de `CODBAIRES`) |
| `UFINFORM` | Código IBGE da UF que informou o registro | Campo de formulários antigos (pré-2010); útil para conferir consistência com `CODMUNRES`/`CODMUNOCOR` |

## 3. Bloco materno/fetal — só preenchido para óbito fetal ou < 1 ano

| Campo | Descrição | Observações |
|---|---|---|
| `IDADEMAE` | Idade da mãe em anos | |
| `ESCMAE` | Escolaridade da mãe em anos (formulário antigo) | Equivalente a `ESC` para a mãe |
| `ESCMAE2010` | Escolaridade da mãe por nível (2010+) | Equivalente a `ESC2010` para a mãe |
| `ESCMAEAGR1` | Escolaridade da mãe agregada (2010+, categorias 00–12) | |
| `SERIESCMAE` | Última série escolar concluída pela mãe | |
| `OCUPMAE` | Ocupação habitual da mãe, código CBO 2002 | Se "aposentada", registra a ocupação anterior |
| `QTDFILVIVO` | Número de filhos vivos | 9 = ignorado |
| `QTDFILMORT` | Número de filhos mortos (não conta o óbito da própria DO) | 9 = ignorado |
| `SEMAGESTAC` | Semanas de gestação (2 dígitos) | 9 = ignorado |
| `GESTACAO` | Faixa de semanas de gestação (formulário antigo, 6 categorias) | Coexiste com `SEMAGESTAC` |
| `GRAVIDEZ` | Tipo de gravidez: 1-Única, 2-Dupla, 3-Tripla ou mais, 9-Ignorada | |
| `PARTO` | Tipo de parto: 1-Vaginal, 2-Cesáreo, 9-Ignorado | |
| `OBITOPARTO` | Momento do óbito em relação ao parto: 1-Antes, 2-Durante, 3-Depois, 9-Ignorado | |
| `PESO` | Peso ao nascer em gramas | Óbitos fetais e < 28 dias |
| `NUMERODN` | Número da Declaração de Nascido Vivo (DN) correspondente | Campo do formulário 2019+ |
| `TPMORTEOCO` | Situação (pós-)gestacional em que ocorreu o óbito: 1-Na gravidez, 2-No parto, 3-No abortamento, 4-Até 42 dias após o parto, 5-43 dias a 1 ano após, 8-Fora desses períodos, 9-Ignorado | Preenchido para óbito de mulher em idade fértil |
| `OBITOGRAV` | Óbito ocorreu durante a gravidez: 1-Sim, 2-Não, 9-Ignorado | |
| `OBITOPUERP` | Óbito no puerpério: 1-Sim (até 42 dias), 2-Sim (43d–1 ano), 3-Não, 9-Ignorado | |
| `MORTEPARTO` | Momento do óbito em relação ao parto, **após investigação** | Ficha de investigação de óbito materno/infantil |
| `TPOBITOCOR` | Momento mais preciso da ocorrência do óbito, após investigação (10 categorias, ex. "durante a gestação", "no puerpério") | Idem |

## 4. Condições, exames e causas do óbito (CID)

| Campo | Descrição | Observações |
|---|---|---|
| `ASSISTMED` | Recebeu assistência médica durante a doença que causou a morte: 1-Sim, 2-Não, 9-Ignorado | |
| `EXAME` | Realização de exame complementar: 1-Sim, 2-Não, 9-Ignorado | |
| `CIRURGIA` | Realização de cirurgia: 1-Sim, 2-Não, 9-Ignorado | |
| `NECROPSIA` | Realização de necropsia: 1-Sim, 2-Não, 9-Ignorado | |
| `LINHAA` | Causa terminal — doença/estado mórbido que causou diretamente a morte (linha A do atestado) | Campo obrigatório; código(s) CID |
| `LINHAB` | Causa antecedente/consequencial que produziu a causa da linha A (linha B) | |
| `LINHAC` | Idem, mais um nível antecedente (linha C) | |
| `LINHAD` | Causa básica declarada no atestado — estado mórbido inicial da cadeia (linha D) | |
| `LINHAII` | Causas contribuintes — outras condições que contribuíram, fora da cadeia principal (parte II) | |
| `CAUSABAS` | **Causa básica do óbito**, já selecionada/codificada em CID-10 | Campo mais importante para análise epidemiológica — ver `tabelas_depara/cid10.csv` |
| `CAUSABAS_O` | Causa básica informada antes da resseleção automática | Útil para auditar mudanças feitas pelo sistema seletor de causa básica |
| `CB_PRE` | Causa selecionada sem resseleção (versão do "novo SCB" – Seletor de Causa Básica) | |
| `CB_ALT` | Variável de sistema/causa básica alterada | Uso interno do processamento do SIM |
| `TP_ALTERA` | Código do tipo de crítica/alteração aplicada à causa básica (ex.: 02-CausaBas em branco, 08-CID implausível, 12-óbito não fetal com causa fetal etc.) | Ver lista completa em `documentacao/Estrutura_do_SIM_2025.pdf` |
| `ATESTADO` | Texto com todos os CIDs informados no atestado (linhas A–D e II combinadas) | Campo textual, útil para conferência manual |
| `ATESTANTE` | Condição do médico que assinou o atestado: 1-Assistente, 2-Substituto, 3-IML, 4-SVO, 5-Outro | |
| `COMUNSVOIM` | Código do município/UF do SVO ou IML que atestou | Obrigatório se `ATESTANTE` = 3 ou 4 |
| `DTATESTADO` | Data em que o atestado foi assinado | |
| `MAT_CLAS` | Classificação de óbito materno (campo mais recente do layout) | |
| `COVID_CLAS` | Classificação relacionada a óbito por/com COVID-19 | Campo introduzido a partir da pandemia |

## 5. Causas externas (acidentes, violência)

| Campo | Descrição | Observações |
|---|---|---|
| `CIRCOBITO` | Tipo de morte violenta: 1-Acidente, 2-Suicídio, 3-Homicídio, 4-Outros, 9-Ignorado | Só se aplica a causas externas |
| `ACIDTRAB` | Indica se foi acidente de trabalho: 1-Sim, 2-Não, 9-Ignorado | |
| `FONTE` | Fonte da informação usada para preencher `CIRCOBITO`/`ACIDTRAB`: 1-Ocorrência policial, 2-Hospital, 3-Família, 4-Outra, 9-Ignorado | |
| `CAUSAMAT` | Código CID da causa externa associada a uma causa materna | Campo calculado pelo sistema |

## 6. Processamento, lote e controle de qualidade

| Campo | Descrição | Observações |
|---|---|---|
| `NUMEROLOTE` | Número do lote de processamento do registro | Campo calculado pelo sistema |
| `DTCADASTRO` | Data de cadastro do óbito no sistema | |
| `STCODIFICA` | Se a instalação era "codificadora" (S/N) | |
| `CODIFICADO` | Se o formulário foi codificado (S/N) | |
| `VERSAOSIST` | Versão do sistema SIM usada no cadastro | |
| `VERSAOSCB` | Versão do Seletor de Causa Básica usada | |
| `DTRECEBIM` | Data de recebimento do registro no nível central | |
| `DTRECORIGA` | Data do recebimento original da DO (campo criado no tratamento federal) | |
| `DTRECORIG` | Mesma finalidade de `DTRECORIGA` (data de recebimento original) | Nome usado em vintages mais antigas do pipeline; provavelmente a mesma informação com grafia diferente |
| `DIFDATA` | Diferença em dias entre `DTOBITO` e a data de recebimento original | Indicador de atraso/oportunidade do registro |
| `OPOR_DO` | Indicador de "oportunidade" da DO (campo criado no tratamento) | Correlaciona com `DIFDATA` |
| `EXPDIFDATA` | Não documentado nos dicionários oficiais consultados | Nome sugere uma variante/expansão de `DIFDATA`; **100% nulo** na base atual — provavelmente não é preenchido para os arquivos baixados |

## 7. Investigação de óbito materno/infantil

| Campo | Descrição | Observações |
|---|---|---|
| `DTINVESTIG` | Data da investigação do óbito | Ficha de investigação de óbito materno/infantil |
| `FONTEINV` | Fonte da investigação: 1-Comitê de Morte Materna/Infantil, 2-Visita domiciliar, 3-Estabelecimento/prontuário, 4-Outros bancos, 5-SVO, 6-IML, 7-Outra, 8-Múltiplas, 9-Ignorado | |
| `DTCADINV` | Data do cadastro da investigação | |
| `DTCONINV` | Data de conclusão da investigação | |
| `DTCADINF` | Data de cadastro — indica se a investigação **infantil** foi feita | Variante de `DTCADINV` para óbito infantil |
| `NUDIASOBCO` | Diferença em dias entre `DTOBITO` e a conclusão da investigação | |
| `NUDIASOBIN` | Mesma ideia de `NUDIASOBCO`, variante para óbito **infantil** | Aparece nas bases estaduais (`DOSP*`) |
| `NUDIASINF` | Indicador de dias relacionado à investigação infantil | Idem, específico de `DOSP*` |
| `FONTES` | Combinação de flags (S/X) indicando quais fontes de investigação foram usadas | |
| `FONTESINF` | Variante de `FONTES` para investigação infantil | Específico de `DOSP*` |
| `TPRESGINFO` | Se a investigação resgatou/corrigiu alguma causa: 01-Não, 02-Sim (resgatou), 03-Sim (corrigiu) | |
| `TPNIVELINV` | Nível do investigador: E-Estadual, R-Regional, M-Municipal | |
| `TPPOS` | Indica se o óbito foi investigado: 1-Sim, 2-Não | |
| `DTCONCASO` | Data de conclusão do caso investigado | |
| `ALTCAUSA` | Indica se houve alteração da causa do óbito após investigação: 1-Sim, 2-Não | |
| `STDOEPIDEM` | Status de "DO Epidemiológica": 1-Sim, 0-Não | |
| `STDONOVA` | Status de "DO Nova": 1-Sim, 0-Não | |

## 8. Campos de cartório e outros não documentados nos dicionários oficiais

Estes campos aparecem na base (provavelmente vindos das variantes `DOSP*`/SEADE) mas **não constam**
em nenhum dos três dicionários oficiais em `documentacao/`. A descrição abaixo é uma inferência a
partir do nome do campo — vale confirmar com a documentação do SEADE ou tratar como campo auxiliar
de baixa prioridade.

| Campo | Inferência | Confiança |
|---|---|---|
| `CODMUNCART` | Código do município do cartório onde o óbito foi registrado | Inferido do nome |
| `CODCART` | Código do cartório (unidade específica) onde o óbito foi registrado | Inferido do nome |
| `NUMREGCART` | Número do registro do óbito no livro do cartório | Inferido do nome |
| `DTREGCART` | Data do registro do óbito no cartório | Inferido do nome |
| `CRM` | Número de registro do médico atestante no Conselho Regional de Medicina | Inferido do nome |
| `TPASSINA` | O dicionário `Estrutura_SIM_Anterior.pdf` lista este campo sem descrição (só o nome) | Possivelmente relacionado ao tipo de assinatura do atestado (manual/digital) |

## Observações gerais sobre a base

- **Valores "ignorados"**: campos com `9`, `99` ou `999` normalmente significam "ignorado" — tratar
  como dado faltante, não como categoria válida.
- **Percentual de nulos alto é esperado** em quase todo o bloco materno/fetal (seção 3) e no bloco de
  investigação (seção 7), porque só se aplicam a subconjuntos específicos de óbitos (fetais/infantis,
  ou óbitos efetivamente investigados) — não é, por si só, um problema de qualidade dos dados.
- **Campos duplicados por nome com sufixo diferente** (`DTRECORIG`/`DTRECORIGA`, `NUDIASOBCO`/
  `NUDIASOBIN`, `FONTES`/`FONTESINF`) provavelmente representam o mesmo conceito em vintages diferentes
  do layout — vale decidir, na limpeza, se serão unificados numa única coluna (coalescendo os valores)
  ou mantidos separados com justificativa no texto do TCC.
- Ver [resumo.md](resumo.md) para a explicação conceitual da base (o que é o SIM, como a DO é
  estruturada) e [CLAUDE.md](CLAUDE.md) para o contexto técnico do projeto.
