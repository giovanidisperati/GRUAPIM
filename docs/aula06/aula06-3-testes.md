---
layout: aula
title: 3. Testes - Unitários e Funcionais
parent: Aula 06 - Service Layers e Testes Automatizados
nav_order: 3
---

## 3. 🧪 Testes Automatizados: Unitários e Funcionais

Os testes automatizados são uma prática importante no desenvolvimento de software. Eles garantem que o sistema funcione conforme o esperado, facilitando a identificação precoce de falhas e permitindo maior confiança durante manutenções e evoluções da aplicação. Nesta fase do projeto, vamos explorar dois tipos de testes amplamente utilizados: **testes unitários** e **testes funcionais**, entendendo suas diferenças, suas aplicações e seus benefícios.

Os **testes unitários** têm como objetivo verificar o comportamento de uma **unidade isolada de código**, geralmente um método ou classe. Em nosso caso, aplicaremos esse tipo de teste aos controladores, validando se, por exemplo, o método `getContactById()` retorna o DTO correto diante de um `Contact` válido retornado pelo repositório. Para isso, simulamos as dependências (como repositórios e mapeadores) com o uso de *mocks* via bibliotecas como Mockito, e utilizamos a anotação `@WebMvcTest` para carregar apenas o contexto necessário do Spring.

Já os **testes funcionais** (também conhecidos como testes de ponta a ponta ou de integração de alto nível) verificam o **comportamento da aplicação como um todo**, a partir da perspectiva do cliente. Simulam requisições HTTP reais e validam a resposta completa da API — status, corpo, headers — utilizando ferramentas como o `MockMvc` ou `TestRestTemplate` combinadas com `@SpringBootTest`. Isso garante que todas as camadas (controller, service, repository) estejam funcionando de forma integrada.

A grande diferença entre os dois tipos de teste é justamente o **nível de isolamento**: enquanto o teste unitário verifica uma peça de código individual em um ambiente controlado, o teste funcional executa um fluxo real do sistema em execução. Ambos são importantes e se complementam: o primeiro nos dá precisão, o segundo, confiança.

Um dos principais benefícios dos testes automatizados está na **refatoração segura do código**. Quando queremos melhorar ou modificar uma implementação — seja otimizando um algoritmo, movendo uma lógica para outro serviço ou alterando um mapeamento de DTO — os testes servem como uma **rede de segurança**. Se todos continuarem passando após a refatoração, sabemos que mantivemos o comportamento original intacto. Isso é ainda mais importante em projetos de médio e grande porte, onde mudanças podem impactar múltiplos pontos do sistema. Lembrem-se das aulas de Engenharia de Software, particularmente onde falamos sobre as práticas de Refatoração de Código Limpo!

Além disso, os testes são cruciais para a **manutenibilidade do software**. Um sistema com testes bem escritos se torna mais confiável e fácil de evoluir, pois a equipe consegue validar rapidamente o impacto de novas funcionalidades ou correções. Eles também atuam como uma forma de documentação viva, pois expressam de maneira clara e executável o comportamento esperado de cada componente. Dessa forma seu custo de manutenção ao longo do ciclo de vida tende a ser bastante controlado em relação aos códigos legados que não possuem testes.

Por isso, nesta etapa do projeto, investiremos tempo para introduzir a implementação de testes — tanto unitários quanto funcionais — cobrindo os cenários positivos e negativos da aplicação. Isso

O objetivo é garantir que os comportamentos esperados da aplicação estejam sempre funcionando corretamente, mesmo diante de futuras alterações no código.

Podemos sintetizar isso da seguinte forma:


| Tipo de Teste      | Foco                          | Exemplo prático                                           |
|--------------------|-------------------------------|-----------------------------------------------------------|
| Teste Unitário     | Testar uma unidade isolada    | Um método de um controller ou serviço                     |
| Teste Funcional    | Testar a aplicação como um todo| Uma chamada HTTP que testa o fluxo completo da API        |
| Teste de Integração| Testar integração entre módulos| O repositório conversando com o banco de dados            |

Para essa aula, focaremos em:

- **Testes unitários** dos *controllers*
- **Testes funcionais** da API, simulando chamadas HTTP

Esses testes nos ajudarão a validar os endpoints, as regras de negócio, o comportamento dos DTOs e as interações com o banco de dados de forma mais robusta.

### 🤔 O que são **mocks?**

Mocks são como **atores substitutos** em uma peça de teatro: eles imitam o comportamento de outros personagens para que a cena possa ser ensaiada, mesmo quando os atores principais não estão presentes. Em testes de software, mocks são **objetos falsos** usados para simular o comportamento de componentes reais (como um banco de dados, um serviço externo ou um repositório) sem realmente executar suas funcionalidades.

Por exemplo, imagine que você está testando uma classe `ContactController`, que depende de um `ContactRepository` para buscar contatos no banco. Em vez de acessar um banco de dados real, você cria um **mock do repositório** que diz: “quando alguém me perguntar pelo contato com ID 1, vou fingir que existe e devolver esse objeto aqui”. Assim, o foco do teste fica apenas no comportamento do controller — e **não no banco**.

Mocks ajudam a deixar os testes **mais rápidos, previsíveis e isolados**, como se estivéssemos testando uma peça do motor sem precisar ligar o carro todo. Eles são especialmente úteis em **testes unitários**, onde queremos validar uma classe específica sem depender de todo o resto da aplicação. Para criar esses “atores substitutos” em Java, usamos ferramentas como o **Mockito**, que permite dizer exatamente o que o mock deve fazer em cada situação.

Usar mocks é uma forma inteligente de garantir que estamos testando **apenas o que importa**, sem efeitos colaterais e sem depender de estruturas externas.

### Como estruturar nossos testes? 😱

Uma das formas mais comuns de estruturarmos nossos testes é por meio da utilização da **estrutura Given-When-Then**.

Essa estrutura é um padrão comum para organizar testes de forma clara e legível, inspirado na técnica **Behavior-Driven Development (BDD)**. Ela ajuda a separar as fases do teste para que qualquer pessoa (inclusive quem não é desenvolvedor) entenda o que está sendo testado. Vamos entender cada parte:

- **Given** (*Dado...*) → representa o **contexto** inicial do teste. É onde preparamos o cenário: criamos objetos, configuramos mocks, definimos entradas etc.

- **When** (*Quando...*) → descreve a **ação** que está sendo testada. Geralmente é a chamada de um método, envio de uma requisição ou invocação de uma operação.

- **Then** (*Então...*) → especifica os **resultados esperados**. Aqui usamos *assertions* para verificar se o comportamento do código foi o correto.

```java
@Test
void deveRetornarUmContatoQuandoIdForValido() {
    // Given
    Long id = 1L;
    Contact contact = new Contact("João", "joao@email.com", "11999999999");
    ReflectionTestUtils.setField(contact, "id", id);

    // When
    when(contactRepository.findById(id)).thenReturn(Optional.of(contact));
    ResponseEntity<ContactResponseDTO> response = contactController.getContactById(id);

    // Then
    assertEquals(HttpStatus.OK, response.getStatusCode());
    assertEquals(id, response.getBody().getId());
}
```

Perceba que ao final dos testes usaremos Assertions para verificar os resultados.

### 🧪 **O que são Assertions**

**Assertions** (ou **afirmações**) são instruções que verificam se o resultado de um teste está **de acordo com o esperado**. Elas são a parte mais importante do *Then* e dizem: “Isso que aconteceu é o que eu esperava?”

Alguns exemplos com `JUnit`:

```java
assertEquals(2, resultado); // Verifica se o valor é 2
assertTrue(lista.isEmpty()); // Verifica se a lista está vazia
assertThrows(IllegalArgumentException.class, () -> metodoQueDeveFalhar()); // Verifica se uma exceção foi lançada
```

Se uma asserção falhar, o teste **falha**, o que indica, portanto, que o código não está se comportando como deveria.

### Testes como ferramenta de qualidade de entrega contínua? ♻️

Quando falamos sobre testes, eles são normalmente executados em:

- Ambiente local, durante o desenvolvimento

- Pipelines de integração contínua (CI), como GitHub Actions, GitLab CI, Jenkins etc., garantindo que o sistema esteja estável antes de realizar deploys

Isso reforça o papel dos testes não só como ferramenta de desenvolvimento, mas também de qualidade de entrega contínua.

Por hora vamos utilizar os testes apenas em ambiente local, mas futuramente abordaremos o uso dele em pipelines de integração contínua.

### É só isso que preciso saber sobre testes? 🥳

Não. Na verdade ainda temos muito a falar: importância da cobertura de testes, inserção de mutantes para QA, organização do código de testes e uso de anotações para evitar duplicação em nossa base de testes.

De forma geral, o importante é entender que nossa base de testes também precisa estar limpa, assim como nossa base de código de produção. Dessa forma, ao alterarmos nossa base de código para introduzir novas features ou modificar features existentes, temos a possibilidade de alterar o código de teste de forma menos trabalhosa.  

Mais adiante na disciplina abordaremos novamente essas questões. Feitas essas considerações, passemos à implementação dos testes!

### Estrutura de diretórios do nosso projeto 📂

Quando concluirmos a implementação de nossos testes (ao término dessa seção), nossa estrutura de diretórios ficará da maneira vista a seguir:


```
├── src
│   ├── main
│   │   ├── java
│   │   │   └── br
│   │   │       └── ifsp
│   │   │           └── contacts
│   │   │               ├── config
│   │   │               │   └── MapperConfig.java
│   │   │               ├── controller
│   │   │               │   ├── AddressController.java
│   │   │               │   └── ContactController.java
│   │   │               ├── dto
│   │   │               │   ├── address
│   │   │               │   │   ├── AddressRequestDTO.java
│   │   │               │   │   └── AddressResponseDTO.java
│   │   │               │   └── contact
│   │   │               │       ├── ContactPatchDTO.java
│   │   │               │       ├── ContactRequestDTO.java
│   │   │               │       └── ContactResponseDTO.java
│   │   │               ├── exception
│   │   │               │   ├── GlobalExceptionHandler.java
│   │   │               │   └── ResourceNotFoundException.java
│   │   │               ├── model
│   │   │               │   ├── Address.java
│   │   │               │   └── Contact.java
│   │   │               ├── repository
│   │   │               │   ├── AddressRepository.java
│   │   │               │   └── ContactRepository.java
│   │   │               └── ContactsApiApplication.java
│   │   └── resources
│   │       ├── application.properties
│   │       └── application-test.properties
│   └── test
│       ├── java
│       │   └── br
│       │       └── ifsp
│       │           └── contacts
│       │               ├── ContactController
│       │               │   ├── ContactControllerFunctionalTest.java
│       │               │   └── ContactControllerUnitTest.java
│       │               └── ContactsApiApplicationTests.java
│       └── resources
│           └── schema.sql
```

Antes de começar, entretanto, certifique-se de que seu `pom.xml` tem as dependências de teste (que o Spring Boot já inclui por padrão):

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

### Entendendo os arquivos `schema.sql` e `application-test.properties`

A introdução dos arquivos `schema.sql` e `application-test.properties` na estrutura do projeto, como visto acima, tem como principal objetivo configurar um ambiente de testes isolado, confiável e controlado — essencial para a execução adequada dos testes funcionais e de integração.

O arquivo `schema.sql` é utilizado para definir explicitamente a estrutura das tabelas do banco de dados durante os testes. Ele contém os comandos SQL responsáveis por criar as tabelas necessárias (como `CONTACT` e `ADDRESS`) com suas colunas, tipos e relacionamentos. Quando usamos o banco H2 em memória, o Spring precisa de instruções claras sobre como construir esse esquema, especialmente quando a configuração `spring.jpa.hibernate.ddl-auto` está desativada. Sem esse arquivo, a aplicação pode lançar erros como “table not found” ou apresentar falhas de mapeamento JPA ao executar testes que dependem do acesso ao banco. Para garantir que nossos testes funcionem, vamos criá-lo. 

Já o `application-test.properties` é o arquivo de configuração específico para o ambiente de testes. Ele permite sobrescrever as configurações padrão da aplicação e adaptar o ambiente exclusivamente para os testes automatizados. Nele, por exemplo, definimos o uso de um banco H2 em memória, um driver JDBC específico (`org.h2.Driver`), e evitamos que o Hibernate tente criar ou modificar o banco automaticamente (`spring.jpa.hibernate.ddl-auto=none`). Também indicamos que o Spring deve utilizar o `schema.sql` como base para criar o banco de dados ao iniciar os testes. Com isso, garantimos que o ambiente seja inicializado sempre da mesma maneira, sem depender de configurações locais ou de acesso a bancos reais como MySQL ou PostgreSQL - tenha em mente que **nunca, sob hipótese alguma, devemos executar nossos testes no banco de dados real de produção**.

Em conjunto, `schema.sql` e `application-test.properties` asseguram que os testes rodem de forma independente, previsível e livre de efeitos colaterais. Isso é fundamental para garantir que os testes automatizados sejam confiáveis, principalmente em processos de integração contínua (CI/CD), onde testes precisam rodar automaticamente a cada nova entrega de código. Se desejarmos, ainda é possível complementar esse ambiente adicionando um arquivo `data.sql` para popular o banco com dados iniciais e facilitar os testes em cenários reais! Por hora, entretanto, vamos seguir com a implementação maios simples.


### Código do `schema.sql`

```sql
CREATE TABLE contact (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL,
    telefone VARCHAR(15) NOT NULL
);

CREATE TABLE address (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    rua VARCHAR(255) NOT NULL,
    cidade VARCHAR(255) NOT NULL,
    estado VARCHAR(2) NOT NULL,
    cep VARCHAR(9) NOT NULL,
    contact_id BIGINT NOT NULL,
    CONSTRAINT fk_address_contact FOREIGN KEY (contact_id) REFERENCES contact(id) ON DELETE CASCADE
);
```

### Código do `application-test.properties`

```properties
spring.datasource.url=jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.sql.init.mode=always
spring.sql.init.schema-locations=classpath:schema.sql
spring.jpa.hibernate.ddl-auto=none
```

### 🎯 Testando o ContactController com Testes Unitários: Classe `ContactControllerUnitTest.java`

```java
package br.ifsp.contacts.ContactController;

@ExtendWith(MockitoExtension.class)
public class ContactControllerUnitTest {

    @Mock
    private ContactRepository contactRepository;

    @Mock
    private ModelMapper modelMapper;

    @InjectMocks
    private ContactController contactController;

    @Test
    void testGetContactById_ReturnsContact() {
            Long id = 1L;      
            Contact contact = new Contact("João", "joao@email.com", "11999999999");
            Address address = new Address();
            address.setRua("Rua A");
            address.setCidade("Cidade A");
            address.setEstado("SP");
            address.setCep("00000-000");
            List<Address> addresses = List.of(address);            
            contact.setAddresses(addresses);
            ReflectionTestUtils.setField(contact, "id", id); // ID via reflexão
        
            ContactResponseDTO dto = new ContactResponseDTO();
            dto.setId(id);
            dto.setNome("João");
            dto.setEmail("joao@email.com");
            dto.setTelefone("11999999999");
        
            when(contactRepository.findById(id)).thenReturn(Optional.of(contact));
            when(modelMapper.map(contact, ContactResponseDTO.class)).thenReturn(dto);
        
            ResponseEntity<ContactResponseDTO> response = contactController.getContactById(id);
        
            assertEquals(HttpStatus.OK, response.getStatusCode());
            assertEquals(id, response.getBody().getId());
    }

    @Test
    void testGetContactById_NotFound() {
        Long id = 999L;
        when(contactRepository.findById(id)).thenReturn(Optional.empty());
        
        assertThrows(ResourceNotFoundException.class, () -> contactController.getContactById(id));
    }
}
```

Nossa classe `ContactControllerUnitTest` é responsável por realizar **testes unitários** no controlador `ContactController`, utilizando a biblioteca **Mockito** para simular as dependências externas, como o repositório de dados (`ContactRepository`) e o `ModelMapper`. O objetivo aqui é testar o comportamento do método `getContactById()` em dois cenários distintos: quando o contato existe e quando ele não é encontrado. Esse tipo de teste isola a lógica da unidade sob teste — no caso, o método do controller — sem depender de banco de dados real, servidor web ou quaisquer outras partes externas da aplicação. Perceba que estamos testando APENAS o método `getContactById()`. Em uma aplicação real testaríamos todos os métodos. Aqui, contudo, para fins de brevidade o teste de um método é suficiente para exemplificarmos a ideia geral.

A classe está anotada com `@ExtendWith(MockitoExtension.class)`, que ativa o suporte do Mockito no JUnit 5, permitindo que as anotações `@Mock` e `@InjectMocks` funcionem corretamente. A anotação `@Mock` indica que os objetos `contactRepository` e `modelMapper` são **mocks**, ou seja, substituições falsas dos objetos reais. É importante ressaltar que os `mocks` são usados no `ContactRepository` e no `ModelMapper` para simular o comportamento dessas dependências externas ao ContactController durante os testes unitários. Isso permite que testar o controller **isoladamente**, sem depender da lógica real do repositório nem do mapeamento entre entidades e DTOs. A anotação `@InjectMocks` diz ao Mockito para criar uma instância real de `ContactController`, injetando nela os mocks criados. Dessa forma, podemos testar a lógica do controller com dependências controladas.

Vamos entender isso melhor separadamente:

#### 🗄️ `ContactRepository` como Mock

O `ContactRepository` é a interface responsável por acessar o banco de dados. No teste unitário, **não queremos acessar o banco real**, pois:

- Tornaria os testes **mais lentos**;
- Os testes poderiam **falhar por motivos externos** (banco fora do ar, sem dados, etc);
- Estaríamos testando o banco, e não a lógica do controller.

Ao usar `@Mock`, você diz:  
> “Eu vou controlar manualmente o que esse repositório retorna quando o método `findById()` for chamado.”

Exemplo:
```java
when(contactRepository.findById(1L)).thenReturn(Optional.of(contact));
```

Com isso, o teste **não consulta o banco**. Ele apenas simula que, ao buscar o ID 1, o repositório retornaria um contato já definido no teste.

#### 🔄 `ModelMapper` como Mock

O `ModelMapper` faz a conversão entre a entidade `Contact` e o DTO `ContactResponseDTO`. Embora o mapeamento não dependa de banco, ele pode envolver regras específicas ou lançar exceções se houver erros.

No teste unitário, não queremos verificar **se o mapeamento funciona corretamente** — isso seria papel de um teste separado (ou já é garantido por testes da própria lib).

Ao usar `@Mock`, você evita que o `ModelMapper` execute a lógica real e garante previsibilidade:

```java
when(modelMapper.map(contact, ContactResponseDTO.class)).thenReturn(dto);
```

Assim, você controla diretamente **qual DTO será retornado**, sem depender da configuração do mapper.

### ✅ Benefícios de Usar Mocks Aqui

1. **Isolamento**: O teste foca exclusivamente no comportamento do controller.
2. **Velocidade**: Sem conexão com banco ou execução de lógica de mapeamento.
3. **Controle**: Você define exatamente o que acontece quando métodos do mock são chamados.
4. **Confiabilidade**: O teste não falha por problemas em outras partes do sistema.

Entendido isso, passemos à análise de cada um dos métodos dessa classe.

### Método `testGetContactById_ReturnsContact()`

Esse método testa o cenário em que o contato **existe** no repositório. Vamos destrinchar as etapas do teste:

1. **Given (cenário)**:
   - É criado um `Contact` com nome, e-mail e telefone.
   - Um `Address` é criado e associado a esse contato.
   - O ID do contato é definido com a ajuda do `ReflectionTestUtils.setField()`, já que o campo `id` é privado e não tem `setId()`.
   - Um `ContactResponseDTO` (o objeto que deve ser retornado ao cliente) também é configurado com os mesmos dados.

2. **When (ação)**:
   - É feito um `when().thenReturn()` para simular que, ao chamar `findById(1L)` no repositório, será retornado o `Contact` criado.
   - Também se configura que, ao mapear esse `Contact` com o `modelMapper`, será retornado o `dto`.

3. **Then (verificação)**:
   - É chamado `getContactById(1L)` no controller.
   - O teste verifica se o status HTTP retornado é 200 OK e se o `id` retornado no corpo da resposta é igual ao esperado.

Esse teste assegura que, **dado um contato existente**, o controller irá corretamente buscar, mapear e retornar o DTO correspondente.

### Método `testGetContactById_NotFound()`

Esse teste cobre o cenário oposto: quando o contato **não existe**.

1. **Given**:
   - Define-se um ID inválido, por exemplo `999L`.
   - Configura-se o repositório para retornar `Optional.empty()` quando `findById(999L)` for chamado.

2. **When/Then**:
   - Usa-se `assertThrows()` para verificar se a chamada `contactController.getContactById(id)` lança uma exceção `ResourceNotFoundException`.

Este teste garante que o controller está corretamente tratando casos em que o recurso buscado não existe, retornando a exceção apropriada, o que no contexto da API equivale a um HTTP 404.

### Em resumo... ✍️

Esses dois testes são exemplos claros de **testes unitários com uso de mocks**. Eles validam o comportamento da classe `ContactController` **sem depender de banco de dados ou do funcionamento real do ModelMapper**. Isso torna os testes rápidos, previsíveis e focados exclusivamente na lógica do controller. Por fim, o uso da estrutura *Given-When-Then* torna o teste mais legível e alinhado às boas práticas de desenvolvimento orientado por testes (TDD ou BDD).

### 🎯 Testando o ContactController com Testes Funcionais: Classe `ContactControllerFunctionalTest.java`

```java
package br.ifsp.contacts.ContactController;

@SpringBootTest
@AutoConfigureMockMvc
@ActiveProfiles("test")
public class ContactControllerFunctionalTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    void shouldCreateContactSuccessfully() throws Exception {
        String json = """
                    {
                        "nome": "Maria Oliveira",
                        "email": "maria@example.com",
                        "telefone": "11988887777",
                        "addresses": [
                            {
                                "rua": "Rua das Rosas",
                                "cidade": "Campinas",
                                "estado": "SP",
                                "cep": "13000-000"
                            }
                        ]
                    }
                """;

        mockMvc.perform(post("/api/contacts")
                .contentType("application/json")
                .content(json))
                .andDo(print()) // <-- aqui exibe a resposta no console
                .andExpect(status().isCreated())
                .andExpect(jsonPath("$.nome").value("Maria Oliveira"))
                .andExpect(jsonPath("$.email").value("maria@example.com"));
    }

    @Test
    void shouldReturnNotFoundWhenContactDoesNotExist() throws Exception {
        mockMvc.perform(get("/api/contacts/9999"))
                .andExpect(status().isNotFound());
    }
}
```

Nossa classe `ContactControllerFunctionalTest` é um exemplo de **teste funcional** construído para validar o comportamento real da API exposta pelo `ContactController`, simulando chamadas HTTP reais como um cliente faria. Esse tipo de teste é necessário para garantir que o sistema funcione corretamente quando todas as camadas estão integradas (controller, service, repository e persistência de dados). Essa classe utiliza a anotação `@SpringBootTest` para carregar o contexto completo da aplicação, incluindo o banco de dados em memória (no perfil `test`), os beans reais e as configurações reais de mapeamento.

O propósito desses testes, portanto, são bem diferentes dos testes unitários que vimos anteriormente: trata-se de validar a **experiência de uso real da API REST** do sistema, com foco no comportamento dos endpoints quando executados de ponta a ponta. Diferente dos testes unitários, aqui não usamos mocks das dependências: o banco é real (embora em memória), o mapeamento via `ModelMapper` está em funcionamento e todas as camadas são exercitadas juntas.

Esses testes são valiosos para detectar falhas de integração, como problemas de mapeamento de DTOs, erros na persistência ou retornos inconsistentes. Quando executados em conjunto com os testes unitários, ajudam a garantir a robustez e confiabilidade do sistema. 

Em relação ao seu funcionamento, a anotação `@AutoConfigureMockMvc` permite que o Spring injete um objeto `MockMvc`, que simula requisições HTTP à aplicação sem a necessidade de subir um servidor real. A anotação `@ActiveProfiles("test")` ativa o perfil de teste, garantindo que as configurações do `application-test.properties` (como uso de banco H2 em memória) sejam utilizadas durante a execução dos testes.

### Método `shouldCreateContactSuccessfully`

Este método testa o **caso positivo de criação de um novo contato via HTTP POST**. Ele constrói uma requisição com JSON contendo os campos `nome`, `email`, `telefone` e uma lista de endereços. A string JSON é enviada ao endpoint `/api/contacts` utilizando o método `post`, e o conteúdo da requisição é definido como `application/json`.

O método então executa a requisição com `mockMvc.perform(...)` e aplica as seguintes validações:

- `andDo(print())`: exibe o conteúdo da requisição e da resposta no console (útil para depuração);
- `andExpect(status().isCreated())`: verifica se o status de resposta HTTP é 201 (Created);
- `andExpect(jsonPath("$.nome").value("Maria Oliveira"))`: verifica se o campo `nome` no corpo da resposta corresponde ao valor enviado;
- `andExpect(jsonPath("$.email").value("maria@example.com"))`: faz a mesma verificação para o campo `email`.

Esse teste garante que, quando uma requisição válida é enviada para criar um contato, o sistema responde corretamente com status 201 e os dados retornados são coerentes com os enviados.

### Método `shouldReturnNotFoundWhenContactDoesNotExist`

Este método verifica o **comportamento do sistema ao tentar buscar um contato inexistente**. Ele simula uma chamada HTTP GET para o endpoint `/api/contacts/9999`, onde o ID 9999 é um número arbitrário assumidamente não existente no banco de dados.

O teste espera que o sistema retorne o status `404 Not Found`, indicando que o recurso não foi encontrado. Isso confirma que o tratamento de exceções com `ResourceNotFoundException` está funcionando como esperado.

### 🧠 Testes Unitários + Funcionais!

Nesta etapa, vimos alguns testes automatizados para o `ContactController`, cobrindo tanto os casos esperados quanto os de erro. Com isso, nossa API começa a se aproximar de um projeto profissional: funcional, testável e documentada. ressaltar, entretanto, que estamos loooonge de completar todos os testes da aplicação. Por isso é importante não negligenciar o desenvolvimento dos testes ao implementarmos nossas features. 🤓