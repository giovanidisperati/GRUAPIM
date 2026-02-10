---
layout: aula
title: "4. Event Storming"
parent: Aula 10 - Domain-Driven Design
nav_order: 4
---

# **4. Event Storming - Descobrindo o Domínio**

Vamos aprender Event Storming, uma técnica prática e colaborativa para descobrir o domínio e construir a linguagem ubíqua.

## 4.1. O Desafio: Como Descobrir o Domínio?

Até agora, falamos muito sobre a importância de **compreender o domínio** e **construir uma linguagem ubíqua**. Mas como fazemos isso na prática?

Como extrair o conhecimento tácito dos especialistas de negócio? Como garantir que desenvolvedores, analistas, product owners e stakeholders compartilhem o mesmo entendimento do problema?

A resposta: **Event Storming**.

## 4.2. O que é Event Storming?

**Event Storming** é uma **técnica de workshop colaborativo e visual** criada por Alberto Brandolini para explorar domínios complexos de forma rápida e eficaz.

O nome vem de:
- **Event** (evento): focamos em "o que acontece" no sistema
- **Storming**: referência a "brainstorming" — é uma atividade dinâmica e de alta energia

### Características principais:

- **Colaborativo**: envolve desenvolvedores, especialistas de negócio, designers, POs
- **Visual**: usa post-its coloridos em uma parede grande
- **Orientado a eventos**: foca no que acontece, não em estruturas de dados
- **Rápido**: em poucas horas, mapeia-se todo um sistema complexo
- **Democrático**: todos contribuem igualmente, sem hierarquias

## 4.3. Por que Event Storming Funciona?

Brandolini (2021) identificou que os métodos tradicionais de levantamento de requisitos têm problemas estruturais:

### Problemas dos métodos tradicionais:

1. **Documentos enormes** que ninguém lê completamente
2. **Reuniões longas** onde poucos falam e muitos se distraem
3. **Diagramas técnicos** (UML, ER) que especialistas de negócio não entendem
4. **Conhecimento concentrado** em poucas cabeças
5. **Falta de alinhamento** — cada um entende o sistema de forma diferente

### Como Event Storming resolve isso:

✅ **Visual e acessível** — post-its coloridos são universais  
✅ **Todos participam ativamente** — não há espectadores  
✅ **Foca no comportamento** — eventos são intuitivos ("Pedido Confirmado", "Pagamento Recebido")  
✅ **Rápido e dinâmico** — mantém energia e engajamento  
✅ **Alinhamento imediato** — conflitos e ambiguidades aparecem rapidamente  

## 4.4. Como Funciona: O Workshop de Event Storming

### 4.4.1. Material Necessário

- **Parede grande** (ou quadro branco gigante)
- **Post-its de cores diferentes** (laranja, azul, rosa, roxo, verde, amarelo)
- **Canetas para todos**
- **Pessoas certas** (especialistas + desenvolvedores + PO/BA)
- **Tempo** (4-8 horas, com intervalos)

### 4.4.2. Fase 1: Tempestade de Eventos (Domain Events)

A primeira fase é **caótica por design**.

**Instrução para o grupo:**  
> "Escrevam nos post-its laranjas TUDO que pode acontecer neste sistema, usando tempo passado. Não se preocupem com ordem ou completude. Coloquem na parede."

**Exemplos de eventos:**
- 🟧 Pedido Realizado
- 🟧 Pagamento Confirmado
- 🟧 Produto Enviado
- 🟧 Entrega Atrasada
- 🟧 Cliente Reclamou
- 🟧 Devolução Solicitada

Nesta fase:
- **Não há ordem cronológica** (ainda)
- **Não há julgamento** — todas as ideias são válidas
- **Especialistas dominam** — eles conhecem o negócio
- **Desenvolvedores aprendem** — ouvem e perguntam

Resultado: **dezenas (às vezes centenas) de eventos na parede**.

### 4.4.3. Fase 2: Ordenação Temporal

Agora o grupo reorganiza os eventos em uma **linha do tempo**.

- Eventos que ocorrem primeiro vão à esquerda
- Eventos subsequentes vão à direita
- Eventos que ocorrem em paralelo ficam na mesma altura vertical

Nesta fase, **começam a aparecer discordâncias**:

- "Espera, o pagamento acontece antes ou depois da confirmação?"
- "Esse evento só acontece se o cliente for pessoa física ou também jurídica?"
- "Às vezes esse fluxo é diferente..."

Essas discussões são **extremamente valiosas** — revelam ambiguidades e regras ocultas do domínio.

### 4.4.4. Fase 3: Adicionando Comandos (Commands)

Agora adicionamos **post-its azuis** representando **comandos** — ações que causam eventos.

```
┌──────────────────┐         ┌──────────────────┐
│  🟦 Confirmar     │   →     │  🟧 Pedido       │
│     Pedido       │         │     Confirmado   │
└──────────────────┘         └──────────────────┘
```

**Comando** = ação que o usuário (ou sistema) executa  
**Evento** = resultado dessa ação

Exemplos:
- 🟦 Adicionar Item ao Carrinho → 🟧 Item Adicionado
- 🟦 Aplicar Cupom de Desconto → 🟧 Desconto Aplicado
- 🟦 Cancelar Pedido → 🟧 Pedido Cancelado

### 4.4.5. Fase 4: Identificando Agregados (Aggregates)

Com eventos e comandos mapeados, começam a aparecer **clusters de comportamento relacionado**.

Por exemplo, todos esses eventos giram em torno de "Pedido":
- Pedido Criado
- Item Adicionado ao Pedido
- Pedido Confirmado
- Pedido Cancelado

Isso sugere a existência de um **Agregado** chamado `Pedido`.

Marcamos com **post-its amarelos grandes** delimitando esses agregados.

### 4.4.6. Fase 5: Identificando Bounded Contexts

Finalmente, o grupo procura **fronteiras naturais** onde:
- A linguagem muda
- Times diferentes trabalham
- Responsabilidades são distintas

Por exemplo:
- **Contexto de Vendas**: Carrinho, Checkout, Pedido
- **Contexto de Pagamento**: Fatura, Pagamento, Reembolso
- **Contexto de Logística**: Envio, Rastreamento, Entrega

Essas fronteiras são marcadas com **fitas** ou **linhas grossas** na parede.

## 4.5. O Produto Final: Um Mapa Visual Compartilhado

Ao final do workshop, a parede contém:

```
┌────────────────────────────────────────────────────────────┐
│              BOUNDED CONTEXT: Vendas                        │
│                                                             │
│  🟦 Criar       →  🟧 Carrinho     (🟡 Carrinho)           │
│     Carrinho          Criado                                │
│                                                             │
│  🟦 Adicionar   →  🟧 Item         (🟡 Item)               │
│     Item            Adicionado                              │
│                                                             │
│  🟦 Finalizar   →  🟧 Pedido       (🟡 Pedido)             │
│     Compra          Criado                                  │
│                                                             │
└────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────┐
│              BOUNDED CONTEXT: Pagamento                     │
│                                                             │
│  🟦 Processar   →  🟧 Pagamento    (🟡 Pagamento)          │
│     Pagamento       Processado                              │
│                                                             │
│  🟦 Confirmar   →  🟧 Pagamento    (🟡 Pagamento)          │
│     Pagamento       Confirmado                              │
│                                                             │
└────────────────────────────────────────────────────────────┘
```

Este mapa é:
- **Compreensível** por todos (técnicos e não-técnicos)
- **Alinhado** — todos participaram da construção
- **Documentação viva** — pode ser fotografado e digitalizado
- **Base para modelagem** — informa decisões de arquitetura

## 4.6. Por que Event Storming é Fundamental para DDD?

Event Storming resolve os problemas centrais que impedem bom DDD:

### 1. Dissolve a barreira comunicacional
Especialistas e desenvolvedores trabalham **lado a lado**, falando a mesma língua (eventos).

### 2. Descobre a Linguagem Ubíqua organicamente
Os termos surgem naturalmente das discussões. Quando alguém diz "Pedido Confirmado" e outro diz "Compra Finalizada", o grupo discute qual termo é mais preciso.

### 3. Revela complexidade oculta rapidamente
Em poucas horas, aparecem regras que levariam semanas em reuniões tradicionais:
- "Ah, mas se for um pedido B2B, o fluxo é diferente..."
- "Nesse caso, temos que enviar um evento para o sistema legado..."

### 4. Identifica Bounded Contexts naturalmente
As fronteiras emergem das próprias discussões — não são impostas artificialmente.

### 5. Cria propriedade compartilhada
Todos contribuíram. Todos entendem. Todos se sentem donos do modelo.

## 4.7. Quando Usar Event Storming?

Event Storming é especialmente útil em:

✅ **Início de projetos complexos** — mapear o domínio antes de codificar  
✅ **Sistemas legados** — entender um sistema obscuro  
✅ **Times novos** — alinhar rapidamente todos os membros  
✅ **Refatorações grandes** — antes de dividir um monólito  
✅ **Introdução de DDD** — ensinar conceitos de forma prática  

## 4.8. Variações de Event Storming

Brandolini (2021) descreve diferentes "sabores" de Event Storming:

### Big Picture Event Storming
- **Objetivo**: Mapear todo o domínio em alto nível
- **Duração**: 4-8 horas
- **Participantes**: 10-30 pessoas (cross-funcional)
- **Resultado**: Visão geral, identificação de Bounded Contexts

### Process Modeling Event Storming
- **Objetivo**: Detalhar um processo específico
- **Duração**: 2-4 horas
- **Participantes**: 5-10 pessoas (foco em um contexto)
- **Resultado**: Fluxo detalhado, agregados, políticas

### Software Design Event Storming
- **Objetivo**: Traduzir modelo em design de software
- **Duração**: 2-4 horas
- **Participantes**: Desenvolvedores + arquiteto
- **Resultado**: Estrutura de classes, APIs, eventos de domínio

## 4.9. Dicas Práticas para Facilitar Event Storming

### Antes do Workshop:

1. **Escolha o escopo** — não tente mapear tudo de uma vez
2. **Convide as pessoas certas** — especialistas são essenciais
3. **Reserve espaço adequado** — sala grande, parede vazia
4. **Compre post-its de qualidade** — barato descola da parede
5. **Explique as regras** — 5 minutos de contexto sobre a técnica

### Durante o Workshop:

6. **Mantenha energia alta** — faça intervalos curtos e frequentes
7. **Não deixe discussões travarem** — marque dúvidas em post-its roxos
8. **Deixe conflitos emergirem** — são valiosos, não evite
9. **Tire fotos frequentemente** — post-its caem, pessoas reorganizam
10. **Não se preocupe com perfeição** — é um processo iterativo

### Depois do Workshop:

11. **Documente digitalmente** — transcreva para Miro, Mural ou similar
12. **Compartilhe amplamente** — tire fotos e distribua
13. **Mantenha atualizado** — Event Storming é vivo, não um artefato estático
14. **Use como referência** — consulte ao modelar código

## 4.10. Conclusão: Do Caos à Clareza

Event Storming pode parecer caótico no início — dezenas de pessoas colando post-its numa parede sem ordem aparente. Mas essa aparente desordem é, na verdade, **o processo de destilação do conhecimento tácito em conhecimento explícito**.

Ao final, o que era confuso e disperso nas cabeças das pessoas se torna **um modelo visual compartilhado e compreensível por todos**.

E mais importante: esse modelo **já está na linguagem do negócio**, pronto para ser traduzido em código que reflete fielmente o domínio.

Na próxima seção, vamos formalizar os conceitos de **Domínio e Subdomínio**, explorando como classificar e priorizar diferentes partes do negócio.
