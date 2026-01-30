---
layout: aula
title: 4. Conclusão e exercícios
parent: Aula 06 - Service Layers e Testes Automatizados
nav_order: 4
---

## 4. 📚 Conclusões da Aula 

Ao longo da Aula demos mais um passo importante na construção de nossa API REST com Spring Boot, focando na **refatoração da arquitetura dos controllers**, **melhoria do mapeamento entre entidades e DTOs**, e na **implementação de testes automatizados** para garantir a qualidade e previsibilidade do sistema.

Vamos recapitular os principais pontos abordados:

Iniciamos a aula realizando a **refatoração completa dos controllers** para que passassem a retornar objetos do tipo `ResponseEntity`. Essa mudança, embora sutil, trouxe vantagens para nossa aplicação:

- Permitiu o controle mais facilitado sobre o código de status HTTP retornado ao cliente (`200 OK`, `201 Created`, `204 No Content`, `404 Not Found`, etc);
- Facilitou o envio de **headers customizados**, **mensagens personalizadas** e **respostas sem corpo** quando necessário;
- Deixou o código mais legível e alinhado às boas práticas RESTful modernas;
- Preparou o terreno para futuras melhorias, como a adição de mensagens de erro customizadas, headers de paginação, etc.

Essa padronização com `ResponseEntity` ajuda a manter a **consistência na comunicação com o cliente**, facilitando tanto o consumo da API por sistemas externos quanto a manutenção da própria aplicação ao longo do tempo.

Além disso, melhoramos o mapeamento das entidades e DTOs via `ModelMapper` em nosso `ContactController`. Isso fez com que nosso código ficasse menos verboso e mais claro.

Por fim, encerramos a aula com a criação de **testes automatizados** — tanto **unitários** quanto **funcionais** — que garantem o bom funcionamento e a estabilidade dos principais endpoints da API.

Realizamos os seguintes testes:

- **Testes unitários com MockMvc** para simular requisições HTTP aos controllers, verificando status de resposta, conteúdo retornado, e integração com os DTOs;
- **Testes de caminho feliz**, simulando casos em que os recursos e chamadas são corretas e espera-se resposta de sucesso de nossa API;
- **Testes de exceção**, simulando casos em que recursos não são encontrados e garantindo que o comportamento da aplicação siga o contrato esperado;

Esses testes aumentam a **confiança no sistema**, facilitam futuras refatorações e reduzem o risco de regressões.

### 📌 Considerações Finais

Desenvolver APIs RESTful robustas não é apenas uma questão de fazer os endpoints funcionarem. É sobre projetar aplicações com **qualidade técnica, clareza conceitual e boas práticas consolidadas**. Nesta aula, vimos que uma pequena refatoração pode ter impacto na manutenibilidade e evolução de um sistema.

O código limpo, os testes bem escritos e a separação de responsabilidades não são luxo — são **necessidades** em projetos de vida longa. À medida que as aplicações crescem, esses fundamentos se tornam cada vez mais valiosos.

Continue investindo nesses princípios. Eles são o que diferencia um sistema improvisado de um sistema profissional. E são o que diferencia um programador iniciante de um bom engenheiro de software.

---

## 🚀 Exercício para a próxima aula

Vimos muitos conceitos até agora. É hora de solidificá-los antes de darmos continuidade na disciplina. Para isso, vamos fazer um exercício prático! Leia o enunciado abaixo e elabore, em duplas, o exercício.

### API de Gerenciamento de Tarefas 👨‍🏭

Você foi contratado para desenvolver uma **API REST para gerenciamento de tarefas pessoais** (to-do list), permitindo que usuários criem, atualizem, consultem e excluam suas tarefas. A API deve ser construída seguindo os princípios RESTful, utilizar verbos HTTP adequadamente, e implementar **paginação**, **validações**, **tratamento de exceções**, e **testes automatizados** (unitários e funcionais).

### 📌 Regras de Negócio

1. Cada **tarefa** deve conter:
   - `id`: identificador único (gerado automaticamente).
   - `titulo`: texto curto e obrigatório.
   - `descricao`: texto opcional.
   - `prioridade`: deve ser `BAIXA`, `MEDIA` ou `ALTA`.
   - `dataLimite`: data final para conclusão da tarefa.
   - `concluida`: `true` ou `false`, indicando se a tarefa já foi finalizada.
   - `categoria`: campo textual obrigatório (ex: “trabalho”, “estudo”, “pessoal”).
   - `criadaEm`: data de criação (preenchida automaticamente).

2. Não é permitido criar tarefas com **dataLimite anterior à data atual**.

3. Tarefas concluídas **não podem ser editadas ou apagadas** — nesse caso, deve ser lançada uma exceção com status **409 (Conflict)**.

### 📥 Funcionalidades obrigatórias

Implemente os seguintes endpoints com seus respectivos verbos HTTP:

| Verbo  | Caminho                     | Descrição                                         |
|--------|-----------------------------|---------------------------------------------------|
| POST   | `/api/tasks`                | Criação de nova tarefa                            |
| GET    | `/api/tasks`                | Listar tarefas com paginação                      |
| GET    | `/api/tasks/{id}`           | Buscar tarefa por ID                              |
| GET    | `/api/tasks/search`         | Buscar tarefas por categoria (`?categoria=...`)   |
| PATCH  | `/api/tasks/{id}/concluir`  | Marcar tarefa como concluída                      |
| PUT    | `/api/tasks/{id}`           | Atualização completa da tarefa                    |
| DELETE | `/api/tasks/{id}`           | Remover tarefa (se ainda não estiver concluída)   |

### ❌ Tratamento de Exceções

Você deverá implementar uma classe `GlobalExceptionHandler` para capturar e retornar respostas adequadas para:

- `ResourceNotFoundException`: retorna 404.
- `InvalidTaskStateException`: retorna 409 quando há tentativa de modificar tarefas concluídas.
- `ValidationException`: retorna 400 com mensagens amigáveis.

### 📃 Validações

Use anotações de Bean Validation para validar os campos da entidade e dos DTOs. Exemplo:

- `titulo` e `categoria` devem ser obrigatórios.
- `prioridade` deve ser um dos valores definidos.

### 📄 Paginação

O endpoint `GET /api/tasks` deverá usar o `Pageable` do Spring e retornar tarefas com suporte a:

- número da página
- tamanho da página
- ordenação por prioridade ou dataLimite

### ✅ Testes Automatizados

Implemente **testes unitários** e **testes funcionais** para pelo menos os seguintes cenários:

- Criar uma tarefa com dados válidos.
- Tentar criar uma tarefa com `dataLimite` inválida.
- Buscar tarefa existente por ID.
- Tentar excluir uma tarefa concluída (e receber erro 409).
- Listar tarefas com paginação.
- Buscar tarefas por categoria.

Use `Mockito` e `MockMvc` (ou `TestRestTemplate`) conforme o tipo de teste.

### 💡 Dicas

- Use `ModelMapper` para conversão entre DTOs e entidades.
- Mapeie corretamente as enums (`@Enumerated`).
- Bônus: mantenha a lógica de negócio isolada em uma **camada de serviço** (`TaskService`).

### 🎯 Objetivos do exercício

Este exercício tem como finalidade consolidar os seguintes conceitos:

- Implementação de API REST com boas práticas.
- Criação e uso de DTOs.
- Manipulação de exceções e mensagens amigáveis.
- Validação de dados com Bean Validation.
- Paginação e ordenação de dados.
- Cobertura de testes unitários e funcionais.

## **Bom trabalho e mãos à obra! 🛠️**