# Confronto:

Para comparar a classificação usando regras determinísitcas e a classificação atribuída pelo agente, seriam utilizados os seguintes critérios:

Ausência de fracionamento e valor atípico: Risco baixo
Presença apenas de valor atípico: Risco médio
Presença de fracionamento: Risco alto

Como o valor atípico é mais provável de ocorrer de forma pontual, foi considerado um risco médio. Já o fracionamento, é mais provável de estar ligado a uma atividade indevida pois é uma combinação de regras mais difícil de acontecer naturalmente, por isso foi considerado um risco alto.

Considerando essas mesmas flags (fracionamento e valor atípico), é esperado que a LLM gere resultados de menos risco no geral, pois pode analisar os casos de maneira mais complexa, considerando a frequência de ocorrência. Um valor atípico poderia ser considerado algo pontual se ocorresse uma vez em um longo período de tempo por exemplo e não acrescentar no valor do risco. Entretanto, a LLM pode analisar também outros fatores como volume de transferências para um mesmo destinatário, distribuição do uso de canais, o que pode acabar acrescentando no nível de risco.

# Regras em escala

- Executar os mesmos comandos para tratamento de dados e criação de regras utilizados no Nível 1 Parte A. Acredito que não seriam necessárias muitas alterações no código original.  
- Realizar o merge do df final, que já inclui o valor atípico, com a tabela resultante da flag de fracionamento utilizando o id da operação como chave.  
- Agrupar a tabela por cliente e realizar a contagem do número de flags como True nas duas colunas e a soma do volume transacionado  
- Ordenar a tabela por contagem do número de flags e por volume e selecionar os 10 primeiros clientes  

# Ferramentas

- Função historico_cliente: Receberá o id do cliente e retornará um DataFrame com as operações realizadas por aquele cliente.   Comando para realizar o filtro df[df["cliente_id"] == cliente_id]  

- Função operacoes_do_dia: Receberá o id do cliente e a data desejada e retornará um DataFrame com as operações realizadas pelo cliente naquele dia.  
Comando para realizar o filtro: df[ (df["id_cliente"] == id_cliente) & (df["data"].dt.date == data.date())]

- FUnção perfil_canal: Recebe o id do cliente e devolve um DataFrame com a distribuição de uso por canal, colunas canal e frequência de uso.  
Comandos para obter a distribuição: df = historico_cliente(cliente_id); df.groupby("canal").size()
- 

# Agente

O agente seria instruído a decidir as ferramentas com base no prompt recebido. Ele iria possuir a descrição de cada ferramenta que ele pode chamar e alguns casos de exemplo de quando elas podem ser úteis. Por exemplo, chamaria a ferramenta de operacoes_do_dia se o prompt espeicificasse que quer analisar algo em um dia específico e para perguntas mais gerais em relação ao numero de transações, volume, media ou mediana chamaria a operacoes_do_dia. Também seria garantido que ele pode chamar mais de uma ferramenta.

# Lote

Com a lista resultante da seção Regras em escala, chamaria o agente para cada cliente com um prompt fixo pedindo um parecer do nível de risco atribuído ao cliente. O prompt e a estrutura de saída seria similar a estrutura utilizada no Nível 1 parte B. Salvaria essas informações diretamente na pasta outputs/. usando logging na função do agente. O nome do arquivo seria gerado a partir do nome do cliente.

# MCP

- Transformar as funções criadas no nível 2 em mcp tools
- Criação do MCP server usando o FastMCP
- Transformar o agente criado no nível 2 em um cliente MCP

## Justificativa de escolha da trilha: 

A trilha escolhida foi a de servidor MCP. Apesar de não impactar diretamente a qualidade da análise, tem um ganho de arquitetura. Desacopla as ferramentas do Nível 2 do agente Python específico desse projeto, permitindo que sejam consumidas por qualquer cliente compatível com MCP (Claude Desktop, outro agente, outro desenvolvedor) sem depender do meu código-fonte.

Optei por essa trilha pois acredito que é possível obter um bom resultado e reduzir consideravelmente a quantidade de análises manuais com a proposta inicial da análise. Assim, considero mais valioso consolidar uma peça de infraestrutura reutilizável e alinhada ao padrão utilizado por diferentes agentes do que aprofundar a lógica de decisão do modelo. 