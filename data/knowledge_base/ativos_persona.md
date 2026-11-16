# Ativos-persona: histórico completo para o dataset simulado

> SIMULADO. Quatro ativos fictícios com história de falhas coerente, pensados para exercitar diagnóstico, reincidência, CBM e decisão de CAPEX. Os registros estão em [`historico_falhas_fsm.csv`](historico_falhas_fsm.csv) e as fichas em [`catalogo_ativos.csv`](catalogo_ativos.csv).
> Data de corte da base: 31/10/2026. "Últimos 12 meses" = 01/11/2025 a 31/10/2026. As métricas abaixo usam operação contínua (8.760 h/ano), taxa de falha constante e excluem registros de `condicao_ambiental`.

| TAG | Ativo | Criticidade | Registros (24 meses) | Falhas em 12 meses | MTBF 12 meses | R(720 h) |
|---|---|---|---|---|---|---|
| CFT-K416-N01 | Câmera PTZ, trecho comum | B | 10 | 5 | 1.752 h | 0,66 |
| CFT-K427-N01 | Câmera PTZ, serra, junto a viaduto | A | 10 | 5 | 1.752 h | 0,66 |
| LAC-PP02-F03-A | Laço de faixa de pesados, praça PP02 | A | 13 | 7 | 1.251 h | 0,56 |
| PFF-K402-OCR-01 | Câmera OCR do pórtico km 402, faixa 1 | A | 8 | 3 (+1 ambiental) | 2.920 h | 0,78 |

## 1. CFT-K416-N01, a câmera do halo noturno

Câmera de trecho comum em que duas causas diferentes dão sintomas parecidos à noite:
- Teia de aranha refletindo o IR (halo, imagem esbranquiçada). Aparece em ciclos de outubro a março (estação quente), com reincidência em menos de 90 dias. O plano prevê limpeza trimestral (PM-CFT-01), mas na estação quente a teia volta antes disso.
- Umidade na caixa de junção (perda total intermitente depois de chuva), em jan/2026 e abr/2026; na segunda vez a caixa foi trocada por uma IP66.

O que o assistente precisa perceber: "problema à noite" não diz muito. Halo aponta para teia; imagem sumindo depois de chuva aponta para a conexão. Uma boa resposta separa as duas hipóteses e sugere um teste para cada uma. Também cabe sugerir apertar a preventiva na época quente, como hipótese para o engenheiro avaliar.

## 2. CFT-K427-N01, o poste que vibra

Câmera de ponto crítico na serra, instalada em mar/2018, com vida útil de 8 anos vencida em mar/2026. O suporte foi reapertado a cada 60 a 80 dias, desgaste do módulo pan/tilt em fev/2026 e, em mai/2026, uma inspeção encontrou trinca na base do poste. A substituição está pendente e as falhas continuam.

O que o assistente precisa perceber: repetir a corretiva só trata o sintoma. É o tipo de caso em que o engenheiro justifica CAPEX (trocar poste e câmera), usando λ e MTBF como evidência e deixando as premissas claras.

## 3. LAC-PP02-F03-A, a faixa de pesados que cobra errado

Laço vulcanizado da faixa de pesados, a de maior impacto na arrecadação, com 7 falhas em 12 meses e dois mecanismos:
- Infiltração na selagem do corte depois de chuva (3 em 12 meses).
- Dano ao pavimento: trilha de roda acima de 20 mm altera a indutância e já rompeu o cabo duas vezes (4 em 12 meses).

O que o assistente precisa perceber: recalibrar só adia o problema. Há base para propor CBM (monitorar a divergência de classificação de PM-LAC-03) e para ligar o caso ao Tema 1 do briefing (trocar o sensor de solo por câmera ou sensor de superfície).

## 4. PFF-K402-OCR-01, a OCR que lê menos placas

Câmera OCR com queda da taxa de leitura abaixo do limiar de 97% (PM-PFF-02) por três motivos:
- Fuligem de diesel na lente (reincidente; a limpeza mensal PM-PFF-01 não está segurando).
- Iluminador IR degradado, com queda só à noite, em out/2025 e set/2026.
- Glare solar no fim da tarde em ago/2025 e jul/2026, registrado como `condicao_ambiental`: não é falha de hardware e não entra no cálculo de λ.

O que o assistente precisa perceber: separar defeito de efeito do ambiente (falso positivo, Tema 2.1) e usar a própria taxa de leitura como variável de CBM.
