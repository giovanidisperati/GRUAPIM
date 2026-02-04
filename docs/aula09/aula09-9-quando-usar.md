---
layout: aula
title: "9. Devo Usar Microsserviços?"
parent: Aula 09 - Introdução aos Microsserviços
nav_order: 9
---

# **9. Devo Usar Microsserviços?**

Embora os microsserviços sejam amplamente discutidos e adotados por muitas empresas, **isso não significa que eles devem ser o padrão para todo projeto**. Como vimos, essa arquitetura traz benefícios reais — mas também desafios significativos. Antes de optar por microsserviços, é essencial refletir sobre **o contexto da sua equipe, do seu produto e da sua infraestrutura**.

Microsserviços são **uma das possíveis abordagens arquiteturais**, não *a* única. Em muitos casos, **um monólito bem estruturado pode atender melhor às suas necessidades** — especialmente no início do desenvolvimento.

## 9.1. Quando Microsserviços Podem Não Ser a Melhor Escolha

### 🔧 Produtos novos ou startups em estágio inicial

No começo de um produto, **muita coisa muda rapidamente**: requisitos, funcionalidades e até a visão do negócio. Separar o sistema em microsserviços nesse estágio **pode criar mais dores do que ganhos**, pois os limites entre serviços ainda não estão claros. A cada mudança no modelo de domínio, você terá que ajustar as interfaces entre serviços — algo **caro e trabalhoso**.

Além disso, pensar "vamos usar microsserviços agora porque no futuro vamos precisar escalar" **pode ser uma armadilha**. Na prática, você ainda não sabe se o produto será um sucesso, nem como ele vai evoluir. É melhor **validar o produto com uma arquitetura simples** e refatorar mais tarde, se necessário — como fizeram empresas como Uber e Flickr, que começaram de forma muito diferente do que são hoje.

### 👩‍💻 Equipes pequenas

Com poucos desenvolvedores, manter microsserviços pode ser **um fardo desnecessário**. Cada serviço exige deploy, monitoramento, testes, integração e operação. Esse "imposto" só se justifica quando há um **time suficientemente grande para se beneficiar da divisão de responsabilidades**. Para times pequenos, manter um monólito modular é mais produtivo — e migrar para microsserviços no futuro é sempre uma opção.

### 📦 Software entregue aos clientes

Se você desenvolve **softwares que são instalados e gerenciados pelos próprios clientes** (como ERPs, CRMs ou sistemas embarcados), os microsserviços podem ser um desafio. Seus clientes talvez estejam acostumados com um instalador simples e local. Pedir que eles configurem Kubernetes ou orquestradores de containers **pode gerar resistência, erros e frustração**. Microsserviços funcionam melhor quando você tem **controle sobre o ambiente de execução**.

## 9.2. Onde Microsserviços Realmente Brilham

### 👥 Equipes grandes e crescimento organizacional

Se sua organização está crescendo e há **muitas pessoas trabalhando no mesmo sistema**, microsserviços podem ajudar a reduzir conflitos entre equipes. Ao dividir o sistema em partes independentes com **responsabilidades bem definidas**, cada equipe pode trabalhar em seu serviço sem travar os demais, **reduzindo a contenção na entrega**. Empresas em estágio de scale-up, com dezenas ou centenas de desenvolvedores, costumam se beneficiar muito dessa separação.

### 🌐 Aplicações SaaS (Software como Serviço)

Sistemas SaaS precisam estar disponíveis **24 horas por dia, 7 dias por semana**, o que torna o deploy e a manutenção mais complexos. Microsserviços permitem **atualizar partes do sistema de forma independente**, com menos risco de interrupção. Além disso, permitem **escalar serviços individualmente**, otimizando custos e desempenho conforme o uso de cada módulo.

### ☁️ Integração com a nuvem e uso de múltiplas tecnologias

Microsserviços funcionam muito bem com plataformas de nuvem. Você pode **escolher a tecnologia certa para cada serviço**, aproveitando ao máximo os recursos disponíveis — como executar um serviço em serverless, outro em uma VM e outro em uma plataforma gerenciada. Essa flexibilidade permite **experimentar e evoluir rapidamente**.

### 📱 Novos canais e transformação digital

Organizações que estão passando por **transformação digital** e precisam expor suas funcionalidades para diferentes canais — web, mobile, APIs públicas, dispositivos IoT — se beneficiam da **composição flexível** dos microsserviços. É mais fácil **reaproveitar partes do sistema** e entregá-las em formatos diferentes para novos tipos de clientes ou parceiros.

### 🔄 Evolução contínua e flexibilidade futura

Microsserviços oferecem um grau alto de **flexibilidade para evoluir o sistema ao longo do tempo**. Você pode refatorar ou substituir um serviço sem mexer no restante, adicionar funcionalidades isoladamente e experimentar abordagens diferentes com menos impacto. Isso é especialmente útil em **sistemas vivos**, que mudam constantemente. Claro, essa liberdade tem um custo — mas, quando bem justificada, pode valer a pena.


