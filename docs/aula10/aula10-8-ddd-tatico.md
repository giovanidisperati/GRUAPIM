---
layout: aula
title: "8. DDD Tático"
parent: Aula 10 - Domain-Driven Design
nav_order: 8
---

# **8. DDD Tático - Padrões de Implementação**

Vamos explorar os padrões táticos do DDD: como implementar o domínio no código.

## 8.1. Design Estratégico vs. Design Tático

Até agora, focamos no **DDD Estratégico**:
- Identificar subdomínios
- Delimitar Bounded Contexts
- Mapear relacionamentos (Context Map)
- Definir linguagem ubíqua

Agora vamos mergulhar no **DDD Tático**: como implementar essas decisões estratégicas em código real.

| Camada          | Pergunta que responde                                              | Conceitos principais                                                                                                                       | Foco                                                  |
| --------------- | ------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------- |
| **Estratégica** | *"Quais partes do negócio existem e como se relacionam?"*          | Domínio, Subdomínio (Core / Supporting / Generic), Bounded Context, Context Map, Linguagem Ubíqua, tipos de relacionamento (ACL, etc.)    | **Desenhar fronteiras** e alinhar times               |
| **Tática**      | *"Como modelar e implementar as regras dentro de cada fronteira?"* | Entidade, Value Object, Aggregate Root, Domain Event, Domain Service, Repository, Factory                                                 | **Codificar comportamento** de forma coesa e testável |

> **Regra de bolso:** *Estratégia define limites; tática preenche conteúdo.*

## 8.2. Entidades (Entities)

### Definição:
Uma **Entidade** é um objeto que possui **identidade única** e **ciclo de vida** ao longo do tempo. Mesmo que seus atributos mudem, sua identidade permanece a mesma.

### Características:
- **Identidade imutável**: geralmente um ID (UUID, Long, etc.)
- **Ciclo de vida**: pode ser criada, modificada, persistida, deletada
- **Igualdade por identidade**: duas entidades são iguais se têm o mesmo ID

### Quando usar:
- Objeto precisa ser rastreado ao longo do tempo
- Importa **qual** instância é, não apenas seus valores
- Exemplo: `User`, `Order`, `Product`, `Task`

### Exemplo:

```java
public class Task {
    private final TaskId id;         // Identidade imutável
    private Title title;             // Pode mudar
    private Priority priority;       // Pode mudar
    private TaskStatus status;       // Pode mudar
    private Deadline deadline;
    
    public Task(TaskId id, Title title, Priority priority, Deadline deadline) {
        this.id = Objects.requireNonNull(id);
        this.title = Objects.requireNonNull(title);
        this.priority = Objects.requireNonNull(priority);
        this.status = TaskStatus.TODO;
        this.deadline = deadline;
    }
    
    // Igualdade por identidade
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Task)) return false;
        Task other = (Task) o;
        return id.equals(other.id);
    }
    
    @Override
    public int hashCode() {
        return id.hashCode();
    }
    
    // Comportamento rico
    public void complete() {
        if (this.status == TaskStatus.DONE) {
            throw new IllegalStateException("Task already completed");
        }
        this.status = TaskStatus.DONE;
    }
    
    public boolean isOverdue() {
        return deadline.isPast() && !status.isDone();
    }
}
```

### ❌ Antipadrão - Entidade Anêmica:
```java
// ERRADO - apenas getters/setters
public class Task {
    private Long id;
    private String title;
    private int status;
    
    // apenas getters e setters
    public void setStatus(int status) { this.status = status; }
}
```

## 8.3. Value Objects (Objetos de Valor)

### Definição:
Um **Value Object** é um objeto **imutável** que é definido por seus valores, não por uma identidade. Dois VOs com os mesmos valores são considerados iguais.

### Características:
- **Sem identidade própria**
- **Imutável**: não tem setters
- **Igualdade por valor**: comparados por todos os campos
- **Autovalidação**: valida-se no construtor

### Quando usar:
- Representa uma medida, quantidade ou descrição
- Não tem sentido rastrear ao longo do tempo
- Exemplos: `Money`, `Email`, `CPF`, `Address`, `DateRange`

### Exemplo:

```java
public record Title(String value) {
    public Title {
        if (value == null || value.isBlank()) {
            throw new IllegalArgumentException("Title cannot be empty");
        }
        if (value.length() < 3 || value.length() > 60) {
            throw new IllegalArgumentException("Title must be between 3 and 60 characters");
        }
    }
}

public record Deadline(LocalDate date) {
    public Deadline {
        Objects.requireNonNull(date, "Deadline cannot be null");
    }
    
    public boolean isPast() {
        return date.isBefore(LocalDate.now());
    }
    
    public boolean isPast(LocalDate referenceDate) {
        return date.isBefore(referenceDate);
    }
}

public record Money(BigDecimal amount, Currency currency) {
    public Money {
        if (amount.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException("Amount cannot be negative");
        }
        Objects.requireNonNull(currency);
    }
    
    public Money add(Money other) {
        if (!this.currency.equals(other.currency)) {
            throw new IllegalArgumentException("Cannot add different currencies");
        }
        return new Money(this.amount.add(other.amount), this.currency);
    }
}
```

### Benefícios dos Value Objects:

✅ **Eliminam "primitive obsession"** (uso excessivo de String, int, etc.)  
✅ **Validação centralizada** (valida no construtor, não em todo lugar)  
✅ **Tornam o código mais expressivo** (`Deadline` vs. `LocalDate`)  
✅ **Imutabilidade** (mais seguro, thread-safe)  
✅ **Comportamento rico** (métodos como `isPast()`, `add()`)  

## 8.4. Aggregates (Agregados) e Aggregate Roots

### Definição:
Um **Aggregate** é um **cluster de objetos de domínio** tratados como uma unidade para mudanças de dados. Um **Aggregate Root** é a entidade que **controla o acesso** ao agregado.

### Regras Fundamentais:

1. **Aggregate Root é a única porta de entrada**
   - Objetos externos não modificam entidades internas diretamente
   - Toda operação passa pelo Root

2. **Invariantes são protegidas pelo Root**
   - O Root garante que regras de negócio nunca são violadas

3. **Transações não cruzam agregados**
   - Cada agregado é uma transação atômica
   - Mudanças em múltiplos agregados = eventual consistency

4. **Referências entre agregados são feitas por ID**
   - Não guarda referência direta para entidade de outro agregado
   - Usa apenas o identificador

### Exemplo: Aggregate `Order`

```java
public class Order { // Aggregate Root
    private final OrderId id;
    private final CustomerId customerId; // apenas ID, não Customer
    private final List<OrderLine> lines; // entidades internas
    private OrderStatus status;
    private Money total;
    
    public Order(OrderId id, CustomerId customerId) {
        this.id = id;
        this.customerId = customerId;
        this.lines = new ArrayList<>();
        this.status = OrderStatus.PENDING;
        this.total = Money.zero(Currency.USD);
    }
    
    // Única forma de adicionar linha - passa pelo Root
    public void addLine(ProductId productId, int quantity, Money unitPrice) {
        // Valida invariante: não pode adicionar em pedido confirmado
        if (status != OrderStatus.PENDING) {
            throw new IllegalStateException("Cannot modify confirmed order");
        }
        
        // Cria linha interna
        OrderLine line = new OrderLine(productId, quantity, unitPrice);
        lines.add(line);
        
        // Atualiza total (mantém invariante: total sempre correto)
        recalculateTotal();
    }
    
    public void confirm() {
        if (lines.isEmpty()) {
            throw new IllegalStateException("Cannot confirm empty order");
        }
        this.status = OrderStatus.CONFIRMED;
    }
    
    private void recalculateTotal() {
        this.total = lines.stream()
            .map(OrderLine::getSubtotal)
            .reduce(Money.zero(Currency.USD), Money::add);
    }
    
    // Getters não permitem modificação
    public List<OrderLine> getLines() {
        return Collections.unmodifiableList(lines);
    }
}

// Entidade interna (não é Aggregate Root)
class OrderLine {
    private final ProductId productId;
    private int quantity;
    private Money unitPrice;
    
    OrderLine(ProductId productId, int quantity, Money unitPrice) {
        this.productId = productId;
        this.quantity = quantity;
        this.unitPrice = unitPrice;
    }
    
    Money getSubtotal() {
        return unitPrice.multiply(quantity);
    }
}
```

### Como Identificar Aggregate Boundaries:

✅ **Use Case Closure**: quais objetos mudam juntos em um caso de uso?  
✅ **Invariantes**: quais regras de negócio precisam ser garantidas atomicamente?  
✅ **Transacional Boundary**: o que precisa ser consistente imediatamente?  

## 8.5. Domain Events (Eventos de Domínio)

### Definição:
Um **Domain Event** é uma mensagem que descreve algo **significativo que já aconteceu** no domínio.

### Características:
- **Tempo passado**: "TaskCompleted", "OrderPlaced", "PaymentReceived"
- **Imutável**: uma vez criado, não muda
- **Carrega dados necessários**: informações relevantes do evento

### Quando usar:
- Comunicar entre Bounded Contexts
- Acionar reações (notificações, relatórios)
- Event Sourcing
- Histórico auditável

### Exemplo:

```java
public record TaskCompletedEvent(
    TaskId taskId,
    UserId completedBy,
    LocalDateTime completedAt
) implements DomainEvent {
    public TaskCompletedEvent {
        Objects.requireNonNull(taskId);
        Objects.requireNonNull(completedBy);
        Objects.requireNonNull(completedAt);
    }
}

// Aggregate publica evento
public class Task {
    private final List<DomainEvent> events = new ArrayList<>();
    
    public void complete(UserId user) {
        if (status == TaskStatus.DONE) return;
        
        this.status = TaskStatus.DONE;
        
        // Publica evento
        events.add(new TaskCompletedEvent(
            this.id,
            user,
            LocalDateTime.now()
        ));
    }
    
    public List<DomainEvent> getDomainEvents() {
        return Collections.unmodifiableList(events);
    }
    
    public void clearEvents() {
        events.clear();
    }
}
```

### Consumindo Eventos:

```java
@Component
public class TaskEventHandler {
    
    @EventListener
    public void on(TaskCompletedEvent event) {
        // Enviar notificação
        notificationService.notifyTaskCompleted(event.taskId());
        
        // Atualizar estatísticas
        analyticsService.recordCompletion(event.taskId());
        
        // Acumular pontos de gamificação
        gamificationService.awardPoints(event.completedBy());
    }
}
```

## 8.6. Repositories (Repositórios)

### Definição:
Um **Repository** é responsável por **persistir e recuperar agregados**. Ele encapsula a lógica de acesso a dados.

### Características:
- Opera em **Aggregate Roots**, não em entidades internas
- Interface no domínio, implementação na infraestrutura
- Métodos expressam intenção de negócio

### Exemplo:

```java
// Interface no domínio
public interface TaskRepository {
    Task findById(TaskId id);
    List<Task> findByOwner(UserId ownerId);
    List<Task> findOverdue();
    void save(Task task);
    void delete(TaskId id);
}

// Implementação na infraestrutura (JPA)
@Repository
class JpaTaskRepository implements TaskRepository {
    
    @PersistenceContext
    private EntityManager entityManager;
    
    @Override
    public Task findById(TaskId id) {
        TaskEntity entity = entityManager.find(TaskEntity.class, id.value());
        return entity != null ? entity.toDomain() : null;
    }
    
    @Override
    public List<Task> findOverdue() {
        String jpql = "SELECT t FROM TaskEntity t WHERE t.deadline < :now AND t.status != :done";
        return entityManager.createQuery(jpql, TaskEntity.class)
            .setParameter("now", LocalDate.now())
            .setParameter("done", TaskStatus.DONE.name())
            .getResultStream()
            .map(TaskEntity::toDomain)
            .collect(Collectors.toList());
    }
    
    @Override
    public void save(Task task) {
        TaskEntity entity = TaskEntity.fromDomain(task);
        if (entityManager.contains(entity)) {
            entityManager.merge(entity);
        } else {
            entityManager.persist(entity);
        }
        
        // Publicar eventos de domínio
        task.getDomainEvents().forEach(eventPublisher::publish);
        task.clearEvents();
    }
}
```

## 8.7. Domain Services (Serviços de Domínio)

### Definição:
Um **Domain Service** contém lógica de domínio que **não pertence naturalmente a nenhuma entidade ou Value Object**.

### Quando usar:
- Operação envolve múltiplos agregados
- Lógica não tem "dono" natural
- Cálculos complexos que não cabem em VOs

### Exemplo:

```java
@Service
public class TaskPrioritizationService {
    
    public Priority calculatePriority(Task task, List<Task> userTasks) {
        // Lógica complexa de negócio
        if (task.getDeadline().isPast()) {
            return Priority.URGENT;
        }
        
        long highPriorityCount = userTasks.stream()
            .filter(t -> t.getPriority() == Priority.HIGH)
            .count();
        
        if (highPriorityCount > 10) {
            return Priority.MEDIUM; // evita tudo ser HIGH
        }
        
        return task.getPriority();
    }
}
```

## 8.8. Arquitetura em Camadas com DDD

```
┌──────────────────────────────────────┐
│         Presentation Layer           │
│  (Controllers, REST APIs, UI)        │
└────────────────┬─────────────────────┘
                 │
┌────────────────▼─────────────────────┐
│        Application Layer             │
│  (Use Cases, Application Services)   │
└────────────────┬─────────────────────┘
                 │
┌────────────────▼─────────────────────┐
│          Domain Layer                │
│  (Entities, VOs, Aggregates,         │
│   Domain Services, Domain Events)    │
└────────────────┬─────────────────────┘
                 │
┌────────────────▼─────────────────────┐
│      Infrastructure Layer            │
│  (Repositories, DB, Messaging,       │
│   External APIs)                     │
└──────────────────────────────────────┘
```

### Exemplo de estrutura de pacotes:

```
com.example.tasks/
├── domain/
│   ├── model/
│   │   ├── Task.java               (Aggregate Root)
│   │   ├── TaskId.java             (Value Object)
│   │   ├── Title.java              (Value Object)
│   │   ├── Priority.java           (Enum/VO)
│   │   └── Deadline.java           (Value Object)
│   ├── events/
│   │   └── TaskCompletedEvent.java (Domain Event)
│   ├── services/
│   │   └── TaskPrioritization.java (Domain Service)
│   └── repository/
│       └── TaskRepository.java     (Interface)
│
├── application/
│   └── TaskApplicationService.java (Use Case orchestration)
│
└── infrastructure/
    ├── persistence/
    │   ├── TaskEntity.java         (JPA Entity)
    │   └── JpaTaskRepository.java  (Implementação)
    └── web/
        └── TaskController.java     (REST API)
```

## 8.9. Conclusão: Táticas para um Domínio Rico

Os padrões táticos do DDD nos permitem:

✅ **Expressar regras de negócio claramente** no código  
✅ **Proteger invariantes** com Aggregates  
✅ **Eliminar primitive obsession** com Value Objects  
✅ **Comunicar mudanças** com Domain Events  
✅ **Isolar persistência** com Repositories  
✅ **Criar modelo rico** (não anêmico)  

Na próxima seção, vamos ver quando usar (e quando **não** usar) DDD, com exemplos práticos e armadilhas comuns.

---

### ⏭️ Próxima Seção

Na próxima seção, vamos explorar **Aplicação Prática** do DDD.

**[→ Ir para Aplicação Prática](aula10-9-aplicacao-pratica)**

---

## Referências Desta Seção

**EVANS, Eric.** *Domain-Driven Design: Tackling Complexity in the Heart of Software.* Boston: Addison-Wesley, 2003.

**VERNON, Vaughn.** *Implementing Domain-Driven Design.* Boston: Addison-Wesley, 2013.

**FOWLER, Martin.** Anemic Domain Model. MartinFowler.com, 2003.
