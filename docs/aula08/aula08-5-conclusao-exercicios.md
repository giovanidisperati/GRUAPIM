---
layout: aula
title: "5. Conclusões e exercícios"
parent: Aula 08 - Segurança de APIs com JWT!
nav_order: 6
---

## 5. É isso? 🙏

Sim, é isso! Chegamos ao final da Aula 08. Essa é, sem dúvidas, a implementação mais complicada que fizemos no Spring (até agora, pelo menos).

A partir daqui, nossa API passou a contar com autenticação e autorização baseadas em JWT, o que permite proteger os dados dos usuários e garantir que apenas operações autorizadas sejam permitidas. Implementamos geração e validação de tokens, criptografia com RSA, controle de acesso via roles e configuração completa do Spring Security para funcionar como um Resource Server stateless.

A abordagem adotada atende bem ao contexto de uma aplicação monolítica, com autenticação centralizada e controle direto sobre os usuários. Com isso, podemos garantir que cada usuário só possa acessar suas próprias tarefas e que administradores possam ter acesso ampliado. A separação entre os componentes (serviço, controlador, conversor e configuração de segurança) também facilita futuras manutenções ou evoluções.

Evidentemente há muito a melhorar se tratando de uma aplicação real, mas essa é uma introdução que balanceia a complexidade inerente dessa implementação com a apresentação dos conceitos envolvidos.

A partir daqui, podemos começar a pensar em desafios mais avançados, como:

- Controle de acesso baseado em roles por endpoint;
- Geração de refresh tokens;
- Integração com OAuth2 e OpenID Connect;
- Cache de dados do usuário para melhorar performance;
- E migração para uma arquitetura baseada em microsserviços.

Por enquanto, temos tudo que precisamos para garantir autenticação segura e acesso controlado na nossa API REST.

## 6. E agora, José? 🦜

Com a segurança implementada, nossa aplicação já é capaz de autenticar usuários e restringir o acesso a recursos protegidos. A partir daqui, podemos evoluir a API para lidar com novos cenários e boas práticas, como:

- Adicionar **refresh tokens**, permitindo sessões mais duradouras sem precisar de login constante.
- Integrar com **provedores de identidade externos**, como **Keycloak**, **Auth0** ou o **Spring Authorization Server**.
- Implementar **controle de acesso por recurso**, garantindo que um usuário só possa acessar ou modificar as próprias tarefas.
- Pensar em soluções de **cache**, como **Redis**, para evitar a consulta ao banco a cada requisição autenticada.
- Embutir mais dados diretamente no JWT, reduzindo chamadas desnecessárias ao banco.
- Preparar a aplicação para uso em **arquiteturas distribuídas**, como em ambientes de microsserviços.

Apesar dessas possibilidades, nosso foco agora deve ser consolidar a base construída. A autenticação com JWT já está ativa e funcional, e as próximas etapas envolvem aplicar essa segurança ao restante da aplicação.

Na próxima aula, vamos introduzir o controle de acesso por roles, diferenciando permissões de usuários comuns e administradores. Também vamos explorar o uso da anotação `@AuthenticationPrincipal` e aplicar as restrições diretamente nos controllers.

Vale destacar que ainda não refatoramos os controladores existentes para aproveitar essa estrutura, e que as `migrations` incluídas no projeto também não foram detalhadas até aqui. Sabem o que isso significa? Que teremos, como sempre...

## Exercícios

Refatore os controladores da aplicação para utilizar a autenticação implementada com JWT. Aplique as regras de negócio descritas no início da aula:

- Usuários só podem acessar, editar ou excluir as **próprias tarefas**.
- Administradores podem visualizar **todas as tarefas**.

### Dicas práticas

- **Inclua o token nas requisições**: ao testar a API no Postman ou outro cliente HTTP, envie o JWT no cabeçalho:

  ```
  Authorization: Bearer <token>
  ```

- **Acesse o usuário logado com `@AuthenticationPrincipal`**:

  ```java
  @GetMapping("/me/tasks")
  public List<TaskResponseDTO> listUserTasks(@AuthenticationPrincipal UserAuthenticated authentication) {
      return taskService.findTasksByUser(authentication.getUser());
  }
  ```

- **Associe a tarefa ao usuário autenticado**:

  ```java
  @PostMapping("/tasks")
  public ResponseEntity<TaskResponseDTO> createTask(@RequestBody TaskRequestDTO dto,
                                                    @AuthenticationPrincipal UserAuthenticated authentication) {
      return ResponseEntity.ok(taskService.createTask(dto, authentication.getUser()));
  }
  ```

- **Proteja endpoints por role com `@PreAuthorize`**:

  ```java
  @PreAuthorize("hasRole('ADMIN')")
  @GetMapping("/admin/tasks")
  public List<TaskResponseDTO> listAllTasks() {
      return taskService.findAll();
  }
  ```