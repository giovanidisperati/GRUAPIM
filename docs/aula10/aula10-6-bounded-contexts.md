---
layout: aula
title: "6. Bounded Contexts"
parent: Aula 10 - Domain-Driven Design
nav_order: 6
---

# **6. Bounded Contexts - Fronteiras Explícitas**

Vamos compreender Bounded Contexts no Solution Space e como mapear subdomínios para contextos.

## 6.1. Do Problem Space ao Solution Space

Como vimos na seção anterior, **Subdomínios** existem no **Problem Space** — são partes do problema de negócio que estamos tentando resolver.

Agora vamos falar de **Bounded Contexts**, que existem no **Solution Space** — são fronteiras de implementação no código.

> 🧠 **Definição:** Um *bounded context* é um **limite explícito onde um determinado modelo de domínio é válido, coeso e consistente**. Dentro desse limite, os termos e comportamentos têm significado específico. Fora dele, o mesmo termo pode significar outra coisa.

## 6.2. O Problema da Ambiguidade Linguística

Muitos desenvolvedores confundem "domínio" com "módulo". O DDD nos mostra que **um mesmo domínio pode ter diferentes interpretações, dependendo do contexto**. Para evitar confusões, introduz-se o conceito de **Bounded Context**, talvez o conceito mais fundamental de todos no DDD.

Por exemplo, pense na palavra **"usuário"**:

* No contexto de autenticação, "usuário" significa um login, senha e conjunto de permissões;
* No contexto de tarefas, "usuário" pode significar um colaborador, com nome, cargo e responsabilidades;
* No contexto de cobrança, "usuário" pode significar um cliente com histórico de pagamentos.

Se usássemos um único modelo `User` para todos os contextos, provavelmente ele teria campos como:

```java
User {
    id;
    login;
    passwordHash;
    email;
    role;
    fullName;
    profilePicture;
    List<Task> tasks;
    PaymentMethod preferredPayment;
    List<Invoice> invoices;
    ...
}
```

Essa classe acabaria servindo a **vários propósitos ao mesmo tempo**, criando um modelo inconsistente, frágil e de difícil manutenção.

## 6.3. A Solução: Delimitar Contextos com Clareza

O DDD nos ensina a **delimitar contextos com clareza**. Dentro de cada contexto, usamos os termos com significados específicos, e **controlamos com rigor as interações entre eles**.

Assim, podemos ter:

* No contexto `Authentication`: `AuthUser { username, passwordHash, role }`
* No contexto `TaskManagement`: `TaskOwner { name, userId, assignedTasks }`
* No contexto `Billing`: `Customer { userId, paymentMethods, invoices }`

E as interações entre contextos são feitas **com regras explícitas de tradução**. Por exemplo, através de eventos, APIs, adaptadores ou mapeamentos.

## 6.4. Exemplo Prático: Universidade

Vamos explorar um exemplo clássico para ilustrar a importância dos Bounded Contexts.

### Cenário: Sistema de uma Universidade

Considere a palavra **"Aluno"** em diferentes departamentos de uma universidade:

#### No Contexto "Matrícula Acadêmica":
```java
public class Student {
    private StudentId id;
    private FullName name;
    private CPF cpf;
    private AcademicProgram program;
    private EnrollmentStatus status; // ACTIVE, SUSPENDED, GRADUATED
    private List<CourseEnrollment> enrolledCourses;
    
    public void enrollInCourse(Course course) { ... }
    public boolean canGraduate() { ... }
}
```

Para este contexto, "Aluno" é alguém que:
- Está matriculado em um programa
- Pode se inscrever em disciplinas
- Tem status acadêmico

#### No Contexto "Biblioteca":
```java
public class LibraryPatron {
    private PatronId id;
    private String name; // nome simplificado
    private Email email;
    private List<BookLoan> activeLoans;
    private int maxLoansAllowed;
    private boolean hasOutstandingFines;
    
    public boolean canBorrowBook() { ... }
    public void returnBook(Book book) { ... }
}
```

Para este contexto, "Aluno" (agora `LibraryPatron`) é alguém que:
- Pode emprestar livros
- Tem limite de empréstimos
- Pode ter multas pendentes

#### No Contexto "Financeiro":
```java
public class PayingStudent {
    private StudentId id;
    private String name; // apenas para referência
    private TuitionPlan plan;
    private List<Payment> payments;
    private Decimal outstandingBalance;
    
    public boolean isUpToDate() { ... }
    public void recordPayment(Payment payment) { ... }
}
```

Para este contexto, "Aluno" (agora `PayingStudent`) é alguém que:
- Tem um plano de pagamento
- Faz mensalidades
- Pode ter débitos pendentes

### Observações Importantes:

1. **Mesmo conceito, modelos diferentes**: "Aluno" significa coisas diferentes em cada contexto
2. **Campos relevantes variam**: CPF importa na Matrícula, mas não na Biblioteca
3. **Comportamentos variam**: `canGraduate()` não faz sentido no contexto Financeiro
4. **Invariantes diferentes**: Regras de validação são específicas de cada contexto

> 💡 **Insight:** Se tentássemos criar uma única classe `Student` para todos esses contextos, teríamos uma "god class" com dezenas de campos e métodos, violando o Princípio da Responsabilidade Única e criando acoplamento desnecessário.

## 6.5. Mapeando Subdomínios para Bounded Contexts

Idealmente, há um **mapeamento 1:1** entre Subdomínios (Problem Space) e Bounded Contexts (Solution Space):

```
PROBLEM SPACE              →          SOLUTION SPACE

Subdomínio: Vendas         →          Context: Sales
Subdomínio: Pagamentos     →          Context: Payments
Subdomínio: Logística      →          Context: Shipping
```

Mas nem sempre isso é possível ou desejável:

### Caso 1: Múltiplos Contextos para um Subdomínio
Um subdomínio complexo pode precisar de vários contextos:

```
Subdomínio: E-commerce     →          Context: Catalog
                           →          Context: ShoppingCart
                           →          Context: Checkout
```

### Caso 2: Um Contexto para Múltiplos Subdomínios
Subdomínios simples podem compartilhar um contexto:

```
Subdomínio: Notificações   →          Context: Communications
Subdomínio: E-mails        →          ↑
```

### Caso 3: Contextos Compartilhados (Shared Kernel)
Dois contextos compartilham parte do modelo (uso raro):

```
Context: Warehouse    ←─→    Context: Shipping
        ↓                           ↓
    (ambos usam modelo de "Location")
```

## 6.6. Lei de Conway e Estrutura Organizacional

Uma observação fundamental de Melvin Conway (1968):

> "Organizações que projetam sistemas estão condenadas a produzir designs que são cópias das estruturas de comunicação dessas organizações."

### Implicação para Bounded Contexts:

**Bounded Contexts devem alinhar-se com a estrutura de times!**

Se você tem:
- **Time de Vendas** → crie um Bounded Context "Sales"
- **Time de Logística** → crie um Bounded Context "Shipping"
- **Time de Pagamentos** → crie um Bounded Context "Payments"

Isso permite que cada time trabalhe de forma **autônoma**, sem bloquear outros times.

### Anti-padrão comum:

```
Time A → trabalha em múltiplos contextos
Time B → trabalha nos mesmos contextos que Time A
         ↓
     Conflitos, bloqueios, dependências
```

### Padrão recomendado:

```
Time A → dono completo do Context: Sales
Time B → dono completo do Context: Payments
         ↓
     Autonomia, deploys independentes
```

## 6.7. Bounded Contexts e Microsserviços

No mundo de microsserviços, **cada microsserviço deve corresponder a um Bounded Context** (ou ser menor que um contexto, nunca maior).

### ✅ Boa Arquitetura:
```
Microservice: sales-service      ← Bounded Context: Sales
Microservice: payment-service    ← Bounded Context: Payments
Microservice: shipping-service   ← Bounded Context: Shipping
```

Cada microsserviço:
- Tem seu próprio banco de dados
- Expõe uma API bem definida
- Mantém sua linguagem ubíqua interna
- Evolui independentemente

### ❌ Arquitetura Problemática:
```
Microservice: user-service       ← serve múltiplos contextos
Microservice: data-service       ← camada técnica (não contexto de negócio)
```

## 6.8. Definindo as Fronteiras de um Bounded Context

Como saber onde termina um contexto e começa outro?

### Sinais de que você precisa de contextos separados:

1. **Mudança de Linguagem**
   - Mesma palavra, significados diferentes
   - Termos que só fazem sentido em uma parte

2. **Mudança de Responsabilidade**
   - Times diferentes
   - Especialistas diferentes
   - Objetivos de negócio diferentes

3. **Mudança de Ritmo**
   - Uma parte muda frequentemente, outra é estável
   - Diferentes ciclos de release

4. **Mudança de Tecnologia**
   - Uma parte precisa de tecnologia específica
   - Requisitos não-funcionais diferentes (latência, throughput)

5. **Mudança de Escala**
   - Uma parte precisa escalar independentemente
   - Padrões de carga muito diferentes

## 6.9. Exemplo Prático: Sistema de E-commerce

Vamos aplicar na prática:

### Bounded Contexts Identificados:

#### 1. **Catalog Context**
- **Responsabilidade**: Gerenciar produtos, categorias, preços
- **Linguagem**: Product, Category, Price, SKU
- **Modelo-chave**: `Product`, `Category`

#### 2. **Shopping Cart Context**
- **Responsabilidade**: Gerenciar carrinhos de compra
- **Linguagem**: Cart, CartItem, AddItem, RemoveItem
- **Modelo-chave**: `ShoppingCart`, `CartItem`

#### 3. **Order Context**
- **Responsabilidade**: Processar pedidos
- **Linguagem**: Order, OrderLine, OrderStatus, PlaceOrder
- **Modelo-chave**: `Order`, `OrderLine`

#### 4. **Payment Context**
- **Responsabilidade**: Processar pagamentos
- **Linguagem**: Payment, Transaction, PaymentMethod, Charge
- **Modelo-chave**: `Payment`, `Transaction`

#### 5. **Shipping Context**
- **Responsabilidade**: Gerenciar envios
- **Linguagem**: Shipment, Tracking, Delivery
- **Modelo-chave**: `Shipment`, `TrackingNumber`

### Relação entre Contextos:

```
Catalog → (informa preços) → ShoppingCart
ShoppingCart → (cria) → Order
Order → (solicita) → Payment
Order → (solicita) → Shipping
```

Cada contexto **não conhece os detalhes internos dos outros**, apenas troca informações via eventos ou APIs bem definidas.

## 6.10. Conclusão: Fronteiras Explícitas, Modelos Limpos

Bounded Contexts são a **ferramenta fundamental para gerenciar complexidade** em sistemas grandes:

✅ **Permitem modelos especializados** para cada parte do negócio  
✅ **Eliminam ambiguidade linguística**  
✅ **Facilitam evolução independente**  
✅ **Alinha arquitetura com organização**  
✅ **Viabilizam microsserviços coerentes**  

Mas os contextos não vivem isolados — eles precisam se comunicar. Na próxima seção, vamos explorar **Context Mapping**: como mapear e gerenciar relacionamentos entre contextos.

---

### ⏭️ Próxima Seção

Na próxima seção, vamos explorar **Context Mapping** e os padrões de relacionamento.

**[→ Ir para Context Mapping](aula10-7-context-mapping)**

---

## Referências Desta Seção

**EVANS, Eric.** *Domain-Driven Design: Tackling Complexity in the Heart of Software.* Boston: Addison-Wesley, 2003.

**VERNON, Vaughn.** *Implementing Domain-Driven Design.* Boston: Addison-Wesley, 2013.

**CONWAY, Melvin E.** How Do Committees Invent? *Datamation*, v. 14, n. 5, p. 28-31, 1968.
