---
layout: aula
title: Aula 11 - Duas soluções para o nosso projeto!
nav_order: 13
has_children: true
permalink: /aula11
---

# **Aula 11 – Duas soluções possíveis para o nosso Projeto**

Na aula anterior, propusemos o projeto de extração de um Microsserviço. Agora, vamos propor duas soluções distintas, com base no todo que fizemos em aula: uma considerando a extração do nosso *core subdomain* e outra considerando a extração de um *support subdomain* (autenticação e autorização).

Bora?

## 1. Contextualizando o Desafio

Antes de mergulharmos nas soluções, vamos revisitar nosso ponto de partida. Atualmente, temos uma aplicação monolítica de lista de tarefas (*To-Do List*). Embora bem estruturada em camadas, toda a lógica de negócio reside em um único processo implantável. Os principais domínios que identificamos são:

1.  **Gerenciamento de Tarefas (`Task Management`)**: O coração da nossa aplicação. Lida com a criação, atualização, conclusão e listagem de tarefas. Este é o nosso **Core Domain**, a funcionalidade que entrega o principal valor ao nosso usuário.
2.  **Identidade e Acesso (`Identity & Access`)**: Responsável por cadastrar usuários, autenticá-los e controlar suas permissões (roles). Este é um **Supporting Subdomain**, pois ele dá suporte à funcionalidade principal, mas não é o diferencial do produto.

A extração de microsserviços, como vimos na Aula 10, é uma decisão estratégica que busca aumentar a autonomia das equipes, a escalabilidade e a resiliência do sistema. A grande questão é: **por onde começar?**

Não existe uma resposta única. A escolha depende dos objetivos de negócio e dos desafios técnicos que queremos resolver primeiro. Vamos explorar duas abordagens estratégicas distintas, mostrando como ficaria a implementação de cada uma.

---

## 2. Solução 1: Extraindo o Core Domain (Gerenciamento de Tarefas)

A primeira abordagem consiste em extrair a funcionalidade mais crítica e central do nosso sistema, o Gerenciamento de Tarefas, para um microsserviço dedicado.

### 2.1. Por que extrair o Core Domain?

Extrair o *Core Domain* é uma estratégia interessante quando:

  * **A lógica de negócio é complexa e muda com frequência**: Isolar o domínio principal permite que a equipe focada nele possa iterar e implantar novas funcionalidades de forma mais rápida e segura, sem impactar o resto do sistema.
  * **A escalabilidade é crítica**: Por exemplo, se no nosso contexto a criação e consulta de tarefas for a operação mais demandada da aplicação, um serviço dedicado a isso pode ser escalado de forma independente para atender a picos de tráfego.
  * **Queremos inovação focada**: Permite que a equipe de produto e desenvolvimento se concentre em melhorar a principal proposta de valor da aplicação com autonomia.

É importante destacarmos que em um sistema real e complexo, é provável, e inclusive muitas vezes desejável, que o *Core Domain* seja implementado através de múltiplos microsserviços. A ideia de que um *Core Domain* equivale a um único microsserviço é uma simplificação que estamos fazendo deliberadamente no nosso exemplo, mas na verdade essa não é a prática adotada visto que usualmente a complexidade de um negócio real é muito maior que a do nosso exemplo: lembrem-se, o *Core Domain* representa a área de negócio que contém a maior complexidade e a principal proposta de valor do produto, e essa complexidade pode ser grande demais para ser gerenciada de forma coesa e eficiente dentro de uma única unidade de implantação (um só microsserviço).

Por exemplo, imagine um *Core Domain* de uma plataforma de e-commerce focado na "Jornada de Compra do Cliente". Este é, sem dúvida, o coração do negócio. No entanto, ele pode ser decomposto em microsserviços menores e mais coesos, como um **Serviço de Carrinho de Compras**, um **Serviço de Precificação e Promoções**, um **Serviço de Orquestração de Pagamento** e um **Serviço de Gestão de Pedidos**. Todos eles pertencem ao mesmo *Core Domain*, pois lidam com a lógica de negócio mais crítica e que muda com frequência, mas cada um tem uma responsabilidade bem definida.

Ou seja, é importante ter isso em mente e não generalizar a nossa solução para o "mundo real", pois aqui nossa preocupação é puramente pedagógica.

### 2.2. Arquitetura Proposta

Nesta solução onde o *Core Domain* será extraído, teremos duas aplicações:

1. **`auth-service` (o antigo monólito)**: Continuará responsável pelo cadastro de usuários, autenticação e geração de tokens JWT. Ele atuará como nosso **Servidor de Autenticação**.
2. **`task-service` (o novo microsserviço)**: Será o responsável exclusivo por toda a lógica de tarefas. Atuará como um **Resource Server**, protegendo seus endpoints e validando os tokens JWT gerados pelo `auth-service`.

O fluxo de comunicação seria o seguinte:

1. O cliente (frontend) se autentica no `auth-service` e obtém um token JWT.
2. Para qualquer operação com tarefas (criar, listar, etc.), o cliente faz a requisição para o `task-service`, enviando o token JWT no cabeçalho `Authorization`.
3. O `task-service` valida o token (verificando sua assinatura com a chave pública do `auth-service`) e, se válido, extrai o `userId` do token para associar a tarefa ao usuário correto.

Perceba que essa arquitetura com dois serviços é um ótimo ponto de partida do ponto de vista pedagógico, mas é natural pensar: e se amanhã tivermos não apenas o `task-service`, mas também um `notification-service`, um `report-service` e um `payment-service`? Ou seja, o que acontece se o sistema crescer? Vamos ter que ficar verificando validade de Tokens em todos os microsserviços?

Se seguirmos o modelo atual, cada um desses serviços teria que ter sua própria cópia da lógica de `SecurityConfig` para validar os tokens. Isso rapidamente vira um pesadelo:

  * **Código duplicado por todo lado**: Qualquer ajuste na validação do token (como adicionar uma nova verificação) precisaria ser replicado em cinco lugares diferentes. A chance de esquecer um e introduzir uma falha de segurança é enorme.
  * **Serviços "poluídos"**: A responsabilidade de cada serviço deveria ser o seu domínio (tarefas, notificações, etc.), e não se preocupar com a infraestrutura de validação de tokens.

É para evitar esse tipo de problema que um padrão muito comum em microsserviços entra em cena: o **API Gateway**. A ideia é simples: em vez de cada serviço validar o token individualmente, colocamos um "porteiro" na frente de todos eles. O API Gateway se torna o **único ponto de entrada** para a nossa aplicação. O fluxo, com um Gateway, ficaria assim:

1. O cliente se autentica no `auth-service` e pega seu token.
2. O cliente envia **todas** as requisições (para tarefas, notificações, etc.) para o API Gateway, sempre com o token.
3. **É o Gateway quem valida o token**. Se estiver tudo certo, ele repassa a requisição para o microsserviço correspondente (que agora não precisa mais se preocupar com isso).

As vantagens são grandes:

* **Lógica centralizada**: A validação de tokens, logs, métricas e outras preocupações "transversais" vivem em um só lugar.
* **Serviços mais limpos**: O `task-service` agora só se preocupa com tarefas. Ele simplesmente confia que, se a requisição chegou até ele, é porque o Gateway já fez o trabalho sujo da segurança. O Gateway pode até facilitar as coisas, extraindo o `userId` do token e passando como um cabeçalho simples (`X-User-ID: 123`) para os serviços internos.

Tecnologias como **Spring Cloud Gateway** podem ser utilizadas para implementar esse padrão.

Portanto, embora para o nosso exercício a validação em cada serviço seja um aprendizado inicial, em um cenário real com múltiplos serviços, o caminho natural seria evoluir para um API Gateway. Novamente: essa perspectiva não está sendo adotada agora para reduzirmos a complexidade.

### 2.3. Passo a Passo para Implementação

Vamos detalhar as etapas para realizar essa extração.

#### **Passo 1: Criar o novo projeto `task-service`**

1. Use o Spring Initializr para criar um novo projeto Spring Boot.
2. Nomeie o grupo como `br.ifsp.edu.todo`.
2. Nomeie o artefato como `task-service`.
3. Adicione as mesmas dependências que temos no projeto original relacionadas a Web, JPA, Validação, Segurança (OAuth2 Resource Server) e o driver do banco de dados, a saber: `Spring Web`, `Spring Data JPA`, `Validation`, `Spring Security`, `OAuth2 Resource Server`, `Lombok `, `MariaDB Driver` (ou `MySQL Driver`, dependendo do seu banco), `Flyway Migration` (estamos utilizando para gerenciar as migrations no projeto todo) e o `H2 Database` (banco de dados para os testes).

#### **Passo 2: Migrar a Lógica de Tarefas**

1. **Mova as classes** do projeto original para o `task-service`:

      * **Model**: `Task`, `Category`, `Priority`.
      * **DTOs**: `TaskRequestDTO`, `TaskResponseDTO`, `PagedResponse`.
      * **Repository**: `TaskRepository`.
      * **Controller**: `TaskController`.
      * **Service**: `TaskService`.
      * **Exceptions**: `InvalidTaskStateException`, `ResourceNotFoundException`.
      * **Mappers**: `PagedResponseMapper`.

Nossa estrutura de diretórios do `task-service` ficaria algo como:

```
task-service/
└── src/
    └── main/
        ├── java/
        │   └── br/
        │       └── ifsp/
        │           └── edu/
        │               └── todo/
        │                   └── task/
        │                       ├── config/          // Configurações, como ModelMapper
        │                       │   └── ModelMapperConfig.java
        │                       ├── controller/
        │                       │   └── TaskController.java
        │                       ├── dto/
        │                       │   ├── page/
        │                       │   │   └── PagedResponse.java
        │                       │   └── task/
        │                       │       ├── TaskRequestDTO.java
        │                       │       └── TaskResponseDTO.java
        │                       ├── exception/
        │                       │   ├── GlobalExceptionHandler.java // Essencial para tratar erros
        │                       │   ├── InvalidTaskStateException.java
        │                       │   ├── ErrorResponse.java        
        │                       │   └── ResourceNotFoundException.java
        │                       ├── mapper/
        │                       │   └── PagedResponseMapper.java
        │                       ├── model/
        │                       │   ├── enumerations/  // Subpacote para enums
        │                       │   │   ├── Category.java
        │                       │   │   └── Priority.java
        │                       │   └── Task.java
        │                       ├── repository/
        │                       │   └── TaskRepository.java
        │                       ├── service/
        │                       │   └── TaskService.java
        │                       └── TaskServiceApplication.java // Classe principal
        │
        └── resources/
            ├── db/migration/    // Scripts do Flyway para o banco de tarefas
            │   └── V1__Create_Task_Table.sql
            └── application.properties
```

2. **Ajuste a Entidade `Task`**:
    A entidade `Task` no projeto original tem uma relação `@ManyToOne` com a entidade `User`. Como o `task-service` não conhecerá a entidade `User` (que pertence a outro serviço), devemos quebrar essa associação direta.

      * **Antes (no monólito):**
        ```java
        @ManyToOne(fetch = FetchType.LAZY)
        @JoinColumn(name = "user_id", nullable = false)
        private User user;
        ```
      * **Depois (no `task-service`):**
        ```java
        @NotNull
        @Column(name = "user_id")
        private Long userId;
        ```

    Agora, o `task-service` armazena apenas o **ID do usuário**, mantendo os serviços desacoplados.

#### **Passo 3: Configurar a Segurança no `task-service`**

O `task-service` precisa validar os tokens JWT. A configuração de segurança (`SecurityConfig.java`) será a de um **Resource Server**.

```java
// SecurityConfig.java no task-service
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {

    @Value("${jwt.public.key}")
    private RSAPublicKey key;

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(auth -> auth
                .anyRequest().authenticated() // Todas as rotas são protegidas
            )
            .oauth2ResourceServer(conf -> conf.jwt(Customizer.withDefaults())) // Habilita validação de JWT
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS));
        return http.build();
    }

    @Bean
    JwtDecoder jwtDecoder() {
        return NimbusJwtDecoder.withPublicKey(this.key).build();
    }
}
```

  * **Atenção**: O `task-service` precisa da **chave pública (`app.pub`)** para validar os tokens. A chave privada (`app.key`) permanece apenas no `auth-service`. Ou seja, temos que mover a `app.pub` para resources no nosso task-service.

#### **Passo 4: Adaptar o `TaskService` e `TaskController`**

A lógica para criar e consultar tarefas agora deve usar o `userId` extraído do token.

  * **`TaskService.java` (método `createTask`)**

    ```java
    // O service agora recebe o userId diretamente
    public TaskResponseDTO createTask(TaskRequestDTO taskDto, Long userId) {
        // ... validações ...
        Task task = modelMapper.map(taskDto, Task.class);
        task.setUserId(userId); // Associa a tarefa ao ID do usuário
        task.setCreatedAt(LocalDateTime.now());
        task.setCompleted(false);
        Task createdTask = taskRepository.save(task);
        return modelMapper.map(createdTask, TaskResponseDTO.class);
    }
    ```

  * **`TaskController.java` (método `createTask`)**
    O controller irá extrair as informações do usuário do token JWT usando a anotação `@AuthenticationPrincipal`.

    ```java
    @PostMapping
    public ResponseEntity<TaskResponseDTO> createTask(@Valid @RequestBody TaskRequestDTO task,
            @AuthenticationPrincipal Jwt jwt) {
        Long userId = jwt.getClaimAsLong("userId"); // Extrai a claim 'userId' do token
        TaskResponseDTO taskResponseDTO = taskService.createTask(task, userId);
        return ResponseEntity.status(HttpStatus.CREATED).body(taskResponseDTO);
    }
    ```

#### **Passo 5: Remover a lógica de tarefas do monólito (`auth-service`)**

No projeto original, remova todas as classes e dependências relacionadas a tarefas que foram migradas. O monólito agora é o `auth-service` e sua única responsabilidade é a identidade.

  * O `auth-service` (`JwtService`) deve ser ajustado para **incluir o `userId` nas claims do token** quando ele é gerado.
    ```java
    // JwtService.java no auth-service
    public String generateToken(User user) {
        Instant now = Instant.now();
        long expire = 3600L;

        var claims = JwtClaimsSet.builder()
                .issuer("auth-service")
                .issuedAt(now)
                .expiresAt(now.plusSeconds(expire))
                .subject(user.getUsername())
                .claim("userId", user.getId()) // <--- ADICIONAR ESTA CLAIM
                .build();

        return jwtEncoder.encode(JwtEncoderParameters.from(claims)).getTokenValue();
    }
    ```

### 2.4. Vantagens e Desvantagens desta Abordagem

| Vantagens | Desvantagens |
| :--- | :--- |
| ✅ Foco total na evolução do **Core Domain**. | ❌ O monólito original ainda retém a complexidade de autenticação. |
| ✅ Escalabilidade isolada para a funcionalidade mais usada. | ❌ Requer um mecanismo de comunicação de rede entre os serviços. |
| ✅ A equipe de tarefas ganha autonomia total de implantação. | ❌ Aumenta a complexidade operacional (dois serviços para gerenciar). |

---

## 3. Solução 2: Extraindo o Supporting Subdomain (Autenticação e Autorização)

A segunda abordagem é extrair o domínio de suporte de **Identidade e Acesso** para um microsserviço dedicado. Esta é uma estratégia muito comum, pois a gestão de identidade é uma funcionalidade genérica, bem delimitada e fundamental para a criação de um ecossistema de serviços coeso.

### 3.1. Por que extrair o Domínio de Suporte?

Extrair um *Supporting Subdomain* como o de autenticação é vantajoso quando:

* **Queremos reutilizar a solução de identidade**: Um serviço de autenticação centralizado pode servir a múltiplos outros serviços ou aplicações no futuro.
* **Buscamos simplificar o Core Domain**: Ao mover a complexidade da segurança para fora, o serviço principal (que continuará com a lógica de tarefas) pode se concentrar exclusivamente em sua regra de negócio.
* **A equipe de segurança/infra é diferente**: Permite que uma equipe especializada gerencie a identidade, enquanto outras equipes consomem essa funcionalidade como um serviço.

### 3.2. Arquitetura Proposta

Nesta solução, a divisão de responsabilidades se inverte em relação à primeira abordagem:

1.  **`auth-service` (o novo microsserviço)**: Será responsável por cadastro, login, gerenciamento de usuários/roles e emissão de tokens JWT. Atuará como nosso **Servidor de Autenticação e Autorização**.
2.  **`task-service` (o antigo monólito, agora focado)**: Continuará responsável pelo gerenciamento de tarefas, mas agora atuará como um **Resource Server**, consumindo e validando os tokens gerados pelo `auth-service`.

O fluxo de comunicação do usuário final seria idêntico ao da Solução 1, mas a origem e o destino das responsabilidades de cada serviço são diferentes.

### 3.3. Perspectiva Estrutural e de Decisão de Negócio

**Do ponto de vista da implementação, o código final resultante desta solução seria praticamente idêntico ao da Solução 1.**. Por esse motivo, não é necessário replicá-lo aqui novamente.

Teríamos um `auth-service` gerando tokens com uma *claim* `userId` e um `task-service` validando esses tokens para autorizar as operações. A diferença crucial não está no código final, mas na **estratégia de migração** e nas **implicações para o negócio**.

Diferente da primeira solução, onde criamos um novo serviço para o *Core Domain*, aqui o processo é o de "esculpir" o serviço de autenticação a partir do monólito existente. O projeto monolítico original não é descartado; pelo contrário, ele é o ponto de partida que será "limpo" e simplificado. Toda a lógica de identidade, cadastro e segurança de autenticação (controllers, services, repositories e entidades como `User` e `Role`) seria movida para um projeto completamente novo: o `auth-service`.

O que resta no projeto original? Exatamente o nosso *Core Domain*. Ele se torna, por definição, o `task-service`. Sua responsabilidade é afunilada, e seu código é reduzido, removendo-se tudo que não diz respeito à gestão de tarefas. Naturalmente, ele precisaria passar por adaptações: a entidade `Task` passaria a referenciar um `userId` em vez de uma entidade `User` completa, e sua configuração de segurança seria alterada para atuar como um *Resource Server*, apenas validando tokens.

A decisão de extrair primeiro um domínio de suporte é frequentemente vista como uma **estratégia pragmática e de menor risco** por várias razões de negócio:

1.  **Criação de um Ativo Estratégico**: Ao isolar a autenticação, a empresa cria imediatamente um ativo reutilizável. Qualquer novo microsserviço (`notification-service`, `report-service`, etc.) que surgir no futuro pode se integrar a este `auth-service` central, acelerando o desenvolvimento e garantindo uma base de usuários unificada.
2.  **Redução da Carga Cognitiva**: A equipe que desenvolve o *Core Domain* (tarefas) é liberada da complexidade de gerenciar segurança, criptografia e tokens. Ela pode focar 100% em entregar valor ao usuário final, consumindo a segurança como uma utilidade. Isso permite maior especialização e agilidade.
3.  **Fundação para o Ecossistema**: Este padrão estabelece uma base sólida para um ecossistema de microsserviços. O `auth-service` se torna a porta de entrada de identidade, e padrões como o API Gateway, discutido na Solução 1, se encaixam perfeitamente neste modelo, centralizando a validação de tokens antes de distribuir as chamadas para os serviços de negócio.
4.  **Risco Controlado**: A lógica de autenticação, por ser mais estável e genérica, geralmente representa um risco menor de extração em comparação com o *Core Domain*, que tende a ser mais volátil e intrinsecamente complexo. Começar por aqui permite que a equipe ganhe experiência com a arquitetura de microsserviços em um domínio bem definido antes de abordar as partes mais críticas do negócio.

Em resumo, esta abordagem prioriza a construção de uma fundação robusta e reutilizável, simplificando a aplicação principal e preparando o terreno para uma expansão futura de forma organizada e escalável.

### 3.4. Vantagens e Desvantagens desta Abordagem

| Vantagens                                                                              | Desvantagens                                                                                                                   |
| :------------------------------------------------------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------- |
| ✅ Cria um serviço de identidade **reutilizável**.                                       | ❌ A lógica de negócio principal (`Task`) ainda reside em um monólito que pode ter outras responsabilidades.                 |
| ✅ **Simplifica o Core Domain**, que não precisa mais se preocupar com segurança.         | ❌ Requer que a equipe do Core Domain integre-se com uma API externa para obter informações do usuário, se necessário.          |
| ✅ Alinha-se bem com padrões de mercado (ex: ter um serviço de SSO).                    | ❌ A complexidade da extração inicial pode ser maior se o acoplamento entre usuário e tarefas for muito alto no código legado. |

---

## 4. Bônus! Uma Terceira Solução: Criando um Serviço Agregador com Anti-Corruption Layer (ACL)

Nesta terceira abordagem, não vamos extrair nada do monólito original. Em vez disso, vamos assumir que a **Solução 1** já foi implementada: temos um `auth-service` e um `task-service` independentes e funcionais. O código da Solução 1 já está disponibilizado no Moodle.

O nosso objetivo agora é criar um terceiro microsserviço, chamado **`report-service`**, com uma responsabilidade muito específica: gerar um relatório de produtividade de um usuário. Para isso, ele precisará consumir dados dos outros dois serviços, atuando como um **orquestrador** ou **agregador**. A ideia aqui é demonstrar uma outra possibilidade de criação de microsserviço, que em um primeiro momento não seria possível já que o nosso escopo original do to-do list era bastante restrito.

### 4.1. Por que criar um Serviço Agregador?

A criação de um serviço agregador como o `report-service` propicia uma série de benefícios:

  * **Simplifica o Cliente (Frontend)**: Em vez de o frontend ter que fazer duas chamadas separadas (uma para o `task-service` para pegar as tarefas e outra para o `auth-service` para pegar os dados do usuário) e juntar as informações no lado do cliente, ele faz uma única chamada para o `report-service`, que entrega os dados já consolidados.
  * **Encapsula a Lógica de Orquestração**: A complexidade de saber "quais serviços chamar e em que ordem" fica contida dentro do `report-service`. Se no futuro for preciso chamar um terceiro serviço (ex: `gamification-service`), o frontend não precisa saber; apenas o `report-service` é alterado.
  * **Cria Novas Visões de Dados**: Permite criar novas representações e junções de dados que não foram originalmente previstas pelos serviços individuais, agregando novo valor ao negócio sem precisar modificar os serviços já existentes e estáveis.
  * **Mantém a Autonomia dos Domínios**: O `task-service` continua não precisando saber nada sobre os detalhes de um usuário, e o `auth-service` não sabe nada sobre tarefas. O `report-service` funciona como uma camada especializada que entende como relacionar esses dois contextos distintos.

### 4.2. Arquitetura Proposta

O fluxo de comunicação para esta nova funcionalidade seria:

1.  O cliente (frontend) se autentica no `auth-service` e obtém um token JWT.
2.  O cliente faz uma requisição para o novo serviço, por exemplo: `GET /reports/productivity/{userId}`, enviando o token JWT no cabeçalho `Authorization`.
3.  O **`report-service`** recebe a requisição. Por ser um *Resource Server*, ele primeiro valida o token JWT para garantir que a chamada é autêntica.
4.  Dentro do `report-service`:
    a. Ele usa um **`RestTemplate`** para fazer uma chamada interna para o `task-service` (ex: `GET /tasks?userId={userId}`) para buscar todas as tarefas daquele usuário, passando o token JWT adiante.
    b. Ele usa outro **`RestTemplate`** para fazer uma chamada para o `auth-service` (ex: `GET /users/{userId}`) para buscar o nome e email do usuário.
5.  O `report-service` junta as informações, monta um DTO de relatório (`ProductivityReportDTO`) e o retorna para o cliente.

### 4.3. O Papel do `RestTemplate` e do Anti-Corruption Layer (ACL)

O `RestTemplate` é a ferramenta clássica do Spring Framework para realizar chamadas HTTP client. No nosso `report-service`, ele será o mecanismo para se comunicar com as APIs REST do `auth-service` e do `task-service`.

Para isso, configuramos um `Bean` de `RestTemplate` na nossa aplicação:

```java
@Configuration
public class AppConfig {
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

E este `RestTemplate` será então injetado nos componentes que precisam fazer as chamadas externas.

Já a ACL é um padrão de design (DDD) importante nesse caso. O `report-service` tem seu próprio domínio (relatórios), e ele **não deve ser "contaminado" pelos modelos de domínio dos outros serviços**. Se o `report-service` usasse diretamente os DTOs do `task-service` (`TaskResponseDTO`) em sua lógica interna, qualquer mudança no `task-service` poderia quebrar o `report-service`.

O ACL previne isso. Ele funciona como uma camada de "tradução" na fronteira do nosso serviço.

**Como implementar o ACL:**

1.  **Modelos Internos**: Dentro do `report-service`, criamos nossas próprias representações de dados, simples e focadas no que precisamos.

    ```java
    // Modelo interno no report-service
    public record UserInfo(Long id, String name, String email) {}
    public record TaskInfo(String title, boolean isCompleted, LocalDate dueDate) {}
    ```

2.  **Modelos Externos (Stubs)**: Criamos DTOs que representam exatamente a estrutura de dados que o serviço externo nos retorna.

    ```java
    // DTO que espelha a resposta do task-service
    public record ExternalTaskDTO(UUID id, String title, String description, boolean completed, String priority, LocalDate dueDate) {}
    ```

3.  **A Camada de Tradução (O ACL)**: Criamos uma classe cuja única responsabilidade é chamar o serviço externo e traduzir o modelo externo para o nosso modelo interno.

    ```java
    @Component
    public class TaskServiceACL {

        private final RestTemplate restTemplate;
        // ... construtor

        public List<TaskInfo> findTasksByUser(Long userId, String token) {
            // Lógica para adicionar o token no header da requisição
            // ...
            
            // Chama a API externa
            ResponseEntity<ExternalTaskDTO[]> response = restTemplate.exchange(
                "http://task-service/api/tasks?userId=" + userId, 
                HttpMethod.GET, 
                httpEntity, // Entidade com headers (token)
                ExternalTaskDTO[].class
            );
            
            // TRADUÇÃO: Mapeia o modelo externo para o interno
            return Arrays.stream(response.getBody())
                         .map(externalTask -> new TaskInfo(
                             externalTask.title(), 
                             externalTask.completed(), 
                             externalTask.dueDate()
                         ))
                         .collect(Collectors.toList());
        }
    }
    ```

A lógica de negócio do `report-service` **só conhece e manipula `UserInfo` e `TaskInfo`**. Ela nunca vê um `ExternalTaskDTO`. Se o `task-service` adicionar um novo campo ao seu DTO, o `report-service` não quebra; apenas a camada de tradução (ACL) precisaria ser ajustada se quiséssemos usar o novo campo.

### 4.4. Vantagens e Desvantagens desta Abordagem

| Vantagens                                                                  | Desvantagens                                                                                                     |
| :------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------- |
| ✅ **Alta Coesão e Baixo Acoplamento**: O ACL garante que os serviços permaneçam desacoplados. | ❌ **Aumento da Complexidade Operacional**: Mais um serviço para implantar, monitorar e gerenciar.                |
| ✅ **Agilidade para Novas Funcionalidades**: Permite criar novas visões de dados sem alterar serviços existentes. | ❌ **Latência de Rede**: Uma requisição do cliente agora dispara múltiplas chamadas na rede interna, aumentando o tempo de resposta. |
| ✅ **Simplificação do Frontend**: Reduz o número de chamadas e a lógica no lado do cliente. | ❌ **Ponto Único de Falha**: Se o `report-service` falhar, a funcionalidade de relatório fica indisponível.        |
| ✅ **Promove o Reúso**: Os serviços base (`auth` e `task`) são genéricos e podem ser consumidos por muitos outros agregadores. | ❌ **Pode virar um "mini-monólito"**: Se muitas lógicas de orquestração forem adicionadas, o serviço pode ficar complexo e difícil de manter. |

No Moodle disponibilizei também o código-fonte com a `report-service`. Deem uma olhada!

---

## 5. Conclusão: Qual Caminho Seguir?

Nesta aula, vimos que a jornada para uma arquitetura de microsserviços não tem um mapa único, mas sim um conjunto de estratégias que aplicamos de acordo com o contexto e a maturidade do nosso sistema. Exploramos três caminhos distintos, cada um com seu momento e propósito.

As **Soluções 1 e 2** representam as clássicas decisões de **decomposição de um monólito**. Elas respondem à pergunta: "Por onde começar a quebrar?".

* A escolha de **extrair o Core Domain (Solução 1)** é o caminho a seguir quando a lógica de negócio principal é o maior gargalo, complexa e precisa evoluir rapidamente, livre das amarras do restante do sistema. O objetivo é dar autonomia total à equipe que cuida da principal proposta de valor do produto.

* Por outro lado, **extrair um Supporting Subdomain (Solução 2)**, como o de identidade, é uma jogada de fundação. Priorizamos a criação de um ativo reutilizável que servirá de base para um ecossistema futuro, enquanto simplificamos os nossos serviços de negócio ao remover deles responsabilidades genéricas.

Já a **Solução 3 (Serviço Agregador)** nos mostra o passo seguinte na jornada: a **composição e evolução** de um sistema que já é distribuído. Ela responde à pergunta: "Como criar novas funcionalidades sobre os serviços que já existem?".

O nosso serviço agregador, protegido por um Anti-Corruption Layer (ACL), nos permite inovar na "superfície" do nosso ecossistema, criando novas visões de dados e simplificando a vida dos clientes (frontends) sem perturbar os serviços base.

Portanto, a pergunta "Qual caminho seguir?" se desdobra para "Qual é o nosso desafio *agora*?".

* O gargalo é a inovação no core? **Decomponha-o (Solução 1)**.
* Precisamos de uma base sólida e reutilizável para o futuro? **Decomponha um serviço de suporte (Solução 2)**.
* Os serviços existentes são estáveis, mas precisamos de uma nova funcionalidade que os combine? **Componha-os com um agregador (Solução 3)**.

Independentemente da escolha, o mais importante é que a decisão seja guiada pelos princípios do **Domain-Driven Design (DDD)**. Identificar corretamente seus subdomínios e *bounded contexts* é o primeiro e mais importante passo para uma arquitetura de microsserviços bem-sucedida e sustentável.