---
layout: aula
title: 7. Tratamento de Exceções e Testes
parent: Aula 07 - Construindo o To-do List
nav_order: 7
---

### 3.6.2. Tratamento Global de Exceções: GlobalExceptionHandler

Após entender como os dados são mapeados dentro da aplicação, precisamos nos preocupar com **o que acontece quando algo sai do esperado**.  

Nem sempre uma requisição será válida, nem todo recurso solicitado existirá, e nem todas as operações serão permitidas — e nossa API precisa reagir a essas situações **de forma padronizada e amigável**. É o que já fizemos anteriormente, mas vamos repetir aqui para fins didáticos.

#### `GlobalExceptionHandler.java`

```java
package br.ifsp.edu.todo.exception;

@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFoundException(ResourceNotFoundException ex) {
        ErrorResponse errorResponse = new ErrorResponse(HttpStatus.NOT_FOUND.value(), ex.getMessage());
        return new ResponseEntity<>(errorResponse, HttpStatus.NOT_FOUND);
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleMethodArgumentNotValid(MethodArgumentNotValidException ex) {
        String errorMessage = ex.getBindingResult().getFieldErrors().stream()
                .map(err -> err.getField() + ": " + err.getDefaultMessage()).collect(Collectors.joining(", "));
        
        ErrorResponse errorResponse = new ErrorResponse(HttpStatus.BAD_REQUEST.value(), errorMessage);
        return new ResponseEntity<>(errorResponse, HttpStatus.BAD_REQUEST);
    }

    @ExceptionHandler(ValidationException.class)
    public ResponseEntity<ErrorResponse> handleValidationException(ValidationException ex) {
        ErrorResponse errorResponse = new ErrorResponse(HttpStatus.BAD_REQUEST.value(), ex.getMessage());
        return new ResponseEntity<>(errorResponse, HttpStatus.BAD_REQUEST);
    }
    
    @ExceptionHandler(InvalidTaskStateException.class)
    public ResponseEntity<ErrorResponse> handleInvalidTaskStateException(InvalidTaskStateException ex) {
        ErrorResponse errorResponse = new ErrorResponse(HttpStatus.CONFLICT.value(), ex.getMessage());
        return new ResponseEntity<>(errorResponse, HttpStatus.CONFLICT);
    }
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGenericException(Exception ex) {
        ErrorResponse errorResponse = new ErrorResponse(HttpStatus.INTERNAL_SERVER_ERROR.value(),
                "An unexpected error occurred");
        return new ResponseEntity<>(errorResponse, HttpStatus.INTERNAL_SERVER_ERROR);
    }

    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<ErrorResponse> handleIllegalArgumentException(IllegalArgumentException ex) {
        ErrorResponse errorResponse = new ErrorResponse(HttpStatus.BAD_REQUEST.value(), ex.getMessage());
        return new ResponseEntity<>(errorResponse, HttpStatus.BAD_REQUEST);
    }
}
```

A `GlobalExceptionHandler` é a classe que efetivamente realiza o tratamento centralizado das exceções em nossa aplicação.
Ela é anotada com `@RestControllerAdvice`, uma especialização do `@ControllerAdvice` combinada com `@ResponseBody`, que intercepta exceções lançadas em toda a aplicação e gera respostas HTTP amigáveis e padronizadas.

Essa classe define métodos como:

Tratamento de `ResourceNotFoundException` → retorna HTTP 404.

Tratamento de `InvalidTaskStateException` → retorna HTTP 409.

Tratamento genérico de outras exceções → retorna HTTP 500.

Cada método constrói um ErrorResponse, define o status correto e retorna uma ResponseEntity<ErrorResponse>. Isso nos permite separar totalmente a lógica de negócio da lógica de tratamento de erro, promovendo a limpeza e a organização da aplicação.

Além disso, ter uma camada de tratamento global facilita a adição de tratamentos personalizados no futuro, como logs de exceções, métricas de falhas ou alertas de erro.

Com isso, garantimos que os clientes da nossa API recebam:

- Códigos de status HTTP adequados (`400`, `404`, `409`, `500`, etc.);
- Mensagens de erro claras e compreensíveis;
- Estruturas de resposta consistentes.

Até aqui, nada de novo!

#### `ResourceNotFoundException.java`

```java
package br.ifsp.edu.todo.exception;

public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }

    public ResourceNotFoundException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

Essa classe representa uma exceção específica para o cenário em que um recurso solicitado (por exemplo, uma tarefa) não é encontrado no sistema. 

Ela estende `RuntimeException`, o que significa que é uma exceção não verificada (unchecked), e por isso não exige tratamento obrigatório no momento de sua propagação.

Sua implementação é bastante simples e elegante: possui apenas um construtor que recebe a mensagem de erro. Essa mensagem será utilizada mais adiante pela camada de tratamento global (`GlobalExceptionHandler`) para gerar a resposta HTTP apropriada.

Essa exceção melhora a legibilidade e a semântica da aplicação, pois ao lançarmos explicitamente uma `ResourceNotFoundException`, deixamos claro qual é o problema ocorrido, em vez de depender de mensagens genéricas.

#### `InvalidTaskStateException.java`

```java
package br.ifsp.edu.todo.exception;

public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }

    public ResourceNotFoundException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

De maneira semelhante, a classe `InvalidTaskStateException` representa situações em que a operação solicitada não é permitida dado o estado atual da tarefa — por exemplo, tentar editar ou excluir uma tarefa que já foi concluída.

Ela também estende `RuntimeException` e possui dois construtores:

- Um que recebe apenas a mensagem de erro.

- Outro que recebe a mensagem de erro e a causa (outra exceção que possa ter originado o erro), permitindo o encadeamento de exceções se necessário.

A criação de exceções específicas como esta é fundamental para a clareza do código: elas tornam o fluxo de erro mais explícito e a manutenção mais fácil, além de favorecer o tratamento diferenciado de casos de erro específicos na camada de exceções globais.

#### `ErrorResponse.java`

```java
package br.ifsp.edu.todo.exception;

@Data
@AllArgsConstructor
public class ErrorResponse {
    private int status;
    private String message;
    private LocalDateTime timestamp;

    public ErrorResponse(int status, String message) {
        this.status = status;
        this.message = message;
        this.timestamp = LocalDateTime.now();
    }
}
```

A classe ErrorResponse é um DTO de erro, criado para padronizar o corpo das respostas de erro que nossa API envia para os clientes. Ela possui três atributos:

- status: o código de status HTTP associado ao erro (ex: 404, 409, 400).

- message: a mensagem de erro detalhada.

- timestamp: o momento exato em que o erro ocorreu.

O uso desse DTO traz vários benefícios:

- Consistência: todas as respostas de erro seguem o mesmo formato.

- Facilidade de análise: tanto humanos quanto sistemas automatizados (como front-ends) podem interpretar e exibir as mensagens de erro de maneira uniforme.

- Rastreamento: o timestamp facilita a investigação de problemas e a correlação de eventos em logs.

Além disso, a classe utiliza anotações Lombok (`@Data` e `@AllArgsConstructor`) para reduzir o boilerplate, mas também oferece um construtor personalizado que define o timestamp automaticamente como `LocalDateTime.now()`, caso ele não seja informado.

Repare que essa classe, apesar de ser um DTO, está no pacote `exception`. Ao posicioná-la no pacote `exception`, estamos reforçando uma decisão **semântica**: o ErrorResponse não é um DTO comum usado em fluxos de sucesso, mas sim um DTO especializado no tratamento de erros.

No nosso projeto de To-Do List, dado que o escopo é controlado e estamos prezando por clareza semântica acima da ortodoxia estrutural, deixar ErrorResponse no pacote exception é uma escolha válida e até recomendada. Mas, para projetos muito grandes e de múltiplas equipes, seria prudente repensar e possivelmente consolidar todos os DTOs no mesmo pacote!

Entendida a camada de tratamento de erros, passemos, finalmente, aos testes!

## 3.6.3. Testes Automatizados

Com o mapeamento de objetos bem resolvido e o tratamento de erros devidamente padronizado, estamos prontos para consolidar a robustez da aplicação: através dos **testes automatizados**.

Testes são a nossa principal ferramenta para **garantir que o sistema se comporte como o esperado** hoje — e continue se comportando assim no futuro, mesmo diante de evoluções ou refatorações.  

No projeto, estruturamos nossos testes de forma a cobrir tanto aspectos **unitários** (camada a camada) quanto aspectos **funcionais** (comportamento da API como um todo).

#### `TaskServiceTest.java`

```java
package br.ifsp.edu.todo.task;

@ExtendWith(MockitoExtension.class)
public class TaskServiceTest {
    
    @Mock
    private TaskRepository taskRepository;

    @Mock
    private ModelMapper modelMapper;

    @Mock
    private PagedResponseMapper pagedResponseMapper;

    @InjectMocks
    private TaskService taskService;
    
    @Test
    void shouldCreateTaskWithValidData() {
        TaskRequestDTO dto = new TaskRequestDTO();
        dto.setTitle("Valid Task");
        dto.setPriority(Priority.MEDIUM);
        dto.setDueDate(LocalDateTime.now().plusDays(2));
        dto.setCategory(Category.WORK);
        
        Task taskEntity = new Task();
        Task savedTask = new Task();
        savedTask.setId(1L);
        savedTask.setTitle("Valid Task");
        
        when(modelMapper.map(dto, Task.class)).thenReturn(taskEntity);
        when(taskRepository.save(any())).thenReturn(savedTask);
        when(modelMapper.map(savedTask, TaskResponseDTO.class)).thenReturn(new TaskResponseDTO());
        
        TaskResponseDTO response = taskService.createTask(dto);
        assertNotNull(response);
    }
    
    @Test
    void shouldThrowValidationExceptionWhenDueDateIsPast() {
        TaskRequestDTO dto = new TaskRequestDTO();
        dto.setTitle("Invalid Task");
        dto.setDueDate(LocalDateTime.now().minusDays(1));
        
        assertThrows(ValidationException.class, () -> taskService.createTask(dto));
    }
    
    @Test
    void shouldFetchTaskById() {
        Task task = new Task();
        task.setId(1L);
        
        when(taskRepository.findById(1L)).thenReturn(Optional.of(task));
        when(modelMapper.map(any(), eq(TaskResponseDTO.class))).thenReturn(new TaskResponseDTO());
        
        TaskResponseDTO response = taskService.getTaskById(1L);
        assertNotNull(response);
    }
    
    @Test
    void shouldThrowErrorWhenDeletingCompletedTask() {
        Task completedTask = new Task();
        completedTask.setId(1L);
        completedTask.setCompleted(true);
        
        when(taskRepository.findById(1L)).thenReturn(Optional.of(completedTask));
        
        assertThrows(InvalidTaskStateException.class, () -> taskService.deleteTask(1L));
    }
    
}
```

Esses testes são **testes unitários**, focados **exclusivamente na lógica da camada de serviço**, usando mocks para isolar o comportamento do repositório (`TaskRepository`), do `ModelMapper`, e do `PagedResponseMapper`. Isso garante que estamos testando apenas a lógica da classe `TaskService`, sem dependências externas.

- **Mock e Injeção**
  - Usamos `@Mock` para criar versões simuladas das dependências.
  - Usamos `@InjectMocks` para injetar essas dependências no `TaskService` real que estamos testando.

Métodos testados:

- **shouldCreateTaskWithValidData()**
  - Testa a criação de uma tarefa válida.
  - Verifica se conseguimos criar uma tarefa quando todos os dados são corretos.
  - Usa `when(...).thenReturn(...)` para simular o comportamento do repositório e do mapeador.

- **shouldThrowValidationExceptionWhenDueDateIsPast()**
  - Testa o cenário em que a data limite (`dueDate`) é passada.
  - Espera que uma `ValidationException` seja lançada.
  
- **shouldFetchTaskById()**
  - Testa a busca de uma tarefa pelo seu ID.
  - Garante que o método de busca retorna corretamente a resposta mapeada.

- **shouldThrowErrorWhenDeletingCompletedTask()**
  - Testa a tentativa de deletar uma tarefa já concluída.
  - Espera que seja lançada uma `InvalidTaskStateException`.


É importante que façamos algumas considerações: todos os testes isolam a lógica do `TaskService`, e testamos tanto fluxos positivos quanto negativos (ex: validação de data e deleção de tarefas concluídas).

Esse teste não fornece cobertura completa de nossa aplicação, mas consegue já cumprir o proposto no exercício e fazer uso dos conceitos que vimos anteriormente sobre testes.

Passemos, agora, aos testes funcionais.

#### `TaskControllerTest.java`

```java
package br.ifsp.edu.todo.task;

@SpringBootTest
@AutoConfigureMockMvc
public class TaskControllerTest {
    @Autowired
    private MockMvc mockMvc;
    
    @Autowired
    private ObjectMapper objectMapper;
    
    @Autowired
    private TaskRepository taskRepository;
    
    @BeforeEach
    void cleanDb() {
        taskRepository.deleteAll();
    }
    
    @Test
    void shouldCreateTask() throws Exception {
        TaskRequestDTO dto = new TaskRequestDTO();
        dto.setTitle("My Task");
        dto.setPriority(Priority.HIGH);
        dto.setDueDate(LocalDateTime.now().plusDays(1));
        dto.setCategory(Category.STUDY);
        
        mockMvc.perform(post("/api/tasks").contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(dto))).andExpect(status().isCreated())
                .andExpect(jsonPath("$.title").value("My Task"));
    }
    
    @Test
    void shouldFailWithPastDueDate() throws Exception {
        TaskRequestDTO dto = new TaskRequestDTO();
        dto.setTitle("Invalid Task");
        dto.setPriority(Priority.MEDIUM);
        dto.setDueDate(LocalDateTime.now().minusDays(1));
        dto.setCategory(Category.WORK);
        
        mockMvc.perform(post("/api/tasks").contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(dto))).andExpect(status().isBadRequest());
    }
    
    @Test
    void shouldGetTaskById() throws Exception {
        Task task = new Task();
        task.setTitle("Search Me");
        task.setPriority(Priority.LOW);
        task.setDueDate(LocalDateTime.now().plusDays(2));
        task.setCategory(Category.OTHER);
        task.setCreatedAt(LocalDateTime.now());
        Task saved = taskRepository.save(task);
        
        mockMvc.perform(get("/api/tasks/" + saved.getId())).andExpect(status().isOk())
                .andExpect(jsonPath("$.title").value("Search Me"));
    }
    
    @Test
    void shouldNotDeleteCompletedTask() throws Exception {
        Task task = new Task();
        task.setTitle("Done Task");
        task.setCompleted(true);
        task.setCategory(Category.PERSONAL);
        task.setDueDate(LocalDateTime.now().plusDays(2));
        task.setCreatedAt(LocalDateTime.now());
        Task saved = taskRepository.save(task);
        
        mockMvc.perform(delete("/api/tasks/" + saved.getId())).andExpect(status().isConflict());
    }
    
    @Test
    void shouldListTasksWithPagination() throws Exception {
        for (int i = 0; i < 10; i++) {
            Task task = new Task();
            task.setTitle("Task " + i);
            task.setCategory(Category.WORK);
            task.setDueDate(LocalDateTime.now().plusDays(3));
            task.setCreatedAt(LocalDateTime.now());
            taskRepository.save(task);
        }
        
        mockMvc.perform(get("/api/tasks?page=0&size=5")).andExpect(status().isOk())
                .andExpect(jsonPath("$.content.length()").value(5));
    }
    
    @Test
    void shouldSearchByCategory() throws Exception {
        Task task = new Task();
        task.setTitle("Work Task");
        task.setCategory(Category.WORK);
        task.setDueDate(LocalDateTime.now().plusDays(2));
        task.setCreatedAt(LocalDateTime.now());
        taskRepository.save(task);
        
        mockMvc.perform(get("/api/tasks/search").param("category", "WORK")).andExpect(status().isOk())
                .andExpect(jsonPath("$.content[0].category").value("WORK"));
    }
}
```

Esses testes são **testes de integração funcional**, realizados com **`MockMvc`**. Aqui simulamos requisições HTTP reais, sem subir o servidor, mas envolvendo de fato toda a stack Spring Boot configurada.

- **Configurações**
  - `@SpringBootTest` indica que queremos inicializar o contexto do Spring.
  - `@AutoConfigureMockMvc` habilita o uso do `MockMvc`.
  - `@Autowired` injeta dependências reais como o `MockMvc`, o `ObjectMapper`, e o `TaskRepository`.

- **`@BeforeEach cleanDb()`**
  - Antes de cada teste, limpamos o banco de dados (H2 em memória) para garantir que os testes sejam independentes.

Métodos testados:

- **shouldCreateTask()**
  - Testa se conseguimos criar uma tarefa válida via `POST /api/tasks`.
  - Verifica se a resposta contém o título esperado.

- **shouldFailWithPastDueDate()**
  - Testa se o sistema rejeita a criação de tarefas com data vencida.

- **shouldGetTaskById()**
  - Testa a recuperação de uma tarefa existente por `GET /api/tasks/{id}`.

- **shouldNotDeleteCompletedTask()**
  - Testa a tentativa de excluir uma tarefa concluída, que deve resultar em erro de conflito (`409 Conflict`).

- **shouldListTasksWithPagination()**
  - Testa a paginação criando múltiplas tarefas e garantindo que apenas 5 tarefas sejam retornadas na página solicitada.

- **shouldSearchByCategory()**
  - Testa a busca de tarefas filtradas pela categoria.


É importante considerar que aqui testamos o fluxo **end-to-end** (entrada HTTP → validação → serviço → resposta HTTP). Além disso, há uso de um **banco de dados real:** o `TaskRepository` salva de fato no banco de dados H2 para os testes funcionarem. Por fim, também temos a **validação de respostas:** por meio da utilização do `jsonPath` para validar atributos específicos da resposta JSON.

#### Resumo dos Testes

Podemos resumir as características gerais dos nossos testes tal como mostrado a seguir:

| Tipo de Teste  | Arquivo                 | Foco Principal                          | Dependências Real/Mocadas | Observações                  |
|----------------|--------------------------|-----------------------------------------|----------------------------|-------------------------------|
| Unitário       | `TaskServiceTest.java`    | Lógica isolada do `TaskService`          | Tudo mockado (Mockito)     | Não interage com banco real.  |
| Funcional      | `TaskControllerTest.java` | Fluxo HTTP completo (`MockMvc`)          | Banco de dados real (H2)    | Simula chamadas reais.        |

Ou seja, concluímos a implementação dos testes da nossa aplicação To-Do List com duas abordagens complementares: testes **unitários** na camada de serviço e testes **funcionais** na camada de controle. Essa estratégia permite que tenhamos confiança **tanto no comportamento interno da aplicação quanto no seu comportamento externo** diante dos usuários.

Ao isolar as responsabilidades (usando mocks nos testes de serviço) e ao validar fluxos reais (simulando requisições HTTP com MockMvc), conseguimos cobrir cenários tanto positivos quanto negativos — desde o simples cadastro de uma nova tarefa até situações de erro como tentativas inválidas de exclusão.

É importante reforçar que boas práticas de testes, como vimos aqui, não são apenas uma exigência burocrática dos projetos, mas também uma grande aliada dos desenvolvedores no dia a dia. Ao construir uma base sólida de testes, reduzimos o medo de refatorar, ganhamos agilidade em novas implementações e entregamos um software com qualidade consistente.

Por fim, vale lembrar: **testar não é apenas encontrar defeitos**, mas também **documentar o comportamento esperado** do sistema. Nosso conjunto de testes torna explícito — e validável — como cada funcionalidade deve operar! 🚀