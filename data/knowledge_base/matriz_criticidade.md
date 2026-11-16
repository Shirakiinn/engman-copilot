# Matriz de criticidade e regra de prioridade do EngMan Copilot

> SIMULADO. Critérios definidos pelo grupo para fins acadêmicos, a partir das dores do briefing da Motiva. Não é um documento oficial da Motiva.

Este documento é a especificação legível da regra de priorização. O texto operacional da regra fica no bloco `# PRIORIZAÇÃO` do [system prompt](../../prompts/system_prompt.md) e é carregado inteiro em toda chamada. Por isso este arquivo não é indexado no vector store: assim existe uma única versão da regra no contexto do modelo. Qualquer mudança aqui precisa ser replicada no prompt no mesmo commit.

## 1. Criticidade do ativo (A, B ou C)

Cada ativo do catálogo recebe duas notas de impacto, de 0 a 3:

| Nota | Impacto na segurança viária | Impacto na arrecadação |
|---|---|---|
| 3 | Ponto crítico (serra, trevo, histórico de acidentes) sem outra câmera cobrindo | Cobrança depende só deste equipamento (faixa de pesados, pórtico Free Flow) |
| 2 | Trecho comum monitorado | Tarifação incorreta possível em faixa de leves |
| 1 | Trecho com cobertura total de outra câmera | Impacto indireto na praça |
| 0 | Sem relação | Sem relação |

Regra de maior impacto: criticidade A se alguma nota for 3; B se a maior nota for 2; C se a maior nota for 0 ou 1.
As notas e a criticidade de cada TAG estão nas colunas `impacto_seguranca`, `impacto_arrecadacao` e `criticidade` de [`catalogo_ativos.csv`](catalogo_ativos.csv), e o assistente as obtém pela tool `consultar_ativo`.

## 2. Efeito da falha

| Código | Efeito | Exemplos |
|---|---|---|
| E1 | Perda total da função sem cobertura redundante | Câmera de serra sem imagem; laço sem pulso; antena e OCR da mesma faixa fora |
| E2 | Degradação da função, ou perda total com cobertura redundante | Imagem com halo noturno; classificação de eixos incorreta; taxa OCR abaixo de 97%; antena fora com OCR da mesma faixa operando |
| E3 | Sem impacto funcional imediato | Achado de inspeção, desgaste, preset levemente deslocado |

## 3. Prioridade = criticidade × efeito

|   | E1 | E2 | E3 |
|---|---|---|---|
| A | P1 | P2 | P3 |
| B | P2 | P3 | P4 |
| C | P3 | P4 | P4 |

Modificador M (aplicado uma única vez, até o teto P1): risco à segurança viária em curso no trecho do ativo (acidente, interdição, neblina) e ativo com impacto na segurança ≥ 2 → sobe um nível. O modificador não se aplica a ativos só de arrecadação.

| Prioridade | Prazo de atendimento |
|---|---|
| P1 | até 4 h |
| P2 | até 24 h |
| P3 | até 72 h |
| P4 | próxima janela preventiva (até 30 dias) |

Desempate: entre ordens de mesma prioridade, vale a ordem de abertura.

## 4. Tipo de intervenção

| Tipo | Quando recomendar |
|---|---|
| Corretiva | Falha funcional presente (E1 ou E2) com causa conhecida ou provável |
| Inspeção | Sintoma intermitente sem causa confirmada, ou achado que precisa de verificação em campo |
| Preventiva | Tarefa do plano por calendário (ou antecipação de tarefa existente) |
| CBM | Propor avaliação quando houver 3+ falhas da mesma `categoria_causa` em 12 meses e existir variável monitorável que antecede a falha (taxa de leitura OCR, divergência de classificação, resistência de isolamento) |

## 5. Critérios proibidos (mitigação do risco de viés)

A prioridade nunca pode considerar: região, praça, fabricante, quem abriu o chamado, insistência do solicitante ou "sensação" de urgência. Toda recomendação mostra a combinação usada (ex.: `A × E2, sem M → P2`), o que torna a decisão auditável.
