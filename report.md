## 1. Arquitetura da Solução RAG

A arquitetura segue o padrão Retrieval-Augmented Generation (RAG), estruturada da seguinte forma:

SYSTEM PROMPT
│
▼
Histórico de mensagens
│
▼
Mensagem do usuário
│
▼
Contexto obtido via Similaridade (RAG)
│
▼
Resposta final do modelo


### Descrição do fluxo

1. **System Prompt**  
   Um prompt inicial define o comportamento do assistente. Ele instruí o modelo a responder *exclusivamente com base no edital oficial do Vestibular Unicamp 2026*. Caso a pergunta não seja relacionada ao edital, o modelo responde que não pode ajudar porque a consulta não se refere ao edital.

2. **Histórico de mensagens**  
   O histórico é recuperado do banco e enviado junto ao modelo em cada requisição, permitindo continuidade da conversa.

3. **Mensagem atual do usuário**  
   A query enviada pelo frontend é anexada ao contexto.

4. **RAG executado em todas as consultas**  
   Foi tomada a decisão de executar o RAG em todas as queries devido a problemas com a integração de ferramentas (tool calling) combinada com streaming SSE.  
   O uso da ferramenta não estava injetando corretamente o contexto no modelo, o que tornava as respostas inconsistentes.  
   Portanto, a solução atual injeta o contexto diretamente nas mensagens a cada requisição.

   O RAG funciona assim:
   - geração de embedding da query,
   - busca de similaridade no index Pinecone,
   - retorno das páginas e trechos relevantes,
   - formatação e anexação ao contexto passado ao modelo.

5. **Resposta do LLM**  
   O modelo recebe:
   - system prompt,
   - histórico,
   - mensagem atual,
   - contexto RAG,
   e produz uma resposta fundamentada no edital.

---

## 2. Decisões Técnicas e Justificativas

### Uso do Pinecone como base vetorial
O Pinecone foi utilizado para armazenamento dos embeddings do edital, pois oferece:
- alta performance para buscas de similaridade,
- API simples,
- escalabilidade e persistência.

### Execução do RAG em todas as queries
Esta decisão foi motivada por limitações técnicas:
- a integração com ferramentas durante streaming SSE não funcionou adequadamente,
- o contexto gerado pelo RAG não chegava ao modelo,
- o model fallback para o comportamento padrão continuava acontecendo sem o contexto.

Executar RAG sempre garante que o modelo receba o conteúdo correto do edital.

### System Prompt adaptado
O prompt inicial contém instruções explícitas para:
- usar exclusivamente o edital,
- ignorar qualquer conhecimento prévio,
- recusar respostas a perguntas irrelevantes ao edital,
- explicar que só pode responder dúvidas relacionadas ao Vestibular Unicamp 2026.

---

## 3. Estratégia de Testes e Validação

Foram feitos testes locais usando:
- perguntas reais do FAQ disponibilizado,
- perguntas específicas sobre datas, exigências, regras do edital,
- perguntas irrelevantes para verificar se o assistente recusa corretamente.

Os resultados foram satisfatórios:
- o modelo recuperou contextos corretos,
- as respostas estavam alinhadas ao edital,
- comportou-se adequadamente ao recusar perguntas fora do contexto.

### Testes automatizados
A integração com pytest ainda não foi concluída.  
Os próximos passos incluem criar testes automatizados para:
- função de embedding,
- consulta ao Pinecone,
- formatação do RAG,
- fluxo de mensagens com contexto,
- comportamento do system prompt.

---

## 4. Avaliação do Chatbot

### Pontos fortes
- Respostas fundamentadas no edital.
- Busca semântica funcionando conforme esperado.
- Estrutura simples e direta do RAG.
- Fluxo de conversa estável com streaming.

### Comportamento observado
- Responde adequadamente perguntas relacionadas ao vestibular.
- Recupera corretamente trechos relevantes do PDF.
- Recusa perguntas fora do escopo, como configurado no system prompt.

---

## 5. Limitações Conhecidas

### Problemas com tool calling
O uso de ferramentas específicas do modelo, quando combinado com SSE, apresentou falhas:
- o modelo não recebia o contexto de retorno corretamente,
- chamadas de ferramentas interrompiam o fluxo de streaming,
- em alguns casos, o modelo ignorava o RAG.

Por isso, optou-se pelo RAG executado diretamente e manualmente em todas as consultas.

### Ineficiência de executar RAG em todas as queries
Idealmente, só deveria executar RAG quando necessário.  
No entanto, devido às limitações descritas, essa abordagem foi adotada como solução estável.

### Ausência de testes automatizados
Ainda não há testes automatizados implementados.  
Isso dificulta garantir consistência ao evoluir o código.

---

## Conclusão

A solução atual entrega um chatbot funcional, consistente e baseado no edital do Vestibular Unicamp 2026.  
Apesar das limitações técnicas com ferramentas e streaming, o sistema final responde com precisão e comporta-se conforme o esperado.

Uma futura versão pode reintroduzir ferramentas com arquitetura ajustada, melhorar a eficiência do RAG e incluir testes automatizados para aumentar a confiabilidade do sistema.

