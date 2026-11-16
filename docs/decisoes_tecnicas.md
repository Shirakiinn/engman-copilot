# Decisões técnicas do EngMan Copilot

> Regra da disciplina: o caminho de execução precisa ser 100% gratuito. Todas as opções abaixo estão na lista aprovada do enunciado. Dados de preço, limites e modelos foram conferidos em 16/11/2026 nas fontes do fim deste documento. Limites de free tier mudam com frequência; confira no console de cada provedor antes de rodar.

## 1. Modelo de linguagem (LLM)

| Critério | Qwen 2.5 7B Instruct, Ollama local (escolhido) | Llama 3.1 8B Instruct (Ollama local) | Gemini Flash / Flash-Lite (Google AI Studio, free tier) | gpt-oss-120b (Groq, free tier) |
|---|---|---|---|---|
| Custo de uso | Zero (roda na máquina ou no Colab) | Zero | Zero dentro dos limites do free tier | Zero dentro dos limites do free tier |
| Licença / termos | Apache 2.0 | Llama 3.1 Community License: uso comercial permitido abaixo de 700 milhões de usuários ativos/mês | Termos da API Google; no free tier o conteúdo enviado pode ser usado para melhorar os produtos do Google | Termos GroqCloud; modelo aberto |
| Português | Fabricante declara 29+ idiomas, incluindo português | Português é um dos 8 idiomas oficialmente suportados; MMLU em português: 62,1 | Esperado alto (modelo grande); não medido pelo grupo | Esperado alto (modelo grande); não medido pelo grupo |
| Janela de contexto | 128K tokens no modelo (32K na configuração padrão); no Ollama o padrão é 4.096, por isso fixamos `num_ctx 16384` | 128K tokens; mesmo ajuste no Ollama | ~1M tokens | 131K tokens |
| Privacidade (roda local?) | Sim: o dado não sai da máquina | Sim | Não: dado vai para a API | Não: dado vai para a API |
| Tool calling | Suportado pelo template; depende do runtime | Suportado oficialmente | Suportado | Suportado (API compatível com OpenAI) |
| Limite de uso | Só o hardware: ≈ 4,7 GB do modelo + ≈ 0,9 GB de KV cache com 16K tokens | Igual | Definido por projeto e exibido no AI Studio | Exemplo publicado: 30 req/min, 1.000 req/dia, 8K tokens/min |
| Integração com Python | `ollama` SDK ou cliente compatível com OpenAI | Igual | SDK `google-genai` | SDK `groq` ou cliente OpenAI |
| Maturidade / documentação | Alta; muito usado em RAG | Alta; o mais documentado entre os abertos | Alta; nomes de modelo mudam a cada poucos meses | Alta; catálogo do free tier muda com frequência |

Escolha: Qwen 2.5 7B Instruct via Ollama local.
1. Privacidade (risco RK09): histórico de falhas e dados de arrecadação são confidenciais. Rodar localmente elimina o envio a terceiros, e o free tier do Gemini permite uso do conteúdo pelo Google.
2. Custo e reprodutibilidade: não depende de chave, de cota diária nem de mudança de catálogo do provedor, e todos do grupo rodam a mesma versão.
3. Licença Apache 2.0: sem cláusula extra (Llama exige atribuição e tem limite de usuários).

Trade-off aceito: um modelo de 7B raciocina e escolhe tools pior que um de 120B ou um Gemini. Compensamos com a arquitetura: as contas ficam nas tools, a prioridade vem de uma matriz com few-shot, o guardrail de saída confere TAGs e números e a temperatura é baixa. Plano B: Llama 3.1 8B no mesmo Ollama, se o tool calling do Qwen falhar no eval set; Groq ou Gemini só para testes rápidos com dado simulado, nunca com dado real.

## 2. Vector store

| Critério | ChromaDB (escolhido) | FAISS |
|---|---|---|
| Custo / licença | Zero / Apache 2.0 | Zero / MIT |
| Filtro por metadados | Nativo (`where` com igualdade, faixa e operadores lógicos) | Não tem: filtrar por metadado exige código próprio |
| Persistência | Embutida (`PersistentClient`) | Manual (`write_index` / `read_index`) |
| Desempenho | Suficiente para milhares de vetores | Mais rápido em busca vetorial pura |
| Integração | Python simples; integra com LangChain | Biblioteca de baixo nível |

Escolha: ChromaDB. O enunciado pede busca filtrada por TAG, tipo de ativo, data e tipo de documento, e o Chroma faz isso nativamente. Trade-off aceito: ele é mais lento que o FAISS, mas com ≈ 40 trechos a diferença é de milissegundos.

## 3. Embeddings

| Critério | multilingual-e5-base, Sentence Transformers local (escolhido) | nomic-embed-text (Ollama) |
|---|---|---|
| Custo | Zero (roda local) | Zero (roda local no Ollama) |
| Licença | MIT | Apache 2.0 |
| Idioma | 94 idiomas, incluindo português | Treinado como modelo de inglês |
| Tamanho máximo do trecho | 512 tokens | 8.192 tokens |
| Dimensão / parâmetros | 768 / 278M | 768 / 137M |
| Privacidade | Local | Local |

Escolha: multilingual-e5-base, porque toda a base está em português. Trade-off aceito: o limite de 512 tokens obriga chunks de até 450 tokens (ver [inventário](../data/knowledge_base/inventario.md#3-como-a-base-será-usada-no-rag)) e os prefixos `query:` e `passage:`.

## 4. Orquestração, interface e ambiente

| Camada | Escolha | Justificativa |
|---|---|---|
| Orquestração | Chamada direta pelo SDK do Ollama, com as três peças do Módulo 3: `chamar_llm()`, histórico e `montar_prompt()` | Os guardrails ficam em código Python legível e testável, sem abstrações extras para depurar. O LangChain fica como opção só para o *retriever* do Chroma, se simplificar. |
| Interface (opcional) | Gradio `ChatInterface` | Poucas linhas e já tem chat; ver o [protótipo visual](mockup_interface.png) |
| Ambiente | Python 3.10+, VS Code ou Jupyter local, ou Google Colab com GPU (quando disponível no plano gratuito) | Os 3 caminhos são gratuitos |
| Versionamento | Git + GitHub público | Obrigatório desde a Sprint 01 |

## 5. Política de contexto (quantificada)

Contagens medidas com o tokenizador `cl100k_base`, usado como aproximação do tokenizador do Qwen (diferença esperada de ±20% em português).

| Componente | Tokens | Regra |
|---|---|---|
| System prompt fixo (sp-v1.1) | 3.550 | Medido (3.527) |
| Contrato das 6 tools | 2.350 | Medido |
| Estado da conversa (TAG ativo + resumo) | 300 | Resumo refeito a cada 6 trocas |
| RAG: até 4 trechos × 450 | 1.800 | top-k = 4, filtro por `tipo_ativo` |
| Resultados de tools deste turno | 1.500 | Até 3 chamadas; histórico limitado a 15 registros (12 meses do CFT-K416-N01 = 712 tokens) |
| Últimas 6 trocas | 3.000 | Resultados de tools de turnos antigos são descartados; fica só a resposta |
| Mensagem atual | 200 | |
| Reserva de saída (`max_tokens`) | 700 | ≈ 250 palavras + rascunho de OS |
| Total | 13.400 | 82% de `num_ctx 16384`, margem de 2.984 (18,2%) |

Por que não colocar a base inteira no contexto: a base já produzida soma ≈ 14 mil tokens e chega a ≈ 24 mil com os documentos da Sprint 3, mais que a janela inteira. Além disso, contexto longo deixa um modelo pequeno mais lento e pior em achar o trecho certo.

Ordem de recorte se o orçamento estourar (o código mede antes de chamar):
1. Reduzir o histórico na íntegra de 6 para 4 trocas (o resumo cobre o resto);
2. Reduzir o RAG de 4 para 3 trechos;
3. Truncar o resultado de histórico de falhas aos 10 registros mais recentes e avisar na resposta.

O prompt fixo e o contrato das tools nunca são cortados.

Memória: no Qwen 2.5 7B (28 camadas, 4 cabeças KV de dimensão 128), o KV cache em fp16 ocupa ≈ 56 KB por token, ou ≈ 0,9 GB para 16.384 tokens. Somado ao modelo quantizado (≈ 4,7 GB), cabe numa GPU de 8 GB ou em 16 GB de RAM.

## 6. Especificação dos guardrails (para implementar na Sprint 3)

Os guardrails são código Python determinístico, não instrução ao modelo. Eles ficam antes (②) e depois (⑦) da chamada ao LLM no [fluxograma](arquitetura.png). Os padrões abaixo foram testados contra os textos do projeto: não confundem TAG, número de OS, registro FSM, POP ou norma com placa ou CPF.

Entrada (②)

| Regra | Como detectar | Ação |
|---|---|---|
| Placa (Mercosul e antiga) | regex `(?<![A-Za-z0-9-])[A-Z]{3}-?\d[A-Z0-9]\d{2}(?![A-Za-z0-9-])` | Troca por `[PLACA]` antes do LLM e do histórico |
| CPF | regex `(?<!\d)\d{3}\.?\d{3}\.?\d{3}-?\d{2}(?!\d)` | Troca por `[CPF]` |
| Injection | termos: "ignore as instruções", "esqueça as regras", "system prompt", "modo desenvolvedor", "a partir de agora você", "[INSTRUÇÃO" | Marca `FLAG_INJECTION` no contexto (não bloqueia) |
| Pedido perigoso | termos: "energizado", "sem desligar", "ligado mesmo", "sem sinalização", "rapidinho", "laudo", "ART", "liberar a faixa" | Marca `FLAG_RISCO` no contexto (não bloqueia) |
| Ativo fora do recorte | termos: "PMV", "painel de mensagem", "callbox", "SAT", "SCA", "estação meteorológica", "WIM", "balança" | Responde direto com a frase R5, sem chamar o LLM |

As regras de injection e de perigo só marcam o pedido, porque palavras como "ligado" geram falso positivo. Quem decide a recusa é o prompt (R4, R8), já avisado pela flag.

Saída (⑦)

| Verificação | Como | Se falhar |
|---|---|---|
| Todo TAG citado existe | regex de TAG + busca no catálogo | Gera de novo, uma vez |
| Número de OS só se a OS foi aberta | regex `OS-SIM-\d{4}-\d{6}` só vale se `abrir_ordem_servico` retornou neste turno | Gera de novo, uma vez |
| `abrir_ordem_servico` só após "confirmo" | a mensagem anterior do usuário contém "confirmo" | Bloqueia a chamada da tool |
| Formato | resposta sobre ativo contém "TAG ativo" e "Nível de confiança" | Gera de novo, uma vez |
| Dado pessoal | mesmas regex de placa e CPF | Mascara |
| Vazamento do prompt | trecho de 8 ou mais palavras igual ao system prompt | Gera de novo, uma vez |

Resposta segura (quando a segunda tentativa também falha): "Não consegui gerar uma resposta verificável para este pedido. Consulte o histórico no FSM ou o responsável pela Engenharia de Manutenção."

## Fontes (conferidas em 16/11/2026)

- Qwen 2.5 (licenças, contexto): [qwenlm.github.io/blog/qwen2.5-llm](https://qwenlm.github.io/blog/qwen2.5-llm) · [README do Qwen2.5-7B-Instruct](https://www.huggingface.co/Qwen/Qwen2.5-7B-Instruct/blob/main/README.md?code=true) · [ficha com idiomas](https://www.promptlayer.com/models/qwen25-7b-instruct)
- Llama 3.1 (idiomas, licença, MMLU em português): [ficha NVIDIA NGC](https://catalog.ngc.nvidia.com/orgs/nvidia/nemo/models/llama-3_1-8b-nemo/1.0) · [model card](https://huggingface.co/Undi95/Meta-Llama-3.1-8B-Instruct-OAS)
- Ollama, contexto padrão de 4.096 e `num_ctx`: [docs.ollama.com/faq](https://docs.ollama.com/faq) · [docs.ollama.com/context-length](https://docs.ollama.com/context-length)
- Gemini API, free tier e uso do conteúdo: [ai.google.dev/gemini-api/docs/pricing](https://ai.google.dev/gemini-api/docs/pricing) · [limites exibidos só no AI Studio](https://benchlm.ai/free-tier/google-ai-studio)
- Groq, free tier: [developersdigest.tech/models/free-api/groq](https://www.developersdigest.tech/models/free-api/groq) · [ficha llama-3.1-8b-instant](https://console.groq.com/docs/model/llama-3.1-8b-instant)
- Chroma × FAISS: [myengineeringpath.dev/tools/chroma-vs-faiss](https://myengineeringpath.dev/tools/chroma-vs-faiss/) · [theneuralbase.com/compare/chromadb-vs-faiss](https://theneuralbase.com/compare/chromadb-vs-faiss/)
- Embeddings: [multilingual-e5-base](https://www.promptlayer.com/models/multilingual-e5-base) · [nomic-embed-text (Ollama)](https://registry.ollama.ai/library/nomic-embed-text) · [artigo Nomic Embed](https://arxiv.org/abs/2402.01613v2)
