---
layout: aula
title: 5. Implementando Services e Controllers
parent: Aula 07 - Construindo o To-do List
nav_order: 5
---


### 3.4. Serviços (`service`)

A seguir, vamos explorar a **camada de serviço**, que é responsável por coordenar os casos de uso da aplicação, centralizar regras de negócio e garantir a integridade das operações. Nesta camada implementamos toda a lógica que regula as operações do sistema, sempre respeitando o princípio de separação de responsabilidades que vimos anteriormente.

#### `TaskService.java`

```java
package br.ifsp.edu.todo.service;

@Service
public class TaskService {
    private final TaskRepository taskRepository;
    private final ModelMapper modelMapper;
    private final PagedResponseMapper pagedResponseMapper;
    
    public TaskService(TaskRepository taskRepository, ModelMapper modelMapper, PagedResponseMapper pagedResponseMapper) {
        this.taskRepository = taskRepository;
        this.modelMapper = modelMapper;
        this.pagedResponseMapper = pagedResponseMapper;
    }
    
    public TaskResponseDTO createTask(TaskRequestDTO taskDto) {
        if (taskDto.getDueDate().isBefore(LocalDateTime.now()))
            throw new ValidationException("Due date cannot be in the past.");
        
        Task task = modelMapper.map(taskDto, Task.class);
        task.setCreatedAt(LocalDateTime.now());
        task.setCompleted(false);
        Task createdTask = taskRepository.save(task);
        return modelMapper.map(createdTask, TaskResponseDTO.class);
    }
    
    public PagedResponse<TaskResponseDTO> getAllTasks(Pageable pageable) {
        Page<Task> tasksPage = taskRepository.findAll(pageable);
        return pagedResponseMapper.toPagedResponse(tasksPage, TaskResponseDTO.class);
    }
    
    public TaskResponseDTO getTaskById(Long id) {
        Task task = taskRepository.findById(id).orElseThrow(() -> new ResourceNotFoundException("Task not found"));
        return modelMapper.map(task, TaskResponseDTO.class);
    }
    
    public PagedResponse<TaskResponseDTO> searchByCategory(String category, Pageable pageable) {
        Category categoryEnum = Category.fromString(category);
        Page<Task> tasks = taskRepository.findByCategory(categoryEnum, pageable);
        return pagedResponseMapper.toPagedResponse(tasks, TaskResponseDTO.class);
    }
    
    public TaskResponseDTO concludeTask(Long id) {
        Task task = taskRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Task not found with ID: " + id));
        
        if (task.isCompleted()) {
            throw new InvalidTaskStateException("Task is already completed.");
        }
        
        task.setCompleted(true);
        Task updatedTask = taskRepository.save(task);
        return modelMapper.map(updatedTask, TaskResponseDTO.class);
    }
    
    public TaskResponseDTO updateTask(Long id, TaskRequestDTO taskDto) {
        if (taskDto.getDueDate().isBefore(LocalDateTime.now()))
            throw new ValidationException("Due date cannot be in the past");
        
        Task existingTask = taskRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Task not found with ID: " + id));
        
        if (existingTask.isCompleted())
            throw new InvalidTaskStateException("Completed tasks cannot be updated");
        
        modelMapper.map(taskDto, existingTask);
        existingTask.setId(id);
        existingTask.setCreatedAt(existingTask.getCreatedAt()); // preserva a data original de criação!
        Task updatedTask = taskRepository.save(existingTask);
        return modelMapper.map(updatedTask, TaskResponseDTO.class);
    }
    
    public void deleteTask(Long id) {
        Task task = taskRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Task not found with ID: " + id));
        
        if (task.isCompleted()) {
            throw new InvalidTaskStateException("Cannot delete a completed task");
        }
        
        taskRepository.delete(task);
    }
}
```

A classe `TaskService` é o **centro da nossa lógica de negócio** no projeto de gerenciamento de tarefas. Ela é a responsável por coordenar as operações sobre as entidades do sistema, garantindo que as regras sejam aplicadas de maneira consistente e que o controller não fique sobrecarregado com decisões que não lhe competem.

A classe é anotada com `@Service`, o que faz com que o Spring reconheça automaticamente essa classe como um *componente de serviço*, permitindo sua injeção em outros pontos do sistema (como nos controllers) de forma automática. Essa anotação é importante porque ajuda a manter a **organização por responsabilidades** dentro do projeto.

As dependências (`TaskRepository`, `ModelMapper`, `PagedResponseMapper`) são injetadas via **construtor**. Essa abordagem de injeção por construtor, ao invés do uso de `@Autowired` nos atributos, traz vários benefícios:
- Permite que os campos sejam `final`, garantindo **imutabilidade** — ou seja, que o serviço não troque de dependência depois de criado.
- Facilita a **testabilidade** — podemos facilmente criar instâncias da `TaskService` passando mocks dessas dependências em testes unitários.
- Deixa a classe mais **autoexplicativa**, pois o construtor mostra claramente quais são suas necessidades para funcionar.

Vamos explicar os métodos da `TaskService`:

- **`createTask(TaskRequestDTO dto)`**  
  - Primeiro, valida se a `dueDate` da tarefa (data limite) é futura. Se for uma data no passado, lançamos uma `ValidationException`.  
  - Depois, o `ModelMapper` é usado para converter o DTO recebido (`TaskRequestDTO`) para uma entidade real (`Task`).  
  - A tarefa recém-criada é inicializada com a data atual (`createdAt = LocalDateTime.now()`) e marcada como não concluída (`completed = false`).  
  - Por fim, a tarefa é salva no banco de dados através do `taskRepository`, e retornamos a resposta no formato `TaskResponseDTO` — ou seja, padronizamos sempre a saída para o cliente.

- **`getAllTasks(Pageable pageable)`**  
  - Faz a consulta paginada no banco através do `taskRepository.findAll(pageable)`.
  - Usa o `PagedResponseMapper` para transformar o `Page<Task>` em um `PagedResponse<TaskResponseDTO>`, que é um DTO nosso mais amigável e controlado (evitando expor detalhes internos do Spring como o objeto `Page`).  
  - Essa separação entre entidades internas e o que expomos para fora é fundamental para garantir que **mudanças internas** (por exemplo, trocarmos a biblioteca de paginação) não quebrem contratos da API.

- **`getTaskById(Long id)`**  
  - Busca uma tarefa pelo ID.
  - Se a tarefa não existir, lança uma `ResourceNotFoundException` com uma mensagem específica.
  - Retorna o DTO da tarefa encontrada.
  
  Esse padrão (`findById().orElseThrow()`) é uma forma moderna e segura de trabalhar com valores opcionais no Java usando `Optional`, evitando `null` e melhorando a legibilidade.

- **`searchByCategory(String category, Pageable pageable)`**  
  - Converte o nome da categoria (`String`) para o `enum Category` usando o método `Category.fromString(category)`.
  - Consulta no repositório todas as tarefas com a categoria correspondente.
  - Usa novamente o `PagedResponseMapper` para devolver o resultado no formato consistente.

- **`concludeTask(Long id)`**  
  - Busca a tarefa pelo ID.
  - Verifica se a tarefa já está marcada como concluída. Se sim, lança uma `InvalidTaskStateException` (evitando que a mesma tarefa seja "reconcluída" várias vezes).  
  - Se não estiver concluída, marca como concluída (`completed = true`) e salva a atualização.

- **`updateTask(Long id, TaskRequestDTO dto)`**  
  - Busca a tarefa existente.
  - Verifica se ela já foi concluída. Se estiver concluída, não é possível alterar — então lançamos uma `InvalidTaskStateException`.
  - Valida se a nova `dueDate` fornecida é futura (não aceitamos alterações que "voltem no tempo").
  - Usa o `ModelMapper` para copiar os dados do DTO para a entidade existente.
  - **Importante**: preservamos campos que não devem ser alterados manualmente, como `id`, `completed` e `createdAt`. Isso é feito explicitamente logo após o mapeamento para garantir que o cliente não consiga sobrescrever essas informações.  
    ```java
    updatedTask.setId(existingTask.getId());
    updatedTask.setCompleted(existingTask.isCompleted());
    updatedTask.setCreatedAt(existingTask.getCreatedAt());
    ```
  - Salva a tarefa atualizada e retorna o DTO.

- **`deleteTask(Long id)`**  
  - Busca a tarefa pelo ID.
  - Verifica se ela foi concluída. Se já estiver concluída, não permitimos a exclusão — para evitar perda de histórico de tarefas finalizadas (uma regra de negócio típica em muitos sistemas de tarefas).  
  - Se não estiver concluída, apagamos do banco usando `deleteById(id)`.

⚠️⚠️ Destaques Importantes:

- A `TaskService` **coordena todos os casos de uso** da aplicação, sem misturar responsabilidade de interação com a web ou com o banco — essas tarefas ficam para o controller e o repository, respectivamente.
- **Regra de negócio** (ex: não alterar tarefas concluídas) é aplicada de forma centralizada, consistente e previsível.
- Uso cuidadoso de **exceções específicas** torna os erros mais compreensíveis e o tratamento no controller mais fácil.
- **Mapeamento com ModelMapper** facilita a conversão entre entidades e DTOs, mas com cuidados manuais em campos sensíveis.
- Separação entre resposta para cliente (`PagedResponse`, `TaskResponseDTO`) e estruturas internas do Spring (`Page`) aumenta a robustez e liberdade evolutiva da API.

Assim, o `TaskService` cumpre seu papel essencial: proteger o domínio da aplicação e fornecer um ponto de entrada claro, seguro e reutilizável para todas as operações relacionadas às tarefas.

Encerrando esta seção, fica claro que a camada de serviço não apenas organiza o fluxo das operações como também **evita duplicação de lógica**, **facilita a manutenção** e **torna a aplicação mais testável**. Ela é o verdadeiro cérebro da aplicação, conectando modelos, DTOs e repositórios em fluxos consistentes de negócio. Com a lógica de negócio centralizada e bem organizada na camada de serviços, resta apenas um elo para completarmos nosso fluxo de aplicação: a exposição dessa lógica para o mundo exterior. Vamos então conhecer a camada de controllers, responsável por receber as requisições dos usuários e orquestrar as operações do sistema.

### 3.5. Controladores (`controller`)

Vamos analisar os **controllers**, que são responsáveis por receber as requisições HTTP, validar entradas e delegar as operações para os serviços. Os controllers atuam como **portões de entrada** da aplicação, traduzindo o mundo externo (requisições REST) para chamadas internas de negócio.

#### `TaskController.java`

```java
package br.ifsp.edu.todo.controller;

@RestController
@RequestMapping("/api/tasks")
public class TaskController {
    private final TaskService taskService;
    
    public TaskController(TaskService taskService) {
        this.taskService = taskService;
    }
    
    @PostMapping
    public ResponseEntity<TaskResponseDTO> createTask(@Valid @RequestBody TaskRequestDTO task) {
        TaskResponseDTO taskResponseDTO = taskService.createTask(task);
        return ResponseEntity.status(HttpStatus.CREATED).body(taskResponseDTO);
    }
    
    @GetMapping
    public ResponseEntity<PagedResponse<TaskResponseDTO>> getAllTasks(Pageable pageable) {
        return ResponseEntity.ok(taskService.getAllTasks(pageable));
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<TaskResponseDTO> getTaskById(@PathVariable Long id) {
        return ResponseEntity.ok(taskService.getTaskById(id));
    }
    
    @GetMapping("/search")
    public ResponseEntity<PagedResponse<TaskResponseDTO>> searchByCategory(@RequestParam String category, Pageable pageable) {
        return ResponseEntity.ok(taskService.searchByCategory(category, pageable));
    }
    
    @PatchMapping("/{id}/finish")
    public ResponseEntity<TaskResponseDTO> concludeTask(@PathVariable Long id) {
        TaskResponseDTO response = taskService.concludeTask(id);
        return ResponseEntity.ok(response);
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<TaskResponseDTO> updateTask(@PathVariable Long id,
            @Valid @RequestBody TaskRequestDTO taskDto) {
        TaskResponseDTO updatedTask = taskService.updateTask(id, taskDto);
        return ResponseEntity.ok(updatedTask);
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteTask(@PathVariable Long id) {
        taskService.deleteTask(id);
        return ResponseEntity.noContent().build();
    }
}
```

A classe `TaskController` é o ponto de entrada da nossa API para manipulação de tarefas. Ela define os **endpoints REST** que os clientes podem utilizar para interagir com o sistema, como criar, buscar, atualizar, concluir e excluir tarefas.

Logo de início, vemos que a classe é anotada com `@RestController` e `@RequestMapping("/api/tasks")`, o que significa que:

- Ela será responsável por tratar requisições HTTP.
- Todos os endpoints definidos aqui terão como prefixo `/api/tasks` (por exemplo: `/api/tasks/1`, `/api/tasks/search`, etc).

Internamente, o `TaskController` injeta a dependência do `TaskService` via construtor. Como vimos anteriormente, isso é uma boa prática, pois favorece a imutabilidade dos atributos (`final`) e facilita a testabilidade da classe.

Agora vamos entender cada um dos métodos:

##### Endpoint de criação de tarefas

```java
@PostMapping
public ResponseEntity<TaskResponseDTO> createTask(@Valid @RequestBody TaskRequestDTO task) {
    TaskResponseDTO taskResponseDTO = taskService.createTask(task);
    return ResponseEntity.status(HttpStatus.CREATED).body(taskResponseDTO);
}
```

- **`@PostMapping`** indica que este método responde a requisições HTTP POST.
- Ele recebe um `TaskRequestDTO` no corpo da requisição (`@RequestBody`) e valida os dados automaticamente com `@Valid`.
- A chamada é repassada para o `taskService.createTask()`, onde a lógica de criação acontece.
- A resposta é encapsulada em um `ResponseEntity` com o status `201 CREATED`, indicando que um novo recurso foi criado com sucesso.

##### Endpoint de listagem paginada de tarefas

```java
@GetMapping
public ResponseEntity<PagedResponse<TaskResponseDTO>> getAllTasks(Pageable pageable) {
    return ResponseEntity.ok(taskService.getAllTasks(pageable));
}
```

- **`@GetMapping`** mapeia este método para requisições HTTP GET em `/api/tasks`.
- Utiliza o `Pageable`, que é injetado automaticamente pelo Spring para suportar paginação e ordenação.
- Retorna uma resposta padronizada usando nosso `PagedResponse`, encapsulada com `ResponseEntity.ok()` para indicar sucesso.

##### Endpoint de busca por ID

```java
@GetMapping("/{id}")
public ResponseEntity<TaskResponseDTO> getTaskById(@PathVariable Long id) {
    return ResponseEntity.ok(taskService.getTaskById(id));
}
```

- Busca uma tarefa específica pelo seu identificador (`id`) passado na URL.
- Se o ID for encontrado, retorna o DTO da tarefa com `200 OK`.
- Se não, o `TaskService` irá lançar uma exceção que será tratada pelo `GlobalExceptionHandler`.

##### Endpoint de busca por categoria

```java
@GetMapping("/search")
public ResponseEntity<PagedResponse<TaskResponseDTO>> searchByCategory(@RequestParam String category, Pageable pageable) {
    return ResponseEntity.ok(taskService.searchByCategory(category, pageable));
}
```

- Permite buscar tarefas que pertencem a uma determinada categoria.
- O parâmetro `category` é passado pela **query string** (ex: `/api/tasks/search?category=STUDY`).
- Também suporta paginação (`Pageable`).

##### Endpoint para concluir tarefa

```java
@PatchMapping("/{id}/finish")
public ResponseEntity<TaskResponseDTO> concludeTask(@PathVariable Long id) {
    TaskResponseDTO response = taskService.concludeTask(id);
    return ResponseEntity.ok(response);
}
```

- Este método permite marcar uma tarefa como concluída.
- Utiliza `@PatchMapping`, que é o verbo apropriado para **atualizações parciais** (alterar apenas o status da tarefa).
- O status de resposta será `200 OK` com os dados da tarefa atualizada.

##### Endpoint para atualização completa de tarefa

```java
@PutMapping("/{id}")
public ResponseEntity<TaskResponseDTO> updateTask(@PathVariable Long id,
        @Valid @RequestBody TaskRequestDTO taskDto) {
    TaskResponseDTO updatedTask = taskService.updateTask(id, taskDto);
    return ResponseEntity.ok(updatedTask);
}
```

- Permite **atualizar todos os dados** de uma tarefa existente (não apenas um campo específico).
- Utiliza `@PutMapping`, que, de acordo com a semântica REST, representa **substituição integral** do recurso.
- O corpo da requisição também é validado automaticamente.

##### Endpoint para excluir tarefa

```java
@DeleteMapping("/{id}")
public ResponseEntity<Void> deleteTask(@PathVariable Long id) {
    taskService.deleteTask(id);
    return ResponseEntity.noContent().build();
}
```

- Exclui uma tarefa com base no ID informado.
- Utiliza `@DeleteMapping`, que mapeia para operações de deleção.
- Retorna um `204 No Content`, indicando que a operação foi bem-sucedida mas que **não há corpo na resposta**.

##### 🌟 Considerações Gerais

Perceba que nessa implementação seguimos a ideia mencionada ao dissertarmos sobre a importância da camada service:

- **Responsabilidade clara:** O controller não possui lógica de negócio, apenas orquestra chamadas ao `TaskService`.
- **Validação:** Usa `@Valid` nos métodos que recebem dados para garantir que as informações estejam corretas antes de tentar persistir no banco.
- **Uso correto de status HTTP:** Cada operação retorna um status condizente com o seu objetivo (201, 200, 204).
- **Separa controle de fluxo da lógica de negócio:** Delega tudo que é mais complexo para a camada de serviço.
- **Padronização:** Utiliza `ResponseEntity` em todos os métodos, garantindo que o formato das respostas seja consistente.

Concluindo a análise dos controllers, percebemos que eles permanecerem **leves e orquestradores**, lidando apenas com aspectos de roteamento, validação inicial e formatação de resposta. Toda a complexidade do sistema já foi devidamente isolada nas camadas anteriores — exatamente como propõe uma boa arquitetura em camadas. 🚀

Ainda falta, entretanto, explorarmos o tratamento de erros, as configurações de mapeamento entre Entidades e os Testes — aspectos transversais que permeiam toda a aplicação e que são fundamentais para garantir robustez e qualidade ao nosso sistema.
