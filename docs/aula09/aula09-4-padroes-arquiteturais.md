---
layout: aula
title: "4. Padrões Arquiteturais"
parent: Aula 09 - Introdução aos Microsserviços
nav_order: 4
---

# **4. Padrões Essenciais de Arquitetura e Comunicação**

Na seção anterior, exploramos os diferentes tipos de monólitos e suas vantagens. Agora vamos conhecer os padrões arquiteturais práticos que viabilizam a implementação de microsserviços: comunicação, descoberta, configuração, resiliência e consistência de dados.

Para que um ecossistema de microsserviços funcione de forma coesa, resiliente e gerenciável, não basta apenas "quebrar o monólito". É preciso adotar um conjunto de padrões arquiteturais testados e aprovados pelas melhores práticas da indústria.

## 4.1. Padrões de Comunicação

### Comunicação Síncrona

Ocorre quando um serviço faz uma requisição a outro e espera (bloqueia) até receber uma resposta. É simples de implementar e ideal para operações de leitura ou comandos que necessitam de uma resposta imediata.

**Tecnologias Comuns:**
- `RestTemplate` (legado) 
- `WebClient` (Spring WebFlux - moderno e reativo)
- APIs REST baseadas em HTTP

### Comunicação Assíncrona

Ocorre quando um serviço envia uma mensagem para um canal (tópico ou fila) e não espera por uma resposta. O serviço destinatário consome a mensagem quando estiver disponível. Este padrão promove baixo acoplamento e maior resiliência, pois o emissor e o receptor não precisam estar online ao mesmo tempo.

**Padrão Comum:** **Publish/Subscribe**, onde um serviço (Publisher) publica um evento em um tópico, e múltiplos serviços (Subscribers) podem consumir esse evento de forma independente.

**Tecnologias Comuns:**
- **RabbitMQ** (message broker tradicional)
- **Apache Kafka** (plataforma de streaming)

## 4.2. Padrões de Descoberta, Roteamento e Acesso

### Service Discovery / Registry

Em um ambiente dinâmico (nuvem, contêineres), os endereços de rede dos serviços mudam constantemente. Um Service Registry atua como uma "lista telefônica" onde cada serviço se registra ao iniciar e informa seu endereço atual. Outros serviços consultam o registro para descobrir como se comunicar.

**Tecnologias Comuns:**
- **Netflix Eureka**
- **Consul**
- **etcd**

### API Gateway

Funciona como um ponto de entrada único (`Single Point of Entry`) para todas as requisições externas. Ele centraliza responsabilidades transversais como:

- **Roteamento:** Encaminha as requisições para o microsserviço correto
- **Autenticação e Autorização:** Valida credenciais (ex: tokens JWT) antes de repassar a chamada
- **Rate Limiting e Caching:** Controla o fluxo de requisições e armazena respostas comuns
- **Transformação de Protocolos:** Converte entre diferentes formatos

**Tecnologia Comum:** **Spring Cloud Gateway**

## 4.3. Padrão de Configuração

### Configuração Centralizada

Gerenciar arquivos de configuração para dezenas de serviços é impraticável. Um servidor de configuração centralizada armazena as configurações de todos os serviços em um local único (geralmente um repositório Git). Os serviços, ao iniciarem, consultam este servidor para obter suas configurações.

**Benefícios:**
- Configurações versionadas (histórico no Git)
- Alterações sem rebuild
- Configurações por ambiente (dev, staging, prod)
- Refresh de configurações em runtime

**Tecnologia Comum:** **Spring Cloud Config**

## 4.4. Padrões de Resiliência e Tolerância a Falhas

Sistemas distribuídos são inerentemente instáveis. A falha de um serviço não pode causar uma falha em cascata no sistema todo.

### Circuit Breaker (Disjuntor)

É um padrão que monitora as chamadas para um serviço remoto. Se o número de falhas ultrapassa um limiar, o "disjuntor abre" e as próximas chamadas falham imediatamente (fail-fast), sem sobrecarregar o serviço instável. Após um tempo, o disjuntor entra em "meio-aberto" para testar se o serviço se recuperou.

**Estados do Circuit Breaker:**
1. **Fechado (Closed):** Funcionamento normal
2. **Aberto (Open):** Falhas imediatas, serviço não é chamado
3. **Meio-aberto (Half-Open):** Testa recuperação com chamadas limitadas

### Retries e Timeouts

- **Retries:** Configurar tentativas automáticas para falhas transitórias
- **Timeouts:** Tempos limite para evitar que um serviço fique bloqueado indefinidamente

**Tecnologia Comum:** Biblioteca **Resilience4j**, que se integra facilmente com o Spring Boot

## 4.5. Padrões de Consistência de Dados

### ACID vs. BASE

Em monólitos com um único banco de dados, as transações **ACID** (Atomicidade, Consistência, Isolamento, Durabilidade) são o padrão. Em microsserviços, com bancos de dados distribuídos, esse modelo é impraticável. Adotamos o modelo **BASE**:

- **B**asically **A**vailable: O sistema garante a disponibilidade
- **S**oft State: O estado do sistema pode mudar ao longo do tempo, mesmo sem novas entradas
- **E**ventually Consistent: O sistema eventualmente atingirá um estado consistente

### Padrão Saga

Para gerenciar "transações" que se estendem por múltiplos serviços, usamos o padrão Saga. Uma saga é uma sequência de transações locais. Se uma transação local falha, a saga executa **transações compensatórias** para reverter o trabalho feito pelas transações anteriores.

**Tipos de Saga:**
1. **Coreografia:** Serviços publicam eventos e reagem a eventos de outros
2. **Orquestração:** Um coordenador central gerencia a saga

**Exemplo prático:**
```
Pedido → Reserva Estoque → Processa Pagamento → Confirma Pedido
        ↓ (falha)         ↓ (compensa)
    Libera Estoque ← Cancela Pagamento ← Cancela Pedido
```

### 💻 Na Prática

Estes padrões não são apenas teoria - são implementados por empresas como:
- **Netflix:** Service Discovery (Eureka), Circuit Breaker
- **Uber:** API Gateway, Event-driven com Kafka
- **Amazon:** Saga Pattern para transações distribuídas
- **Spotify:** Configuração centralizada