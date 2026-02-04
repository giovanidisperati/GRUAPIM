---
layout: aula
title: "10. Resumo e Exercício"
parent: Aula 09 - Introdução aos Microsserviços
nav_order: 10
---

# **10. Resumo da Ópera e Exercício**


As arquiteturas de microsserviços podem oferecer um enorme grau de **flexibilidade** na escolha de tecnologia, no manuseio de robustez e escalabilidade, na organização de equipes e muito mais. Essa flexibilidade é, em parte, o motivo pelo qual muitas pessoas estão adotando essas arquiteturas. Mas os microsserviços trazem consigo um grau significativo de **complexidade**, e você precisa garantir que essa complexidade seja justificada. Para muitos, eles se tornaram uma arquitetura de sistema padrão, a ser usada em praticamente todas as situações. No entanto, o autor ainda pensa que são uma escolha arquitetural cujo uso deve ser justificado pelos problemas que você está tentando resolver; muitas vezes, abordagens mais simples podem entregar resultados muito mais facilmente.

No entanto, muitas organizações, especialmente as maiores, mostraram o quão eficazes os microsserviços podem ser. Quando os conceitos centrais dos microsserviços são devidamente compreendidos e implementados, eles podem ajudar a criar arquiteturas capacitadoras e produtivas que podem ajudar os sistemas a se tornarem mais do que a soma de suas partes.

---

## Exercício

### **Atividade Prática-Teórica: Planejando a Evolução da sua API**

Agora que temos uma base conceitual sobre o universo dos microsserviços e entendemos a jornada que nos trouxe até aqui, é hora de um desafio prático-teórico. O objetivo desta atividade é conectar esses novos conceitos diretamente com o trabalho que vocês já realizaram: a API REST que cada equipe desenvolveu com Spring Boot no projeto intermediário da disciplina.

Antes de apresentarmos formalmente os conceitos de **Domain-Driven Design (DDD)** na próxima aula, vamos usar essa oportunidade para que vocês deem um passo à frente. Este exercício é uma preparação, uma oportunidade para investigar e aplicar um dos pilares mais importantes para o design de microsserviços.

### **A Missão da sua Equipe:**

Com base no projeto de API Spring Boot que vocês criaram, sua equipe deverá realizar um exercício de planejamento arquitetural. O trabalho consiste em quatro etapas principais:

#### 1. Pesquisa Autônoma sobre DDD

A equipe deverá pesquisar o conceito fundamental de **"Bounded Context" (Contexto Delimitado)** do Domain-Driven Design. O foco não é se tornar um especialista, mas entender:
- O que é um Bounded Context?
- Por que ele é usado para organizar a complexidade de um software?
- Como ele ajuda a definir fronteiras lógicas dentro de um sistema?

#### 2. Análise do Domínio do seu Projeto

Olhem para a API que vocês construíram, mas desta vez, com uma nova perspectiva. Esqueçam por um momento as camadas técnicas (Controller, Service, Repository) e concentrem-se nas **capacidades de negócio** que o sistema possui. Perguntem-se: "Quais são as grandes áreas de responsabilidade da nossa aplicação?".

#### 3. Mapeamento dos Bounded Contexts

Com base na pesquisa e na análise do seu projeto, identifiquem e mapeiem os potenciais Bounded Contexts. Listem esses contextos e descrevam brevemente a responsabilidade de cada um. Por exemplo, em um e-commerce, poderíamos ter contextos como "Gestão de Catálogo", "Processamento de Pedidos" e "Controle de Inventário".

#### 4. Planejamento de Extração Estratégica

Nem todos os contextos precisam virar um microsserviço imediatamente. A extração é um processo estratégico. Escolham **1 ou 2 Bounded Contexts** que vocês consideram os candidatos ideais para serem os **primeiros microsserviços** a serem extraídos do seu monólito. Justifiquem a escolha com base em critérios como:

* **Taxa de Mudança:** É uma parte do sistema que muda com muita frequência?
* **Necessidades de Escalabilidade:** Essa funcionalidade exige escalar de forma independente do resto do sistema?
* **Criticidade para o Negócio:** É uma função tão crítica que se beneficiaria de um isolamento maior (robustez)?

Evidentemente nossos projetos não estão em produção, mas tentem pensar criticamente em como essa aplicação se comportaria no "mundo real" e, principalmente, considerar como isso afetaria as questões de escala da aplicação.

### **Formato da Entrega:**

Cada equipe deve preparar um documento simples (2 a 3 páginas) contendo:

* Um breve resumo do que vocês entenderam por "Bounded Context".
* A lista (ou um diagrama simples) dos Bounded Contexts identificados no seu projeto.
* A escolha dos 1 ou 2 microsserviços estratégicos e a justificativa clara para essa decisão. Aqui vocês descrever, também, o raciocínio que os levou às escolhas feitas pela equipe.

Este exercício não tem uma única resposta "certa". O objetivo é o processo de análise, discussão e o desenvolvimento do pensamento crítico sobre arquitetura de software. Os resultados e as dúvidas desta atividade servirão como ponto de partida para a nossa próxima aula.



# **Bom trabalho!⚒️**
