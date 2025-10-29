# Projeto AWS: Orquestração de Workflows com Step Functions

Este repositório é parte de um desafio de projeto do bootcamp **Santander Code Girls 2025** (parceria com a [Dio.me](https://www.dio.me/)) e foca na exploração de workflows automatizados usando o AWS Step Functions.

O objetivo é documentar os conceitos, componentes, benefícios e aplicações práticas deste serviço de orquestração serverless.

---

## AWS Step Functions: Orquestração Serverless de Workflows

O AWS Step Functions é um serviço nativo da AWS e sem servidor (serverless) que fornece recursos de orquestração. Ele é usado para criar fluxos de trabalho (também chamados de Máquinas de Estado) para construir aplicações distribuídas, automatizar processos de negócios, orquestrar microsserviços e criar pipelines de dados e machine learning.

### O que é e Como Funciona

#### O que é (Definição)

O Step Functions é um orquestrador de fluxo de trabalho que se destaca por ser totalmente gerenciado e sem servidor. Ele é uma ótima opção para a orquestração de fluxos de trabalho.

#### Como Funciona (Orquestração)

Na abordagem de orquestração:

- O Step Functions atua como um orquestrador central.
- Ele é responsável por chamar cada microsserviço.
- Determina a ordem de execução (em sequência ou em paralelo).
- Manipula as respostas individuais do serviço ao longo do caminho e, por fim, compila o resultado final.

Esta abordagem separa a lógica de orquestração do código da aplicação, o que reduz significativamente a complexidade do código.

### Componentes Chave de um Workflow

Um fluxo de trabalho (ou Máquina de Estado) no Step Functions é uma série de etapas orientadas a eventos. Os fluxos de trabalho são definidos usando a **Amazon States Language (ASL)**.

#### 1. Tipos de Workflow (Standard vs. Express)

Existem dois tipos principais de fluxos de trabalho, escolhidos durante a criação da máquina de estado:

| Tipo de Workflow | Propósito Ideal | Características Principais | Cobrança |
| :--- | :--- | :--- | :--- |
| **Padrão (Standard)** | Fluxos de longa duração (até um ano), duráveis e que exigem auditoria. Adequado para orquestrar ações não idempotentes (como processamento de pagamentos). | Histórico de execução completo (até 90 dias). Modelo de execução *exactly-once*. | Cobrado por transição de estado. |
| **Expresso (Express)** | Alto volume de processamento de eventos, ingestão de dados IoT, backends de aplicativos móveis. Execuções de curta duração (até 5 minutos). | Histórico de execução baseado em nível de log (via CloudWatch Logs). Modelo de execução *at-least-once* (adequado para ações idempotentes). | Cobrado por número de execuções, duração total e memória consumida. |

#### 2. Estados e Lógica

Cada passo é um **Estado**. O Step Functions suporta lógica complexa através de diferentes tipos de estados:

- **Estado de Tarefa (`Task State`):** Representa uma unidade de trabalho executada por outro Serviço da AWS, como uma função Lambda ou uma integração otimizada com mais de 200 serviços.
- **Estado Choice (`Choice`):** Permite ramificações lógicas e tomadas de decisão no fluxo com base nas entradas.
- **Estado Parallel (`Parallel`):** Permite que passos sejam executados simultaneamente.
- **Estado Map (`Map`):** Permite que um conjunto de passos do fluxo de trabalho seja executado em cada item de um conjunto de dados. O **Map Distribuído** permite processamento de dados em grande escala e alta simultaneidade.

#### 3. Padrões de Integração

O Step Functions se comunica com outros serviços usando três padrões de integração principais:

- **Resposta à Solicitação (`Request-Response`):** O Step Functions avança para o próximo estado assim que obtém uma resposta HTTP. (Suportado por Padrão e Expresso).
- **Executar Tarefa (`Run a Job - .sync`):** O Step Functions aguarda até que a tarefa do serviço chamado seja concluída. (Suportado apenas por fluxos de trabalho Padrão para serviços otimizados).
- **Aguardar Retorno de Chamada (`Wait for Callback - .waitForTaskToken`):** O Step Functions pausa e espera que um serviço externo envie um token de tarefa de volta para que o fluxo possa continuar. (Suportado apenas por fluxos de trabalho Padrão para serviços como SQS, SNS e Lambda).

### Benefícios e Vantagens

O uso do Step Functions é recomendado quando:

- **Tratamento de Erros Integrado:** Oferece mecanismos de tratamento de erros e lógica de repetição (`retry logic`) no nível do fluxo de trabalho, o que aumenta a confiabilidade. Os estados `Task`, `Parallel` e `Map` podem usar um campo `Retry` para tentar novamente estados com falha.
- **Gerenciamento de Estado:** É necessário manter o estado em operações de longa duração. Os fluxos de trabalho padrão podem suportar processos de longa duração por até um ano.
- **Orquestração Complexa:** A integração de microsserviços envolve processos complexos de várias etapas.
- **Observabilidade:** O uso de um orquestrador melhora o monitoramento. Ele fornece um editor visual para projetar e depurar fluxos de trabalho complexos, simplificando o gerenciamento.
- **Serverless:** É um serviço totalmente gerenciado, o que significa que não há infraestrutura para pré-provisionar.
- **Otimização de Custos:** Você não incorre em custos quando não há fluxos de trabalho em execução.

### Exemplos de Uso

O Step Functions pode ser usado para orquestrar tarefas complexas e introduzir interações humanas ou processamento em massa:

- **Orquestração de Tarefas:** Criar fluxos de trabalho que orquestram uma série de etapas em ordem específica, onde a saída de uma tarefa serve como entrada para a próxima.
- **Processos de Aprovação Humana:** Incluir etapas de aprovação humana no fluxo de trabalho usando o padrão de retorno de chamada (`.waitForTaskToken`).
  - *Exemplo:* Um processo de aprovação de empréstimo onde o Step Functions orquestra a validação de dados, verificação de crédito e, se houver alto risco, encaminha para uma revisão manual.
- **Processamento de Dados Paralelo:** Usar o estado `Parallel` para processar dados de entrada em etapas simultâneas.
- **ETL e Processamento de Grande Escala:** Ideal para executar trabalhos ETL. O `Distributed Map` permite copiar dados CSV em grande escala e lidar com processamento distribuído.
- **Automação e Integração:** Pode ser usado para iniciar fluxos de trabalho em resposta a eventos (ex: via `EventBridge Pipes`) e integrar-se a outros serviços para processamento de dados, como o `AWS Glue`.

### Tipos de Insights Possíveis de Adquirir

A capacidade de monitoramento e observabilidade do Step Functions é uma de suas principais vantagens:

- **Visualização e Depuração de Fluxo:** Capacidade de visualizar, editar e depurar o fluxo de trabalho da aplicação no console. É possível examinar o estado de cada etapa no fluxo.
- **Histórico de Execução Detalhado:**
  - Para **Fluxos de Trabalho Padrão**, é possível recuperar o histórico de execução completo por até 90 dias.
  - Para **Fluxos de Trabalho Expressos**, o histórico de execução pode ser inspecionado no `CloudWatch Logs` ou no console.
- **Métricas de Desempenho e Otimização:** As métricas de execução devem ser monitoradas para otimizar o fluxo de trabalho. O sistema fornece informações detalhadas sobre os estados com falha.
- **Automação de Resposta a Eventos:** O Step Functions pode ser usado como um destino de eventos no `Amazon EventBridge`, permitindo que você configure a automação para responder a alterações de estado ou status.
