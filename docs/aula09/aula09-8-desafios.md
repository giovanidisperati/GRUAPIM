---
layout: aula
title: "8. Desafios dos Microsserviços"
parent: Aula 09 - Introdução aos Microsserviços
nav_order: 8
---

# **8. Os Desafios (Pain Points) dos Microsserviços**

Apesar de suas muitas vantagens, a adoção de uma arquitetura de microsserviços **traz também uma série de complexidades**. Antes de migrar para esse modelo, é importante fazer uma análise equilibrada entre os **benefícios e os custos**. Muitos dos desafios enfrentados aqui são, na verdade, **inerentes a qualquer sistema distribuído** — ou seja, também podem surgir em um *monólito distribuído* mal estruturado.


## 8.1. Experiência do Desenvolvedor (Developer Experience - DX)

À medida que o número de serviços cresce, **a rotina de desenvolvimento se torna mais pesada**. Em um monólito, o desenvolvedor pode rodar tudo localmente com um simples comando. Já em microsserviços, é comum precisar subir diversos serviços para testar algo — o que **pode consumir muitos recursos da máquina**, especialmente se estiver usando runtimes pesados como a JVM.

Rodar 10 ou mais microsserviços localmente pode ser impraticável. Isso leva a discussões como "devo desenvolver diretamente na nuvem?" — o que pode atrasar o ciclo de feedback e dificultar a produtividade. Uma abordagem mais prática é **limitar o escopo local de desenvolvimento** (ex: rodar apenas os serviços essenciais), embora isso possa gerar conflitos com culturas de "propriedade coletiva" do código.

## 8.2. Sobrecarga Tecnológica

Microsserviços não exigem, mas permitem o uso de várias tecnologias diferentes — linguagens, bancos de dados, frameworks. Esse poder de escolha pode ser tentador e, em muitos casos, **as equipes acabam adotando um "combo" de ferramentas novas ao mesmo tempo**, o que gera uma curva de aprendizado e manutenção considerável.

A diversidade tecnológica só vale a pena se for usada com parcimônia e propósito. É comum empresas acharem que, ao adotar microsserviços, também precisam usar Kubernetes, mensageria, múltiplas linguagens, bancos especializados etc. Mas a verdade é que **cada nova tecnologia adiciona complexidade**, e isso pode atrasar entregas e aumentar o custo de manutenção.

**Dica:** introduza tecnologias **à medida que os problemas surgirem**. Não é necessário usar Kafka ou Kubernetes se você tem apenas três serviços e consegue gerenciá-los bem com ferramentas simples.


## 8.3. Custo

No curto prazo, é muito comum que os **custos aumentem** com a adoção de microsserviços. São mais processos rodando, mais máquinas ou containers, mais tráfego de rede, mais armazenamento e ferramentas de suporte — sem falar em **licenciamento e operações**.

Além disso, há o **custo de aprendizado e adaptação da equipe**. O tempo para entender novas práticas, modelar os serviços corretamente e automatizar o deploy impacta diretamente na entrega de novas funcionalidades.

Se a sua organização tem foco principal em **redução de custos**, pode ser que os microsserviços tragam mais dor de cabeça do que benefícios. Por outro lado, se o objetivo for **acelerar o crescimento**, atender mais usuários ou liberar funcionalidades em paralelo, a arquitetura distribuída pode ajudar a gerar mais valor — e, assim, mais receita.

## 8.4. Relatórios

Em um monólito, os dados geralmente estão centralizados em um único banco. Isso facilita a geração de relatórios: basta consultar diretamente a base (ou uma réplica de leitura) para gerar dashboards, gráficos ou análises.

Nos microsserviços, **os dados ficam espalhados por vários bancos isolados**, o que torna mais difícil realizar análises globais. Se os dados de vendas estão em um serviço, os de clientes em outro, e os de produtos em um terceiro, é preciso encontrar maneiras de agregá-los.

Soluções incluem o uso de **data lakes**, **pipelines de streaming** com Kafka ou o envio regular de dados para um **repositório de relatórios unificado**. Mas todas essas opções exigem **esforço adicional e novas tecnologias**.

## 8.5. Monitoramento e Solução de Problemas

No monólito, monitorar e debugar é relativamente simples: se a aplicação caiu ou está lenta, o impacto é visível. Em microsserviços, **o sistema continua funcionando mesmo que partes estejam falhando** — o que torna os problemas mais sutis e difíceis de detectar.

Além disso, há **muitos serviços, cada um com logs e métricas próprios**, e entender o que está acontecendo requer ferramentas adequadas de **observabilidade**. Um único serviço travado em 100% de CPU pode não parecer crítico, mas pode estar afetando silenciosamente a experiência do usuário.

Ferramentas como **Grafana, Prometheus, Jaeger e Loki** ajudam muito nesse cenário, mas exigem investimento em cultura e infraestrutura para serem bem aproveitadas.

## 8.6. Segurança

Em um monólito, a maior parte dos dados trafega **dentro do mesmo processo**. Já nos microsserviços, os dados circulam **entre processos via rede**, o que expõe o sistema a novos riscos: **interceptação, manipulação de mensagens e acessos indevidos**.

É fundamental adotar práticas como:

* Criptografia de dados em trânsito (TLS);
* Autenticação e autorização entre serviços (por exemplo, com tokens JWT);
* Validação de contratos e limites de acesso.

Microsserviços exigem uma **abordagem mais cuidadosa de segurança** — tanto no nível de rede quanto no de aplicação.

## 8.7. Testes

Testar microsserviços é mais desafiador. Quanto maior o número de serviços envolvidos, **mais difícil é garantir que tudo funcione em conjunto**. Testes de ponta a ponta se tornam pesados, lentos e propensos a erros falsos — como falhas causadas por um serviço estar fora do ar, e não por um bug real.

Isso pode levar a **retorno decrescente sobre os testes automatizados tradicionais**. Em vez disso, é recomendável adotar **testes de contrato**, **testes em produção controlados** (como canary releases) e **entregas progressivas** com controle de impacto.

## 8.8. Latência

Ao dividir uma lógica que antes rodava localmente em vários serviços separados, as chamadas passam a trafegar pela rede, o que introduz **atrasos de comunicação**. Cada requisição agora envolve:

* Serializar os dados;
* Enviar pela rede;
* Esperar resposta.

Isso pode aumentar significativamente o tempo de resposta de algumas operações. O impacto varia conforme o volume de chamadas e a arquitetura da rede — e deve ser **medido e monitorado** com ferramentas de rastreamento, como **Jaeger**.

Por isso, migrar para microsserviços deve ser um processo **incremental**, com monitoramento constante da **latência de ponta a ponta**.

## 8.9. Consistência de Dados

Em sistemas monolíticos, é comum contar com **transações de banco de dados** para garantir que múltiplas operações ocorram juntas ou não ocorram — o chamado *all-or-nothing*. Em microsserviços, como os dados estão espalhados em bancos diferentes, **transações distribuídas são raras e arriscadas**.

Por isso, é preciso adotar modelos como:

* **Sagas**, onde uma sequência de etapas é executada em diferentes serviços com compensações em caso de falha;
* **Consistência eventual**, aceitando que os dados estejam temporariamente fora de sincronia, mas se resolvam ao longo do tempo.

Esses conceitos exigem uma mudança profunda na forma como pensamos e tratamos os dados — o que pode ser difícil para quem está migrando de sistemas legados.

