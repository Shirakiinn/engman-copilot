# POPs simulados do EngMan Copilot

> **SIMULADO. NÃO USAR EM CAMPO.** Procedimentos fictícios escritos pelo grupo só para dar forma ao retorno da tool `consultar_procedimento`. Em qualquer situação real, valem o POP oficial da Motiva, a NR-10, a NR-35 e as regras internas de trabalho em via.

Estes POPs não entram no vector store: são entregues inteiros pela tool `consultar_procedimento`. Assim os pré-requisitos de segurança não são cortados no meio por chunking (ver [inventário](inventario.md)).

---

## POP-CFT-01: Limpeza de cúpula, teste de comunicação, ajuste de foco e reaperto de suporte (CFTV PTZ)

- Versão: 1.0-SIM · Tipo de intervenção: preventiva, inspeção ou corretiva · Plano: PM-CFT-01, PM-CFT-02, PM-CFT-04

#### Pré-requisitos de segurança (obrigatórios, antes de qualquer passo)
1. Equipe com treinamento válido de trabalho em altura (NR-35) e, para a caixa de junção, de eletricidade (NR-10).
2. Análise de risco da atividade emitida e assinada pelo responsável.
3. Sinalização do acostamento conforme regra interna de trabalho em via.
4. Plataforma elevatória inspecionada; não operar com chuva ou vento acima do limite do equipamento.
5. Para abrir a caixa de junção: circuito desenergizado e bloqueado conforme procedimento oficial.

#### Passos
1. Registrar no FSM o início do atendimento com o TAG.
2. Pedir ao CCO que leve a câmera ao preset de manutenção.
3. Inspecionar a cúpula: teias, ninhos, barro, fuligem, trincas, condensação interna.
4. Limpar a cúpula com o kit óptico (sem solvente abrasivo).
5. Verificar o aperto do suporte com torquímetro.
6. Testar a comunicação: o CCO confirma ping, vídeo sem travamento e resposta aos comandos PTZ.
7. Ajustar o foco e conferir os presets com o CCO.
8. Se o sintoma for noturno, pedir ao CCO um teste de IR com a câmera no preset noturno.
9. Pedir ao CCO a validação da imagem (dia e IR) antes de liberar a área.
10. Registrar no FSM: causa encontrada, peças trocadas e fotos antes e depois.

Critério de aceite: imagem nítida em todos os presets e sem halo no IR, validada pelo CCO.

---

## POP-LAC-02: Teste elétrico, resselagem e ajuste de laço indutivo

- Versão: 1.0-SIM · Tipo de intervenção: preventiva, inspeção ou corretiva · Plano: PM-LAC-01, PM-LAC-02, PM-LAC-04

#### Pré-requisitos de segurança (obrigatórios, antes de qualquer passo)
1. Faixa fechada pela Operação e sinalizada conforme regra interna de trabalho em via. Nunca trabalhar com a faixa aberta ao tráfego.
2. Análise de risco da atividade emitida e assinada pelo responsável.
3. Equipe com treinamento de eletricidade (NR-10) para trabalhar no gabinete do detector.
4. Detector desenergizado antes de desconectar o laço.

#### Passos do teste elétrico
1. Registrar no FSM o início do atendimento com o TAG.
2. Desconectar o laço do detector.
3. Medir a indutância do laço e comparar com a faixa do fabricante.
4. Medir a resistência de isolamento entre laço e terra com o megômetro.
5. Medir a continuidade do laço e do cabo alimentador.
6. Inspecionar a selagem do corte e medir a trilha de roda sobre o laço.
7. Reconectar, energizar o detector e verificar o pulso com passagem de veículo de teste.
8. Registrar no FSM as medições e a condição do pavimento.

Resselagem, quando a selagem estiver rompida: limpar e secar o corte, aplicar o selante epóxi, respeitar o tempo de cura antes de liberar a faixa e refazer o ajuste de sensibilidade (PM-LAC-04).

Critério de aceite: medições dentro da faixa e 20 passagens seguidas classificadas corretamente.

---

## POP-PFF-01: Limpeza de lentes OCR e IR e teste de antenas (Pórtico Free Flow)

- Versão: 1.0-SIM · Tipo de intervenção: preventiva, inspeção ou corretiva · Plano: PM-PFF-01, PM-PFF-03, PM-PFF-04

#### Pré-requisitos de segurança (obrigatórios, antes de qualquer passo)
1. Equipe com treinamento válido de trabalho em altura (NR-35) e, para o painel elétrico, de eletricidade (NR-10).
2. Janela noturna de baixo tráfego; faixa sinalizada ou interditada pela Operação.
3. Análise de risco da atividade emitida e assinada pelo responsável.
4. Painel elétrico do pórtico só pode ser aberto desenergizado e bloqueado conforme procedimento oficial.

#### Passos
1. Registrar no FSM o início do atendimento com o TAG.
2. Avisar o sistema de pista que a faixa entra em manutenção (as transações passam a ser marcadas).
3. Limpar as lentes OCR e os iluminadores IR com o kit óptico.
4. Para antenas: medir potência e ROE com o analisador de RF e ler a TAG de teste.
5. Verificar o sincronismo NTP do equipamento no servidor de pista.
6. Pedir a validação de 50 leituras seguidas (OCR ≥ 97% e TAG ≥ 99,5%).
7. Registrar no FSM as medições e a taxa de leitura antes e depois.

Critério de aceite: taxas de leitura acima dos limiares do plano PM-PFF-02.
