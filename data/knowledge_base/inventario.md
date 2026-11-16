# Inventário da base de conhecimento do EngMan Copilot

> Aqui não tem dado real da Motiva. Os documentos marcados como simulados foram inventados pelo grupo, com números plausíveis, e dizem isso no próprio arquivo. Os marcados como reais vêm do briefing do Challenge. A rodovia, os fabricantes (FAB-*), os modelos e os registros do FSM são fictícios.

Data de corte da base simulada: 31/10/2026. "Últimos 12 meses" = 01/11/2025 a 31/10/2026.

## 1. Documentos

| ID | Documento | Conteúdo e campos | Origem | Volume | Situação | Como o assistente acessa |
|---|---|---|---|---|---|---|
| DOC-01 | [Catálogo de ativos](catalogo_ativos.csv) | tag, tipo_ativo, componente, fabricante, modelo, km, sentido, localização, data_instalacao, vida_util_anos, impacto_seguranca, impacto_arrecadacao, criticidade, redundância, cobertura_redundante | simulado | 32 ativos (14 PTZ, 10 laços, 8 de pórtico) | pronto | Tool `consultar_ativo` (não indexado) |
| DOC-02 | [Plano de manutenção preventiva](plano_manutencao_preventiva.csv) | id_tarefa, tipo_ativo, tarefa, periodicidade, responsável, tempo estimado, recursos, condição de execução, POP | simulado | 13 tarefas | pronto | RAG (1 chunk por tarefa) |
| DOC-03 | [Histórico de falhas do FSM](historico_falhas_fsm.csv) | id_registro, tag, data, sintoma, categoria_causa, causa identificada, tipo de intervenção, tempo de reparo, peça trocada, condição ambiental, reincidência em 90 dias | simulado | 72 registros, 24 meses | pronto | Tools `consultar_historico_falhas` e `calcular_metricas_confiabilidade` (não indexado) |
| DOC-04 | [Procedimentos operacionais (POPs)](pops_simulados.md) | Pré-requisitos de segurança, passos numerados, critério de aceite | simulado | 3 POPs (CFT-01, LAC-02, PFF-01) | pronto | Tool `consultar_procedimento`, POP inteiro (não indexado) |
| DOC-05 | Fichas técnicas dos equipamentos | Função, falhas típicas, faixas de operação de câmera PTZ, laço + detector, antena de TAG e câmera OCR | real (descrição e falhas típicas do briefing) + simulado (especificações) | 3 fichas | Sprint 3 | RAG |
| DOC-06 | [Matriz de criticidade](matriz_criticidade.md) | Notas de impacto, criticidade A/B/C, efeito E1–E3, matriz P1–P4, modificador M, prazos, critérios proibidos | simulado | 1 | pronto | Copiada no bloco `# PRIORIZAÇÃO` do prompt (não indexado) |
| DOC-07 | [Glossário de siglas](glossario.md) | Sigla, significado, uso no projeto | real (siglas do briefing) | 28 siglas | pronto | RAG (1 chunk a cada ~10 siglas) |
| DOC-08 | Normas e requisitos de segurança | Resumo de NR-10, NR-35 e regras internas de trabalho em via, com link para o texto oficial e a data de vigência | real (resumo) | 1 | Sprint 3 | RAG, sempre citado como resumo |
| DOC-09 | Documentos do briefing Motiva | Contexto da empresa, ondas de inovação, Temas 1, 2 e 2.1 | real | 1 | disponível (briefing do Challenge) | RAG |
| DOC-10 | Boletins e relatos de campo | Relato textual do técnico, condição ambiental, TAG, data; técnico pseudonimizado (T-01, T-02…) | simulado | 10+ | Sprint 3 | RAG com filtro por TAG |
| DOC-11 | [Ativos-persona](ativos_persona.md) | História de 4 ativos com falhas encadeadas | simulado | 4 | pronto | Fora do RAG (ver nota) |

Sobre o DOC-08: a NR-10 ganhou nova redação (Portaria MTE nº 737, de 29/05/2026), que só passa a valer em 01/06/2027. Até lá vale a redação de 2004. O resumo precisa guardar as duas versões com o metadado de vigência, senão o RAG pode trazer a versão errada (risco RK12). A NR-35 também foi alterada em 2026 (Portaria MTE nº 1.259, de 15/07/2026).

Sobre o DOC-11: o arquivo descreve o que o assistente deveria concluir em cada caso. Se fosse indexado, o modelo leria a resposta esperada e o eval set ficaria contaminado. Ele serve para o grupo e para gerar dados, não para o RAG.

## 2. Uma fonte de verdade por dado

| Dado | Fonte única | Por que não vai para o vector store |
|---|---|---|
| Ficha e criticidade do ativo | `consultar_ativo` | Dado estruturado com consulta exata por TAG. Busca semântica poderia trazer o TAG vizinho (CFT-K416-N01 × S01). |
| Histórico de falhas e métricas | `consultar_historico_falhas` / `calcular_metricas_confiabilidade` | Contagem e conta precisam de todos os registros da janela, não dos "4 mais parecidos". |
| POP | `consultar_procedimento` | O chunking pode separar os pré-requisitos de segurança dos passos. O POP é entregue inteiro. |
| Regra de prioridade | Bloco `# PRIORIZAÇÃO` do prompt | Usada em quase todo turno e pequena (≈ 400 tokens). Fica no prompt para não depender da recuperação. |
| Plano, fichas, normas, glossário, briefing, boletins | ChromaDB | Texto descritivo consultado por assunto. É onde a busca semântica ajuda. |

Se uma tool e um trecho recuperado discordarem, vale a regra R7 do prompt: mostrar as duas fontes, usar a da tool e marcar `DIVERGÊNCIA DE CADASTRO`.

## 3. Como a base será usada no RAG

Tamanho estimado: o que já está pronto soma uns 14 mil tokens. Com o que entra na Sprint 3 (fichas, uns 2,5 mil; normas, 2 mil; briefing, 3 mil; boletins, 2,5 mil), a base vai a uns 24 mil tokens. Isso não cabe na janela de 16.384 junto com prompt, tools e conversa (Aula 02), e é por isso que dado estruturado vai por tool e texto vai por RAG.

Parte indexada: uns 12 mil tokens (DOC-02, 05, 07, 08, 09 e 10), perto de 40 trechos.

Tamanho do trecho: de 300 a 450 tokens, com sobreposição de 50. Os motivos:
1. O `multilingual-e5-base` lê no máximo 512 tokens por trecho e trunca o resto sem avisar. Chunks de até 450 deixam margem para o prefixo `passage: ` e para a diferença entre tokenizadores.
2. Recuperamos 4 trechos (top-k = 4): 4 × 450 = 1.800 tokens, que cabem no orçamento da [política de contexto](../../docs/decisoes_tecnicas.md#5-política-de-contexto-quantificada).
3. O corte é feito primeiro pela unidade lógica (uma tarefa do plano, um item de norma, um boletim) e só depois por tamanho. Assim cada trecho tem um id citável, como `[PM-CFT-01]`, que a regra de FORMATO exige.

Metadados guardados com cada trecho:

| Campo | Exemplo | Uso |
|---|---|---|
| `id_documento` / `id_item` | DOC-02 / PM-CFT-01 | Citação da fonte na resposta |
| `tipo_documento` | plano_preventivo, ficha_tecnica, norma_resumo, glossario, briefing, boletim_campo | Filtro por tipo de pergunta |
| `tipo_ativo` | cftv_ptz, laco_indutivo, portico_free_flow, geral | Filtro pelo tipo do TAG ativo |
| `tag` | CFT-K416-N01 (só boletins) | Filtro exato em boletins |
| `data` / `versao` | 2026-10-31 / 1.0-SIM | Descartar versão vencida (risco RK12) |
| `origem` | real, simulado | Transparência na resposta |

Consulta típica: `where = {"$and": [{"tipo_ativo": {"$in": [<tipo do TAG ativo>, "geral"]}}, {"versao": <vigente>}]}`.

## 4. O que fica fora da base (LGPD e confidencialidade)

| Fica fora | Motivo | Como garantimos |
|---|---|---|
| Nome, matrícula, CPF ou telefone de empregado ou terceirizado | Dado pessoal (LGPD, art. 5º, I) sem necessidade para o diagnóstico | Nos boletins, o técnico aparece só por código (T-01, T-02…) |
| Placas, imagens de veículos ou de ocupantes capturadas por OCR ou CFTV | Dado pessoal de terceiros; finalidade de cobrança, não de manutenção | O assistente trabalha com taxa de leitura agregada, nunca com leituras individuais |
| Dados de clientes de TAG e de cobrança | Dado pessoal e financeiro | Fora do escopo da tool de pórtico |
| Arrecadação por praça, valores de contrato, custos reais | Confidencialidade operacional e comercial | Impacto expresso só como nota de 0 a 3 |
| Qualquer dado real da Motiva | Regra do Challenge e confidencialidade | Só dado simulado no repositório; `data/real/` no `.gitignore` |
