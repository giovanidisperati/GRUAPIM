---
layout: aula
title: "7. Context Mapping"
parent: Aula 10 - Domain-Driven Design
nav_order: 7
---

# **7. Context Mapping - Cartografia das Relações**

Vamos aprender a mapear relacionamentos entre contextos usando Context Maps e seus padrões.

## 7.1. O Problema: Contextos Não Vivem Isolados

Até agora, aprendemos a identificar e delimitar Bounded Contexts — fronteiras onde um modelo de domínio é válido e consistente. Mas há um problema fundamental:

**Bounded Contexts não existem em isolamento.**

Eles precisam se comunicar, trocar dados e coordenar operações. Por exemplo:
- O contexto de **Pedidos** precisa saber se o **Pagamento** foi aprovado
- O contexto de **Envio** precisa conhecer os **Itens do Pedido**
- O contexto de **Notificações** precisa ser informado quando um **Pedido** muda de status

Como gerenciamos essas interações sem criar acoplamento e confusão?

## 7.2. O que é um Context Map?

> 🧠 **Definição:** Um *Context Map* é um **diagrama que mostra todos os Bounded Contexts de um sistema e os relacionamentos entre eles**, incluindo os padrões de integração e as dependências.

O Context Map é uma **ferramenta estratégica** que:
- Documenta a arquitetura atual
- Revela dependências e riscos
- Guia decisões de integração
- Facilita comunicação entre times

## 7.3. Estudo de Caso: Sistema de Companhia Aérea

Vamos explorar um exemplo clássico para ilustrar Context Mapping.

### Cenário: Sistema de Reservas de Voos

Uma companhia aérea tem os seguintes Bounded Contexts:

#### 1. **Flight Operations Context**
- **Responsabilidade**: Gerenciar operações de voo (tripulação, aeronaves, rotas)
- **Modelo principal**: `Flight`, `Aircraft`, `Crew`

#### 2. **Booking Context**
- **Responsabilidade**: Reservas de passageiros
- **Modelo principal**: `Booking`, `Passenger`, `Seat`

#### 3. **Ticketing Context**
- **Responsabilidade**: Emissão de bilhetes
- **Modelo principal**: `Ticket`, `TicketNumber`, `BoardingPass`

#### 4. **Loyalty Program Context**
- **Responsabilidade**: Programas de fidelidade e milhas
- **Modelo principal**: `Member`, `Miles`, `Tier`

#### 5. **Pricing Context**
- **Responsabilidade**: Precificação dinâmica de voos
- **Modelo principal**: `Fare`, `PriceRule`, `Promotion`

### Relacionamentos Entre Contextos:

```
┌─────────────────┐
│   Flight Ops    │
│   (Upstream)    │
└────────┬────────┘
         │
         │ (fornece dados de voos)
         ▼
┌─────────────────┐         ┌─────────────────┐
│    Booking      │  ──→    │    Pricing      │
│  (Downstream)   │  (consulta preços)        │
└────────┬────────┘         └─────────────────┘
         │
         │ (cria bilhete após confirmação)
         ▼
┌─────────────────┐         ┌─────────────────┐
│   Ticketing     │  ──→    │  Loyalty Prog   │
│                 │  (acumula milhas)         │
└─────────────────┘         └─────────────────┘
```

## 7.4. Padrões de Relacionamento Entre Contextos

O DDD define **padrões específicos** para classificar relacionamentos entre contextos. Esses padrões deixam explícito:
- Quem tem mais poder na relação
- Quem pode mudar contratos
- Como os dados fluem
- Qual lado precisa se adaptar

Vamos explorar os principais padrões.

---

## 7.5. Partnership (Parceria)

### Definição:
Dois contextos **dependem mutuamente** um do outro para ter sucesso. Ambos os times coordenam evolutivamente suas interfaces.

### Características:
- Comunicação bidirecional forte
- Sucesso interdependente
- Coordenação de releases
- Compromisso de compatibilidade

### Quando usar:
- Times próximos e colaborativos
- Ambos os contextos evoluem juntos
- Falha de um impacta o outro diretamente

### Exemplo:
```
┌─────────────────┐ ←─→ ┌─────────────────┐
│   Booking       │     │  Flight Ops     │
│   Context       │     │   Context       │
└─────────────────┘     └─────────────────┘
    Partnership
```

**Booking** precisa de dados de voos em tempo real.  
**Flight Ops** precisa saber quais voos têm reservas.  
→ Times trabalham juntos, coordenam mudanças.

---

## 7.6. Shared Kernel (Núcleo Compartilhado)

### Definição:
Um **subconjunto do modelo de domínio** é compartilhado entre dois ou mais contextos. Mudanças nesse núcleo requerem coordenação.

### Características:
- Código compartilhado (mesma biblioteca, mesmo repositório)
- Acoplamento explícito e aceito
- Mudanças coordenadas
- Geralmente pequeno (10-20% do modelo)

### Quando usar:
- Dois contextos realmente precisam do mesmo modelo
- Custo de duplicação é maior que custo de coordenação
- Times têm comunicação frequente

### Quando EVITAR:
- Times geograficamente distribuídos
- Ciclos de release muito diferentes
- Baixa comunicação entre times

### Exemplo:
```
┌─────────────────┐     ┌─────────────────┐
│   Booking       │     │   Pricing       │
│   Context       │     │   Context       │
└────────┬────────┘     └────────┬────────┘
         │                       │
         └───────┬───────────────┘
                 │
         ┌───────▼────────┐
         │  Shared Kernel  │
         │  (FlightId,     │
         │   RouteInfo)    │
         └─────────────────┘
```

Ambos compartilham a definição de `FlightId` e `RouteInfo`.  
→ Qualquer mudança nessas classes precisa ser coordenada.

> ⚠️ **Aviso:** Shared Kernel introduz acoplamento forte. Use com moderação.

---

## 7.7. Customer-Supplier (Cliente-Fornecedor)

### Definição:
Um contexto **fornece** (Upstream) serviços ou dados para outro contexto **consumidor** (Downstream). O Upstream define a API, mas considera as necessidades do Downstream.

### Características:
- Relação de poder desequilibrada (Upstream tem mais)
- Consumidor depende do fornecedor
- Fornecedor tem compromisso com consumidor
- Negociação de features e contratos

### Quando usar:
- Upstream provê serviço essencial para Downstream
- Ambos os times têm comunicação
- Upstream se compromete a atender Downstream

### Exemplo:
```
┌─────────────────┐
│  Flight Ops     │  (Upstream - Fornecedor)
│                 │
└────────┬────────┘
         │
         │ API: GET /flights/{id}
         ▼
┌─────────────────┐
│   Booking       │  (Downstream - Cliente)
│                 │
└─────────────────┘
```

**Flight Ops** fornece dados de voos via API.  
**Booking** consome esses dados.  
→ Flight Ops coordena com Booking ao mudar API.

---

## 7.8. Conformist (Conformista)

### Definição:
O contexto Downstream **se conforma** totalmente ao modelo do Upstream, **sem poder de negociação**. O Upstream não se compromete a atender necessidades do Downstream.

### Características:
- Downstream tem zero influência
- Usa modelo do Upstream "como está"
- Sem camada de tradução
- Aceitação de limitações

### Quando usar:
- Upstream é externo (SaaS, API pública)
- Upstream não negocia mudanças
- Custo de anticorrupção é muito alto

### Quando EVITAR:
- Modelo do Upstream polui seu domínio
- Você tem poder de negociação

### Exemplo:
```
┌─────────────────┐
│  Payment Gateway│  (Upstream - externo)
│  (Stripe API)   │
└────────┬────────┘
         │
         │ usa modelo Stripe diretamente
         ▼
┌─────────────────┐
│   Billing       │  (Downstream - conformista)
│   Context       │
└─────────────────┘
```

**Billing** usa diretamente os objetos da API do Stripe.  
→ Aceita as limitações da API sem tradução.

---

## 7.9. Anti-Corruption Layer (ACL - Camada Anticorrupção)

### Definição:
O contexto Downstream cria uma **camada de tradução** que isola seu modelo do modelo do Upstream. Protege o domínio interno de "corrupção" por modelos externos.

### Características:
- Camada de adaptação explícita
- Traduz modelos externos para internos
- Protege a pureza do domínio
- Isola de mudanças no Upstream

### Quando usar:
- Modelo do Upstream não se alinha com seu domínio
- Upstream pode mudar sem aviso
- Você quer proteger seu modelo interno
- Integração com sistemas legados

### Exemplo:
```
┌─────────────────┐
│  Legacy ERP     │  (Upstream - sistema legado)
│                 │
└────────┬────────┘
         │
         │ XML SOAP complexo
         ▼
┌─────────────────┐
│       ACL       │  ← Camada de tradução
│  (Translator)   │
└────────┬────────┘
         │
         │ modelo limpo e simples
         ▼
┌─────────────────┐
│   Order Mgmt    │  (Downstream - protegido)
│   Context       │
└─────────────────┘
```

**Order Management** não quer se contaminar com o modelo confuso do ERP.  
→ ACL traduz XML complexo para modelo de domínio limpo.

### Implementação da ACL:

```java
// Modelo externo (do ERP legado)
class LegacyOrder {
    String ord_id;
    String cust_ref;
    List<LegacyItem> items;
    int status; // 0=pending, 1=confirmed, 2=shipped
}

// Modelo interno (do nosso domínio)
class Order {
    OrderId id;
    CustomerId customer;
    List<OrderLine> lines;
    OrderStatus status; // enum limpo
}

// ACL - Camada Anticorrupção
class OrderAdapter {
    public Order fromLegacy(LegacyOrder legacy) {
        return new Order(
            new OrderId(legacy.ord_id),
            new CustomerId(legacy.cust_ref),
            legacy.items.stream()
                .map(this::toOrderLine)
                .collect(toList()),
            mapStatus(legacy.status)
        );
    }
    
    private OrderStatus mapStatus(int code) {
        return switch(code) {
            case 0 -> OrderStatus.PENDING;
            case 1 -> OrderStatus.CONFIRMED;
            case 2 -> OrderStatus.SHIPPED;
            default -> throw new IllegalArgumentException();
        };
    }
}
```

---

## 7.10. Open Host Service (OHS)

### Definição:
Um contexto Upstream define uma **API pública** bem documentada que pode ser usada por **múltiplos** consumidores Downstream.

### Características:
- API pública e documentada
- Versionamento e estabilidade
- Múltiplos consumidores
- Contrato explícito

### Quando usar:
- Upstream serve múltiplos clientes
- API deve ser estável
- Consumidores diversos (internos e externos)

### Exemplo:
```
                ┌─────────────────┐
                │  Product Catalog│  (Open Host Service)
                │   API v2.0      │
                └────────┬────────┘
                         │
         ┌───────────────┼───────────────┐
         │               │               │
         ▼               ▼               ▼
   ┌─────────┐   ┌─────────┐   ┌─────────┐
   │ Web App │   │ Mobile  │   │ Partner │
   │         │   │   App   │   │   API   │
   └─────────┘   └─────────┘   └─────────┘
```

---

## 7.11. Published Language (PL)

### Definição:
A comunicação entre contextos usa uma **linguagem compartilhada e bem documentada** (geralmente JSON Schema, XML Schema, Protobuf).

### Características:
- Formato padrão de dados
- Documentação formal
- Independente de implementação
- Estabilidade de contrato

### Quando usar:
- Múltiplos contextos precisam trocar dados
- Integração com sistemas externos
- Necessidade de versionamento formal

### Exemplo:
```json
// Published Language: OrderPlaced Event
{
  "event_type": "OrderPlaced",
  "version": "1.0",
  "order_id": "ORD-12345",
  "customer_id": "CUST-789",
  "total": 199.99,
  "currency": "USD",
  "timestamp": "2026-02-05T14:30:00Z"
}
```

Múltiplos contextos (Shipping, Billing, Loyalty) consomem este evento.  
→ O schema JSON é a "linguagem publicada".

---

## 7.12. Separate Ways (Caminhos Separados)

### Definição:
Dois contextos **não se integram**. Cada um resolve seu problema de forma independente, aceitando duplicação.

### Características:
- Zero integração
- Duplicação de dados ou funcionalidades
- Independência total

### Quando usar:
- Custo de integração > benefício
- Contextos realmente independentes
- Duplicação é aceitável

### Exemplo:
```
┌─────────────────┐     ┌─────────────────┐
│  Internal CRM   │ ✗   │  Public Website │
│                 │     │                 │
└─────────────────┘     └─────────────────┘
     (não se integram)
```

CRM mantém lista de clientes internamente.  
Website mantém cadastro próprio de usuários.  
→ Dois sistemas, dois bancos, sem sincronização.

---

## 7.13. Visualizando o Context Map

Um Context Map completo mostra todos os relacionamentos:

```
┌──────────────────────────────────────────────────────────────┐
│                    CONTEXT MAP                                │
└──────────────────────────────────────────────────────────────┘

┌─────────────┐
│  Flight Ops │ (OHS)
└──────┬──────┘
       │ Customer-Supplier
       ▼
┌─────────────┐  Partnership  ┌─────────────┐
│   Booking   │ ←─────────→   │   Pricing   │
└──────┬──────┘               └─────────────┘
       │ ACL
       ▼
┌─────────────┐               ┌─────────────┐
│  Ticketing  │  Conformist   │  Legacy ERP │
└──────┬──────┘ ─────────→    └─────────────┘
       │ Published Language (OrderPlaced Event)
       ▼
┌─────────────┐
│   Loyalty   │
└─────────────┘
```

---

## 7.14. Conclusão: Mapeando para Gerenciar Complexidade

Context Maps são ferramentas essenciais para:

✅ **Documentar arquitetura** — todos veem os relacionamentos  
✅ **Identificar riscos** — dependências críticas ficam explícitas  
✅ **Guiar decisões** — qual padrão usar em cada integração  
✅ **Facilitar comunicação** — linguagem compartilhada entre times  
✅ **Planejar evolução** — onde introduzir ACL, onde migrar para OHS  

Na próxima seção, vamos mergulhar nos **padrões táticos do DDD**: Entities, Value Objects, Aggregates e muito mais.

---

### ⏭️ Próxima Seção

Na próxima seção, vamos explorar **DDD Tático** e padrões de implementação.

**[→ Ir para DDD Tático](aula10-8-ddd-tatico)**

---

## Referências Desta Seção

**EVANS, Eric.** *Domain-Driven Design: Tackling Complexity in the Heart of Software.* Boston: Addison-Wesley, 2003.

**VERNON, Vaughn.** *Implementing Domain-Driven Design.* Boston: Addison-Wesley, 2013.

**KHONONOV, Vlad.** *Learning Domain-Driven Design: Aligning Software Architecture and Business Strategy.* Sebastopol: O'Reilly Media, 2021.
