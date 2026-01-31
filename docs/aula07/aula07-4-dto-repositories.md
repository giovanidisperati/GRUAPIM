---
layout: aula
title: 4. DTOs e Repositories
parent: Aula 07 - Construindo o To-do List
nav_order: 4
---


### 3.2. Data Transfer Objects (`dto`)

Agora que conhecemos os modelos de domínio, vamos analisar como estruturamos a comunicação de dados entre o cliente e a nossa aplicação: os **DTOs**. Os DTOs nos permitem expor apenas as informações necessárias de forma controlada, além de validar a entrada de dados de maneira eficiente e desacoplada do modelo de domínio.

#### `TaskRequestDTO.java`

```java
package br.ifsp.edu.todo.dto.task;

@Data
public class TaskRequestDTO {
 @NotBlank
    @Size(min = 10, max = 100)
    private String title;

    @Size(max = 255)
    private String description;

    @NotNull
    private Priority priority;

    @NotNull
    @FutureOrPresent
    private LocalDateTime dueDate;

    private boolean completed;

    @NotNull
    private Category category;
}
```
A classe `TaskRequestDTO` representa o modelo de **entrada de dados** que a aplicação espera receber do cliente quando ele quiser criar ou atualizar uma tarefa. Ou seja, sempre que um usuário submete uma requisição para cadastrar ou alterar uma tarefa, os dados são mapeados para este DTO.

Principais características:
- **Validações com Bean Validation**:
  - `@NotBlank` e `@Size(min = 10, max = 100)` no `title`, garantindo que o título seja obrigatório e esteja entre 10 e 100 caracteres.
  - `@Size(max = 255)` para limitar o tamanho da `description`.
  - `@NotNull` no `priority`, `dueDate` e `category`, assegurando que esses campos sejam sempre informados.
  - `@FutureOrPresent` na `dueDate`, impedindo a criação de tarefas com data limite no passado.
- **Campos**:
  - `title`, `description`, `priority`, `dueDate`, `completed`, `category`.
  - Note que **não há campo `id` nem `createdAt`** no `TaskRequestDTO`, pois essas informações são geradas e controladas internamente pela aplicação e não devem ser manipuladas pelo usuário. 🙂

Em resumo, o `TaskRequestDTO` protege o modelo de domínio de dados inválidos e padroniza a estrutura de entrada de dados para as operações de criação e atualização.

#### `TaskResponseDTO.java`

```java
package br.ifsp.edu.todo.dto.task;

@Data
public class TaskResponseDTO {
    private Long id;
    private String title;
    private String description;
    private Priority priority;
    private LocalDateTime dueDate;
    private boolean completed;
    private Category category;
    private LocalDateTime createdAt;
}
```

A classe `TaskResponseDTO` define o modelo de **saída de dados** enviado de volta ao cliente quando uma tarefa é consultada, criada ou atualizada. É a estrutura que encapsula todas as informações necessárias para o cliente visualizar ou tratar a resposta da API.

Principais características:
- **Campos retornados**:
  - `id`: identificador único da tarefa.
  - `title`: título da tarefa.
  - `description`: descrição da tarefa.
  - `priority`: prioridade da tarefa.
  - `dueDate`: data limite para conclusão.
  - `completed`: status de conclusão.
  - `category`: categoria associada.
  - `createdAt`: data em que a tarefa foi criada.

Aqui, diferente do `TaskRequestDTO`, incluímos informações **geradas pela aplicação**, como `id` e `createdAt`, que não fazem sentido serem enviados pelo usuário, mas são muito relevantes para o consumidor da API!

#### 📄 `PagedResponse.java`

```java
package br.ifsp.edu.todo.dto.page;

@Data
@AllArgsConstructor
public class PagedResponse<T> {
    private List<T> content;
    private int page;
    private int size;
    private long totalElements;
    private int totalPages;
    private boolean last;
}
```

Aqui temos a primeira diferença em relação ao que fizemos anteriormente: a classe `PagedResponse<T>` é um DTO genérico criado para padronizar as respostas paginadas da API. Dessa forma, conseguimos garantir que todas as respostas paginadas da nossa aplicação sigam um formato consistente, facilitando o consumo por parte de front-ends ou integrações. Em vez de retornarmos diretamente um `Page<T>`, que é uma estrutura interna do Spring, adaptamos os dados para esse DTO mais controlado e amigável para o cliente.

Principais características:
- **Campos**:
  - `content`: lista de elementos da página (do tipo genérico `T`).
  - `pageNumber`, `pageSize`, `totalElements`, `totalPages`: metadados sobre a paginação.
  - `first`, `last`: flags indicando se estamos na primeira ou na última página.

Assim, os DTOs servem como **fronteira de segurança** e **contrato de comunicação** da nossa API. Eles reforçam a separação entre o que acontece internamente na aplicação e o que é exposto externamente, promovendo segurança, clareza e controle sobre a evolução da interface pública do sistema. Com os DTOs estruturando as entradas e saídas de dados, precisamos garantir que essas informações sejam corretamente persistidas no banco de dados. Vamos então explorar a camada de repositórios, que isola e facilita essa comunicação com o armazenamento permanente.

### 3.3. Repositórios (`repository`)

Com os dados e suas representações de entrada e saída definidos, precisamos agora garantir a **persistência** dessas informações. Para isso, utilizamos os **repositórios**, que fornecem uma abstração poderosa para comunicação com o banco de dados, reduzindo significativamente o esforço necessário para operações CRUD e permitindo métodos de consulta personalizados.

#### `TaskRepository.java`

```java
package br.ifsp.edu.todo.repository;
public interface TaskRepository extends JpaRepository<Task, Long> {

    Page<Task> findByCategory(Category category, Pageable pageable);
}
```

A interface `TaskRepository` representa a camada de acesso a dados da nossa aplicação, sendo responsável pela comunicação direta com o banco de dados. Ela estende `JpaRepository<Task, Long>`, o que nos permite herdar diversos métodos prontos para realizar operações CRUD (`findAll`, `save`, `deleteById`, `findById`, etc.) sem precisar implementá-los manualmente. Lembrem-se que vimos isso nas aulas anteriores!

Além disso, a `TaskRepository` define um método personalizado:

```java
Page<Task> findByCategory(Category category, Pageable pageable);
```

Esse método permite buscar tarefas filtrando por uma determinada categoria, já com suporte a paginação. O Spring Data JPA interpreta o nome do método (pela convenção *query method naming*, lembram-se?!) e gera automaticamente a consulta necessária para o banco de dados.

Com isso, conseguimos facilmente construir consultas específicas apenas declarando métodos na interface, sem a necessidade de escrever JPQL ou SQL manualmente — o que torna o desenvolvimento mais ágil e o código mais limpo.

Essa abordagem é especialmente útil para aplicações como a nossa, onde queremos que o repositório atue como um componente **focado apenas em persistência de dados**, enquanto toda a lógica de negócio permanece na camada de serviço.

Dessa forma, a camada de repositórios isola o acesso à base de dados e libera as demais camadas — principalmente a de serviço — da preocupação com detalhes técnicos de persistência. Esse isolamento é crucial para garantir a flexibilidade e testabilidade da nossa aplicação. Sabendo como armazenar e recuperar informações, surge agora uma pergunta importante: quem será o responsável por coordenar a lógica que conecta tudo isso? Para responder a essa questão, vamos mergulhar na camada de serviço, onde reside a inteligência operacional da aplicação.
