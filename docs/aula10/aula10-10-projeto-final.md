---
layout: aula
title: "10. Projeto Final"
parent: Aula 10 - Domain-Driven Design
nav_order: 10
---

# **10. Projeto Final - Extração de Microsserviço**

Vamos detalhar o **Projeto Final da disciplina**: extração de um microsserviço aplicando DDD.

## Projeto Final da Disciplina

Na aula anterior, vocês exploraram e mapearam seus **Bounded Contexts** e identificaram potenciais candidatos para se tornarem microsserviços. Agora, o desafio é levar essa teoria para a prática! Vocês vão iniciar a jornada de **transformar um pedaço da sua aplicação em um microsserviço independente**, aplicando diretamente os conceitos estratégicos e táticos do DDD.

Sua equipe deverá escolher **um (1) Bounded Context** do projeto intermediário que desenvolveram para a disciplina. O objetivo é criar um **novo projeto Spring Boot** que represente esse microsserviço isolado.

---

## Etapas da Atividade

### 1. Revisão e Delimitação Final do Bounded Context Escolhido

- Revisem o **Bounded Context** que sua equipe selecionou para a extração. Verifiquem se ele faz sentido à luz do que foi apresentado nessa aula.
- Reforcem a **linguagem ubíqua** específica desse contexto. Quais são os termos, entidades e operações que só fazem sentido aqui?
- Documentem brevemente a fronteira: o que pertence a este microsserviço e o que não pertence? Quais são as **responsabilidades exclusivas** dele?

**Perguntas para responder**:
- Este contexto tem vocabulário próprio?
- As regras de negócio são específicas deste contexto?
- Este contexto pode evoluir independentemente dos outros?
- Existe (ou poderia existir) uma equipe dedicada a este contexto?

---

### 2. Criação do Novo Projeto do Microsserviço

- Criem um **novo projeto Spring Boot** separado do monólito original. Este será o repositório ou módulo do seu microsserviço.
- Dê a ele um nome que reflita seu **Bounded Context** (ex: `task-management-service`, `identity-access-service`).

**Estrutura recomendada**:
```
task-management-service/
├── src/
│   └── main/
│       └── java/
│           └── com/seuprojeto/task/
│               ├── domain/           # Modelos de domínio
│               │   ├── model/
│               │   ├── events/
│               │   ├── services/
│               │   └── repository/  # Interfaces
│               ├── application/      # Casos de uso
│               └── infrastructure/   # Implementações
│                   ├── persistence/
│                   ├── web/
│                   └── messaging/
├── pom.xml (ou build.gradle)
└── README.md
```

---

### 3. Migração do Modelo de Domínio e Lógica de Negócio (DDD Tático)

- **Migrem** ou **recriem** as classes de domínio relevantes (`Entidades`, `Value Objects`, `Aggregate Roots`) para dentro do novo projeto do microsserviço.
- Garanta que a **lógica de negócio** associada a essas classes esteja encapsulada dentro delas, seguindo os princípios do DDD tático (comportamento rico, invariantes nos agregados).
- Reorganize a estrutura de pacotes do novo microsserviço para refletir o **DDD**, utilizando pacotes como `domain/`, `application/`, `infrastructure/` dentro do seu contexto (ex: `com.seuprojeto.task.domain`, `com.seuprojeto.task.application`).

**Aplique os padrões táticos**:
- ✅ **Entities** com identidade e ciclo de vida
- ✅ **Value Objects** imutáveis e autovalidados
- ✅ **Aggregates** com invariantes protegidas
- ✅ **Domain Events** para comunicação
- ✅ **Repositories** com interfaces no domínio
- ✅ **Domain Services** quando lógica não pertence a entidade

---

### 4. Definição da API Externa (Contrato do Microsserviço)

O microsserviço precisará expor uma API para que outras partes do sistema (incluindo o monólito original) possam interagir com ele.

- **Definam e implementem endpoints REST** (Controllers) no novo microsserviço que permitam realizar as operações principais do seu **Bounded Context**.
- Lembrem-se: essa API deve ser **pública e bem definida**, expondo apenas o necessário e evitando vazar detalhes internos do modelo de domínio do microsserviço. Use **DTOs** (Data Transfer Objects) para entrada e saída de dados, jamais expondo diretamente as entidades de domínio internas.

**Exemplo de endpoints**:
```
POST   /api/tasks           - Criar tarefa
GET    /api/tasks/{id}      - Buscar tarefa
PUT    /api/tasks/{id}      - Atualizar tarefa
DELETE /api/tasks/{id}      - Deletar tarefa
POST   /api/tasks/{id}/complete - Completar tarefa
GET    /api/tasks/overdue   - Listar tarefas atrasadas
```

**DTOs recomendados**:
```java
// Request DTOs (entrada)
public record CreateTaskRequest(
    String title,
    String description,
    String priority,
    LocalDate deadline
) {}

// Response DTOs (saída)
public record TaskResponse(
    String id,
    String title,
    String description,
    String priority,
    String status,
    LocalDate deadline,
    String ownerId
) {}
```

---

### 5. Adaptação do Monólito Original

No monólito original, onde o contexto extraído costumava viver:

- **Removam** as classes e lógicas que foram migradas para o novo microsserviço.
- **Substituam** as chamadas internas a essa lógica por chamadas para a **nova API REST** do microsserviço.

**Exemplo de integração**:
```java
// Antes (monólito)
@Service
public class OrderService {
    @Autowired
    private TaskRepository taskRepository;
    
    public void createOrderProcessingTask(Order order) {
        Task task = new Task(...);
        taskRepository.save(task);
    }
}

// Depois (monólito chamando microsserviço)
@Service
public class OrderService {
    @Autowired
    private RestTemplate restTemplate;
    
    public void createOrderProcessingTask(Order order) {
        CreateTaskRequest request = new CreateTaskRequest(
            "Process order " + order.getId(),
            "...",
            "HIGH",
            LocalDate.now().plusDays(2)
        );
        
        restTemplate.postForObject(
            "http://task-service/api/tasks",
            request,
            TaskResponse.class
        );
    }
}
```

---

### 6. Desafio Extra (Opcional): Camada Anti-Corrupção

Se houver complexidade na integração ou na tradução de modelos entre o monólito e o novo microsserviço, considerem a criação de uma **Camada Anti-Corrupção (ACL)** no monólito.

Essa camada atuaria como um tradutor, adaptando o modelo do novo microsserviço para o modelo ainda existente no monólito, isolando o acoplamento.

**Exemplo de ACL**:
```java
@Component
public class TaskServiceAdapter {
    @Autowired
    private RestTemplate restTemplate;
    
    // Traduz do modelo do monólito para o modelo do microsserviço
    public ExternalTask createTask(InternalTaskData data) {
        CreateTaskRequest request = new CreateTaskRequest(
            data.getDescription(),
            data.getNotes(),
            mapPriority(data.getImportanceLevel()),
            data.getDueDate()
        );
        
        TaskResponse response = restTemplate.postForObject(
            "http://task-service/api/tasks",
            request,
            TaskResponse.class
        );
        
        // Traduz resposta de volta para modelo do monólito
        return new ExternalTask(
            response.id(),
            response.title(),
            TaskStatus.fromString(response.status())
        );
    }
    
    private String mapPriority(int level) {
        return switch(level) {
            case 1, 2 -> "LOW";
            case 3, 4 -> "MEDIUM";
            case 5 -> "HIGH";
            default -> "MEDIUM";
        };
    }
}
```

---

## Exemplo Completo: Contexto "Gerenciamento de Tarefas"

Suponha que vocês extraíram o **Bounded Context "Gerenciamento de Tarefas"** para um novo microsserviço `task-service`.

### No novo Microsserviço (`task-service`):

**Estrutura de pastas**:
```
src/main/java/com/edtech/task/
├── domain/
│   ├── model/
│   │   ├── Task.java              # Aggregate Root
│   │   ├── TaskId.java            # Value Object
│   │   ├── Title.java             # Value Object
│   │   ├── Priority.java          # Enum/VO
│   │   └── Deadline.java          # Value Object
│   ├── events/
│   │   └── TaskCompletedEvent.java
│   ├── services/
│   │   └── TaskPrioritization.java
│   └── repository/
│       └── TaskRepository.java    # Interface
├── application/
│   └── TaskApplicationService.java
└── infrastructure/
    ├── persistence/
    │   ├── TaskEntity.java
    │   └── JpaTaskRepository.java
    └── web/
        ├── TaskController.java
        └── dto/
            ├── CreateTaskRequest.java
            └── TaskResponse.java
```

**Classes como** `Task`, `Title`, `Priority`, `Deadline`, `TaskId` (Value Objects e Aggregate Root) estariam no pacote `domain`.

**Um** `TaskApplicationService` para coordenar operações no `application`.

**Um** `TaskController` no `infrastructure` (ou diretamente na raiz da API) com endpoints como `POST /tasks`, `PUT /tasks/{id}/complete`, `GET /tasks/{id}`.

**Os DTOs** para `POST /tasks` seriam `CreateTaskRequest` (com título, prioridade, etc.), e para `GET /tasks/{id}` seria `TaskResponse` (com status, data, etc.).

---

### No Monólito Original:

- Onde antes havia um `TaskService` monolítico, agora ele faria uma **chamada HTTP** para `http://task-service/api/tasks` para criar uma tarefa.
- Pode ser necessário um cliente REST (ex: `RestTemplate` ou `WebClient` do Spring) para fazer essas chamadas.

---

## Formato da Entrega

Cada equipe deve entregar:

### 1. Documentação da Extração (2-3 páginas)

**Bounded Context Escolhido**:
- Qual foi o Bounded Context selecionado para extração?
- Qual a justificativa? (baseada em taxa de mudança, escalabilidade, criticidade)

**Context Map Simples**:
- Um diagrama (mesmo que simples, em texto ou PlantUML) mostrando o novo microsserviço e como ele se relaciona com o monólito original.
- Indiquem o **tipo de relacionamento** (ex: Customer/Supplier, ACL, etc.)

**Linguagem Ubíqua e Modelagem Tática**:
- Descrevam os principais **elementos táticos (Entidades, VOs, Agregados)** que foram migrados
- Como eles aplicam a **linguagem ubíqua** desse contexto?

**Contrato do Microsserviço**:
- Listem os principais **endpoints REST** que o novo microsserviço expõe
- Quais são os **DTOs** de entrada/saída? (nome e principais campos)

**Adaptação do Monólito**:
- Expliquem como o monólito original foi adaptado para se comunicar com o novo microsserviço
- Usaram chamadas REST diretas? WebClient? ACL?

---

### 2. Repositórios de Código

- O **novo repositório (ou pasta/módulo)** do microsserviço Spring Boot extraído.
- O **repositório original do monólito**, mostrando as modificações para se integrar ao novo serviço.

**Importante**:
- Código deve compilar e rodar
- README com instruções de como executar
- Commits devem ser claros e organizados

---

### 3. Demonstração (Obrigatória)

- Preparem uma breve demonstração da sua aplicação, mostrando o monólito chamando (ou interagindo de alguma forma) com o novo microsserviço.
- Estejam prontos para explicar o código e as decisões de design tomadas.

**Sugestões para demo**:
- Mostrar requisição HTTP sendo enviada
- Mostrar resposta sendo recebida
- Mostrar logs de ambos os lados
- Demonstrar comportamento rico do domínio

---

## Critérios de Avaliação

O projeto será avaliado considerando:

### Estratégia (40%)
- [ ] Bounded Context bem justificado
- [ ] Linguagem ubíqua identificada e documentada
- [ ] Context Map mostrando relacionamentos
- [ ] Decisões estratégicas fundamentadas

### Tática (40%)
- [ ] Uso correto de Entities, VOs e Aggregates
- [ ] Comportamento rico (não modelo anêmico)
- [ ] Invariantes protegidas
- [ ] Repositórios e Domain Events (se aplicável)
- [ ] Separação domain/application/infrastructure

### Implementação (20%)
- [ ] Código compila e executa
- [ ] API REST bem definida com DTOs
- [ ] Integração entre monólito e microsserviço funciona
- [ ] Documentação clara
- [ ] README com instruções

---

## Dicas Finais

✅ **Comece simples**: não tente extrair todo o sistema de uma vez  
✅ **Documente decisões**: por que escolheram este contexto? por que este padrão?  
✅ **Teste a integração**: garanta que monólito e microsserviço conversam  
✅ **Revise o DDD**: releia as seções sobre padrões táticos  
✅ **Peça feedback**: mostre para o professor antes da entrega final  

---

## Entrega

**Local**: Moodle  
**Formato**: ZIP contendo:
- Documentação (PDF ou Markdown)
- Código-fonte (ambos repositórios)
- README com instruções

---

# **Bom trabalho!🛠️**

Este projeto consolida **tudo que você aprendeu** sobre DDD, microsserviços e arquitetura de software. É a oportunidade de aplicar teoria em prática, cometer erros em ambiente controlado e aprender profundamente.

**Boa sorte!** 🚀

---

## Referências Finais

**EVANS, Eric.** *Domain-Driven Design: Tackling Complexity in the Heart of Software.* Boston: Addison-Wesley, 2003.

**VERNON, Vaughn.** *Implementing Domain-Driven Design.* Boston: Addison-Wesley, 2013.

**VERNON, Vaughn.** *Domain-Driven Design Distilled.* Boston: Addison-Wesley, 2016.

**KHONONOV, Vlad.** *Learning Domain-Driven Design: Aligning Software Architecture and Business Strategy.* Sebastopol: O'Reilly Media, 2021.

---

**[⬅️ Voltar ao Índice Principal](Aula10)**
