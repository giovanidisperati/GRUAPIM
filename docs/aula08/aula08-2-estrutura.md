---
layout: aula
title: "2. Estrutura do Projeto"
parent: Aula 08 - Segurança de APIs com JWT!
nav_order: 3
---

## 2. Estrutura do Projeto 

Vamos dar continuidade, como mencionamos inicialmente, à aplicação de To-do List. Nossa aplicação ganhará novas classes e pastas relacionadas à segurança, conforme o projeto evolui. A seguir, apresentaremos passo a passo os principais arquivos e explicações, mantendo a mesma organização e estilo da Aula 07.

Após a implementação, a estrutura do projeto ficará como mostrado a seguir:

```
.
├── src/
│   └── main/
│       └── java/
│           └── br/ifsp/edu/todo/
│               ├── config/
│               │   ├── ModelMapperConfig.java
│               │   └── SecurityConfig.java
│               ├── controller/
│               │   ├── AuthenticationController.java
│               │   ├── TaskController.java
│               │   └── UserController.java
│               ├── dto/
│               │   ├── authentication/
│               │   │   ├── AuthenticationDTO.java
│               │   │   └── UserRegistrationDTO.java
│               │   ├── page/
│               │   │   └── PagedResponse.java
│               │   └── task/
│               │       ├── TaskRequestDTO.java
│               │       └── TaskResponseDTO.java
│               ├── exception/
│               │   ├── ErrorResponse.java
│               │   ├── GlobalExceptionHandler.java
│               │   ├── InvalidTaskStateException.java
│               │   ├── ResourceNotFoundException.java
│               │   └── UserAlreadyExistsException.java
│               ├── mapper/
│               │   └── PagedResponseMapper.java
│               ├── model/
│               │   ├── enumerations/
│               │   │   ├── Category.java
│               │   │   ├── ERole.java
│               │   │   └── Priority.java
│               │   ├── Role.java
│               │   ├── Task.java
│               │   ├── User.java
│               │   └── UserAuthenticated.java
│               ├── repository/
│               ├── security/
│               │   └── CustomJwtAuthenticationConverter.java
│               ├── service/
│               │   ├── AuthenticationService.java
│               │   ├── JwtService.java
│               │   ├── TaskService.java
│               │   ├── UserDetailService.java
│               │   ├── UserService.java
│               │   └── TodoApplication.java
│   └── resources/
│       ├── db/migration/
│       │   ├── V1_CreateTables.sql
│       │   ├── V2_InsertDefaultRoles.sql
│       │   └── V3_InsertDefaultUsers.sql
│       ├── static/
│       ├── templates/
│       ├── app.key
│       ├── app.pub
│       └── application.properties
├── test/
│   └── java/
│       └── br/ifsp/edu/todo/
│           └── task/
│               ├── TaskControllerTest.java
│               ├── TaskServiceTest.java
│           └── TodoApplicationTests.java
```

Vamos começar explorando as configurações de segurança com JWT, a estrutura de autenticação e o novo fluxo de controle de acesso!
