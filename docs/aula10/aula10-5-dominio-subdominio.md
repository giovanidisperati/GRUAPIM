---
layout: aula
title: "5. Domínio e Subdomínio"
parent: Aula 10 - Domain-Driven Design
nav_order: 5
---

# **5. Domínio e Subdomínio**

Vamos explorar os conceitos de Domínio e Subdomínio, e como classificá-los estrategicamente.

## 5.1. O que é Domínio?

O **domínio** é o ponto de partida do DDD. Trata-se do **conjunto de regras, atividades e conhecimentos específicos** do problema que a aplicação se propõe a resolver. Em outras palavras, o domínio é o **assunto central** do seu sistema.

> 🧠 **Definição:** O domínio representa o "mundo real" que o software está modelando. Ele inclui os termos, conceitos e regras de negócio relevantes para os usuários e stakeholders da aplicação.

Por exemplo, no caso de uma aplicação de gerenciamento de tarefas (To-Do List), o domínio inclui conceitos como:

* Tarefa (`Task`)
* Responsável (`Owner`)
* Prioridade (`Priority`)
* Categoria (`Category`)
* Data de vencimento (`DueDate`)
* Conclusão (`Completed`)
* Atribuição e regras de modificação

O DDD nos orienta a **modelar o domínio com riqueza e precisão**, para que o sistema reflita fielmente a lógica do negócio, evitando simplificações técnicas que distorçam ou empobreçam o modelo.

## 5.2. Problem Space vs. Solution Space

Para compreender profundamente domínios e subdomínios, precisamos primeiro distinguir dois espaços conceituais fundamentais no DDD:

### Problem Space (Espaço do Problema)

O **Problem Space** representa **o que estamos tentando resolver** — o problema de negócio em si, independentemente de como vamos implementá-lo tecnologicamente.

No Problem Space, lidamos com:
- **Domínio**: o problema geral
- **Subdomínios**: partes distintas do problema

**Características:**
- Foco no **negócio**
- Independente de tecnologia
- Definido pelos **especialistas de domínio**
- Responde "**O QUE**" precisamos resolver

### Solution Space (Espaço da Solução)

O **Solution Space** representa **como vamos resolver** — a arquitetura e implementação do software.

No Solution Space, lidamos com:
- **Bounded Contexts**: fronteiras de implementação
- **Context Maps**: como os contextos se relacionam
- Código, APIs, bancos de dados

**Características:**
- Foco na **implementação**
- Decisões técnicas e arquiteturais
- Definido pelos **desenvolvedores e arquitetos**
- Responde "**COMO**" vamos resolver

### A Relação Entre os Dois Espaços:

```
PROBLEM SPACE              →              SOLUTION SPACE
(Negócio)                                 (Implementação)

┌──────────────────┐                     ┌──────────────────┐
│   DOMÍNIO        │                     │   APLICAÇÃO      │
│                  │                     │                  │
│  ┌────────────┐  │                     │  ┌────────────┐  │
│  │ Subdomínio │  │  ──────────→        │  │  Bounded   │  │
│  │   Core     │  │     mapeia          │  │  Context   │  │
│  └────────────┘  │                     │  └────────────┘  │
│                  │                     │                  │
│  ┌────────────┐  │                     │  ┌────────────┐  │
│  │ Subdomínio │  │  ──────────→        │  │  Bounded   │  │
│  │ Supporting │  │     mapeia          │  │  Context   │  │
│  └────────────┘  │                     │  └────────────┘  │
│                  │                     │                  │
└──────────────────┘                     └──────────────────┘
```

> 💡 **Importante:** Subdomínios existem no **Problem Space** (são partes do problema de negócio). Bounded Contexts existem no **Solution Space** (são fronteiras de implementação no código). Idealmente, há um mapeamento 1:1, mas nem sempre isso é possível ou desejável.

## 5.3. Subdomínios: Dividindo o Problema

Em sistemas mais complexos, o domínio principal normalmente se divide em **partes menores com responsabilidades bem definidas**, chamadas **subdomínios**. Essa separação ajuda a organizar melhor o sistema, atribuindo a cada parte um foco específico.

> 🧠 **Definição:** Subdomínios são divisões internas do domínio principal. Cada subdomínio representa uma **área funcional coesa**, com seu próprio conjunto de regras, entidades e operações.

## 5.4. Classificação Estratégica dos Subdomínios

O DDD classifica os subdomínios em **três categorias estratégicas**, segundo Khononov (2021) e Evans (2003):

### 5.4.1. Core Domain (Domínio Central)

**Definição:** A parte do sistema que fornece **vantagem competitiva** à organização. É o que torna o produto único e valioso.

**Características:**
- Onde a empresa **se diferencia** dos competidores
- Contém as regras de negócio **mais complexas e valiosas**
- Justifica o **maior investimento** de tempo e recursos
- Deve ser desenvolvido **internamente**, nunca terceirizado
- Evolui **constantemente** conforme o negócio cresce

**Exemplos:**
- **Netflix**: Algoritmo de recomendação personalizada
- **Uber**: Sistema de matching motorista-passageiro em tempo real
- **Amazon**: Motor de precificação dinâmica e logística preditiva
- **Nubank**: Análise de crédito em tempo real sem burocracia

**Perguntas para identificar Core Domain:**
- "Se tirarmos isso do sistema, ainda teríamos um produto diferenciado?"
- "Nossos competidores fazem isso melhor ou pior que nós?"
- "Isso é o que nossos clientes realmente valorizam?"

### 5.4.2. Supporting Subdomain (Subdomínio de Suporte)

**Definição:** Apoia o Core Domain, mas **não é o diferencial** do produto. É necessário para o sistema funcionar, mas não é inovador.

**Características:**
- Suporta o Core Domain
- Pode ser **desenvolvido internamente**, mas com menos investimento
- Pode usar **soluções prontas** adaptadas
- Geralmente específico do domínio (não genérico)
- Menor complexidade que o Core

**Exemplos:**
- **E-commerce**: Sistema de gestão de usuários (não é o que vende, mas precisa existir)
- **Banco digital**: Cadastro de clientes (necessário, mas não o diferencial)
- **SaaS**: Gerenciamento de planos e assinaturas

**Perguntas para identificar Supporting:**
- "Isso é essencial para o Core funcionar?"
- "Outros concorrentes têm exatamente a mesma funcionalidade?"
- "Podemos usar uma solução pronta e adaptar?"

### 5.4.3. Generic Subdomain (Subdomínio Genérico)

**Definição:** Problemas **comuns a vários domínios**, com soluções amplamente disponíveis no mercado.

**Características:**
- **Não é específico** do seu negócio
- Soluções prontas **funcionam bem**
- Candidato ideal para **compra** ou **uso de bibliotecas**
- Baixo valor estratégico
- Não justifica desenvolvimento customizado

**Exemplos:**
- Autenticação (OAuth, Auth0, Keycloak)
- Envio de e-mails (SendGrid, Mailgun)
- Geração de relatórios (JasperReports, Crystal Reports)
- Armazenamento de arquivos (S3, Google Cloud Storage)
- Sistema de cache (Redis, Memcached)
- Notificações push (Firebase, OneSignal)

**Perguntas para identificar Generic:**
- "Todo mundo precisa disso do mesmo jeito?"
- "Existe uma biblioteca ou SaaS que resolve bem isso?"
- "Desenvolver isso internamente agrega valor ao negócio?"

## 5.5. Tabela Comparativa: Core vs. Supporting vs. Generic

| Critério | Core Domain | Supporting Subdomain | Generic Subdomain |
|----------|-------------|----------------------|-------------------|
| **Valor estratégico** | ⭐⭐⭐⭐⭐ Muito alto | ⭐⭐⭐ Médio | ⭐ Baixo |
| **Complexidade** | Alta | Média | Baixa |
| **Investimento** | Máximo | Moderado | Mínimo |
| **Equipe** | Melhor time | Time competente | Pode terceirizar |
| **Decisão Build vs Buy** | Sempre BUILD | Pode adaptar | Sempre BUY |
| **Velocidade de mudança** | Alta (evolui com negócio) | Média | Baixa (estável) |
| **Diferencial competitivo** | Sim | Não | Não |

## 5.6. Exemplo Prático: Sistema de To-Do List

Vamos classificar os subdomínios do nosso exemplo de sistema de tarefas:

| Subdomínio | Classificação | Justificativa |
|------------|---------------|---------------|
| **Gestão de Tarefas** | 🟢 **Core** | É o diferencial do produto — regras de priorização, categorização inteligente, workflows personalizados |
| **Autenticação de Usuários** | 🟡 **Supporting** | Necessário, mas não é o que vende. Pode usar OAuth2 + JWT com alguma customização |
| **Envio de Notificações** | 🔵 **Generic** | Enviar e-mail/push é igual em qualquer sistema. Use SendGrid ou similar |
| **Relatórios Gerenciais** | 🟡 **Supporting** | Importante para gestão, mas não o core. Pode usar biblioteca pronta com customização |
| **Gestão de Permissões (ACL)** | 🟡 **Supporting** | Necessário para multi-tenancy, mas pode usar frameworks existentes |

## 5.7. Implicações da Classificação: Construir vs. Comprar

A classificação estratégica dos subdomínios **orienta decisões de investimento**:

### Para Core Domains:
✅ **Invista pesadamente**  
✅ **Equipe sênior dedicada**  
✅ **Desenvolvimento interno**  
✅ **Testes extensivos**  
✅ **Monitoramento rigoroso**  
✅ **Documentação detalhada**  
✅ **Experimentação e inovação**

### Para Supporting Subdomains:
⚠️ **Invista moderadamente**  
⚠️ **Equipe competente**  
⚠️ **Pode adaptar soluções prontas**  
⚠️ **Testes adequados**  
⚠️ **Documentação básica**

### Para Generic Subdomains:
❌ **Não reinvente a roda**  
✅ **Use bibliotecas, frameworks ou SaaS**  
✅ **Equipe pode ser júnior (integração)**  
✅ **Testes de integração são suficientes**  
✅ **Documentação: referência à lib externa**

> 💡 **Regra de ouro:** Não gaste tempo de desenvolvedores seniores construindo coisas genéricas que já existem prontas. Guarde esse talento para o Core Domain.

## 5.8. Identificando o Verdadeiro Core Domain

Muitas organizações cometem o erro de **identificar incorretamente seu Core Domain**. Sinais de alerta:

### 🚩 Sinais de que você identificou errado:

1. **Todo mundo diz que "tudo é core"**
   - Isso indica falta de foco estratégico
   - Core Domain deve ser 20-30% do sistema, não 100%

2. **O "core" é algo genérico**
   - "Nosso core é autenticação segura"
   - Se todos fazem igual, não é core

3. **O investimento está espalhado**
   - Core Domain deve receber **mais recursos** que os outros

4. **Terceirização do core**
   - Se você terceiriza algo, não é core

### ✅ Sinais de que você acertou:

1. **É onde você inova**
2. **Clientes pagam por isso**
3. **Competidores querem copiar**
4. **Regras de negócio mudam frequentemente**
5. **Stakeholders discutem intensamente sobre ele**

## 5.9. Subdomínios e Microsserviços

A relação entre subdomínios e microsserviços:

> **Boa prática:** Cada microsserviço deve corresponder a um subdomínio (ou bounded context), nunca a uma camada técnica.

**❌ Errado (divisão por camada técnica):**
- `user-service`
- `database-service`
- `authentication-service`

**✅ Correto (divisão por subdomínio):**
- `task-management-service` (Core)
- `user-access-service` (Supporting)
- `notification-service` (Generic)

## 5.10. Conclusão: O Problem Space Orienta o Solution Space

Compreender domínios e subdomínios é **essencial antes de projetar a arquitetura**. Eles vivem no Problem Space e orientam as decisões do Solution Space:

1. **Identifique subdomínios** → partes do problema de negócio
2. **Classifique estrategicamente** → Core / Supporting / Generic
3. **Aloque recursos apropriadamente** → invista no Core
4. **Decida Build vs. Buy** → construa Core, compre Generic
5. **Mapeie para Bounded Contexts** → (próxima seção)

Na próxima seção, vamos explorar **Bounded Contexts** — como traduzimos os subdomínios do Problem Space para fronteiras de implementação no Solution Space.

---

### ⏭️ Próxima Seção

Na próxima seção, vamos explorar **Bounded Contexts** e fronteiras explícitas.

**[→ Ir para Bounded Contexts](aula10-6-bounded-contexts)**

---

## Referências Desta Seção

**EVANS, Eric.** *Domain-Driven Design: Tackling Complexity in the Heart of Software.* Boston: Addison-Wesley, 2003.

**KHONONOV, Vlad.** *Learning Domain-Driven Design: Aligning Software Architecture and Business Strategy.* Sebastopol: O'Reilly Media, 2021.

**VERNON, Vaughn.** *Domain-Driven Design Distilled.* Boston: Addison-Wesley, 2016.
