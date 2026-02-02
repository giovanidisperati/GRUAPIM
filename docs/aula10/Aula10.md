---
layout: aula
title: Aula 10 - Introdução ao Domain Driven Design
nav_order: 12
has_children: true
permalink: /aula10
---

# Aula 10 – Domínio, Subdomínio e Bounded Contexts com DDD

Na Aula anterior vimos os conceitos introdutório de microsserviços.

Agora, vamos focar-nos em compreender os **conceitos de domínio, subdomínio, bounded contexts e mapeamento de contextos** propostos por Eric Evans no **Domain-Driven Design (DDD)**. Esses conceitos são **fundamentais para o entendimento da arquitetura de microsserviços**, que será explorada na sequência das próximas aulas.

Vamos ver os conceitos que nos ajudam a decompor sistemas complexos de forma lógica e coerente, com base em regras de negócio reais e não apenas em critérios técnicos. Essa base teórica é o que permitirá que a transição para microsserviços seja feita de forma consciente, orientada pelo **modelo de negócio**, e não apenas por decisões arbitrárias de código.

---

## 1. O que é DDD e por que ele importa?

Antes de entrarmos na arquitetura de microsserviços do ponto de vista prático, é importante compreender o conceito que serve como um dos fundamentos lógicos para esse tipo de arquitetura: o **Domain-Driven Design (DDD)**.

Desenvolver aplicações em larga escala exige muito mais do que saber escrever endpoints REST ou persistir dados com JPA. É necessário ter clareza sobre **como organizar o software de forma a refletir o mundo real que ele pretende modelar**. Quando essa estruturação é feita de maneira confusa ou acidental, surgem sintomas comuns: código duplicado, lógica de negócio espalhada, classes que crescem sem controle, e times que não conseguem evoluir a aplicação sem quebrar funcionalidades existentes.

Foi nesse contexto que Eric Evans, em seu livro clássico *“Domain-Driven Design: Tackling Complexity in the Heart of Software”* (2003), propôs uma abordagem focada em **colocar o domínio no centro do desenvolvimento de software**. Em vez de partir da tecnologia, o DDD parte do **modelo de negócio**, da compreensão profunda do problema que estamos tentando resolver, como eixo organizador da arquitetura.

### 1.1. Motivação: o que o DDD resolve?

Eric Evans percebeu que muitos sistemas “cresciam para os lados”, misturando código técnico (como autenticação, logging, persistência) com regras de negócio importantes (como cálculo de imposto, regras de vencimento, fluxo de aprovação, etc.) sem critério definido. Isso tornava o sistema:

* Difícil de **compreender** (a lógica de negócio estava diluída em detalhes técnicos);
* Difícil de **evoluir** (cada modificação impactava partes imprevisíveis do sistema);
* Difícil de **testar** (as regras não estavam isoladas de suas dependências);
* Difícil de **escalar em equipe** (vários desenvolvedores trabalhando sobre a mesma área com sobreposição de responsabilidades).

O DDD surge como **uma forma de estruturar o sistema para que ele reflita com fidelidade o problema que está resolvendo**, criando **limites explícitos** entre os diferentes aspectos do negócio. Ao adotar essa abordagem, conseguimos criar um modelo conceitual claro, que guia não apenas o código, mas também a comunicação entre as pessoas envolvidas no projeto.

### 1.2. O que significa "Dirigido pelo Domínio"?

A palavra **"domínio"** se refere ao conjunto de regras, processos e conhecimento específico de um determinado negócio ou problema que a aplicação resolve. “Design dirigido por domínio” significa organizar o software em torno desse **conhecimento central**. Isso envolve:

* **Modelar com profundidade os conceitos do negócio**, e não apenas mapear tabelas para entidades;
* Trabalhar em estreita colaboração com **especialistas do domínio** (clientes, analistas, operadores do sistema) para entender o vocabulário e as regras envolvidas;
* Desenvolver uma **linguagem ubíqua** (ubiquitous language), que seja falada igualmente por desenvolvedores e especialistas;
* **Isolar diferentes partes do sistema** em **bounded contexts**, evitando ambiguidade e acoplamento desnecessário entre modelos;
* E, como consequência, permitir que diferentes partes do sistema **evoluam de forma independente**, com times focados em subdomínios específicos.

Portanto, o DDD não é um padrão de projeto, nem uma arquitetura pronta. Ele é uma **abordagem estratégica** que orienta a construção do software a partir daquilo que realmente importa: **o conhecimento sobre o negócio**.

### 1.3. Quando o domínio não existe, a bagunça domina

Imagine que uma *edtech* resolve criar um **sistema de gerenciamento de tarefas colaborativas** para seus instrutores e alunos. Nos primeiros sprints tudo cabe em um único repositório, com poucos endpoints REST. Mas, três meses depois, chegam novas demandas:

| Nova demanda                   | Por que apareceu?                                                                        |
| ------------------------------ | ---------------------------------------------------------------------------------------- |
| **Controle de permissões**     | Cada curso quer regras próprias de quem pode editar ou excluir tarefas.                  |
| **Atribuição de responsáveis** | Tutores precisam delegar tarefas a monitores e acompanhar progresso.                     |
| **Priorização e categorias**   | A coordenação quer uma visão rápida de pendências “urgentes” versus “melhoria contínua”. |
| **Notificações multicanal**    | Alunos pedem aviso no app; instrutores preferem e-mail.                                  |
| **Relatórios gerenciais**      | A diretoria quer dashboards semanais por curso e equipe.                                 |

#### 1.3.1. **Sem DDD: o "bolovo" que cresce sem parar**

O time segue empilhando código nas mesmas camadas genéricas:

```
src/
 ├─ service/
 │   ├─ UserService.java
 │   ├─ TaskService.java
 │   └─ ReportService.java
 ├─ controller/
 │   ├─ TaskController.java
 │   └─ AuthController.java
 └─ utils/
     └─ NotificationUtils.java
```

1. **UserService** agora contém 3 000 linhas, mistura hash de senha com lógica de ACL.
2. **TaskService** faz *query* em três tabelas e ainda dispara e-mails.
3. Quando um estagiário altera `NotificationUtils`, quebra autorização sem querer, pois tudo roda na mesma transação.

Refatorar dói, testar dá trabalho, e cada *hotfix* vira corrida contra o tempo.

#### 1.3.2. **Com DDD: clareza de limites**

A partir dessa situação, a alternativa é aplicar o DDD e "quebrar" o código. Os analistas de produto sentam com usuários finais e mapeiam **subdomínios**, que são pedaços do negócio que fazem sentido por si só:

| Subdomínio (Bounded Context) | Responsabilidade-chave                        | Exemplos de artefatos                          |
| ---------------------------- | --------------------------------------------- | ---------------------------------------------- |
| **Task Management**          | Criar, priorizar, atribuir e concluir tarefas | `Task`, `Priority`, evento `TaskAssigned`      |
| **Identity & Access**        | Login, senhas, ACL, tokens                    | `User`, `Role`, serviço `AuthService`          |
| **Notification**             | Disparar alertas (push, e-mail, SMS)          | `NotificationPreference`, evento `TaskOverdue` |
| **Reporting**                | Gerar relatórios e KPIs por curso/equipe      | `TaskSnapshot`, consulta OLAP                  |

Cada contexto vira um **módulo isolado** (em um mono-repositório ou em microsserviços) com:

* **Modelo de domínio próprio** – classes e regras que só fazem sentido ali.
* **API contratada** – troca de dados via eventos ou DTOs, jamais expondo entidades de outro contexto (Camada *Anti-Corruption*).
* **Banco dedicado ou esquema lógico separado** – permite versionar sem travar os demais times.

> **Decisão tática:** *Task Management* define `Task` como Aggregate Root e só expõe o identificador do usuário (`UserId`). Quando dispara `TaskAssignedEvent`, o módulo *Notification* decide como alertar o responsável.

Em termos de organização prática de código, teríamos algo como:

```
collab-tasks/
├── pom.xml                        # parent-pom: dependências e plugins comuns
│
├── task-management/               # Contexto “Task Management”
│   ├── pom.xml
│   └── src/main/java/com/example/taskmgmt/
│       ├── domain/               # Entidades, Aggregates, VOs
│       ├── application/          # Use cases, Application Services
│       ├── infrastructure/       # JPA, Messaging, REST Adapters
│       └── config/               # Beans, eventos, segurança local
│
├── identity-access/               # Contexto “Identity & Access”
│   └── … (mesma divisão domain/application/infra/config)
│
├── notification/                  # Contexto “Notification”
│   └── … (implementações de e-mail, push, SMS)
│
├── reporting/                     # Contexto “Reporting”
│   └── … (consultas OLAP, exportação CSV/PDF)
│
└── build-scripts/                 # CI, Dockerfiles, Compose, Helm charts
```

Ou seja, temos:

| Pasta             | Para que serve                         | Ganho concreto                         |
| ----------------- | -------------------------------------- | -------------------------------------- |
| `domain/`         | Regras de negócio puras, sem framework | Testes rápidos e independentes         |
| `application/`    | Orquestra entidades, publica eventos   | Fluxo de caso de uso bem localizado    |
| `infrastructure/` | Detalhes técnicos (JPA, REST, Kafka)   | Troca de tecnologia sem afetar regras  |
| `config/`         | Beans e wiring Spring do contexto      | Evita “classe de configuração gigante” |


E para que seja feita a integração entre esses contextos (domínios), teremos:

* **Eventos de domínio** (ex.: `TaskAssignedEvent`) publicados em broker (`Kafka`, `RabbitMQ` ou outbox): elimina dependência direta entre módulos.
* **APIs REST** só quando realmente necessárias; cada módulo define seus próprios DTOs e nunca exporta entidades!

A partir disso, caso a solução vá para o lado de microserviços separados, basta extrair cada pasta-módulo para um repositório próprio ou manter a raiz apenas como orquestração (Docker Compose, Helm, Terraform). O princípio continua: **dividir por subdomínio antes de dividir por tecnologia!**.

<!-- #### 1.3.3. **E como isso apareceria no código?**

```java
// Task aggregate (Task Management)
public class Task {
    private TaskId id;
    private Title title;
    private Priority priority;
    private UserId assignee;        // só o identificador, não a entidade User
    private TaskStatus status;

    public DomainEvent assignTo(UserId newAssignee) {
        this.assignee = newAssignee;
        return new TaskAssignedEvent(this.id, newAssignee);
    }
}

// Event handler (Notification)
public class TaskAssignedListener {
    @TransactionalEventListener
    public void on(TaskAssignedEvent event) {
        var pref = preferenceRepo.findByUserId(event.assignee());
        notifier.send(pref.channel(), "Você recebeu uma tarefa!");
    }
}
```

* **Claridade de limite** – `Task` não conhece detalhes de notificação.
* **Baixo acoplamento** – mudar o canal de alerta não obriga *build* de Task Management.
* **Testabilidade** – cada contexto tem suíte própria de testes unitários e de contrato. -->

#### 1.3.3. **Benefícios percebidos em produção**

Dentre os benefícios, temos

| Antes                                              | Depois                                                   |
| -------------------------------------------------- | -------------------------------------------------------- |
| Deploy monolítico travava times                    | Times independentes versionam contextos sem conflito     |
| Correções em notificação derrubavam login          | Falha em Notification não afeta autenticação             |
| Relatórios lentos competiam com escrita de tarefas | Reporting gera *snapshots* assíncronos sem bloquear CRUD |

#### 1.3.4. **E a moral da história...**

É que quando ignoramos o **domínio**, o código cresce como um emaranhado único, onde qualquer mudança puxa um fio que pode rasgar o resto. Ao **modelar subdomínios**, damos nomes claros aos problemas, isolamos regras de negócio específicas e criamos um caminho natural para escalar o sistema e o time.

### 1.4. O DDD como base para Microsserviços

Com a explicação acima deve ter ficado claro: o DDD é frequentemente citado como uma **abordagem natural para a decomposição de microsserviços**, pois ambos compartilham o mesmo objetivo: **quebrar sistemas grandes e complexos em unidades menores e coesas**.

* Microsserviços bem projetados são, na prática, **bounded contexts autônomos**.
* Subdomínios bem definidos ajudam a **evitar microsserviços arbitrários** (por camada ou por funcionalidade técnica).
* A linguagem ubíqua ajuda a **padronizar a comunicação entre times** e entre serviços.

Por isso, antes de sair quebrando seu monólito em dezenas de serviços, é essencial entender quais são os **limites naturais do seu sistema**, e o DDD é a ferramenta certa para fazer isso com segurança e clareza.

---

## 2. Termos-chave do Domain-Driven Design (DDD)

Agora que compreendemos a motivação do DDD e por que ele se torna essencial ao estruturar aplicações complexas, especialmente em cenários de microsserviços, é hora de conhecermos seus **principais conceitos estruturantes**. Esses conceitos fornecem a base para toda a modelagem orientada ao domínio, e serão indispensáveis nas próximas aulas práticas, quando começarmos a extrair serviços do nosso projeto atual.

### 2.1. Domínio

O **domínio** é o ponto de partida do DDD. Trata-se do **conjunto de regras, atividades e conhecimentos específicos** do problema que a aplicação se propõe a resolver. Em outras palavras, o domínio é o **assunto central** do seu sistema.

> 🧠 **Definição:** O domínio representa o “mundo real” que o software está modelando. Ele inclui os termos, conceitos e regras de negócio relevantes para os usuários e stakeholders da aplicação.

Por exemplo, no caso da nossa aplicação de gerenciamento de tarefas (To-Do List), o domínio inclui conceitos como:

* Tarefa (`Task`)
* Responsável (`Owner`)
* Prioridade (`Priority`)
* Categoria (`Category`)
* Data de vencimento (`DueDate`)
* Conclusão (`Completed`)
* Atribuição e regras de modificação

O DDD nos orienta a **modelar o domínio com riqueza e precisão**, para que o sistema reflita fielmente a lógica do negócio, evitando simplificações técnicas que distorçam ou empobreçam o modelo.

### 2.2. Subdomínio

Em sistemas mais complexos, o domínio principal normalmente se divide em **partes menores com responsabilidades bem definidas**, chamadas **subdomínios**. Essa separação ajuda a organizar melhor o sistema, atribuindo a cada parte um foco específico.

> 🧠 **Definição:** Subdomínios são divisões internas do domínio principal. Cada subdomínio representa uma **área funcional coesa**, com seu próprio conjunto de regras, entidades e operações.

O DDD classifica os subdomínios em três categorias:

| Tipo de Subdomínio       | Descrição                                                                        | Exemplo na Task API                            |
| ------------------------ | -------------------------------------------------------------------------------- | ---------------------------------------------- |
| **Core Domain**          | Parte central e mais valiosa do sistema — o que diferencia o produto no mercado. | Gestão de Tarefas                              |
| **Supporting Subdomain** | Apoia o domínio central, mas não é o diferencial principal.                      | Autenticação, gerenciamento de usuários        |
| **Generic Subdomain**    | Problemas genéricos e resolvidos por bibliotecas ou padrões prontos.             | Envio de e-mails, geração de relatórios, cache |

Essa classificação é **estratégica**: no Core Domain, investimos mais tempo, cuidado e testes, pois é onde a inovação está. Já subdomínios genéricos podem até ser terceirizados ou resolvidos com soluções prontas.

No nosso projeto:

* **Tarefas (Task)** → é o **core domain**: é onde está o diferencial da aplicação;
* **Usuários e autenticação** → são **supporting**: são necessários, mas não o foco principal;
* **Notificações** → são **generic**: podem ser reutilizadas por outros sistemas, inclusive.

### 2.3. Bounded Context (Contexto Delimitado)

Muitos desenvolvedores confundem “domínio” com “módulo”. O DDD nos mostra que **um mesmo domínio pode ter diferentes interpretações, dependendo do contexto**. Para evitar confusões, introduz-se o conceito de **Bounded Context**, talvez o conceito mais fundamental de todos no DDD.

> 🧠 **Definição:** Um *bounded context* é um **limite explícito onde um determinado modelo de domínio é válido, coeso e consistente**. Dentro desse limite, os termos e comportamentos têm significado específico. Fora dele, o mesmo termo pode significar outra coisa.

Por exemplo, pense na palavra **"usuário"**:

* No contexto de autenticação, “usuário” significa um login, senha e conjunto de permissões;
* No contexto de tarefas, “usuário” pode significar um colaborador, com nome, cargo e responsabilidades.

Se usássemos um único modelo `User` para todos os contextos, provavelmente ele teria campos como:

```java
User {
    id;
    login;
    passwordHash;
    email;
    role;
    fullName;
    profilePicture;
    list<Task> tasks;
    ...
}
```

Essa classe acabaria servindo a **vários propósitos ao mesmo tempo**, criando um modelo inconsistente, frágil e de difícil manutenção.

O DDD nos ensina a **delimitar contextos com clareza**. Dentro de cada contexto, usamos os termos com significados específicos, e **controlamos com rigor as interações entre eles**.

Assim, podemos ter:

* No contexto `Auth`: `AuthUser { username, password, role }`
* No contexto `Task`: `TaskUser { name, assignedTasks }`

E as interações entre contextos são feitas **com regras explícitas de tradução**. Por exemplo, através de eventos, APIs, adaptadores ou mapeamentos.

### 2.4. Linguagem Ubíqua

Um dos conceitos mais revolucionários do DDD é a **linguagem ubíqua (ubiquitous language)**.

> 🧠 **Definição:** A linguagem ubíqua é um **vocabulário comum, compartilhado entre desenvolvedores e especialistas de negócio**, que reflete os conceitos do domínio. Essa linguagem deve ser **utilizada no código-fonte, nos testes, na modelagem, na documentação e nas conversas** do time.

Ou seja, não se trata apenas de conversar com o cliente usando termos do negócio, trata-se de **usar esses mesmos termos nas classes, métodos, pacotes, testes e commits do projeto**.

Exemplo no nosso projeto de to-do:

| Conceito de negócio | Implementação correta (linguagem ubíqua) |
| ------------------- | ---------------------------------------- |
| Concluir tarefa     | `taskService.concludeTask(id)`           |
| Data limite         | `dueDate` (e não `expiration`)           |
| Prioridade          | Enum `Priority.HIGH`, `MEDIUM`, `LOW`    |
| Categoria           | Enum `Category.STUDY`, `WORK`, etc.      |

Usar a linguagem ubíqua evita ambiguidades e promove clareza. Se a equipe estiver discutindo “o que acontece quando um responsável conclui uma tarefa atrasada?”, todos devem conseguir **visualizar essa lógica diretamente no código**, com termos familiares ao negócio.

### 2.5. Domínio vs. Modelo de Dados

É importante entender que o modelo de domínio **não é igual** ao modelo de dados (banco de dados relacional).

* O **modelo de domínio** expressa **comportamentos e regras do negócio**, incluindo entidades ricas, invariantes, agregados e políticas.
* O **modelo de dados** é focado em **armazenar informações** de forma eficiente, com tabelas, chaves e normalizações.

O DDD recomenda que o **modelo de domínio seja o centro do design**, e não apenas uma representação da estrutura do banco!

Isso é especialmente importante em microsserviços, onde **cada serviço pode ter seu próprio modelo de dados**, mas precisa manter **coerência com o modelo do seu domínio**.

Por exemplo, quando projetamos um microsserviço de “Task Management”, primeiro modelamos a entidade `Task`, seus estados e eventos (`TaskAssigned`, `TaskCompleted`). Só depois decidimos se isso virará uma tabela `tasks`, um JSON no Mongo ou um documento em DynamoDB. O banco pode mudar sem mexer na lógica de negócio, porque ela vive no modelo de domínio. Assim, cada serviço mantém seu próprio banco do jeito que fizer mais sentido, mas a coerência das regras permanece intacta onde realmente importa: no código que expressa o domínio.

Ou seja, no **modelo de domínio**, a classe agregada `Task` carrega valor semântico e regras:

```java
public class Task {
    private final TaskId id;              // Value Object: UUID + validação
    private Title title;                  // Value Object: ≥ 3 e ≤ 60 caracteres
    private Priority priority;            // Enum: LOW, MEDIUM, HIGH
    private UserId assignee;              // Apenas o identificador do usuário
    private TaskStatus status;            // Enum com regras internas
    private Deadline deadline;            // Value Object calcula atraso
    private List<DomainEvent> events = new ArrayList<>();

    public void complete() {
        if (status != TaskStatus.DONE) {
            status = TaskStatus.DONE;
            events.add(new TaskCompletedEvent(id));
        }
    }
}
```

Note que `Task` guarda **comportamento** (`complete()`), **invariantes** (deadline não nulo) e **tipos ricos** como `Title` e `Deadline`. Já no **modelo de dados**, focado em persistir rápido e responder *queries*, as mesmas informações podem ser decompostas e normalizadas:

```sql
CREATE TABLE tasks (
    task_id         CHAR(36)  PRIMARY KEY,
    title           VARCHAR(60) NOT NULL,
    priority        SMALLINT      NOT NULL,  -- 0=LOW,1=MEDIUM,2=HIGH
    assignee_id     CHAR(36)      NULL,
    status          SMALLINT      NOT NULL,  -- 0=TODO,1=DOING,2=DONE
    deadline_date   DATE          NULL,
    created_at      TIMESTAMP     DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_tasks_assignee_status ON tasks (assignee_id, status);
```

Ou seja: o banco não precisa saber o que é `Deadline` como tipo especializado nem guardar eventos; ele apenas armazena `deadline_date` e um inteiro para `priority`. Se um dia você mover os dados para MongoDB ou acrescentar uma tabela `task_events`, a classe `Task` e suas regras de negócio continuam iguais.

### 2.6. DDD — Estratégia × Tática

No Domain-Driven Design existe uma **camada estratégica** e uma **camada tática**. Entender a diferença evita que o time discuta detalhes de classe antes de saber *onde* cada regra pertence.

| Camada          | Pergunta que responde                                              | Conceitos principais                                                                                                                                                     | Foco                                                  |
| --------------- | ------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------- |
| **Estratégica** | *“Quais partes do negócio existem e como se relacionam?”*          | Domínio, Subdomínio (Core / Supporting / Generic), Bounded Context, Context Map, Linguagem Ubíqua, tipos de relacionamento (ACL, Shared Kernel, Customer-Supplier, etc.) | **Desenhar fronteiras** e alinhar times               |
| **Tática**      | *“Como modelar e implementar as regras dentro de cada fronteira?”* | Entidade, Value Object, Aggregate Root, Domain Event, Domain Service, Repository, Factory                                                                                | **Codificar comportamento** de forma coesa e testável |

> **Regra de bolso:** *Estratégia define limites; tática preenche conteúdo.*

1. **Estratégia**: comece mapeando subdomínios, definindo bounded contexts e documentando as integrações no *Context Map*. Isso orienta decisões de time-split, versionamento e, no futuro, microsserviços.
2. **Tática**: dentro de cada contexto, crie um modelo rico: entidades com identidade, objetos-valor imutáveis, agregados que garantem invariantes e eventos que propagam fatos sem acoplamento.

Com as fronteiras claras, os padrões táticos ganham sentido: um `Task` só precisa conhecer `UserId` (e não a entidade `User`) porque **estrategicamente** decidimos isolar `Task Management` de `Identity & Access`.

### 2.7. Elementos táticos do DDD

| Conceito           | O que é                                                                | Sinal de que precisa dele                                               |
| ------------------ | ---------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| **Entidade**       | Objeto com identidade imutável (ID) e ciclo de vida longo              | Precisa rastrear a mesma coisa ao longo do tempo                        |
| **Value Object**   | Objeto imutável, comparado por valor, sem identidade própria           | Representa uma medida ou descrição - Ex.: `Money`, `Title`              |
| **Aggregate Root** | Entidade que **encapsula** um cluster de objetos e garante invariantes | Vários objetos mudam juntos — ex.: `Task` controla `assignee`, `status` |
| **Domain Event**   | Mensagem que descreve algo **já ocorrido** no domínio                  | Outros contextos ou camadas precisam reagir a um fato                   |

#### Exemplo na *Task API*

```java
// Value Object: título
public record Title(String value) {
    public Title {
        if (value == null || value.length() < 3 || value.length() > 60)
            throw new IllegalArgumentException("Título inválido");
    }
}

// Aggregate Root + Entidade
public class Task {
    private final TaskId id;           // identidade imutável
    private Title title;               // VO imutável
    private Priority priority;         // Enum é VO simples
    private UserId assignee;           // apenas o identificador
    private TaskStatus status;
    private Deadline deadline;         // VO
    private final List<DomainEvent> events = new ArrayList<>();

    /** Regra de negócio: concluir tarefa */
    public void complete() {
        if (status == TaskStatus.DONE) return;
        status = TaskStatus.DONE;
        events.add(new TaskCompletedEvent(id));
    }
}

// Domain Event
public record TaskCompletedEvent(TaskId taskId) implements DomainEvent {}
```

* **Entidade**: `Task` possui ciclo de vida, seu `id` não muda.
* **Value Objects**: `Title` evita títulos vazios; como é imutável, não existe “estado quebrado”.
* **Aggregate Root**: somente métodos de `Task` podem modificar seu próprio estado, garantindo que “tarefa concluída não pode voltar ao status anterior”.
* **Domain Event**: `TaskCompletedEvent` é disparado após mudança de estado, permitindo que **Notification** ou **Reporting** reajam sem acoplamento.

### 2.8. Elementos **estratégicos** do DDD

A parte estratégica do Domain-Driven Design decide **onde** cada conjunto de regras vive, **quem** é dono de cada parte do negócio e **como** essas partes conversam. Os elementos abaixo formam o alicerce que antecede qualquer diagrama de classes ou script SQL.

| Elemento estratégico                                                                                 | Para que serve                                                                                        | Resultado prático no projeto                                                                                      |
| ---------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| **Domínio**                                                                                          | Define *qual problema principal* o software resolve.                                                  | Mantém o time focado no propósito real do sistema.                                                                |
| **Subdomínio**                                                                                       | Fragmenta o domínio em áreas funcionais coesas. Classifica em **Core**, **Supporting** e **Generic**. | Orienta investimento de esforço: core recebe mais atenção; genéricos podem virar bibliotecas ou serviços prontos. |
| **Bounded Context**                                                                                  | Delimita o espaço onde um modelo de domínio é consistente e a linguagem tem significado único.        | Evita que “Usuário”, “Pedido” ou “Carga” mudem de sentido conforme atravessam o código.                           |
| **Context Map**                                                                                      | Mostra *como* e *por onde* os bounded contexts trocam dados (REST, eventos, ACL, Shared Kernel…).     | Guias claros de integração; reduz surpresa quando um contexto evolui.                                             |
| **Tipos de Relacionamento** (ACL, Shared Kernel, Customer–Supplier, Conformist, Published Language…) | Padronizam as regras de dependência entre contextos.                                                  | Deixa explícito quem pode mudar contrato, quem é consumidor passivo e onde é preciso camada de tradução.          |
| **Linguagem Ubíqua**                                                                                 | Vocabulário comum entre especialistas e devs, usado em código, testes e conversa diária.              | Código se torna leitura natural para qualquer pessoa do negócio; reduz mal-entendidos.                            |
| **Visão Organizacional** (Times por Contexto)                                                        | Atribui ownership a equipes alinhadas aos bounded contexts.                                           | Facilitam deploys independentes, pipelines separados e metas claras por domínio.                                  |

#### Como aplicar no dia a dia

1. **Mapeie o domínio com o negócio** (event storming, entrevistas).
2. **Quebre em subdomínios** e classifique: *core*, *supporting*, *generic*.
3. **Desenhe bounded contexts** para cada subdomínio ou grupo de regras que precise de modelo próprio.
4. **Documente o Context Map** com os padrões de relacionamento escolhidos.
5. **Aloque times** ou responsáveis por contexto, não por camada técnica.
6. **Refatore a linguagem**: renomeie classes, endpoints, testes para refletir exatamente o vocabulário do contexto.

> **Resumo:** se a tática preenche o código com entidades ricas e agregados, a estratégia garante que esses modelos surjam **nos lugares certos**, conversando pelos **canais certos**, na **língua certa**.

Vamos abordar alguns desses principais conceitos mais à fundo, porque entendê-los pode nos auxiliar inclusive em contextos em que não estamos aplicando diretamente o DDD!

---

## 3. Linguagem Ubíqua (Ubiquitous Language)

A **linguagem ubíqua**, como vimos acima, é uma das ideias centrais e mais poderosas do Domain-Driven Design. Introduzida por Eric Evans como uma estratégia para integrar as visões dos especialistas de negócio e dos desenvolvedores, ela atua como um elo entre o mundo real (o problema) e o código-fonte (a solução). Mais do que um artifício retórico, ela é um **instrumento técnico e estratégico** que molda a própria estrutura do software.

### 3.1. Um problema comum: nomes genéricos, código obscuro

Em muitos sistemas legados, é comum encontrar nomes de classes e métodos como:

```java
DataManager, ServiceImpl, ProcessData, Handler, doStuff()
```

Esses nomes não revelam nada sobre o que o sistema faz. Eles são **genéricos, imprecisos e não têm valor semântico**. Pior ainda: muitas vezes a equipe usa termos diferentes para se referir à mesma coisa, o que gera confusão tanto na comunicação entre pessoas quanto na leitura do código.

No DDD, esse cenário é combatido de forma ativa. O objetivo é que os desenvolvedores **falem com o especialista de negócio usando os mesmos termos que aparecem no código**, e que o especialista possa ler trechos do código e entendê-los, mesmo sem formação técnica.

### 3.2. A linguagem ubíqua como ferramenta de modelagem

A construção da linguagem ubíqua **não é tarefa exclusiva dos desenvolvedores**. Ela deve surgir da **colaboração constante com os especialistas do domínio**, que são aquelas pessoas que realmente conhecem o problema que está sendo resolvido (gerentes, usuários, operadores, analistas de negócio, etc.).

Durante sessões de levantamento de requisitos, refinamento ou *event storming*, o time deve anotar os **termos que surgem com frequência**, como:

* “Uma tarefa pode ser atribuída a um responsável”
* “A data limite precisa ser ajustada caso o projeto atrase”
* “Um usuário só pode editar tarefas em aberto”
* “Há categorias fixas: Estudo, Trabalho, Lazer, etc.”

Cada um desses termos pode, e deve, se refletir diretamente em estruturas do código:

| Termo do negócio    | No código                                  |
| ------------------- | ------------------------------------------ |
| Tarefa              | `Task`                                     |
| Responsável         | `Owner`, `assignedTo`, `User`              |
| Data limite         | `dueDate`                                  |
| Concluída           | `completed`, `isDone()`                    |
| Categoria           | `Category` (enum)                          |
| Atrasada            | `isOverdue()`                              |
| Projeto             | `Project`                                  |
| Permissão de edição | `canEdit()` ou regra de domínio no serviço |

Isso transforma o código-fonte em **uma extensão natural da linguagem de negócio**.

### 3.3. Exemplo no nosso projeto: Task API

Vamos ilustrar com um exemplo real do nosso projeto de API de tarefas.

#### Cenário:

> "Usuários podem criar tarefas, que possuem título, descrição, prioridade, categoria e data limite. Tarefas podem ser marcadas como concluídas. Só o dono da tarefa pode editá-la."

#### Código com linguagem genérica e técnica:

```java
public class DataEntity {
    private String field1;
    private String field2;
    private boolean flag;
}
```

#### Código com linguagem ubíqua:

```java
public class Task {
    private String title;
    private String description;
    private Priority priority;
    private Category category;
    private LocalDateTime dueDate;
    private boolean completed;
    private User owner;

    public boolean isOverdue() {
        return !completed && LocalDateTime.now().isAfter(dueDate);
    }

    public boolean canBeEditedBy(User user) {
        return this.owner.equals(user) && !this.completed;
    }
}
```

Observe como os **termos do domínio estão diretamente refletidos no código**, e com métodos que **expressam regras de negócio**, não apenas acessos a dados. Essa forma de codificação é mais expressiva, fácil de ler e de manter.

Aqui há uma ligação direta com os conceitos que vimos sobre Clean Code! Quando você usa **linguagem ubíqua** e deixa as regras de negócio explícitas dentro das classes de domínio, já está aplicando vários princípios de *Clean Code*:

* **Nomes que revelam intenção**: `Task`, `isOverdue()` e `canBeEditedBy()` dizem exatamente o que fazem, evitando comentários supérfluos.
* **Baixo acoplamento e alta coesão**: a classe `Task` concentra apenas o que diz respeito a uma tarefa; ela não dispara e-mail nem consulta banco, mantendo um único motivo para mudar.
* **Código que conta uma história**: ao ler o método `canBeEditedBy(user)`, qualquer pessoa entende a regra de permissão sem precisar vasculhar controllers ou services.

Ou seja, modelar o domínio e escolher bons nomes faz o código limpo surgir quase como consequência: funções curtas, sem condicionais espalhadas, com objetos ricos que se validam sozinhos. Isso simplifica testes, refatorações e onboarding de novos devs, os mesmos objetivos defendidos pelo *Clean Code*!

### 3.4. Linguagem ubíqua como contrato entre pessoas e código

A adoção da linguagem ubíqua também **reduz o atrito na comunicação entre desenvolvedores e especialistas**. Por exemplo:

* O analista de negócios diz: “Se a tarefa estiver atrasada, ela deve aparecer em vermelho”.
* O desenvolvedor diz: “Ok, vou usar o `isOverdue()` do objeto `Task` para aplicar essa lógica no front-end”.

Esse tipo de troca se torna natural, porque ambos estão falando **o mesmo idioma** e o código vira uma **representação exata do modelo mental** de todos os envolvidos no projeto.

Aqui no nosso contexto as coisas são bastante simples, mas a força da **linguagem ubíqua** aparece de verdade quando o domínio tem **regras intrincadas**, como no clássico exemplo de **logística de contêineres** apresentado por Eric Evans.

Nesse sistema, cada *carga* navega por uma *rota* (Itinerary) composta por vários *trechos* (Legs) a bordo de *viagens* (Voyages).  Durante o trajeto surgem *eventos de manuseio* (Handling Events) - embarque, desembarque, armazenagem, inspeção alfandegária - que podem alterar o *status de entrega* (Delivery Status). Analistas e operadores portuários falam exatamente nesses termos; ao adotar o mesmo vocabulário no código, desenvolvedores evitam traduções mentais e preservam a lógica de negócios com precisão.

```java
// Modelo de domínio simplificado do livro de Evans
public class Cargo {
    private TrackingId trackingId;           // VO: valida formato — ex. "ABC1234567"
    private RouteSpecification spec;         // origem, destino, deadline
    private Itinerary itinerary;             // lista de Legs já planejados
    private Delivery delivery;               // estado atual calculado a cada evento

    /** Reavalia o status quando um evento chega do sistema portuário */
    public void deriveDeliveryProgress(HandlingEvent lastEvent) {
        this.delivery = Delivery.derivedFrom(itinerary, lastEvent, spec);
    }

    /** Regra de negócio: rota ainda atende à especificação? */
    public boolean isMisrouted() {
        return !itinerary.satisfies(spec);
    }
}
```

Quando o **especialista** diz: “Essa carga está *misrouted* porque desembarcou em Singapura em vez de Tóquio”, o **desenvolvedor** responde: “O método `isMisrouted()` já detectou isso e disparou um alerta de replanejamento”. Ambos utilizam os mesmos substantivos e verbos: *carga*, *evento de manuseio*, *rota*, *misrouted*, criando um **contrato vivo** entre conversas de negócio, testes automatizados e código de produção. Em domínios complexos como logística, saúde ou finanças, essa coincidência de linguagem é o que impede regras elaboradas de virarem um emaranhado de *ifs* dispersos, garantindo que as mudanças continuem **compreensíveis, rastreáveis e seguras** ao longo do tempo.

### 3.5. Práticas para adotar a linguagem ubíqua

Dentre as práticas para adotarmos a linguagem ubíqua, podemos elencar

| Prática                                             | Descrição                                                                                            |
| --------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| **Conversar com especialistas**                     | Evite criar nomes por conta própria — escute como o cliente fala                                     |
| **Refletir termos reais no código**                 | Use nomes que façam sentido no domínio, não nomes técnicos genéricos                                 |
| **Evitar siglas e jargões técnicos desnecessários** | Prefira `deadline` a `dt_limite`, `owner` a `usr_fk`                                                 |
| **Evitar traduções forçadas**                       | Se o cliente fala “task”, não insista em “tarefa” no código só por patriotismo                       |
| **Padronizar termos em todo o sistema**             | Use sempre os mesmos nomes para os mesmos conceitos, em DTOs, entidades, eventos, testes e endpoints |
| **Refatorar nomes com frequência**                  | À medida que o entendimento evolui, ajuste os nomes para manter o alinhamento conceitual             |

### 3.6. Impacto nos testes, commits e documentação

* **Testes:** nomes de testes devem refletir casos de uso do negócio
  Ex: `shouldMarkTaskAsCompleted_WhenUserIsOwnerAndTaskIsOpen()`

* **Commits:** mensagens devem refletir mudanças no domínio
  Ex: `feat(task): adicionar verificação de atraso baseado em dueDate`

* **Documentação:** usar o mesmo vocabulário nas histórias de usuário, nos casos de uso e no código

Em resumo: a linguagem ubíqua é **mais do que um padrão de nomeação**: ela é o **coração da comunicação entre o negócio e o código**. Ao adotá-la com disciplina, garantimos que todos os envolvidos no projeto falem a mesma língua e compartilhem o mesmo modelo mental.

## 3.7. Event Storming: o Domínio em Colaboração

Uma das formas mais eficazes e colaborativas de construir a **Linguagem Ubíqua** e identificar os **Subdomínios** e **Bounded Contexts** é através de uma técnica chamada **Event Storming**. Criado por Alberto Brandolini, o Event Storming é uma oficina dinâmica e visual que reúne especialistas de negócio e desenvolvedores para mapear o fluxo de eventos em um sistema, revelando o coração do domínio.

### 3.7.1. O que é Event Storming?

Event Storming é uma abordagem facilitada para explorar domínios de negócio complexos, focando nos **Eventos de Domínio**. A ideia central é que todos os participantes – desde a área de negócio até a equipe técnica – se reúnem em uma sala grande (ou usam uma ferramenta online colaborativa), munidos de muitos *post-its* de cores diferentes.

A oficina gira em torno de identificar e sequenciar os eventos que acontecem no sistema. Um **Evento de Domínio** é algo significativo que **já aconteceu** no negócio e que os especialistas reconhecem como relevante. Eles são expressos no passado, como "Tarefa Criada", "Usuário Autenticado" ou "Pagamento Processado".

#### Como funciona:

1.  **Linha do Tempo de Eventos:** Os participantes começam colocando **eventos de domínio (laranja)** em ordem cronológica, do passado para o futuro. "O que acontece depois disso?" é a pergunta que impulsiona a discussão.
2.  **Comandos e Agregados:** Para cada evento, busca-se o **comando (azul)** que o disparou (ex: "Criar Tarefa" leva a "Tarefa Criada") e o **agregado (amarelo)** responsável por executar esse comando (ex: a `Task` é o agregado que "cria a tarefa").
3.  **Sistemas Externos e Pessoas:** Identificam-se sistemas externos (rosa) que interagem com o domínio e os atores (bonequinhos) que executam os comandos.
4.  **Políticas e Regras:** Discussões sobre "o que dispara o quê" e "por que isso acontece" revelam políticas e regras de negócio (verde).
5.  **Hotspots de Discussão:** Pontos de incerteza, conflito ou complexidade são marcados como "hotspots" (vermelho) para serem discutidos e aprofundados.

### 3.7.2. Por que usar Event Storming?

* **Visão Compartilhada:** Garante que todos os envolvidos no projeto (negócio e tecnologia) construam uma compreensão comum do sistema, utilizando a mesma **Linguagem Ubíqua**.
* **Identificação de Limites:** À medida que os eventos são mapeados, naturalmente surgem agrupamentos de eventos e comandos que pertencem a uma mesma área de responsabilidade. Esses agrupamentos são fortes candidatos a **Subdomínios** e **Bounded Contexts**.
* **Foco no Comportamento:** Em vez de focar em dados ou telas, o Event Storming concentra-se nos **comportamentos** do sistema e nas **decisões de negócio**, o que é crucial para um design orientado ao domínio.
* **Descoberta de Invariantes:** Ao discutir os fluxos e as regras, as invariantes do sistema (regras que nunca devem ser violadas) vêm à tona, ajudando a modelar os Agregados de forma mais robusta.
* **Engajamento e Colaboração:** É uma atividade divertida e visual que incentiva a participação ativa, quebrando barreiras entre as áreas.

### 3.7.3. Exemplo na Task API com Event Storming

Vamos imaginar uma pequena sessão de Event Storming para o nosso projeto de gerenciamento de tarefas.

**Eventos identificados (post-its laranja):**
* "Usuário Criado"
* "Usuário Autenticado"
* "Tarefa Criada"
* "Tarefa Atribuída"
* "Tarefa Concluída"
* "Notificação Enviada"
* "Relatório de Tarefas Gerado"

**Agrupando e descobrindo Contextos:**

À medida que os eventos são dispostos, percebemos que:

* "Usuário Criado" e "Usuário Autenticado" naturalmente se agrupam em um contexto de **Identidade & Acesso**.
* "Tarefa Criada", "Tarefa Atribuída" e "Tarefa Concluída" formam o núcleo do contexto de **Gerenciamento de Tarefas**.
* "Notificação Enviada" aponta para um contexto de **Notificação**.
* "Relatório de Tarefas Gerado" para um contexto de **Relatórios**.

**Comandos e Políticas:**

* Para "Tarefa Criada", o comando é "Criar Nova Tarefa", executado pelo `Task` (Agregado), disparado por um "Usuário" (Ator).
* Para "Notificação Enviada", a política pode ser "Quando a Tarefa é Atribuída, Enviar Notificação ao Atribuído".

Com o Event Storming, essas divisões e as interações entre elas (quem dispara o quê) se tornam visíveis e claras para toda a equipe, servindo como um mapa visual para a arquitetura do software.

Para se aprofundar um pouco mais no Event Storming, acesse o link a seguir: [Event Storming — Guia básico](https://medium.com/@jonesroberto/event-storming-guia-básico-216498f5dd2d)

Na próxima seção, veremos **como identificar subdomínios na prática**, usando heurísticas que nos ajudam a decidir **onde estão os limites naturais** de cada parte do sistema, base para uma futura divisão em microsserviços.

---

## 4. Heurísticas para Identificar Subdomínios na Prática

Até aqui, vimos que um **subdomínio** é uma **área funcional coesa** dentro do domínio maior do sistema, e que o DDD recomenda que cada subdomínio seja implementado dentro de um **bounded context** próprio. Essa divisão permite que diferentes partes da aplicação sejam desenvolvidas, testadas e evoluídas de forma **independente, consistente e alinhada com as regras de negócio**.

No entanto, surge uma pergunta natural:

> “Como saber o que é um subdomínio e onde ele começa e termina?”

Essa é, talvez, a pergunta mais difícil — e mais importante — ao adotar DDD na prática. Por isso, nesta seção, vamos apresentar **heurísticas** para ajudar você e sua equipe a **identificarem subdomínios com base em critérios objetivos e observáveis**.

### 4.1. Heurísticas para descobrir subdomínios

A seguir, são apresentados um conjunto de perguntas, ou heurísticas, que você pode aplicar **ao analisar um módulo, classe ou funcionalidade**. Quanto mais dessas características estiverem presentes, maior a chance de você estar diante de um **subdomínio candidato**.

#### ✅ Heurística 1: Vocabulário distinto?

* Esse módulo usa termos próprios que não aparecem no restante do sistema?
* Os significados dos termos mudam em contextos diferentes?

> Exemplo: "usuário" pode significar login e senha no `Auth`, mas representar um colaborador com tarefas no `Task`.

#### ✅ Heurística 2: Regras de negócio próprias?

* Há validações, cálculos ou fluxos de decisão que só fazem sentido nesse pedaço do sistema?
* Essas regras são discutidas com especialistas do negócio?

> Exemplo: no módulo `Task`, não se pode editar uma tarefa já concluída.

#### ✅ Heurística 3: Ciclo de vida independente?

* Os dados desse módulo têm um ciclo de criação, atualização e exclusão próprios?
* Os fluxos desse módulo não precisam esperar por decisões externas?

> Exemplo: tarefas podem ser criadas, concluídas e apagadas sem relação direta com o fluxo de login do usuário.

#### ✅ Heurística 4: Equipe ou papel diferente no negócio?

* Existe uma equipe (real ou imaginária) que seria responsável apenas por essa parte do sistema?
* O especialista de negócio que define as regras dessa área é diferente de outras partes?

> Exemplo: quem cuida de permissões e autenticação pode ser o time de infraestrutura; quem define o fluxo de tarefas é o time de operações.

#### ✅ Heurística 5: Frequência de mudança distinta?

* Esse módulo muda com frequência diferente de outras partes?
* As regras desse módulo tendem a evoluir mais rápido (ou mais devagar) do que o restante do sistema?

> Exemplo: regras de tarefas (prioridades, categorias, prazos) mudam com frequência conforme o negócio muda. Regras de autenticação mudam mais raramente.

#### ✅ Heurística 6: Escalabilidade ou desempenho específico?

* Esse módulo tem características de carga, latência ou volume muito distintas?
* Há justificativas técnicas para separá-lo, como escalabilidade ou resiliência?

> Exemplo: o módulo de notificações (e-mails ou SMS) pode precisar de filas e paralelismo, enquanto o cadastro de usuários pode ser síncrono.

### 4.2. Aplicando heurísticas ao nosso projeto: Task API

Vamos aplicar essas heurísticas para analisar o que já temos em nosso projeto atual. Considere os três pacotes principais:

#### 📦 `auth` – autenticação e segurança

* **Vocabulário próprio**: `token`, `JWT`, `credenciais`, `autenticação`
* **Regras específicas**: login com senha, expiração de token, geração de JWT
* **Frequência de mudança**: baixa — muda apenas se a estratégia de segurança mudar
* **Equipe separada?** Sim, normalmente responsabilidade de times de infraestrutura

👉 **Subdomínio identificado**: `Authentication` (supporting subdomain)

#### 📦 `user` – dados do usuário

* **Vocabulário**: `nome`, `e-mail`, `perfil`, `papel`
* **Regras**: associação de tarefas, permissões
* **Frequência de mudança**: média — novo perfil, papéis, e-mail etc.
* **Equipe dedicada?** Sim, pode haver um time de RH, cadastro ou administração

👉 **Subdomínio identificado**: `User Management` (supporting subdomain)

#### 📦 `task` – gestão de tarefas

* **Vocabulário**: `título`, `prioridade`, `categoria`, `vencimento`, `concluída`
* **Regras complexas**: não pode editar tarefa concluída, não pode excluir tarefa alheia
* **Frequência de mudança**: alta — regras e categorias podem mudar com o negócio
* **Core business?** Sim — é o diferencial da aplicação

👉 **Subdomínio identificado**: `Task Management` (core domain)

### 4.3. Mapeando os subdomínios no código

A partir dessas observações, poderíamos começar a organizar o código em pacotes ou módulos separados, como:

```
src/
├── auth/          → JWT, autenticação, login
├── user/          → cadastro, papéis, permissões
├── task/          → entidades, serviços e controladores relacionados às tarefas
```

No futuro, essa separação pode ser evoluída para:

* Módulos Maven ou Gradle separados
* Containers Docker distintos
* Microsserviços independentes

Mas mesmo em um monólito, **essa separação lógica já ajuda muito na clareza, manutenção e escalabilidade** do projeto.

Na próxima seção, vamos estudar **como representar visualmente os relacionamentos entre subdomínios**, usando um conceito chamado **Context Map** — ferramenta para entender como os módulos se comunicam e para planejar a extração de microsserviços.

---

## 5. Mapeamento de Contextos e Comunicação (Context Map)

Aprendemos a identificar **subdomínios** dentro de uma aplicação real. Agora, chegou o momento de compreender como **esses subdomínios se relacionam entre si**. Para isso, o Domain-Driven Design propõe uma ferramenta visual e conceitual chamada **Context Map** (Mapa de Contextos).

O Context Map é fundamental para arquiteturas baseadas em microsserviços, pois permite **entender como os diferentes bounded contexts trocam informações entre si**. É através dele que decidimos onde aplicar integrações via API REST, eventos assíncronos, filas, ou até bancos de dados compartilhados (quando inevitável). Em projetos monolíticos, o mapa também orienta **módulos internos e fronteiras de código**.

### 5.1. O que é um Context Map?

> 🧠 **Definição:** O Context Map (Mapa de Contextos) é um **diagrama que representa os diferentes bounded contexts do sistema** e **como eles se relacionam entre si**. Ele explicita os limites, os contratos e os mecanismos de integração entre contextos distintos.

Enquanto o **bounded context** define o limite interno de consistência de um modelo de domínio, o **context map define o que existe entre esses limites**: como os modelos se comunicam, como os times colaboram e como os dados são transferidos entre fronteiras.

### 5.2. Exemplo simples de Context Map

Para o nosso projeto de To-Do List, que já possui separações entre os subdomínios `Auth`, `User` e `Task`, o **context map inicial** poderia ser representado assim:

```
+-------------------+          REST          +-------------------+
|    Auth Context   |  <-------------------  |   Task Context     |
| - Login           |                       | - Criação de Tarefa |
| - Token JWT       |                       | - Conclusão         |
+-------------------+                       +-------------------+
         |                                           ^
         |                                           |
         |                         Ownership (ref.) |
         |                                           |
         v                                           |
+-------------------+           API / Shared Kernel |
|   User Context    |  <--------------------------->|
| - Cadastro        |                               |
| - Perfis / Roles  |                               |
+-------------------+                               |
                                                    |
    ←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←
           Comunicação Assíncrona (eventual)
```

Esse diagrama mostra:

* `Auth` fornece tokens JWT consumidos por `Task`;
* `Task` depende de `User` para resolver o dono da tarefa (`owner`);
* `User` e `Task` podem compartilhar parte do modelo (ex: ID e nome do usuário) — configurando um **Shared Kernel**;
* Em um cenário mais avançado, `Task` poderia se manter atualizado sobre mudanças no usuário via **eventos assíncronos** disparados por `User`.

### 5.3. Tipos de Relacionamento no Context Map

O DDD descreve vários **padrões de relacionamento entre contextos**. Abaixo, destacamos os principais, aplicáveis tanto a microsserviços quanto a monólitos modulados.

| Tipo de Relacionamento         | Descrição                                                                                                                   | Exemplo prático                                                                        |
| ------------------------------ | --------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| **Shared Kernel**              | Dois contextos compartilham um pequeno modelo comum e versionado em conjunto.                                               | `User` e `Task` compartilham o identificador do usuário                                |
| **Customer / Supplier**        | Um contexto depende diretamente de outro e consome seus dados/modelo.                                                       | `Task` é consumidor de `Auth` (precisa validar tokens)                                 |
| **Conformist**                 | Um contexto aceita o modelo do outro sem adaptar. É um cliente passivo.                                                     | `Task` usa diretamente o DTO de `User`, sem abstrair                                   |
| **Anticorruption Layer (ACL)** | Um contexto consumidor cria um tradutor que adapta o modelo do fornecedor para o seu próprio modelo interno.                | `Task` consome `User` via API, mas converte para `TaskUser`                            |
| **Open Host Service**          | Um contexto fornece uma API pública bem definida que outros podem consumir.                                                 | `Auth` expõe `/login` e `/token` para todos os serviços                                |
| **Published Language**         | Os contextos comunicam-se usando um contrato público e bem documentado, como eventos assíncronos ou schemas compartilhados. | `User` publica eventos como `UserCreated`, `UserUpdated` em uma fila Kafka ou RabbitMQ |

Esses padrões ajudam a **evitar acoplamento excessivo**, **clarear responsabilidades** e **orientar decisões técnicas** sobre como os serviços (ou módulos) devem se integrar.

### 5.4. Regras de ouro para relacionamentos saudáveis

* **Evite depender diretamente do modelo interno de outro contexto** → Prefira contratos públicos ou DTOs específicos.
* **Quando precisar se integrar, defina explicitamente o tipo de relacionamento** → Customer/Supplier? ACL? Shared Kernel?
* **Estabeleça fronteiras claras e documentadas** → APIs, eventos, modelos compartilhados devem ser explícitos.
* **Não use o mesmo banco de dados entre contextos diferentes** → Isso quebra a independência dos contextos e gera acoplamento acidental.
* **Prefira integração assíncrona quando possível** → Melhora a escalabilidade e reduz dependência em tempo de execução.

### 5.5. Context Map em projetos monolíticos

Mesmo em projetos monolíticos (como o nosso até agora), o Context Map pode ser aplicado:

* **Separando pacotes ou módulos internos** com base nos bounded contexts;
* Criando **interfaces explícitas** entre os módulos, mesmo que sejam métodos Java;
* Mantendo **contratos estáveis** entre as camadas (DTOs, interfaces, serviços);
* Facilitando, no futuro, a extração de microsserviços — já que os limites já estão definidos.

### 5.6. Ferramentas para desenhar Context Maps

Você pode representar context maps com ferramentas simples como:

* Diagramas de caixa e seta no **draw\.io**, **Lucidchart**, **Miro** ou **Excalidraw**;
* Anotações no código com pacotes `auth.*`, `user.*`, `task.*`;
* Mapas de dependência usando **arquitetura hexagonal** ou **Clean Architecture**, por exemplo;
* Representações visuais em **event storming** (usando sticky notes físicos ou digitais).

### 5.7. O Context Map como bússola estratégica

Em equipes ágeis, o Context Map também é útil para:

* **Distribuir responsabilidades entre times** (cada time pode assumir um contexto);
* **Planejar releases ou refatorações** (extração gradual de microsserviços);
* **Negociar contratos entre equipes** (APIs, eventos, permissões);
* **Gerenciar dependências** e evitar cascatas de mudanças entre módulos.

Por isso, ele não é apenas uma ferramenta técnica — mas também **organizacional e estratégica**.

Agora veremos como todos esses conceitos se relacionam com **microsserviços na prática** — e por que tentar “fazer microsserviços sem DDD” pode nos levar a sistemas frágeis, redundantes e difíceis de manter.

---

## 6. A Conexão entre DDD e Microsserviços

Ao longo desta aula, aprendemos que o Domain-Driven Design nos ajuda a entender, dividir e expressar sistemas complexos a partir de seus **conceitos de negócio**. Agora, vamos compreender por que **DDD é uma base sólida para arquiteturas de microsserviços** — e como ele nos guia na construção de serviços pequenos, autônomos, bem delimitados e orientados a domínio.

> ⚠️ Antes de pensar em extrair microsserviços, precisamos **identificar contextos delimitados e subdomínios coesos**. Sem isso, apenas fragmentamos um sistema de maneira arbitrária — o que tende a aumentar a complexidade, e não resolvê-la.

### 6.1. Por que microsserviços precisam de DDD?

Microsserviços prometem várias vantagens:

* Escalabilidade independente
* Entregas mais rápidas por times pequenos
* Maior resiliência a falhas
* Flexibilidade tecnológica

Entretanto, **sem um mapeamento de domínios bem feito**, essas promessas frequentemente se perdem. O que acontece, na prática:

| Sem DDD                                                              | Com DDD                                                           |
| -------------------------------------------------------------------- | ----------------------------------------------------------------- |
| Serviços definidos por conveniência técnica (ex: por CRUD ou tabela) | Serviços definidos por limites de negócio (bounded contexts)      |
| Modelo de dados compartilhado entre serviços                         | Cada serviço tem seu próprio modelo, alinhado ao domínio          |
| Alto acoplamento entre APIs                                          | Comunicação controlada, baseada em contratos claros               |
| Mudança em um serviço exige alterar outros                           | Serviços evoluem com mais autonomia                               |
| Entendimento frágil do que cada serviço faz                          | Cada serviço tem propósito claro e está vinculado a um subdomínio |

### 6.2. Microsserviço ≠ Recurso REST

Muitas equipes caem no erro de criar microsserviços com base em **entidades isoladas**:

* `task-service`: gerencia tarefas
* `user-service`: gerencia usuários
* `notification-service`: envia mensagens

Na prática, esses serviços acabam sendo apenas **CRUDs distribuídos** — sem autonomia real, sem lógica de negócio relevante, e **com alto acoplamento entre si**, pois o fluxo real exige múltiplas chamadas entre eles para fazer qualquer coisa útil.

> 🎯 Microsserviços de verdade representam **casos de uso ou domínios significativos**, e não apenas entidades de banco.

### 6.3. De bounded context → para microsserviço

O mapeamento que fizemos anteriormente com `Auth`, `User` e `Task` é o ponto de partida para uma futura divisão em microsserviços:

| Subdomínio            | Bounded Context | Futuro microsserviço?          |
| --------------------- | --------------- | ------------------------------ |
| Autenticação          | `AuthContext`   | ✅ `auth-service`               |
| Usuários e permissões | `UserContext`   | ✅ `user-service`               |
| Gestão de tarefas     | `TaskContext`   | ✅ `task-service` (core domain) |

Se estruturarmos corretamente esses contextos **desde agora** — mesmo dentro de um monólito —, estaremos **preparando o código, os testes e os contratos** para uma eventual extração futura com menor atrito.

### 6.4. Comunicação entre serviços: como o DDD ajuda

Uma das maiores dificuldades em arquiteturas de microsserviços é **lidar com a comunicação entre eles**. O DDD nos ajuda a:

* **Definir quem fala com quem** (via Context Map)
* **Controlar acoplamento** com padrões como *Anticorruption Layer*, *Open Host Service*, *Published Language*
* **Separar modelos internos dos externos** com DTOs, interfaces, eventos
* **Decidir onde aplicar sincronia (REST) e assíncronia (mensageria)**

Exemplo:

* O `task-service` não precisa saber como funciona o modelo completo de `user-service`. Ele pode expor apenas o `TaskOwner`, um objeto com `id` e `nome`.
* O `user-service` publica eventos como `UserRoleChanged` que o `task-service` consome, atualizando permissões de edição.

### 6.5. Time-to-market e times independentes

Com bounded contexts bem definidos e alinhados a subdomínios estratégicos, é possível **alocar equipes independentes para cada contexto**:

* Time A: cuida do `auth-service`
* Time B: cuida do `user-service`
* Time C: foca no `task-service`

Cada equipe:

* Possui autonomia de deploy
* Assume ownership do domínio
* Evolui regras e funcionalidades específicas sem impactar outros serviços

Esse modelo reduz gargalos, evita conflitos e **promove ownership real**, um dos maiores desafios em projetos complexos.

### 6.6. Quando *não* usar microsserviços?

DDD não exige microsserviços. Ele pode (e deve!) ser aplicado em **monólitos bem estruturados**. Microsserviços são recomendados quando:

* Há escala organizacional (múltiplos times)
* Existem necessidades claras de escalar partes diferentes do sistema separadamente
* O sistema é grande, com muitas regras de negócio que evoluem de forma autônoma

> 💡 Antes de dividir, organize. Primeiro os **bounded contexts**, depois (talvez) os **microsserviços**.

### 6.7. Resumo: DDD como base arquitetural

| DDD Fornece…                                | Microsserviços Precisam de…                |
| ------------------------------------------- | ------------------------------------------ |
| Subdomínios e Contextos Delimitados         | Serviços autônomos e focados em domínio    |
| Contratos claros de integração              | APIs bem definidas, eventos controlados    |
| Modelos expressivos e ubíquos               | Código de negócio compreensível e testável |
| Separação entre modelos internos e externos | Baixo acoplamento entre serviços           |
| Estratégia para crescimento controlado      | Arquitetura escalável e sustentável        |

Na próxima seção, finalizaremos com uma **síntese de boas práticas** e **principais erros a evitar ao aplicar DDD** — especialmente em times iniciantes.

---

## 7. Boas Práticas e Armadilhas Comuns no uso de DDD

Domain-Driven Design é, antes de tudo, uma **abordagem estratégica para enfrentar sistemas complexos**. E como toda abordagem poderosa, ela pode ser **mal utilizada** — o que não apenas neutraliza seus benefícios, mas pode até gerar mais confusão e acoplamento.

Nesta seção, reunimos **boas práticas recomendadas por especialistas e pela literatura**, além de **erros frequentes cometidos por times iniciantes**, com sugestões concretas de como evitá-los.

### 7.1. Boas práticas ao aplicar DDD

| Prática                                                      | Descrição                                                                                                                                          |
| ------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| ✅ **Comece pelo negócio, não pelo código**                   | Converse com especialistas do domínio, observe o vocabulário, entenda o problema real antes de desenhar tabelas ou endpoints.                      |
| ✅ **Adote e cultive a linguagem ubíqua**                     | Use os mesmos termos em reuniões, documentação, testes, código e nome de variáveis. Essa consistência acelera o entendimento e reduz ambiguidades. |
| ✅ **Modele com base nos comportamentos, não só nos dados**   | Entidades não são apenas conjuntos de campos — elas devem encapsular regras, decisões e estados do domínio.                                        |
| ✅ **Separe modelos de domínio de modelos de infraestrutura** | Evite misturar regras de negócio com lógica de banco, frameworks ou serialização.                                                                  |
| ✅ **Identifique e respeite os bounded contexts**             | Cada contexto deve ter sua própria linguagem, modelo e fronteiras técnicas e organizacionais bem definidas.                                        |
| ✅ **Modele casos de uso reais, não apenas operações CRUD**   | Ex: `concluirTarefa()` faz mais sentido do que `updateTaskFlag()`. A intenção deve ser expressa claramente no código.                              |
| ✅ **Escreva testes com linguagem do negócio**                | Testes são ótima forma de reforçar o modelo. Um bom teste revela o que o sistema faz com termos claros e domínios bem definidos.                   |
| ✅ **Comece com um monólito modular**                         | Você não precisa extrair microsserviços no início. Dominar o domínio e separar contextos em módulos é o passo mais importante.                     |
| ✅ **Documente o Context Map**                                | Mesmo que informal, mantenha um diagrama ou descrição dos contextos e seus relacionamentos. Isso guia decisões futuras.                            |

### 7.2. Armadilhas comuns (e como evitá-las)

| Armadilha                                                       | Problema causado                                                                       | Como evitar                                                                                            |
| --------------------------------------------------------------- | -------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| ❌ **Transformar DDD em “camadas técnicas” sem foco no domínio** | Classes como `XService`, `YManager`, `ZUtils` que não representam nenhum conceito real | Comece pela linguagem do negócio. Modele com base em entidades e comportamentos reais                  |
| ❌ **Modelo anêmico (sem comportamento)**                        | Entidades viram apenas estruturas de dados, sem regras ou decisões                     | Encapsule lógica dentro das entidades, com métodos como `isOverdue()`, `canBeEditedBy()` etc.          |
| ❌ **Usar DDD em sistemas simples demais**                       | Gera sobrecarga sem ganho real                                                         | Avalie a complexidade do domínio. Use DDD quando há regras, decisões e variações relevantes no negócio |
| ❌ **Confundir subdomínio com “entidade”**                       | Tentar criar um microsserviço por tabela do banco                                      | Subdomínios representam **funções do negócio**, não objetos do banco                                   |
| ❌ **Compartilhar banco de dados entre contextos**               | Altíssimo acoplamento e quebra de encapsulamento                                       | Use APIs, eventos ou antifraudes (ACLs) para comunicação. Cada contexto tem seu modelo e persistência  |
| ❌ **Adotar eventos sem bounded context claro**                  | Gera uma “eventosfera” caótica, sem controle ou sentido                                | Só publique eventos de negócio com significado e com consumidores bem definidos                        |
| ❌ **Ignorar colaboração com o especialista do domínio**         | O modelo vira uma abstração técnica irrelevante                                        | O domínio é construído em colaboração — mantenha o diálogo constante com os usuários e analistas       |
| ❌ **Microsserviços antes de entender o domínio**                | Só adiciona complexidade, sem autonomia real                                           | Primeiro separe contextos, depois (se necessário) extraia microsserviços                               |

### 7.3. Dica de ouro: *“Entenda primeiro. Modele depois.”*

A principal armadilha do DDD é **tratar como técnica o que é, na verdade, filosofia e estratégia**.

* DDD não é sobre criar `AggregateRoot.java`
* DDD não é sobre usar `@Entity` com `@Service`
* DDD não é sobre separar pastas com nomes bonitos

DDD é **sobre entender profundamente o domínio de negócio**, encontrar os limites naturais de cada parte, **explicitar os contratos entre elas** e garantir que **as decisões corretas sejam expressas no código** — com clareza, com coesão, e com autonomia.

### 7.4. Quando parar de modelar?

É comum ouvir: “mas quando o modelo está bom o suficiente?”

Uma boa regra é:

> **Pare de modelar quando você consegue implementar as funcionalidades mais importantes do sistema de forma expressiva, compreensível e sem gambiarras.**

Se você ainda precisa “dar um jeitinho” para contornar inconsistências, criar validações espalhadas ou reimplementar lógicas em vários lugares... provavelmente ainda falta refinar o modelo ou dividir melhor os contextos.

### 7.5. Conclusão

O Domain-Driven Design, quando bem aplicado, nos ajuda a **desenvolver sistemas mais compreensíveis, sustentáveis e alinhados com a realidade do negócio**. Ele **não é uma bala de prata**, mas é um conjunto de princípios que **evita que a complexidade do sistema cresça de forma descontrolada**.

Mais do que um padrão de arquitetura, o DDD nos convida a sermos **modeladores conscientes**, **desenvolvedores curiosos** e **colaboradores ativos do entendimento do problema**.

Se adotado com propósito, ele transforma não apenas o código — mas a forma como equipes inteiras pensam, falam e constroem software.

---

# 8. Projeto Final da disciplina

Na aula anterior, vocês exploraram mapearam seus **Bounded Contexts** e identificaram potenciais candidatos para se tornarem microsserviços. Agora, o desafio é levar essa teoria para a prática! Vocês vão iniciar a jornada de **transformar um pedaço da sua aplicação em um microsserviço independente**, aplicando diretamente os conceitos estratégicos e táticos do DDD.

Sua equipe deverá escolher **um (1) Bounded Context** do projeto intermediário que desenvolveram para a disciplina. O objetivo é criar um **novo projeto Spring Boot** que represente esse microsserviço isolado.

### Etapas da Atividade:

1.  **Revisão e Delimitação Final do Bounded Context Escolhido:**
    * Revisem o **Bounded Context** que sua equipe selecionou para a extração. Verifiquem se ele faz sentido à luz do que foi apresentado nessa aula.
    * Reforcem a **linguagem ubíqua** específica desse contexto. Quais são os termos, entidades e operações que só fazem sentido aqui?
    * Documentem brevemente a fronteira: o que pertence a este microsserviço e o que não pertence? Quais são as **responsabilidades exclusivas** dele?

2.  **Criação do Novo Projeto do Microsserviço:**
    * Criem um **novo projeto Spring Boot** separado do monólito original. Este será o repositório ou módulo do seu microsserviço.
    * Dê a ele um nome que reflita seu **Bounded Context** (ex: `task-management-service`, `identity-access-service`).

3.  **Migração do Modelo de Domínio e Lógica de Negócio (DDD Tático):**
    * **Migrem** ou **recriem** as classes de domínio relevantes (`Entidades`, `Value Objects`, `Aggregate Roots`) para dentro do novo projeto do microsserviço.
    * Garanta que a **lógica de negócio** associada a essas classes esteja encapsulada dentro delas, seguindo os princípios do DDD tático (comportamento rico, invariantes nos agregados).
    * Reorganize a estrutura de pacotes do novo microsserviço para refletir o **DDD**, utilizando pacotes como `domain/`, `application/`, `infrastructure/` dentro do seu contexto (ex: `com.seuprojeto.task.domain`, `com.seuprojeto.task.application`).

4.  **Definição da API Externa (Contrato do Microsserviço):**
    * O microsserviço precisará expor uma API para que outras partes do sistema (incluindo o monólito original) possam interagir com ele.
    * **Definam e implementem endpoints REST** (Controllers) no novo microsserviço que permitam realizar as operações principais do seu **Bounded Context**.
    * Lembrem-se: essa API deve ser **pública e bem definida**, expondo apenas o necessário e evitando vazar detalhes internos do modelo de domínio do microsserviço. Use **DTOs** (Data Transfer Objects) para entrada e saída de dados, jamais expondo diretamente as entidades de domínio internas.

5.  **Adaptação do Monólito Original (Camada Anti-Corrupção - Desafio Extra, opcional):**
    * No monólito original, onde o contexto extraído costumava viver, **removam** as classes e lógicas que foram migradas para o novo microsserviço.
    * **Substituam** as chamadas internas a essa lógica por chamadas para a **nova API REST** do microsserviço.
    * **Desafio Extra (Camada Anti-Corrupção):** Se houver complexidade na integração ou na tradução de modelos entre o monólito e o novo microsserviço, considerem a criação de uma **Camada Anti-Corrupção (ACL)** no monólito. Essa camada atuaria como um tradutor, adaptando o modelo do novo microsserviço para o modelo ainda existente no monólito, isolando o acoplamento.

### Exemplo de Aplicação (para um contexto de "Gerenciamento de Tarefas"):

Suponha que vocês extraíram o **Bounded Context "Gerenciamento de Tarefas"** para um novo microsserviço `task-service`.

**No novo Microsserviço (`task-service`):**

* Estrutura de pastas: `src/main/java/com/edtech/task/domain`, `com/edtech/task/application`, `com/edtech/task/infrastructure`.
* Classes como `Task`, `Title`, `Priority`, `Deadline`, `TaskId` (Value Objects e Aggregate Root) estariam no pacote `domain`.
* Um `TaskApplicationService` para coordenar operações no `application`.
* Um `TaskController` no `infrastructure` (ou diretamente na raiz da API) com endpoints como `POST /tasks`, `PUT /tasks/{id}/complete`, `GET /tasks/{id}`.
* Os DTOs para `POST /tasks` seriam `CreateTaskRequest` (com título, prioridade, etc.), e para `GET /tasks/{id}` seria `TaskResponse` (com status, data, etc.).

**No Monólito Original:**

* Onde antes havia um `TaskService` monolítico, agora ele faria uma **chamada HTTP** para `http://task-service/api/tasks` para criar uma tarefa.
* Pode ser necessário um cliente REST (ex: `RestTemplate` ou `WebClient` do Spring) para fazer essas chamadas. Pesquisem!

### Formato da Entrega:

Cada equipe deve entregar:

1.  **Documentação da Extração (2-3 páginas):**
    * **Bounded Context Escolhido:** Qual foi o Bounded Context selecionado para extração e qual a sua justificativa (baseada em taxa de mudança, escalabilidade, criticidade)?
    * **Context Map Simples:** Um diagrama (mesmo que simples, em texto ou PlantUML) mostrando o novo microsserviço e como ele se relaciona com o monólito original. Indiquem o **tipo de relacionamento** (ex: Customer/Supplier).
    * **Linguagem Ubíqua e Modelagem Tática:** Descrevam os principais **elementos táticos (Entidades, VOs, Agregados)** que foram migrados e como eles aplicam a **linguagem ubíqua** desse contexto.
    * **Contrato do Microsserviço:** Listem os principais **endpoints REST** que o novo microsserviço expõe e quais são os **DTOs** de entrada/saída (nome e principais campos).
    * **Adaptação do Monólito:** Expliquem como o monólito original foi adaptado para se comunicar com o novo microsserviço (chamadas REST, e se houver, menção à ACL).

2.  **Repositórios de Código:**
    * O **novo repositório (ou pasta/módulo)** do microsserviço Spring Boot extraído.
    * O **repositório original do monólito**, mostrando as modificações para se integrar ao novo serviço.

3.  **Demonstração (obrigatória):**
    * Preparem uma breve demonstração da sua aplicação, mostrando o monólito chamando (ou interagindo de alguma forma) com o novo microsserviço.
    * Estejam prontos para explicar o código e as decisões de design tomadas.

Entrega no Moodle!

# **Bom trabalho!🛠️**