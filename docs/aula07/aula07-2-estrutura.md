---
layout: aula
title: 2. Estrutura de diretórios
parent: Aula 07 - Construindo o To-do List
nav_order: 2
---

## 2. Estrutura de Pastas e Pacotes

Nossa aplicação será organizada da seguinte forma:

```
src/main/java/br/ifsp/edu/todo/
├── config
│   └── ModelMapperConfig.java
├── controller
│   └── TaskController.java
├── dto
│   ├── page
│   │   └── PagedResponse.java
│   └── task
│       ├── TaskRequestDTO.java
│       └── TaskResponseDTO.java
├── exception
│   ├── ErrorResponse.java
│   ├── GlobalExceptionHandler.java
│   ├── InvalidTaskStateException.java
│   └── ResourceNotFoundException.java
├── mapper
│   └── PagedResponseMapper.java
├── model
│   ├── Category.java
│   ├── Priority.java
│   └── Task.java
├── repository
│   └── TaskRepository.java
├── service
│   └── TaskService.java
└── TodoApplication.java
```

Essa estrutura promove a separação clara de responsabilidades, facilita a manutenção, melhora a escalabilidade do projeto e segue a divisão em camadas que já vínhamos adotando nas aulas anteriores.

A camada `model` é responsável pelas entidades JPA que representam nossas tabelas no banco de dados.

- As entidades possuem anotações do pacote `jakarta.persistence`, como `@Entity`, `@Id`, `@GeneratedValue`, `@Enumerated`, entre outras.
- O mapeamento segue o padrão ORM (Object-Relational Mapping) para garantir a persistência correta dos dados.
- Utilizamos enums para representar valores fixos, como as prioridades das tarefas.

A camada `repository` é responsável pela interação direta com o banco de dados.

- Utilizamos a interface `JpaRepository` para herdar métodos padrão de CRUD.
- Métodos personalizados foram adicionados para suportar funcionalidades como busca por categoria.

A camada `dto` define objetos de transferência de dados, separando o modelo de domínio da representação utilizada nos endpoints.

- `TaskRequestDTO`: encapsula os dados enviados pelo cliente para criação ou atualização de tarefas.
- `TaskResponseDTO`: representa a resposta da API para o cliente.
- `PagedResponse`: padroniza respostas paginadas.

Todos os DTOs contam com anotações de validação (`@NotBlank`, `@Size`, `@Future`, etc.) para garantir a integridade dos dados no momento da entrada.

A camada `mapper` contém classes utilitárias para conversão entre entidades e DTOs, facilitando a adaptação dos modelos de domínio para exposições externas.

- Utilizamos o `ModelMapper` (configurado na pasta `config`) para automatizar os mapeamentos.

A camada `exception` é responsável pelo tratamento centralizado de erros.

- Utilizamos um `GlobalExceptionHandler` anotado com `@ControllerAdvice` para capturar e tratar exceções de forma padronizada.
- Definimos exceções customizadas para representar erros de domínio, como `ResourceNotFoundException` (recurso não encontrado) e `InvalidTaskStateException` (tentativa de operação inválida em tarefas concluídas).
- As respostas de erro são estruturadas utilizando o DTO `ErrorResponse`.

A camada `service` centraliza a lógica de negócio da aplicação, atuando como uma ponte entre o controller e o repositório.

- A classe `TaskService`, anotada com `@Service`, concentra toda a lógica de manipulação de tarefas: criação, atualização, conclusão e exclusão, bem como validações adicionais que não podem ser garantidas apenas com Bean Validation.
- A lógica de verificação de regras de negócio, como **impedir a modificação ou exclusão de tarefas já concluídas**, é implementada aqui.
- Essa abordagem evita a duplicação de código e garante que regras de negócio sejam mantidas de maneira **consistente e centralizada** em um único ponto do sistema.
- A injeção de dependência é feita via construtor, permitindo que os atributos sejam `final`, promovendo imutabilidade e facilitando a criação de mocks para testes unitários.
- A camada de serviço orquestra as operações necessárias: chama o repositório para acessar os dados e, quando necessário, utiliza o ModelMapper para transformar entidades em DTOs ou vice-versa.

**Importante:**  
A camada `service` **não implementa lógica de persistência** (não executa diretamente operações no banco) e **não implementa lógica de interface** (não formata diretamente respostas HTTP). Ela simplesmente **coordena o fluxo de dados e a execução das regras de negócio**