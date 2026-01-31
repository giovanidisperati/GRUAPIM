---
layout: aula
title: 3. Camada Model
parent: Aula 07 - Construindo o To-do List
nav_order: 3
---

## 3. Código-fonte do To-Do List

Para deixar a explicação mais clara e facilitar o entendimento da estrutura do projeto, organizamos a apresentação das classes seguindo a ordem em que naturalmente elas se conectam dentro da aplicação: primeiro vamos ver o modelo de domínio, que representa os dados que manipulamos; depois os DTOs, que são usados para trocar informações com quem consome a API; na sequência os repositórios, que salvam e recuperam os dados no banco; logo depois a camada de serviços, onde reunimos e coordenamos as regras de negócio; e, por fim, os controllers, que expõem tudo isso através dos endpoints da API. Essa sequência ajuda a construir o raciocínio de dentro para fora — do coração da aplicação até a porta de entrada! 🤩

### 3.1. Modelos de Domínio (`model`)

Vamos começar nossa análise pela **camada de modelo**, onde definimos as **entidades** que representam o núcleo de informação da nossa aplicação. Tudo no sistema — criação de tarefas, atualizações, conclusões, consultas — gira em torno desses modelos. Entender suas estruturas é essencial para compreendermos o comportamento do sistema como um todo.

#### `Task.java`

```java
package br.ifsp.edu.todo.model;

@NoArgsConstructor
@Data
@Entity
public class Task {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NotBlank
    @Size(min = 10, max = 100)
    private String title;

    @Size(max = 255)
    private String description;

    @NotNull
    @Enumerated(EnumType.STRING)
    private Priority priority;

    @NotNull
    private LocalDateTime dueDate;

    private boolean completed;

    @NotNull
    @Enumerated(EnumType.STRING)
    private Category category;

    private LocalDateTime createdAt;
}
```

A classe `Task` representa a entidade principal da nossa aplicação de gerenciamento de tarefas. Ela está anotada com `@Entity`, o que indica que será mapeada para uma tabela no banco de dados pela JPA (Jakarta Persistence API). Cada instância de `Task` corresponde a uma linha na tabela.

Entre seus atributos, temos:

- `id`: a chave primária da tarefa, gerada automaticamente com estratégia de incremento (`IDENTITY`).
- `title`: o título da tarefa, obrigatório (`@NotBlank`) e limitado entre 10 e 100 caracteres para garantir descrições concisas e claras.
- `description`: um campo opcional para detalhes adicionais, limitado a 255 caracteres.
- `priority`: a prioridade da tarefa, obrigatoriamente preenchida, armazenada como texto (`EnumType.STRING`) para facilitar a leitura no banco de dados.
- `dueDate`: a data limite para conclusão da tarefa, também obrigatória.
- `completed`: um booleano que indica se a tarefa foi concluída ou não.
- `category`: a categoria à qual a tarefa pertence (ex: trabalho, estudo, pessoal), também obrigatoriamente preenchida e mapeada como texto no banco.
- `createdAt`: o timestamp de criação da tarefa, preenchido no momento em que a tarefa é criada.

A classe utiliza ainda o `Lombok` para gerar automaticamente métodos como getters, setters e o construtor padrão (`@NoArgsConstructor` e `@Data`), reduzindo o código repetitivo (boilerplate) e deixando a implementação mais limpa e focada apenas nas informações essenciais da entidade.

#### `Category.java`

```java
package br.ifsp.edu.todo.model;

public enum Category {
    STUDY, WORK, LEISURE, HEALTH, FAMILY, FRIENDS, PERSONAL, OTHER;
    
    public static Category fromString(String value) {
        return Arrays.stream(values()).filter(c -> c.name().equalsIgnoreCase(value)).findFirst()
                .orElseThrow(() -> new IllegalArgumentException("Invalid category: " + value));
    }
}
```

A classe `Category` define uma enumeração que representa as categorias possíveis para uma tarefa no nosso sistema. As opções incluem valores como `STUDY`, `WORK`, `LEISURE`, `HEALTH`, entre outros, possibilitando que o usuário classifique melhor suas tarefas de acordo com áreas da vida. 

Além disso, a enumeração implementa o método utilitário `fromString(String value)`, que permite converter uma string recebida — como em entradas de usuários ou dados de APIs — em um valor válido da enumeração. Esse método percorre todas as categorias existentes e realiza uma comparação `case-insensitive` para encontrar o valor correspondente. Caso a string informada não corresponda a nenhuma categoria conhecida, é lançada uma `IllegalArgumentException`, garantindo que apenas categorias válidas sejam aceitas. Essa abordagem reforça a robustez da nossa aplicação ao evitar erros silenciosos e inconsistências nos dados. Isso será bastante útil na nossa `TaskService`, como veremos mais adiante. 

#### `Priority.java`

```java
package br.ifsp.edu.todo.model;

public enum Priority {
    HIGH,
    MEDIUM,
    LOW
}
```

A classe `Priority` define uma enumeração simples que representa os níveis de prioridade que uma tarefa pode ter no sistema. As opções disponíveis são `HIGH` (alta), `MEDIUM` (média) e `LOW` (baixa). Essa enumeração é usada para classificar a importância relativa de cada tarefa, permitindo que o usuário ou a aplicação deem tratamento diferenciado de acordo com a prioridade definida. Por ser uma enumeração básica sem métodos adicionais, seu papel principal é fornecer um conjunto fixo e seguro de valores que podem ser atribuídos às tarefas, garantindo consistência e evitando a utilização de valores inválidos no sistema.

Concluindo esta seção, percebemos que as entidades `Task`, `Priority` e `Category` formam a espinha dorsal do nosso domínio, definindo tanto os dados persistidos quanto as restrições de valores possíveis. A clareza e a correção nesta camada são fundamentais, pois qualquer erro aqui propaga-se para todas as demais camadas. Agora que compreendemos como o núcleo da nossa aplicação — o modelo de domínio — está estruturado, é hora de avançarmos para uma camada igualmente fundamental: a dos DTOs. São eles que estabelecerão a comunicação segura entre o mundo externo e os nossos modelos internos.
