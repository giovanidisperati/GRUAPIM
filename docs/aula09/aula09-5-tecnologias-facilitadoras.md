---
layout: aula
title: "5. Tecnologias Facilitadoras"
parent: Aula 09 - Introdução aos Microsserviços
nav_order: 5
---

# **5. Tecnologias Facilitadoras**

Ao iniciar com **microsserviços**, não é necessário adotar um conjunto enorme de tecnologias novas logo de início. O ideal é começar com uma base simples e, conforme os desafios forem surgindo, adotar tecnologias que resolvam problemas reais da sua arquitetura.

## 5.1. Agregação de Logs e Rastreamento Distribuído 

Conforme você começa a lidar com diversos microsserviços rodando ao mesmo tempo, **entender o que está acontecendo no sistema fica mais difícil**. Erros podem surgir em um serviço e se propagar para outro.

Por isso, um dos primeiros passos recomendados é implementar uma **ferramenta de agregação de logs**. Ela coleta os logs de todos os serviços e os centraliza em um único painel.

**Ferramentas Comuns:**
- **Humio**
- **AWS CloudWatch**
- **Azure Monitor**
- **Google Cloud Logging**
- **ELK Stack** (Elasticsearch, Logstash, Kibana)

**IDs de Correlação:** Um identificador único que acompanha todo o ciclo de uma requisição entre serviços, facilita muito o rastreamento.

À medida que a arquitetura cresce, também se torna importante usar **ferramentas de rastreamento distribuído**:
- **Jaeger** (open source)
- **Lightstep**
- **Honeycomb**
- **OpenTelemetry**

## 5.2. Contêineres e Kubernetes 

Idealmente, cada microsserviço deve rodar **de forma isolada**. Contêineres são uma maneira leve e eficiente de conseguir isso.

**Vantagens dos Contêineres:**
- Tempo de inicialização rápido
- Consumo menor de recursos que VMs
- Isolamento de processos
- Portabilidade entre ambientes

Depois de começar a usar contêineres, você provavelmente vai precisar orquestrá-los. É aí que entra o **Kubernetes**.

**No entanto, não é necessário começar com Kubernetes.** Se você tem apenas alguns microsserviços, soluções mais simples podem bastar. Só pense em Kubernetes quando o gerenciamento da infraestrutura começar a virar um gargalo.

**Se possível, use serviços gerenciados:**
- **GKE** (Google Kubernetes Engine)
- **EKS** (Amazon Elastic Kubernetes Service)
- **AKS** (Azure Kubernetes Service)

## 5.3. Streaming de Dados 

Mesmo em uma arquitetura distribuída, os microsserviços **precisam trocar informações** — e, muitas vezes, isso precisa acontecer em tempo real.

Ferramentas de **streaming de dados**, como o **Apache Kafka**, se tornaram populares por permitir comunicação assíncrona, escalável e resiliente.

**Com Kafka você pode:**
- Publicar e consumir mensagens entre serviços
- Garantir que os dados fluam mesmo que um serviço esteja temporariamente indisponível
- Processar streams em tempo real

**Ferramentas Complementares:**
- **ksqlDB** (processamento de streams)
- **Apache Flink** (análises em tempo real)
- **Debezium** (captura mudanças de bancos relacionais)

## 5.4. Nuvem Pública e Serverless 

À medida que sua arquitetura cresce, a complexidade da infraestrutura também aumenta. É nesse ponto que os **serviços gerenciados da nuvem pública** se tornam aliados poderosos.

**Provedores Principais:**
- **AWS (Amazon Web Services)**
- **Azure (Microsoft)**
- **Google Cloud Platform (GCP)**

### Serverless (Function as a Service)

O modelo **serverless** permite subir funções, APIs ou componentes sem precisar se preocupar com servidores.

**Tecnologias:**
- **AWS Lambda**
- **Google Cloud Functions**
- **Azure Functions**

**Benefícios:**
- Escala automaticamente
- Você paga apenas pelo que usa
- Reduz custos operacionais
- Acelera o desenvolvimento

**Casos de Uso Ideal:**
- Tarefas event-driven
- Processar uploads
- Responder a chamadas HTTP
- Lidar com eventos de fila
