---
layout: aula
title: 1. Implementando DTOs
parent: Aula 05 - Aprimorando nossas APIs Rest
nav_order: 1
---
## 1. Implementando DTOs (Data Transfer Objects)

Vimos anteriormente, na Aula 04, alguns conceitos iniciais sobre DTOs, objetos que representam os dados que realmente precisam ser transmitidos ou recebidos por sua aplicação, sem os detalhes internos de implementação e persistência. Dessa forma, a camada de apresentação fica desacoplada das entidades, reforçando a segurança, evitando ciclos de serialização e tornando a aplicação mais robusta e flexível a mudanças. Ao expor as entidades JPA diretamente na API, podemos incorrer em uma série de inconvenientes que comprometem tanto a segurança quanto a arquitetura do sistema e, por isso, justifica-se o uso de DTOs. Vamos relembrar algumas vantagens de utilizarmos esse padrão:

Em primeiro lugar, há o risco de **expor dados sensíveis**: nem todos os campos internos de uma entidade devem ser acessíveis ao cliente, mas, ao enviar as entidades sem filtragem, é possível que informações privadas ou confidenciais sejam inadvertidamente reveladas. Apenas enviar objetos inteiros em formato JSON, portanto, pode não ser (e em muitos casos não é) o mais adequado. Com DTOs, escolhemos exatamente quais campos serão incluídos na resposta da API. Isso dá ao desenvolvedor controle total sobre a exposição de dados, evitando vazamentos acidentais de informações.

Outro aspecto é a **fragilidade do design**: se as entidades JPA forem diretamente utilizadas como formato de resposta e entrada da API, qualquer mudança na estrutura dessas entidades — seja para fins de manutenção, otimização ou adaptação às regras de negócio — pode quebrar as integrações existentes, pois os clientes dependem daquele formato exato. Com DTOs, é possível evoluir a estrutura dos dados de sua aplicação sem afetar o contrato da API. Podemos alterar as entidades livremente e adaptar os DTOs conforme necessário, mantendo compatibilidade com os consumidores da API.

Além disso, DTOs permitem aplicar validações específicas ao contexto da API, como campos obrigatórios apenas na criação (@NotBlank, @Size, etc.). Isso evita poluir as entidades com validações específicas de entrada de dados e permite uma **validação contextualizada** — útil quando os requisitos de entrada diferem dos requisitos de persistência.

Outro ponto importante a se considerar é que muitas vezes a resposta da API precisa incluir dados formatados, campos computados ou até mesmo combinar informações de múltiplas fontes. Com DTOs, você pode criar estruturas sob medida para a resposta da API, como incluir contadores, nomes concatenados, mensagens personalizadas, flags booleanas calculadas etc.

Em resumo:

- Entidades → representações do banco de dados (persistência)
- DTOs → representações dos dados trafegados na API (camada de apresentação)

Embora exija mais código inicialmente, o uso de DTOs é considerado boas práticas de engenharia de software, especialmente em APIs públicas, e contribui significativamente para a qualidade do projeto ao longo do tempo. **Existem ao menos duas formas distintas de implementar DTOs**: implementar um DTO para a request e response, ou implementar DTOs distintos para request e response. 

Há situações em que o uso de um único DTO tanto para entrada (request) quanto para saída (response) é aceitável e até recomendável, especialmente em contextos mais simples. Um exemplo são APIs extremamente simples e estáticas, com poucos endpoints, como um `GET /ping` ou um `POST /login`, e que não têm a expectativa de crescimento ou mudanças estruturais — nesse caso, utilizar um único DTO pode economizar tempo e reduzir complexidade. Da mesma forma, em aplicações internas ou protótipos, como APIs utilizadas apenas por desenvolvedores em ambientes controlados (por exemplo, um painel administrativo interno), essa separação pode ser adiada sem comprometer a manutenibilidade ou segurança. Além disso, em situações em que os dados esperados na requisição são exatamente os mesmos que serão retornados na resposta — ou seja, quando não há campos sensíveis, timestamps ou dados internos da aplicação — também é viável utilizar o mesmo DTO para ambos os sentidos, já que não há riscos de exposição indevida ou inconsistência no contrato da API.

Por outro lado, existem diversos cenários em que não é recomendado utilizar o mesmo DTO para requisição e resposta. Um desses casos ocorre quando há relacionamentos complexos entre entidades — por exemplo, em estruturas do tipo Cliente → Pedidos → Produtos —, onde o que é enviado ao servidor pode ser bastante diferente do que precisa ser retornado ao cliente. Outro caso comum é quando há lógica de negócios envolvida na resposta, como o cálculo de campos derivados, a exemplo de um campo `valorTotal` calculado com base na `quantidade` e no `preço`. Também é importante separar DTOs quando a aplicação possui múltiplos tipos de clientes consumindo a mesma API, como administradores, usuários públicos ou clientes mobile, que exigem visões distintas do mesmo recurso - nesse tipo de situação o uso de outros padrões, evidentemente, também serão necessários em conjunto com os DTOs. Os abordaremos futuramente na disciplina. Além disso, em aplicações que exigem maior controle sobre segurança e auditoria, é necessário evitar a exposição de dados sensíveis ou internos do sistema, o que torna o uso de DTOs separados uma prática importante para garantir a integridade e a segurança da informação.

Nesses casos em que estamos em projetos que buscam segurança, clareza e escalabilidade, separar DTOs para requisição (RequestDTO) e resposta (ResponseDTO) é uma prática recomendada. Uma das principais vantagens dessa separação é evitar a exposição de dados desnecessários ou sensíveis. Com um ResponseDTO, controlamos exatamente quais informações serão retornadas ao cliente, omitindo campos técnicos como `createdAt`, `updatedAt`, chaves estrangeiras como `contact_id`, ou ainda flags de controle interno como `isDeleted`, `isVerified` e `passwordHash`. Esses campos fazem sentido no contexto interno da aplicação e não devem ser expostos externamente.

Além disso, essa separação impede que o cliente envie dados que não deveria ou não poderia definir. Por exemplo, no caso de um ContactRequestDTO, o cliente pode fornecer apenas informações como nome, email e endereços, enquanto campos como `id` ou `createdAt` devem ser gerados e controlados exclusivamente pelo servidor. Dessa forma, garantimos que o cliente não tenha acesso indevido a propriedades que fogem do seu escopo de atuação.

Outro benefício importante está na possibilidade de aplicar validações específicas para os dados de entrada. Os RequestDTOs geralmente utilizam anotações como `@NotBlank`, `@Email`, `@Size` ou `@Pattern` para garantir a integridade dos dados recebidos, enquanto os ResponseDTOs não exigem esse tipo de validação, já que os dados já passaram por todas as regras de negócio do servidor antes de serem retornados ao cliente. 

Separar os DTOs também facilita a evolução da API. Com o tempo, novos requisitos podem exigir a inclusão de campos na resposta que não precisam estar presentes na requisição, ou a descontinuação de campos que antes eram obrigatórios na entrada. Além disso, essa separação permite criar diferentes versões do ResponseDTO para públicos distintos, como administradores, usuários finais ou aplicações mobile, adaptando a resposta conforme o contexto de uso.

Por fim, ao separar os DTOs, a documentação gerada automaticamente por ferramentas como Swagger ou OpenAPI se torna mais precisa e compreensível. Com contratos distintos para entrada e saída, a API pode ser melhor documentada e mais fácil de entender por desenvolvedores que a consumirão, evitando ambiguidade e promovendo uma comunicação clara entre cliente e servidor.

### 📌 Em resumo

| Situação                        | Recomendação               |
|--------------------------------|----------------------------|
| API simples, dados triviais    | Pode usar o mesmo DTO      |
| Aplicação robusta e escalável  | Separar request/response   |
| Dados sensíveis envolvidos     | Separar request/response   |
| Requisitos de validação distintos | Separar request/response   |
| Uso de Swagger/OpenAPI         | Separar para clareza       |


Vamos verificar o código-fonte para criação dos DTOs da nossa aplicação. 


### 1.1 🤔 Fazer validações no DTO ou na Entidade?

Em relação a questão de validação mencionada acima, vamos traduzir o exemplo dado nessa discussão muito relevante que ocorreu no Stack Overflow sobre onde colocar as validações no seu sistema: se na **entidade JPA**, se no **DTO**, ou em ambos. [Spring REST API validation - should be in DTO or in Entity? (Stack Overflow)](https://stackoverflow.com/questions/42280355/spring-rest-api-validation-should-be-in-dto-or-in-entity).

>Imagine que você tem uma entidade `User` com um campo `name`, e sua lógica de negócio exige que esse campo nunca seja nulo. Você também tem um `UserDTO` com o mesmo campo `name`.
>
>Suponha que todas as suas validações, tanto na entidade quanto no DTO, são feitas utilizando a API `jakarta.validation`.
>
>Se você validar apenas no controller (ou seja, validar o DTO com `@Valid`), você estará protegido contra a persistência de dados inválidos — mas apenas para requisições recebidas. Se houver um serviço interno que manipule diretamente a entidade (sem passar por uma requisição HTTP, por exemplo), ele poderá acabar salvando uma entidade inválida no banco de dados sem que você perceba, a menos que haja uma restrição na própria coluna do banco (como `NOT NULL`).
>
>Então você pode pensar: “OK, vou mover as anotações de validação do DTO para a entidade e pronto!”. Bem, sim… e não.
>
>Se você validar apenas na entidade, você de fato estará protegido tanto contra dados inválidos vindos de requisições externas quanto contra erros internos na camada de serviço. No entanto, isso pode trazer um problema de **desempenho**. Segundo Anghel Leonard, no livro *Spring Boot Persistence Best Practices*, toda vez que você carrega uma entidade do banco de dados, o Hibernate consome memória e CPU para manter o estado dessa entidade no contexto de persistência, mesmo que ela esteja em “modo somente leitura”.
>
>Agora pense: se o campo `name` estiver nulo e você validar isso apenas na entidade, o que acontece?
>
>1. Você inicia uma transação.
>2. Carrega a entidade do banco.
>3. Modifica a entidade.
>4. Tenta persistir.
>5. O Hibernate valida.
>6. A transação é revertida.
>7. E todo esse esforço (bastante custoso) foi feito **só para no fim dar erro e descartar tudo**.
>
>Isso poderia ter sido evitado com uma validação mais simples e imediata — por exemplo, no DTO, **antes mesmo de começar qualquer interação com o banco de dados**."

Ou seja, a partir dos argumentos acima fica evidente que uma boa prática é a de validação em ambas as camadas: tanto de transporte, quanto de persistência. O *tradeoff* é um eventual impacto de performance, mas que pode se mostrar negligente quando considerada a economia de recursos que se obtém evitando-se operações desnecessárias na camada de persistência.

### 1.2 🤔 Implementar os DTOs como Classes ou por meio de Records?

Ao implementar DTOs (Data Transfer Objects) em Java, podemos optar por duas abordagens principais: o uso de **classes tradicionais** ou de **`records`**, um recurso introduzido no Java 14 e estabilizado a partir do Java 16. Ambas as formas cumprem o mesmo propósito — transportar dados entre diferentes camadas da aplicação, como entre a camada de serviço e a camada de apresentação (ou entre client e server) — mas apresentam diferenças significativas quanto à concisão, imutabilidade, compatibilidade e flexibilidade. A escolha entre uma ou outra abordagem depende das necessidades do projeto e das preferências da equipe de desenvolvimento.

### DTOs como Classes

A abordagem tradicional utiliza classes Java no estilo POJO (Plain Old Java Object), com atributos privados, métodos getters e setters, construtores e, opcionalmente, sobrescrita de métodos como `equals()`, `hashCode()` e `toString()`. 

**Exemplo:**

```java
public class ContactResponseDTO {
    private String nome;
    private String email;

    public ContactResponseDTO(String nome, String email) {
        this.nome = nome;
        this.email = email;
    }

    public String getNome() {
        return nome;
    }

    public String getEmail() {
        return email;
    }
}
```

A principal vantagem do uso de classes é a **flexibilidade**. Podemos incluir lógica interna nos métodos, adicionar métodos auxiliares, sobrecargar construtores e até mesmo usar herança. Isso permite, por exemplo, criar hierarquias de DTOs ou adicionar comportamentos mais elaborados ao objeto. Além disso, essa abordagem é ideal para sistemas legados ou bibliotecas que exigem POJOs clássicos, como alguns recursos do Jackson em versões mais antigas. Frameworks e projetos em versões antigas do Java podem simplesmente não possuir suporte aos `records`.

Por outro lado, um ponto negativo é o **boilerplate**: classes DTO podem se tornar longas e repetitivas, especialmente em aplicações grandes que lidam com muitos atributos. Isso pode tornar o código mais difícil de manter a longo prazo. Entretanto, é possível diminuir o boilerplate com bibliotecas como o **Lombok**, que veremos nas próximas seções.

##### Herança em DTOs? 😱

Apesar de termos citado acima que o uso de classes para implementação de DTOs é mais flexível, temos que ter em mente que **a herança em DTOs é um anti-pattern na maioria dos casos ⚠️** 

Embora tecnicamente **possível**, usar **herança em DTOs geralmente não é recomendado** — e por um motivo muito importante: **DTOs não representam entidades do domínio com uma hierarquia semântica "é-um"**, mas sim **estruturas planas e transitórias de dados** usadas para transporte entre camadas ou sistemas.

##### 🚫 **Por que herança não faz sentido em DTOs na maioria dos casos?**

1. **Violação do princípio de responsabilidade única**  
   DTOs devem ter **uma única função clara**: transportar dados. Ao adicionar herança, começamos a introduzir um comportamento "estrutural" que remete à modelagem de domínio — e isso **mistura responsabilidades** que deveriam estar separadas.

2. **A herança pressupõe uma relação semântica forte ("é-um")**  
   Se criamos `ClienteDTO` que herda de `PessoaDTO`, estamos dizendo que **todo Cliente é uma Pessoa**, **em todos os contextos**. Mas e se a API de clientes for consumida por um sistema que **não conhece** o DTO de Pessoa? Ou pior: e se o DTO de Pessoa tiver campos que não fazem sentido para Cliente?  
   → Isso quebra o **princípio da substituição de Liskov** e compromete o reuso.

3. **Aumenta o acoplamento entre componentes**  
   A herança cria uma dependência forte entre DTOs, o que dificulta a manutenção e evolução do código — especialmente em sistemas distribuídos ou com múltiplos consumidores. Cada alteração na superclasse impacta todas as subclasses.

4. **Dificulta a documentação, o versionamento e a serialização**  
   Ferramentas como Swagger/OpenAPI perdem clareza ao lidar com hierarquias de DTOs. Além disso, algumas bibliotecas de serialização (como Jackson) exigem configurações adicionais para lidar com polimorfismo, o que torna o sistema mais complexo sem necessidade.

#####  ✅ **Composição é a abordagem recomendada**

**Composição** resolve todos os problemas anteriores: ao invés de herdar de uma superclasse, o DTO **declara explicitamente os campos que precisa**, ou **utiliza outros DTOs como campos compostos**.

#####  🧱 Exemplo com composição (boa prática):

```java
public record PessoaDTO(String nome, String email) {}

public record ClienteDTO(PessoaDTO pessoa, String numeroCartaoFidelidade) {}
```

Essa abordagem:
- ✅ Separa os contextos
- ✅ Reduz acoplamento
- ✅ Torna a API mais clara
- ✅ É mais fácil de manter, testar e documentar

#####  📌 Quando herança *pode* ser aceitável?

Em casos como:

- Aplicações internas, controladas e pequenas
- Frameworks que impõem herança em contratos (por exemplo, quando se usa `BaseRequestDTO`, `BaseResponseDTO` com metadados comuns)
- Ambientes onde a equipe **consciente dos riscos** opta pela herança para **reuso puramente estrutural**

Mesmo assim, **é preciso pesar os custos**, pois esses benefícios normalmente **podem ser obtidos com composição**, de forma mais segura e modular.

**DTOs representam dados de transporte, não estruturas de domínio**. E por isso, **usar composição é quase sempre a melhor escolha.**

🔎 **Lembrem-se sempre**: para fazer boas escolhas técnicas, é essencial **compreender os conceitos por trás das ferramentas**. Só assim conseguimos tomar decisões conscientes, alinhadas ao propósito do código e às necessidades reais do projeto.

### DTOs como Records

Com a introdução dos `records` no Java, a linguagem passou a oferecer uma maneira muito mais concisa de declarar classes imutáveis que servem unicamente para armazenar dados. Um `record` em Java automaticamente cria os campos, construtor, métodos getters, além de `equals()`, `hashCode()` e `toString()`.

**Exemplo:**

```java
public record ContactResponseDTO(String nome, String email) {}
```

O maior benefício do uso de `records` está na sua simplicidade e imutabilidade. Com poucas linhas, temos uma estrutura de dados clara, imutável e segura, ideal para representar objetos de transporte em APIs REST. Isso se alinha a boas práticas modernas que favorecem a imutabilidade e o uso de objetos simples para troca de dados.

No entanto, os `records` têm suas limitações. Por serem imutáveis, não permitem que seus campos sejam alterados após a construção do objeto, o que pode ser uma barreira em cenários que exigem mutabilidade. Além disso, `records` não suportam herança (embora possam implementar interfaces) e oferecem menos flexibilidade para incluir lógica interna elaborada. Em frameworks mais antigos ou bibliotecas que esperam um POJO com getters e setters tradicionais, o uso de `records` pode não funcionar corretamente.

🧠 **Mas e em APIs complexas? Posso usar records?**

**Sim! E muitas equipes fazem isso.** A chave está em manter a **função do DTO simples**: transportar dados. Se você adota uma arquitetura bem separada (com serviços, conversores, validadores e mapeadores bem definidos), o DTO pode — e deve — continuar sendo apenas um recipiente de dados. 

Exemplo: mesmo em uma API com dezenas de endpoints, como um sistema de e-commerce, é comum encontrar registros como:

```java
public record ProductResponseDTO(
    Long id,
    String nome,
    BigDecimal preco,
    int estoqueDisponivel
) {}
```

Ou seja: se o DTO **não precisa de lógica complexa ou mutabilidade**, mesmo em APIs grandes, **o record continua sendo uma excelente escolha**.

#### Comparativo entre Classes e Records


| Situação                                 | Usar `record` | Usar `class` |
|------------------------------------------|---------------|--------------|
| DTO simples (sem lógica)                 | ✅             | -            |
| Necessidade de lógica de transformação   | -             | ✅            |
| Integração com bibliotecas legadas       | -             | ✅            |
| Herança entre tipos de DTO               | -             | ✅            |
| Aplicações modernas com arquitetura limpa| ✅             | -            |

A escolha entre usar `records` ou classes tradicionais para representar DTOs deve ser feita com base nos requisitos do projeto. Em APIs simples ou aplicações modernas, `records` costumam ser a escolha ideal por sua concisão e imutabilidade. Já em contextos que compatibilidade com frameworks antigos, as classes ainda são a opção mais robusta.

## 1.3 ✅ **(Finalmente) Implementando nossos DTOs**

Como visto acima, temos duas maneiras de implementar nossos DTOs: com classes ou records. Ao longo da disciplina vamos abordar ambas as formas.

Nesse primeiro momento vamos implementar os DTOs de nossa aplicação fazendo o uso de classes e os separaremos em DTOs de `Request` e DTOs de `Response`. Vamos usar o `ModelMapper` para mapear nossos records e utilizar o `Lombok` para diminuir código boilerplate.

Faremos dessa forma por um único motivo: explorar esse tipo de implementação e ferramenta no Java. Nem sempre vamos trabalhar com projetos que utilizam versões modernas da linguagem e, em muitos casos, os sistemas em produção são legados e ainda utilizam Java 8. O Java 17 também é LTS e tem suporte prolongado, mas isso não significa que todos os projetos estejam migrados para ele — e muito menos para o Java 21.

Essa abordagem nos permite aprender conceitos fundamentais do ecossistema Java de forma mais completa: veremos como o `Lombok` pode nos ajudar na redução de código repetitivo, como funcionam os mapeamentos manuais e automáticos com `ModelMapper`, e entenderemos as diferenças práticas entre uma modelagem com classes e uma com records. Mais adiante, teremos a oportunidade de refatorar os mesmos DTOs utilizando `records` e comparar os impactos de cada abordagem, tanto na legibilidade quanto na manutenibilidade do código. 

O **Lombok** é uma biblioteca Java que ajuda a reduzir o código repetitivo (boilerplate) em classes, especialmente em projetos que utilizam muitos DTOs ou modelos com getters, setters, construtores e métodos como `toString` ou `equals`. Através de anotações simples o Lombok gera automaticamente esses métodos em tempo de compilação, tornando o código mais limpo, legível e fácil de manter. Ele é amplamente utilizado em aplicações Spring Boot e facilita o desenvolvimento sem comprometer a estrutura da aplicação. As anotações do **Lombok** como `@Getter`, `@Setter`, `@AllArgsConstructor`, `@NoArgsConstructor` e `@Data` servem para eliminar a repetição de código "boilerplate" nas classes Java.  

- `@Getter` e `@Setter` geram automaticamente os métodos *get* e *set* para todos os atributos da classe (ou para um atributo específico, se usados diretamente sobre ele).  
- `@NoArgsConstructor` cria um construtor sem argumentos (necessário, por exemplo, para frameworks como o JPA).  
- `@AllArgsConstructor` gera um construtor com todos os atributos da classe como parâmetros.  
- `@Data` combina várias anotações úteis: `@Getter`, `@Setter`, `@ToString`, `@EqualsAndHashCode` e `@RequiredArgsConstructor`, cobrindo boa parte das necessidades de uma classe simples de modelo ou DTO.  

Essas anotações deixam o código mais limpo, reduzindo a verbosidade típica do Java.

Para configurar a biblioteca **Lombok**, adicione a seguinte dependência no `pom.xml`:

```xml
<dependency>
	<groupId>org.projectlombok</groupId>
	<artifactId>lombok</artifactId>
	<version>1.18.28</version>
	<scope>provided</scope>
</dependency>
```

Em nosso contexto, de uma aplicação pequena e com pouquíssimas regras de negócio, entretanto, é importante salientar que **não faz sentido adotar uma implementação mais complexa a não ser por fins pedagógicos de demonstração de estruturas e ferramentas**, o que é exatamente nosso caso. Poderíamos tranquilamente criar os getters e setters "na mão" sem grande prejuízo de tempo. 

É preciso entender que optar por estruturas excessivamente sofisticadas em sistemas simples pode nos levar ao chamado Overengineering — ou “superengenharia”. Esse termo descreve a prática de criar soluções desnecessariamente complexas para problemas simples. Em vez de facilitar, o excesso de abstrações, padrões ou camadas técnicas pode dificultar a manutenção, tornar o código mais difícil de entender e até mesmo atrapalhar a produtividade da equipe.

Em outras palavras: só porque algo é possível tecnicamente, não significa que seja a melhor escolha para aquele momento ou projeto. Ou, como diria sua mãe: não é porque você pode, que você deve. Uma arquitetura deve ser proporcional à complexidade e às necessidades da aplicação. Por isso, ainda que exploremos ferramentas como DTOs separados, mapeamentos automáticos e uso de bibliotecas auxiliares, é essencial entender que essas decisões devem sempre ser tomadas com base no contexto, na equipe e nos objetivos do sistema — e não apenas por modismos ou pelo desejo de usar todas as tecnologias disponíveis.

Dito isso, e entendendo o contexto em que nossa aplicação está inserida, vamos organizar nossos DTOs por meio da seguinte **📁 Estrutura de Diretórios**

```
src/
└── main/
    └── java/
        └── br/
            └── ifsp/
                └── contacts/
                    ├── config/
                    │   └── MapperConfig.java
                    ├── dto/
                    │   ├── contact/
                    │   │   ├── ContactRequestDTO.java
                    │   │   ├── ContactResponseDTO.java
                    │   │   └── ContactPatchDTO.java
                    │   │
                    │   └── address/
                    │       ├── AddressRequestDTO.java
                    │       └── AddressResponseDTO.java
```

✨ Separar os DTOs em `Request` e `Response` nos ajuda a ter mais **clareza e controle sobre o fluxo de dados** que entra e sai da nossa aplicação. Perceba, também, que iremos criar um `ContactPatchDTO`, que será utilizado para atualizarmos um contato por meio de uma requisição `PATCH`. Os motivos para isso serão explorados quando abordarmos sua implementação. 🧑🏻‍💻

Vamos começar a explorar as implementações pelos DTOs que transportam os endereços.

### ✅ DTOs de Endereço (`Address`)

### 📥 `AddressRequestDTO.java`

```java
package br.ifsp.contacts.dto.address;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class AddressRequestDTO {
        @NotBlank(message = "A rua não pode estar vazia")
        String rua;

        @NotBlank(message = "A cidade não pode estar vazia")
        String cidade;

        @NotBlank(message = "O estado não pode estar vazio")
        @Size(min = 2, max = 2, message = "O estado deve ter exatamente 2 letras")
        @Pattern(regexp = "[A-Z]{2}", message = "O estado deve ser representado por duas letras maiúsculas")
        String estado;

        @NotBlank(message = "O CEP não pode estar vazio")
        @Pattern(regexp = "\\d{5}-\\d{3}", message = "O CEP deve estar no formato 99999-999")
        String cep;
}
```

Esse DTO, implementado como **classe com Lombok**, representa o **corpo da requisição** para criação ou atualização de um endereço. Ele contém apenas os campos que o cliente deve fornecer, com anotações de validação para garantir a integridade dos dados enviados.

#### 📤 `AddressResponseDTO.java`

```java
package br.ifsp.contacts.dto.address;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class AddressResponseDTO {
        private Long id;
        private String rua;
        private String cidade;
        private String estado;
        private String cep;
}
```

Essa **classe** representa a **resposta que a API retorna** ao cliente. Com o uso do Lombok, eliminamos boilerplate como getters e construtores. O campo `id` é incluído porque se trata de um dado **gerado pelo sistema** e importante para a leitura e manipulação dos dados pelo consumidor da API.

### ✅ DTOs de Contato (`Contact`)

### 📥 `ContactRequestDTO.java`

```java
package br.ifsp.contacts.dto.contact;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class ContactRequestDTO {
        @NotBlank(message = "O nome não pode estar vazio")
        private String nome;

        @NotBlank(message = "O email não pode estar vazio")
        @Email(message = "Formato de email inválido")
        private String email;

        @NotBlank(message = "O telefone não pode estar vazio")
        @Size(min = 8, max = 15, message = "O telefone deve ter entre 8 e 15 caracteres")
        @Pattern(regexp = "\\d+", message = "O telefone deve conter apenas números")
        private String telefone;

        @NotEmpty(message = "O contato deve ter pelo menos um endereço")
        private List<AddressRequestDTO> addresses;
}
```

A classe `ContactRequestDTO` representa os dados que o cliente envia para **criar ou atualizar** um contato. A estrutura mantém as validações exigidas pela aplicação e requer pelo menos um endereço, garantindo a integridade dos dados recebidos pela API.


### 📤 `ContactResponseDTO.java`

```java
package br.ifsp.contacts.dto.contact;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class ContactResponseDTO {
        private Long id;
        private String nome;
        private String email;
        private String telefone;
        private List<AddressResponseDTO> addresses;
}

```

O `ContactResponseDTO` representa os dados que a API retorna ao cliente ao consultar um contato. Ele inclui o `id`, informações pessoais do contato (nome, email, telefone) e a lista de endereços associados, já convertidos para `AddressResponseDTO`. É usado exclusivamente para **respostas** e nunca para envio de dados ao servidor.

### 📤  `ContactPatchDTO.java`

```java
package br.ifsp.contacts.dto.contact;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class ContactPatchDTO {
    private Optional<String> nome = Optional.empty();
    private Optional<String> email = Optional.empty();
    private Optional<String> telefone = Optional.empty();
}
```

O `ContactPatchDTO` foi criado especificamente para atender ao **endpoint PATCH**, que permite **atualizações parciais** de um recurso — no nosso caso, um `Contact`.

Usamos `Optional<String>` em cada campo para representar claramente a **presença ou ausência de um valor na requisição**. Isso nos ajuda a:

- Saber se o cliente quer ou não atualizar determinado campo.
- Evitar atualizar campos com `null` acidentalmente.
- Tornar o código de atualização mais expressivo e seguro, sem precisar verificar `null` diretamente.

Criamos um DTO exclusivo para PATCH pelos seguintes motivos:

- O PATCH não exige todos os campos (como `nome`, `email` e `telefone`), mas sim **somente os que o cliente deseja modificar**.
- Os DTOs de `Request` e `Response` são pensados para representar requisições completas (POST/PUT) e respostas completas (GET).
- Um DTO exclusivo com `Optional` representa perfeitamente a **semântica de atualização parcial**, garantindo clareza no contrato da API e facilitando a manutenção e validação.

Essa abordagem torna a API mais robusta, bem documentada, e alinhada às boas práticas de desenvolvimento RESTful.

## 1.4 ✡️ Conversão entre Entidades e DTOs 

Fazer a conversão entre entidades e DTOs é uma prática fundamental em APIs bem projetadas. Para reforçar o que vimos anteriormente: as entidades representam o modelo de domínio da aplicação e contêm toda a lógica de negócios e mapeamento com o banco de dados, incluindo relacionamentos complexos, anotações de persistência e campos internos que não devem ser expostos. Já os DTOs (Data Transfer Objects) são estruturas mais simples, voltadas exclusivamente para transportar dados entre o cliente e o servidor.

Quando recebemos uma requisição ou retornamos uma resposta, portanto, queremos converter os dados de uma Entidade para um DTO e vice-versa. Para facilitar esse processo de conversão e evitar a escrita manual de código repetitivo, podemos utilizar a biblioteca **ModelMapper**, que mapeia automaticamente os campos entre objetos com nomes semelhantes. Ela ajuda a manter o código limpo e padronizado, além de reduzir erros e acelerar o desenvolvimento. Por isso, configuramos um `@Bean` do ModelMapper na classe principal da aplicação, permitindo que ele seja injetado e utilizado em qualquer parte do sistema para conversões consistentes entre entidades e DTOs.

No contexto do Spring Framework, um *Bean* é um objeto cuja instância é criada, configurada e gerenciada automaticamente pelo Spring, por meio do seu container de Inversão de Controle (IoC). Quando anotamos um método com `@Bean`, estamos informando ao Spring que o objeto retornado por aquele método deve ser registrado no contexto da aplicação como um componente gerenciado. Isso significa que o Spring cuidará do ciclo de vida desse objeto e permitirá que ele seja injetado em outras partes do sistema com o uso da anotação `@Autowired`.

Por exemplo, ao configurarmos um método `modelMapper()` anotado com `@Bean`, o Spring criará uma instância da classe `ModelMapper`, armazenará essa instância em seu contexto interno e a disponibilizará para uso em toda a aplicação. Quando uma classe precisar de um `ModelMapper`, basta declarar um campo anotado com `@Autowired`, e o Spring se encarregará de injetar a instância configurada.

Esse comportamento tem várias vantagens. Primeiro, evita a criação repetida de instâncias de objetos que poderiam ser reaproveitados, promovendo reutilização e economia de recursos. Além disso, ao centralizar a criação e configuração dos objetos, favorece a manutenção e o teste do código, já que os componentes não são fortemente acoplados às suas dependências. Em outras palavras, os beans contribuem para uma arquitetura mais flexível, coesa e desacoplada, permitindo que o desenvolvedor foque na lógica de negócio em vez de se preocupar com detalhes de instanciamento e configuração.

### 🛑 ESPERE! Antes de prosseguir, vamos relembrar os conceitos de Inversão de Controle e Injeção de Dependência

Os termos "container de inversão de controle" (IoC Container) e "container de injeção de dependência" (DI Container) são frequentemente utilizados como sinônimos, e essa confusão é compreensível, já que ambos os conceitos estão intimamente relacionados. No entanto, existe uma distinção sutil entre eles que ajuda a compreender melhor o funcionamento interno do Spring e de frameworks semelhantes.

Inversão de Controle (IoC) é um princípio de design que propõe uma mudança na forma como o código lida com a criação e o gerenciamento de objetos. Em vez de o próprio código instanciar e controlar seus objetos diretamente, essa responsabilidade é delegada a um container, que passa a cuidar desse processo. Esse container é o IoC Container, responsável por instanciar classes, resolver e injetar dependências, inicializar objetos e gerenciar seu ciclo de vida ao longo da execução da aplicação. O programador, portanto, apenas declara o que precisa, e o container provê as instâncias apropriadas no momento adequado.

Dentro desse processo, a injeção de dependência (DI) surge como uma técnica concreta para realizar a inversão de controle. Por meio da injeção de dependência, o container fornece automaticamente as dependências que uma classe necessita — geralmente serviços, repositórios ou outras estruturas — sem que a própria classe tenha que criá-las. Isso pode ser feito de diferentes formas: via construtor, via métodos `set`, ou até diretamente nos atributos da classe, por meio de anotações como `@Autowired`.

O IoC Container, portanto, representa um conceito mais amplo, englobando todo o gerenciamento dos componentes da aplicação, enquanto o DI Container é um subconjunto especializado dessa infraestrutura, focado exclusivamente no fornecimento de dependências entre objetos. Podemos dizer que todo DI Container é um IoC Container, mas o inverso não é necessariamente verdadeiro, já que a inversão de controle vai além da simples injeção — ela pode envolver, por exemplo, a gestão do ciclo de vida dos objetos, configuração dinâmica, escopos, eventos, aspectos transversais (AOP), entre outros recursos.

No contexto do Spring Framework, essa estrutura é implementada principalmente pelas interfaces `ApplicationContext` e `BeanFactory`. O Spring oferece um IoC Container completo, com suporte robusto à injeção de dependência. Quando utilizamos anotações como `@Component`, `@Autowired` ou declaramos um `@Bean` em uma classe de configuração, estamos, na prática, utilizando a funcionalidade de DI provida pelo IoC Container do Spring para automatizar a construção e o fornecimento dos nossos objetos de forma segura, reutilizável e desacoplada.

Portanto, o IoC Container é a base sobre a qual o Spring se estrutura, e a injeção de dependência é uma das principais ferramentas disponíveis nesse modelo. Essa arquitetura nos permite construir aplicações mais limpas, testáveis e de fácil manutenção, separando claramente a lógica de negócio da infraestrutura e tornando o código mais declarativo e orientado a contratos.

Para mais informações, consulte a [Introdução aos Beans no Spring Framework](https://docs.spring.io/spring-framework/reference/core/beans/introduction.html).

Entendidas as diferenças entre IoC e DI, vamos continuar o que estávamos fazendo anteriormente: a implementação do ModelMapper em nosso projeto.

### Como configurar o ModelMapper?

Para configurar a biblioteca **ModelMapper**, adicione a seguinte dependência no `pom.xml`:

```xml
<dependency>
    <groupId>org.modelmapper</groupId>
    <artifactId>modelmapper</artifactId>
    <version>3.1.1</version>
</dependency>
```

Configure o bean na classe `MapperConfig`, criada no pacote config:

```java
package br.ifsp.contacts.config;

@Configuration
public class MapperConfig {

        @Bean
        public ModelMapper modelMapper() {
                ModelMapper modelMapper = new ModelMapper();
                return modelMapper;
        }
}
```

Agora temos que atualizar nossos controllers para utilizarmos os DTOs em nossas requisições e respostas ao invés das Entidades.

Será que você consegue, a partir das configurações acima, refatorar os controllers da nossa aplicação? 🤓

Para evitar duplicação de apresentação código (e também mais uma refatoração, já que temos que adicionar a paginação no próximo exercício!), vamos apresentá-los posteriormente. De qualquer forma, dê uma pausinha e caso não tenha conseguido fazer os exercícios da última aula, tente refatorar os controllers para que trabalhem com nossos DTOs.