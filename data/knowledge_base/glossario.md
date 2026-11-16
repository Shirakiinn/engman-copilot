# Glossário de siglas do EngMan Copilot

> Origem: real (siglas citadas no briefing Motiva × FIAP). As definições foram redigidas pelo grupo.

| Sigla | Significado | Uso no projeto |
|---|---|---|
| CBM | *Condition-Based Maintenance*: manutenção baseada na condição | Intervir quando uma variável monitorada (ex.: taxa OCR) cruza um limiar, e não pelo calendário |
| CCO | Centro de Controle Operacional | Onde o operador vê os alarmes e as imagens de CFTV |
| CFTV | Circuito Fechado de TV | Câmeras PTZ de monitoramento da rodovia |
| E1 / E2 / E3 | Efeito da falha (perda total / degradação / sem impacto) | Entrada da matriz de prioridade |
| FSM | *Field Service Management*: sistema de gestão do serviço de campo | Fonte do histórico de falhas (aqui, simulado) |
| HS WIM | *High-Speed Weigh-In-Motion*: pesagem em movimento | Fora do escopo desta versão |
| IR | Infravermelho | Iluminação noturna das câmeras; a teia reflete o IR e gera halo |
| LDA | *Life Data Analysis*: análise de dados de vida (ex.: Weibull) | Evolução futura da tool de métricas |
| λ (lambda) | Taxa de falha (falhas por hora de operação) | λ = nº de falhas ÷ horas de operação |
| MTBF | *Mean Time Between Failures*: tempo médio entre falhas | MTBF = 1/λ, com taxa constante |
| MTTR | *Mean Time To Repair*: tempo médio de reparo | Média de `tempo_reparo_h` no histórico |
| NR-10 | Norma Regulamentadora 10: segurança em instalações e serviços em eletricidade | Motivo da recusa de orientação em equipamento energizado |
| NR-35 | Norma Regulamentadora 35: trabalho em altura | Pré-requisito em câmeras e pórticos |
| OCR | *Optical Character Recognition*: leitura óptica de placas | Câmeras dos pórticos Free Flow |
| OS | Ordem de Serviço | Redigida pelo assistente e aberta só com confirmação |
| P1 a P4 | Níveis de prioridade de atendimento | Saída da matriz de criticidade |
| PLC | Controlador lógico programável | Recebe o pulso dos laços na pista |
| PMV | Painel de Mensagens Variáveis | Fora do escopo desta versão |
| POP | Procedimento Operacional Padrão | Retornado pela tool `consultar_procedimento` |
| PTZ | *Pan-Tilt-Zoom* | Câmera motorizada com presets |
| R(t) | Confiabilidade: probabilidade de operar sem falha até o tempo t | R(t) = e^(−λt) no modelo exponencial |
| RCM | *Reliability-Centered Maintenance*: manutenção centrada em confiabilidade | Referência metodológica do briefing |
| RGA | *Reliability Growth Analysis*: análise de crescimento de confiabilidade | Tendência pelo β de Crow-AMSAA na tool de métricas |
| SAT | Sistema Analisador de Tráfego | Fora do escopo desta versão |
| SCA | Sistema Ambiental: estações meteorológicas automatizadas (como aparece no briefing) | Fora do escopo desta versão |
| TAG | Código único do ativo; também o *tag* eletrônico do veículo no Free Flow | No projeto, "TAG" sem qualificação é o código do ativo |
| WCM | *World Class Manufacturing* | Referência metodológica do briefing |
