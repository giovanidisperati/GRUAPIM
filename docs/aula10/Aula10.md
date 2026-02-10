---
layout: aula
title: Aula 10 - Domain-Driven Design
nav_order: 12
has_children: true
permalink: /aula10
---

# **Aula 10 – Domain-Driven Design (DDD)**

**Domínio, Subdomínios, Bounded Contexts e Modelagem Estratégica**

---

## 📋 Índice da Aula

Esta aula está organizada em **10 seções navegáveis** que cobrem desde os fundamentos teóricos do DDD até sua aplicação prática na extração de microsserviços. Cada seção foi cuidadosamente estruturada para proporcionar uma progressão pedagógica adequada ao ensino superior.

---

### **Seções Teóricas Fundamentais (1-4)**

Estas seções estabelecem as bases conceituais e filosóficas do Domain-Driven Design:

1. **[Introdução ao DDD](aula10-1-introducao)** ⭐ COMEÇAR AQUI
   - O que é DDD e por que ele importa?
   - Motivação histórica e contexto de Eric Evans
   - Quando o domínio não existe, a bagunça domina
   - DDD como abordagem estratégica

2. **[Os Três Tipos de Complexidade](aula10-2-complexidade)**
   - Complexidade da Solução Técnica (Acidental)
   - Complexidade do Legado (Acidental)
   - Complexidade do Domínio (Essencial)
   - O foco deslocado: Tecnologia vs. Negócio
   - DDD-Lite e o Modelo Anêmico

3. **[Linguagem Ubíqua - A Base da Colaboração](aula10-3-linguagem-ubiqua)**
   - A falha da comunicação tradicional
   - Knowledge Crunching: aprendizado mútuo
   - O perigo de pular a linguagem
   - Curando a anemia: repensando os setters
   - O desafio do idioma (Português vs. Inglês)

4. **[Event Storming - Descobrindo o Domínio](aula10-4-event-storming)**
   - O que é Event Storming?
   - Como funciona: post-its e eventos
   - Descoberta de comandos e agregados
   - Identificação de fronteiras naturais
   - Por que Event Storming é fundamental?

---

### **Seções de Modelagem Estratégica (5-7)**

Estas seções exploram como estruturar e organizar o software de acordo com o domínio:

5. **[Domínio e Subdomínio](aula10-5-dominio-subdominio)**
   - Espaço do Problema (Problem Space)
   - Classificação estratégica dos subdomínios
   - Core Domain, Supporting e Generic
   - Implicações: Construir vs. Comprar
   - Identificando o verdadeiro Core

6. **[Bounded Contexts - Fronteiras Explícitas](aula10-6-bounded-contexts)**
   - Espaço da Solução (Solution Space)
   - Mapeando conceitos: Subdomínio → Contexto
   - Exemplo: "Aluno" vs "Cliente" na Universidade
   - Lei de Conway e estrutura organizacional
   - Alinhamento Problem Space → Solution Space

7. **[Context Mapping - Cartografia das Relações](aula10-7-context-mapping)**
   - O Mapa de Contexto (Context Map)
   - Estudo de caso: Companhia Aérea
   - Padrões de relacionamento
   - Partnership, Shared Kernel, Customer-Supplier
   - Conformist e Anti-Corruption Layer (ACL)

---

### **Seções de Implementação e Prática (8-10)**

Estas seções conectam a teoria à prática, com padrões táticos e aplicação real:

8. **[DDD Tático - Padrões de Implementação](aula10-8-ddd-tatico)**
   - Design Estratégico vs. Design Tático
   - Entities (Entidades)
   - Value Objects (Objetos de Valor)
   - Aggregates (Agregados) e Aggregate Roots
   - Repositories e Domain Services
   - Exemplos práticos em Spring Boot

9. **[Aplicação Prática - Do Conceito ao Código](aula10-9-aplicacao-pratica)**
   - Quando usar DDD?
   - Quando NÃO usar DDD?
   - Estudos de caso reais
   - Armadilhas comuns
   - Dicas de ouro para modelagem
   - Conclusão: DDD como filosofia

10. **[Projeto Final - Extração de Microsserviço](aula10-10-projeto-final)**
    - Descrição do projeto final da disciplina
    - Etapas da atividade
    - Revisão e delimitação do Bounded Context
    - Migração do modelo de domínio
    - Adaptação do monólito original
    - Formato de entrega

---

## 🎯 Objetivos de Aprendizado

Ao final desta aula, você será capaz de:

- ✅ Compreender os fundamentos filosóficos e teóricos do Domain-Driven Design
- ✅ Distinguir entre complexidade essencial e acidental no desenvolvimento de software
- ✅ Aplicar a Linguagem Ubíqua para melhorar a colaboração entre times
- ✅ Conduzir workshops de Event Storming para descoberta do domínio
- ✅ Identificar e classificar subdomínios (Core, Supporting, Generic)
- ✅ Delimitar Bounded Contexts com base em fronteiras linguísticas
- ✅ Mapear relacionamentos entre contextos usando Context Maps
- ✅ Implementar padrões táticos do DDD (Entities, VOs, Aggregates)
- ✅ Decidir quando usar (e quando não usar) DDD
- ✅ Extrair um microsserviço de um monólito aplicando DDD estratégico

---

## 📚 Referências Bibliográficas Principais

Esta aula é baseada nas seguintes obras seminais e contemporâneas do DDD:

1. **EVANS, Eric.** *Domain-Driven Design: Tackling Complexity in the Heart of Software.* Boston: Addison-Wesley, 2003. (O "Livro Azul")

2. **VERNON, Vaughn.** *Implementing Domain-Driven Design.* Boston: Addison-Wesley, 2013. (O "Livro Vermelho")

3. **VERNON, Vaughn.** *Domain-Driven Design Distilled.* Boston: Addison-Wesley, 2016.

4. **KHONONOV, Vlad.** *Learning Domain-Driven Design: Aligning Software Architecture and Business Strategy.* Sebastopol: O'Reilly Media, 2021.

5. **BRANDOLINI, Alberto.** *Introducing EventStorming.* Leanpub, 2021.

6. **FOWLER, Martin.** Anemic Domain Model. MartinFowler.com, 2003.

---

## 🔗 Conexão com as Aulas Anteriores

Esta aula é a continuação natural da **Aula 09 - Introdução aos Microsserviços**, onde vocês:

- Mapearam Bounded Contexts do projeto intermediário
- Identificaram candidatos para extração de microsserviços
- Justificaram escolhas baseadas em critérios estratégicos

Agora, vamos aprofundar os fundamentos teóricos que sustentam essas decisões e aprender como aplicá-los na prática.