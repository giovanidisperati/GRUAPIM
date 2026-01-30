---
layout: aula
title: 2. Refatorando os controllers com ResponseEntity
parent: Aula 06 - Service Layers e Testes Automatizados
nav_order: 2
---

## 2. Refatorando nossos Controllers com `ResponseEntity`

Após a implementação das refatorações mencionadas acima (`ResponseEntity`, melhoria na documentação Swagger, injeção de dependência via construtor) chegamos à seguinte implementação do Controller.

```java
package br.ifsp.contacts.controller;

@Tag(name = "Contatos", description = "API para gerenciamento de contatos")
@Validated
@RestController
@RequestMapping("/api/contacts")
public class ContactController {

        private final ContactRepository contactRepository;
        private final ModelMapper modelMapper;

        public ContactController(ContactRepository contactRepository, ModelMapper modelMapper) {
                this.contactRepository = contactRepository;
                this.modelMapper = modelMapper;
        }

        /**
         * Retorna uma lista paginada de todos os contatos.
         * 
         * @param pageable informações de paginação
         * @return página de contatos
         */
        @Operation(summary = "Listar todos os contatos", description = "Retorna uma lista paginada de todos os contatos cadastrados no sistema")
        @ApiResponses(value = {
                        @ApiResponse(responseCode = "200", description = "Contatos encontrados com sucesso"),
                        @ApiResponse(responseCode = "403", description = "Acesso negado", content = @Content)
        })
        @GetMapping
        public ResponseEntity<Page<ContactResponseDTO>> getAllContacts(Pageable pageable) {
                Page<Contact> contacts = contactRepository.findAll(pageable);
                Page<ContactResponseDTO> responseDTO = contacts
                                .map(contact -> modelMapper.map(contact, ContactResponseDTO.class));
                return ResponseEntity.ok(responseDTO);
        }

        /**
         * Busca um contato pelo ID.
         * 
         * @param id identificador do contato
         * @return contato encontrado
         * @throws ResourceNotFoundException se o contato não for encontrado
         */
        @Operation(summary = "Buscar contato por ID", description = "Retorna um contato específico com base no ID fornecido")
        @ApiResponses(value = {
                        @ApiResponse(responseCode = "200", description = "Contato encontrado com sucesso"),
                        @ApiResponse(responseCode = "404", description = "Contato não encontrado"),
                        @ApiResponse(responseCode = "403", description = "Acesso negado", content = @Content)
        })
        @GetMapping("/{id}")
        public ResponseEntity<ContactResponseDTO> getContactById(@PathVariable Long id) {
                Contact contact = contactRepository.findById(id)
                                .orElseThrow(() -> new ResourceNotFoundException("Contato não encontrado: " + id));

                ContactResponseDTO responseDTO = modelMapper.map(contact, ContactResponseDTO.class);
                return ResponseEntity.ok(responseDTO);
        }

        /**
         * Cria um novo contato.
         * 
         * @param dto dados do contato a ser criado
         * @return contato criado
         */
        @Operation(summary = "Criar novo contato", description = "Cria um novo contato com os dados fornecidos")
        @ApiResponses(value = {
                        @ApiResponse(responseCode = "201", description = "Contato criado com sucesso"),
                        @ApiResponse(responseCode = "400", description = "Dados inválidos"),
                        @ApiResponse(responseCode = "403", description = "Acesso negado", content = @Content)
        })
        @PostMapping
        public ResponseEntity<ContactResponseDTO> createContact(@Valid @RequestBody ContactRequestDTO dto) {
                Contact contact = modelMapper.map(dto, Contact.class);
                contact.getAddresses().forEach(address -> address.setContact(contact));
                Contact saved = contactRepository.save(contact);
                ContactResponseDTO responseDTO = modelMapper.map(saved, ContactResponseDTO.class);
                return ResponseEntity.status(HttpStatus.CREATED).body(responseDTO);
        }

        /**
         * Atualiza um contato existente.
         * 
         * @param id  identificador do contato
         * @param dto novos dados do contato
         * @return contato atualizado
         * @throws ResourceNotFoundException se o contato não for encontrado
         */
        @Operation(summary = "Atualizar contato", description = "Atualiza todos os dados de um contato existente")
        @ApiResponses(value = {
                        @ApiResponse(responseCode = "200", description = "Contato atualizado com sucesso"),
                        @ApiResponse(responseCode = "400", description = "Dados inválidos"),
                        @ApiResponse(responseCode = "404", description = "Contato não encontrado"),
                        @ApiResponse(responseCode = "403", description = "Acesso negado", content = @Content)
        })
        @PutMapping("/{id}")
        public ResponseEntity<ContactResponseDTO> updateContact(@PathVariable Long id,
                        @Valid @RequestBody ContactRequestDTO dto) {
                Contact existingContact = contactRepository.findById(id)
                                .orElseThrow(() -> new ResourceNotFoundException("Contato não encontrado: " + id));

                modelMapper.map(dto, existingContact);
                existingContact.getAddresses().forEach(address -> address.setContact(existingContact));

                Contact updated = contactRepository.save(existingContact);
                ContactResponseDTO responseDTO = modelMapper.map(updated, ContactResponseDTO.class);
                return ResponseEntity.ok(responseDTO);
        }

        /**
         * Atualiza parcialmente um contato existente.
         * 
         * @param id  identificador do contato
         * @param dto dados a serem atualizados
         * @return contato atualizado
         * @throws ResourceNotFoundException se o contato não for encontrado
         */
        @Operation(summary = "Atualizar contato parcialmente", description = "Atualiza apenas os campos especificados de um contato existente")
        @ApiResponses(value = {
                        @ApiResponse(responseCode = "200", description = "Contato atualizado com sucesso"),
                        @ApiResponse(responseCode = "404", description = "Contato não encontrado"),
                        @ApiResponse(responseCode = "403", description = "Acesso negado", content = @Content)
        })
        @PatchMapping("/{id}")
        public ResponseEntity<ContactResponseDTO> updateContactPartial(@PathVariable Long id,
                        @RequestBody ContactPatchDTO dto) {
                Contact existingContact = contactRepository.findById(id)
                                .orElseThrow(() -> new ResourceNotFoundException("Contato não encontrado: " + id));
                dto.getNome().ifPresent(existingContact::setNome);
                dto.getEmail().ifPresent(existingContact::setEmail);
                dto.getTelefone().ifPresent(existingContact::setTelefone);
                Contact saved = contactRepository.save(existingContact);
                ContactResponseDTO responseDTO = modelMapper.map(saved, ContactResponseDTO.class);
                return ResponseEntity.ok(responseDTO);
        }

        /**
         * Exclui um contato.
         * 
         * @param id identificador do contato
         * @return resposta sem conteúdo
         * @throws ResourceNotFoundException se o contato não for encontrado
         */
        @Operation(summary = "Excluir contato", description = "Remove permanentemente um contato do sistema")
        @ApiResponses(value = {
                        @ApiResponse(responseCode = "204", description = "Contato excluído com sucesso"),
                        @ApiResponse(responseCode = "404", description = "Contato não encontrado"),
                        @ApiResponse(responseCode = "403", description = "Acesso negado", content = @Content)
        })
        @DeleteMapping("/{id}")
        public ResponseEntity<Void> deleteContact(@PathVariable Long id) {
                if (!contactRepository.existsById(id)) {
                        throw new ResourceNotFoundException("Contato não encontrado: " + id);
                }
                contactRepository.deleteById(id);
                return ResponseEntity.noContent().build();
        }

        /**
         * Busca contatos pelo nome.
         * 
         * @param name     nome ou parte do nome a ser pesquisado
         * @param pageable informações de paginação
         * @return lista paginada de contatos que correspondem ao critério de busca
         */
        @Operation(summary = "Buscar contatos por nome", description = "Retorna uma lista paginada de contatos cujo nome contém o termo pesquisado")
        @ApiResponses(value = {
                        @ApiResponse(responseCode = "200", description = "Busca realizada com sucesso"),
                        @ApiResponse(responseCode = "403", description = "Acesso negado", content = @Content)
        })
        @GetMapping("/search")
        public ResponseEntity<Page<ContactResponseDTO>> searchContactsByName(@RequestParam String name,
                        Pageable pageable) {
                Page<Contact> contacts = contactRepository.findByNomeContainingIgnoreCase(name, pageable);
                Page<ContactResponseDTO> responseDTO = contacts
                                .map(contact -> modelMapper.map(contact, ContactResponseDTO.class));
                return ResponseEntity.ok(responseDTO);
        }
}
```

Vamos usar essa oportunidade para relembrar (e reforçar) os conceitos que vimos anteriormente. Passemos à análise do nosso código, método a método!

### 🔎 Construtor da classe

Nosso construtor ficou da seguinte forma na nova versão.

```java
public ContactController(ContactRepository contactRepository, ModelMapper modelMapper) {
    this.contactRepository = contactRepository;
    this.modelMapper = modelMapper;
}
```

Nessa versão refatorada implementamos a **injeção de dependência via construtor**, que é considerada uma prática superior à injeção feita diretamente em atributos com `@Autowired` no desenvolvimento com Spring Boot. Isso se deve a diversos motivos técnicos e de boas práticas que promovem código mais limpo, seguro, testável e fácil de manter.

Um dos principais benefícios da injeção via construtor é a **possibilidade de declarar os atributos como `final`**, o que garante que essas dependências nunca serão modificadas após a inicialização da classe. Isso reforça o princípio da imutabilidade, contribuindo para a segurança do código e reduzindo a possibilidade de exceções do tipo `NullPointerException`.

Além disso, essa abordagem **facilita os testes unitários**. Ao utilizar construtores, o desenvolvedor pode instanciar a classe sob teste passando diretamente objetos mockados ou stubs das dependências, sem depender do contexto do Spring para realizar injeções automáticas. Isso torna o código mais independente, modular e fácil de testar em isolamento.

Outro ponto positivo é a **clareza e a legibilidade do código**. Quando os construtores são usados explicitamente, é fácil visualizar todas as dependências de uma classe logo de início, sem a necessidade de examinar todos os campos ou anotações espalhadas pela classe. Esse aspecto torna o código mais autodescritivo, o que facilita a compreensão tanto por outros desenvolvedores quanto por ferramentas de análise estática.

A injeção via construtor também ajuda a evitar **dependências ocultas ou parcialmente injetadas**. A injeção em atributos ocorre após a construção do objeto, o que pode gerar problemas em métodos anotados com `@PostConstruct` ou em inicializações que dependam dessas injeções logo na criação do bean. Já com o construtor, as dependências são garantidas no momento da criação do objeto, tornando o ciclo de vida mais previsível e seguro. Por exemplo, imagine um cenário em que uma classe ReportService utiliza uma dependência chamada EmailService, injetada com @Autowired, e dentro do método init(), anotado com @PostConstruct, tenta usá-la imediatamente. Existe o risco de emailService ainda não estar injetado nesse momento, resultando em um NullPointerException difícil de diagnosticar. Ao utilizar injeção via construtor, no entanto, esse risco desaparece, pois o Spring só consegue instanciar o objeto ReportService se todas as dependências passadas no construtor forem resolvidas previamente. Assim, ao chegar no @PostConstruct, temos a certeza de que o objeto está completamente pronto para uso, com todas as dependências já disponíveis. O código abaixo mostra esse cenário:

```java
@Component
public class ReportService {

    @Autowired
    private EmailService emailService;

    private String status;

    @PostConstruct
    public void init() {
        // Tentamos usar o emailService logo após a construção do bean
        this.status = emailService.sendReport("Relatório inicial");
    }
}
```

Se o `EmailService` ainda não tiver sido injetado no momento em que o método init() é executado, isso pode gerar um `NullPointerException`. Esse tipo de erro é sutil, porque depende do ciclo de vida do Spring, do modo como o bean é criado, se há proxies envolvidos, e da ordem de inicialização dos beans. Embora o Spring costume cuidar disso corretamente na maioria dos casos, é um risco real quando usamos `@Autowired` diretamente no atributo.

Além disso, essa prática de injeção via construtor se integra muito bem com ferramentas como o **Lombok**. Ao declarar os atributos como `final`, é possível usar a anotação `@RequiredArgsConstructor`, que automaticamente gera um construtor com todos os campos necessários, eliminando a necessidade de escrever código adicional, ao mesmo tempo em que preserva todos os benefícios da injeção via construtor.

Naturalmente, há exceções. Em classes muito simples ou com apenas uma dependência, a utilização direta de `@Autowired` pode ser aceitável. E se uma classe possui muitas dependências (acima de 4 ou 5), isso pode ser um sinal de que ela está assumindo responsabilidades demais, sendo mais adequado refatorá-la antes de decidir pela forma de injeção. 

Em resumo, a injeção de dependência via construtor promove **imutabilidade, testabilidade, legibilidade e segurança**, além de favorecer a manutenção e evolução do código. Por essas razões, ela é atualmente a abordagem **mais recomendada no ecossistema Spring**, especialmente em classes anotadas com `@Service`, `@Controller`, `@Component` e `@RestController`.

Vamos agora analisar os métodos!

### 🔎 Método 1: `getAllContacts(Pageable pageable)`

```java
/**
* Retorna uma lista paginada de todos os contatos.
* 
* @param pageable informações de paginação
* @return página de contatos
*/
@Operation(summary = "Listar todos os contatos", description = "Retorna uma lista paginada de todos os contatos cadastrados no sistema")
@ApiResponses(value = {
    @ApiResponse(responseCode = "200", description = "Contatos encontrados com sucesso"),
    @ApiResponse(responseCode = "403", description = "Acesso negado", content = @Content)
})
@GetMapping
public ResponseEntity<Page<ContactResponseDTO>> getAllContacts(Pageable pageable) {
    Page<Contact> contacts = contactRepository.findAll(pageable);
    Page<ContactResponseDTO> responseDTO = contacts
        .map(contact -> modelMapper.map(contact, ContactResponseDTO.class));
    return ResponseEntity.ok(responseDTO);
}
```

Acabamos de ver esse método na explicação acima, mas vamos reforçar os conceitos.

Este método é responsável por retornar todos os contatos cadastrados, com suporte à paginação. A anotação `@GetMapping` expõe o método via HTTP GET na URL `/api/contacts`.

Como vimos na aula anterior, o parâmetro `Pageable pageable` é um objeto especial do Spring Data que encapsula informações sobre a requisição de paginação: número da página (`page`), tamanho da página (`size`) e critério de ordenação (`sort`). Esses parâmetros podem ser passados diretamente pela URL. Por exemplo:

```
GET /api/contacts?page=0&size=10&sort=nome,asc
```

Internamente, o método chama `contactRepository.findAll(pageable)` para buscar os dados já paginados no banco de dados. O retorno é um `Page<Contact>`, uma interface que representa uma "página" de objetos com metadados como total de elementos, número da página, número total de páginas etc.

Em seguida, cada objeto `Contact` é mapeado para `ContactResponseDTO` usando o `ModelMapper`, uma biblioteca que converte objetos com base em seus atributos de mesmo nome (configurada e adicionada ao projeto na aula anterior). O método `map()` da interface `Page` é usado aqui:

```java
Page<ContactResponseDTO> responseDTO = contacts
        .map(contact -> modelMapper.map(contact, ContactResponseDTO.class));
```

Aqui, `contacts` é um objeto do tipo `Page<Contact>`, ou seja, uma lista paginada de contatos. O método `map()` da interface `Page` aplica uma transformação a cada elemento da lista interna, retornando um novo `Page` com os elementos convertidos — nesse caso, de `Contact` para `ContactResponseDTO`.

A transformação é feita através de uma `Function<T, R>`, fornecida por meio de uma expressão lambda. Uma `Function<T, R>` é uma interface funcional da API de funções do Java (`java.util.function`) que representa uma função que recebe um valor do tipo `T` e retorna um valor do tipo `R`. Em outras palavras, é uma função que transforma um valor de um tipo em outro. Vamos exemplificar com um código simples que transforma um inteiro em uma String:

```java
Function<Integer, String> intToString = num -> "Número: " + num;

String resultado = intToString.apply(10);
// resultado: "Número: 10"
```

Esse trecho de código acima utiliza a interface funcional `Function<T, R>` da API do Java para definir uma função que recebe um número inteiro (`Integer`) como entrada e retorna uma `String` como saída.

Na linha:

```java
Function<Integer, String> intToString = num -> "Número: " + num;
```

estamos criando uma variável chamada `intToString` do tipo `Function<Integer, String>`. Isso significa que essa função aceitará um valor do tipo `Integer` (número inteiro) e retornará um valor do tipo `String`. A implementação é feita com uma expressão lambda: `num -> "Número: " + num`, o que quer dizer que, ao receber um número `num`, a função concatenará a string `"Número: "` com esse número, produzindo uma nova string como resultado.

Em seguida, temos a linha:

```java
String resultado = intToString.apply(10);
```

Aqui, estamos chamando o método `apply` da função, passando o valor `10` como argumento. Isso faz com que a função seja executada e retorne a string `"Número: 10"`, que é atribuída à variável `resultado`.

Portanto, ao final da execução desse trecho, a variável `resultado` conterá o valor `"Número: 10"`. Esse exemplo ilustra de forma simples como a interface `Function` pode ser usada para representar transformações de dados no estilo funcional.

Da mesma forma, o modelMapper.map(...) é invocado para cada Contact e transforma o objeto em seu correspondente ContactResponseDTO. A expressão `contact -> modelMapper.map(contact, ContactResponseDTO.class)` é uma `Function<Contact, ContactResponseDTO>`!

💡Caso a explicação não tenha ficado suficientemente clara, consulte a [Documentação da interface Function (Java 8)](https://docs.oracle.com/javase/8/docs/api/java/util/function/Function.html) e também o artigo da Baeldung, [Baeldung: Java 8 Functional Interfaces](https://www.baeldung.com/java-8-functional-interfaces).

Por fim, o método retorna o resultado com `ResponseEntity.ok(responseDTO)`, encapsulando o conteúdo e o código de status HTTP 200 (OK).

Feitas essas considerações, passemos ao próximo método!



### 🔎 Método 2: `getContactById(Long id)`

```java
/**
* Busca um contato pelo ID.
* 
* @param id identificador do contato
* @return contato encontrado
* @throws ResourceNotFoundException se o contato não for encontrado
*/
@Operation(summary = "Buscar contato por ID", description = "Retorna um contato específico com base no ID fornecido")
@ApiResponses(value = {
    @ApiResponse(responseCode = "200", description = "Contato encontrado com sucesso"),
    @ApiResponse(responseCode = "404", description = "Contato não encontrado"),
    @ApiResponse(responseCode = "403", description = "Acesso negado", content = @Content)
})
@GetMapping("/{id}")
public ResponseEntity<ContactResponseDTO> getContactById(@PathVariable Long id) {
    Contact contact = contactRepository.findById(id)
        .orElseThrow(() -> new ResourceNotFoundException("Contato não encontrado: " + id));

    ContactResponseDTO responseDTO = modelMapper.map(contact, ContactResponseDTO.class);
    return ResponseEntity.ok(responseDTO);
}
```

Este método busca um contato específico a partir de seu `id`. A anotação `@PathVariable` indica que o valor da variável `id` será extraído diretamente da URL (por exemplo: `/api/contacts/5`), como vimos nas aulas anteriores.

O método tenta encontrar o contato via `contactRepository.findById(id)`. Caso o contato não exista, é lançada uma exceção personalizada `ResourceNotFoundException`, que será tratada pelo `GlobalHandlerException` (visto na aula 04), um `@ControllerAdvice` global da aplicação.

O código utiliza dois recursos importantes em aplicações modernas com Spring Boot: o método `.orElseThrow()` da classe `Optional`, e as anotações do Swagger (mais precisamente, da especificação OpenAPI 3) para documentação automática da API. Vamos entender cada parte com mais profundidade.

O método `.orElseThrow()` é uma forma moderna e concisa de tratar situações em que um valor pode ou não estar presente, representado por um objeto `Optional`. Vimos o uso de `Optional` nas aulas 03, 04 e 05. Mesmo assim, vamos reforçar o que é e como funciona. No contexto do código:

```java
Contact contact = contactRepository.findById(id)
    .orElseThrow(() -> new ResourceNotFoundException("Contato não encontrado: " + id));
```

A chamada `contactRepository.findById(id)` retorna um `Optional<Contact>` — ou seja, um recipiente que **pode ou não conter** um objeto `Contact`. Isso é uma forma segura de evitar exceções como `NullPointerException`, forçando o desenvolvedor a lidar com a possibilidade de ausência do valor.

O método `.orElseThrow()` verifica se há um valor presente. Caso **haja**, o valor é retornado. Caso **não haja**, a função passada como argumento é executada, lançando uma exceção customizada — no caso, uma `ResourceNotFoundException` com uma mensagem personalizada. Isso permite que o controle de fluxo seja feito de maneira limpa, sem necessidade de `if (contact == null)` ou blocos `try/catch`.

Esse padrão torna o código mais expressivo, seguro e aderente ao paradigma funcional introduzido com o `Optional` a partir do Java 8.

Além disso, o trecho de código também utiliza anotações da biblioteca `springdoc-openapi` (implementação da especificação OpenAPI para projetos Spring Boot) para gerar automaticamente documentação interativa da API — geralmente acessível via `/swagger-ui.html` ou `/swagger-ui/index.html`. 

```java
@Operation(summary = "Buscar contato por ID", description = "Retorna um contato específico com base no ID fornecido")
```

Essa anotação descreve o propósito do endpoint. O campo `summary` é uma descrição curta e objetiva, enquanto o `description` pode conter mais detalhes. Essa informação será exibida na interface gráfica do Swagger e também usada na geração de documentação no formato JSON/YAML da OpenAPI.

```java
@ApiResponses(value = {
    @ApiResponse(responseCode = "200", description = "Contato encontrado com sucesso"),
    @ApiResponse(responseCode = "404", description = "Contato não encontrado"),
    @ApiResponse(responseCode = "403", description = "Acesso negado", content = @Content)
})
```

Jã essa estrutura acima define **quais respostas o endpoint pode retornar**, informando:

- O **código HTTP** (`responseCode`) que pode ser devolvido,
- A **descrição textual** do que esse código representa no contexto da operação,
- E opcionalmente, o **conteúdo da resposta** via `@Content` (nesse exemplo, deixado vazio).

Essas anotações ajudam outras pessoas desenvolvedoras — e até o próprio time — a compreender rapidamente o comportamento esperado da API, suas possíveis respostas e os erros que podem ocorrer, tudo de forma automatizada e centralizada.

Em resumo, o método `.orElseThrow()` traz elegância e segurança ao tratamento de dados opcionais, eliminando verificações manuais de nulo e oferecendo uma forma fluente de lançar exceções customizadas. Já as anotações `@Operation` e `@ApiResponses` são fundamentais para gerar documentação automática, interativa e padronizada da API, que tornam o sistema mais compreensível, fácil de usar e alinhado às boas práticas de desenvolvimento de software moderno. 😊

Por fim, se o contato for encontrado, ele é convertido para `ContactResponseDTO` usando o `ModelMapper` e retornado dentro de um `ResponseEntity`.

### 🔎 Método 3: `createContact(ContactRequestDTO dto)`

```java
/**
* Cria um novo contato.
* 
* @param dto dados do contato a ser criado
* @return contato criado
*/
@Operation(summary = "Criar novo contato", description = "Cria um novo contato com os dados fornecidos")
@ApiResponses(value = {
    @ApiResponse(responseCode = "201", description = "Contato criado com sucesso"),
    @ApiResponse(responseCode = "400", description = "Dados inválidos"),
    @ApiResponse(responseCode = "403", description = "Acesso negado", content = @Content)
})
@PostMapping
public ResponseEntity<ContactResponseDTO> createContact(@Valid @RequestBody ContactRequestDTO dto) {
    Contact contact = modelMapper.map(dto, Contact.class);
    contact.getAddresses().forEach(address -> address.setContact(contact));
    Contact saved = contactRepository.save(contact);
    ContactResponseDTO responseDTO = modelMapper.map(saved, ContactResponseDTO.class);
    return ResponseEntity.status(HttpStatus.CREATED).body(responseDTO);
}
```

Este método é responsável por criar um novo contato. A anotação `@PostMapping` define que ele será acessado via HTTP POST. A anotação `@RequestBody` indica que o corpo da requisição será convertido para um objeto `ContactRequestDTO`, que representa os dados recebidos. Relembrando o que vimos nas aulas anteriores, a anotação `@RequestBody` no Spring Boot tem um papel importante em métodos que recebem dados enviados pelo cliente no corpo de uma requisição HTTP — geralmente em requisições do tipo `POST`, `PUT` ou `PATCH`. 

Quando essa anotação é aplicada a um parâmetro de um método no controller, como no caso:

```java
public ResponseEntity<ContactResponseDTO> createContact(@Valid @RequestBody ContactRequestDTO dto)
```

ela **instrui o Spring** a **deserializar automaticamente o corpo da requisição (normalmente em JSON)** para uma instância da classe especificada, neste caso, `ContactRequestDTO`. Ou seja, o Spring irá:

1. Ler o corpo da requisição HTTP (por exemplo, um JSON enviado via POST);
2. Usar uma biblioteca de mapeamento de objetos (por padrão, o Jackson) para converter os dados para o tipo `ContactRequestDTO`;
3. Entregar esse objeto já populado ao método do controller, pronto para ser utilizado na lógica de negócios.

Por exemplo, se o cliente enviar a seguinte requisição:

```http
POST /api/contacts
Content-Type: application/json

{
  "nome": "Maria Oliveira",
  "email": "maria@email.com",
  "telefone": "11999999999",
  "addresses": [
    {
      "rua": "Rua das Flores",
      "cidade": "São Paulo",
      "estado": "SP",
      "cep": "01234-567"
    }
  ]
}
```

O Spring automaticamente transforma esse corpo JSON em um objeto `ContactRequestDTO` com os respectivos campos preenchidos — sem que o desenvolvedor precise escrever manualmente qualquer código de parsing ou conversão.

Além disso, como o DTO é anotado com `@Valid`, o Spring também realiza **validação automática dos campos** com base nas anotações de Bean Validation (como `@NotBlank`, `@Email`, etc.). Se houver qualquer erro de validação, o Spring lançará uma exceção do tipo `MethodArgumentNotValidException`, que pode ser tratada globalmente para retornar uma resposta padronizada de erro.

Portanto, o uso do `@RequestBody` promove maior desacoplamento entre os dados da requisição e a lógica da aplicação, além de permitir a integração limpa com validação e documentação automática da API.

O objeto DTO é convertido diretamente para a entidade `Contact` com o `ModelMapper`. Contudo, como a entidade `Contact` tem uma relação bidirecional com `Address` (endereços), é necessário que cada endereço tenha seu campo `contact` corretamente associado. Isso é feito manualmente com:

```java
contact.getAddresses().forEach(address -> address.setContact(contact));
```

Aqui estamos usando, novamente, a interface funcional do Java para facilitar nossa sintaxe. A linha acima é equivalente a:

```java
for (Address address : contact.getAddresses()) {
    address.setContact(contact);
}
```

De qualquer forma, seja usando a forma funcional (com `Lambda`) ou imperativa (com `foreach`), sem esse trecho o campo `contact` dentro de cada `Address` ficaria `null`. Como resultado:

- O JPA/Hibernate não conseguiria gerar corretamente a chave estrangeira (`contact_id`, por exemplo) na tabela de endereços;
- Poderiam surgir erros de integridade referencial no banco de dados;

Depois de configurar as associações, o contato é salvo via `contactRepository.save(contact)`. O objeto salvo (que agora tem `id` e os endereços com `contact_id`) é convertido para `ContactResponseDTO` e retornado com código 201 (CREATED) usando:

```java
return ResponseEntity.status(HttpStatus.CREATED).body(responseDTO);
```

E assim, mais um método de nosso Controller está refatorado. Passemos ao próximo!

### 🔎 Método 4: `updateContact(Long id, ContactRequestDTO dto)`

```java
/**
* Atualiza um contato existente.
* 
* @param id  identificador do contato
* @param dto novos dados do contato
* @return contato atualizado
* @throws ResourceNotFoundException se o contato não for encontrado
*/
@Operation(summary = "Atualizar contato", description = "Atualiza todos os dados de um contato existente")
@ApiResponses(value = {
    @ApiResponse(responseCode = "200", description = "Contato atualizado com sucesso"),
    @ApiResponse(responseCode = "400", description = "Dados inválidos"),
    @ApiResponse(responseCode = "404", description = "Contato não encontrado"),
    @ApiResponse(responseCode = "403", description = "Acesso negado", content = @Content)
})
@PutMapping("/{id}")
public ResponseEntity<ContactResponseDTO> updateContact(@PathVariable Long id, @Valid @RequestBody ContactRequestDTO dto) {
    Contact existingContact = contactRepository.findById(id)
        .orElseThrow(() -> new ResourceNotFoundException("Contato não encontrado: " + id));

    modelMapper.map(dto, existingContact);
    existingContact.getAddresses().forEach(address -> address.setContact(existingContact));
    Contact updated = contactRepository.save(existingContact);
    ContactResponseDTO responseDTO = modelMapper.map(updated, ContactResponseDTO.class);
    return ResponseEntity.ok(responseDTO);
}
```

O método `updateContact` é responsável por **atualizar completamente** os dados de um contato já existente. Aqui utilizamos a anotação `@PutMapping`, que, conforme o padrão REST que vimos na Aula 03, indica que a atualização deve substituir **integralmente** o recurso identificado.

O processo começa com a tentativa de buscar o contato no banco de dados pelo `id` informado na URL. Caso o contato não seja encontrado, a exceção personalizada do tipo `ResourceNotFoundException` é lançada, retornando uma resposta HTTP 404 (Not Found).

Se o contato for encontrado, o objeto `ContactRequestDTO` enviado no corpo da requisição é mapeado para o objeto `Contact` existente usando o `ModelMapper`. O mesmo processo que usamos anteriormente 😊

Nesse método (`updateContact`), o mapeamento automático do `ModelMapper` é usado para substituir os dados antigos pelos novos — com exceção de associações que exigem tratamento especial, como a relação entre `Contact` e seus `Address`.

Logo após, associamos os endereços ao contato por meio da seguinte operação:

```java
existingContact.getAddresses().forEach(address -> address.setContact(existingContact));
```

Esse trecho percorre a lista de endereços (`addresses`) associados ao contato e, para **cada um deles**, define explicitamente o `Contact` pai. Esse *bind reverso* é necessário porque, ao mapear o DTO para a entidade, o `ModelMapper` **não estabelece automaticamente** essa associação de volta. Em outras palavras, os endereços sabem seus próprios dados (rua, cidade, etc.), mas não sabem a quem pertencem — e isso precisa ser explicitado manualmente.

Isso acontece porque os objetos `Address` recebidos via DTO não carregam, por padrão, a referência para o contato pai (`Contact`). Essa referência é essencial para manter a integridade da relação bidirecional entre as entidades e permitir que o JPA gere corretamente a chave estrangeira na tabela de endereços.

**Nós poderíamos ter configurado isso criando `TypeMaps` personalizados**, que permitem instruir o `ModelMapper` a **ignorar ou tratar de forma específica certos campos**, como listas aninhadas ou relações bidirecionais. Por exemplo, para **evitar que o `ModelMapper` substitua diretamente a lista de endereços sem realizar o vínculo reverso com o `Contact`**, podemos configurar um `TypeMap` que **ignore o mapeamento automático do campo `addresses`** e depois fazer o controle manual. Dessa forma, **ganharíamos um pouco mais de clareza e previsibilidade no mapeamento**, especialmente em estruturas mais complexas com relacionamentos bidirecionais. Isso serve o propósito de evitar possíveis erros de persistência, como violação de integridade referencial, e facilita a manutenção do código, pois o mapeamento torna-se explícito apenas onde realmente é necessário.

Novamente: não optamos por essa configuração justamente porque estamos seguindo uma abordagem **evolutiva e pedagógica** no desenvolvimento do projeto — o objetivo é **compreender passo a passo os motivos por trás de cada decisão**, construindo conhecimento sólido antes de introduzir abstrações mais avançadas.

Após esse ajuste, o contato é salvo novamente no repositório (persistido no banco de dados), e a entidade atualizada é convertida para `ContactResponseDTO`, que será retornado ao cliente com uma resposta `200 OK`.

### 🔎 Método 5: `updateContactPartial(Long id, ContactPatchDTO dto)`

```java
/**
* Atualiza parcialmente um contato existente.
* 
* @param id  identificador do contato
* @param dto dados a serem atualizados
* @return contato atualizado
* @throws ResourceNotFoundException se o contato não for encontrado
*/
@Operation(summary = "Atualizar contato parcialmente", description = "Atualiza apenas os campos especificados de um contato existente")
@ApiResponses(value = {
    @ApiResponse(responseCode = "200", description = "Contato atualizado com sucesso"),
    @ApiResponse(responseCode = "404", description = "Contato não encontrado"),
    @ApiResponse(responseCode = "403", description = "Acesso negado", content = @Content)
})
@PatchMapping("/{id}")
public ResponseEntity<ContactResponseDTO> updateContactPartial(@PathVariable Long id, @RequestBody ContactPatchDTO dto) {
    Contact existingContact = contactRepository.findById(id)
        .orElseThrow(() -> new ResourceNotFoundException("Contato não encontrado: " + id));
    dto.getNome().ifPresent(existingContact::setNome);
    dto.getEmail().ifPresent(existingContact::setEmail);
    dto.getTelefone().ifPresent(existingContact::setTelefone);
    Contact saved = contactRepository.save(existingContact);
    ContactResponseDTO responseDTO = modelMapper.map(saved, ContactResponseDTO.class);
    return ResponseEntity.ok(responseDTO);
}
```

O método `updateContactPartial` tem como objetivo realizar a **atualização parcial** de um contato existente no sistema, utilizando o padrão HTTP **PATCH**. Esse padrão indica que apenas parte dos dados de um recurso será modificada, ao contrário do PUT, que exige a substituição completa. Lembrem-se que abordamos esse conceito nas aulas anteriores. 🤓

Ao receber uma requisição com o identificador (`id`) do contato e um corpo contendo apenas os campos que devem ser atualizados, o método começa realizando uma busca pelo contato correspondente no repositório. Caso não exista, uma exceção do tipo `ResourceNotFoundException` é lançada, resultando em uma resposta HTTP 404 (Not Found).

O DTO utilizado neste método é o `ContactPatchDTO`, cuja principal característica é que seus atributos são declarados como `Optional<String>`. Essa escolha é intencional e permite diferenciar com clareza se um campo foi de fato enviado na requisição ou não. Por exemplo, `Optional.empty()` indica que o campo não foi enviado, enquanto `Optional.of("valor")` indica que o campo foi fornecido e deve ser processado.

A lógica do método utiliza o método `ifPresent` de cada `Optional` para atualizar apenas os campos que estão presentes. Veja o trecho:

```java
dto.getNome().ifPresent(existingContact::setNome);
dto.getEmail().ifPresent(existingContact::setEmail);
dto.getTelefone().ifPresent(existingContact::setTelefone);
```

Cada chamada acima verifica se o campo está presente no DTO. Se estiver, o valor é passado ao método `set` correspondente do objeto `Contact` existente. Caso não esteja, o campo permanece inalterado.

**🤔 E essa sintaxe com uso de `::`?**

A expressão `existingContact::setNome` (bem como as demais) representa uma **referência de método** (ou *method reference*), uma funcionalidade introduzida no Java 8 que permite passar métodos como argumentos de forma mais concisa e legível. No contexto do código `dto.getNome().ifPresent(existingContact::setNome);`, essa referência está sendo usada para aplicar uma função ao valor presente em um `Optional`.

Mais especificamente, `existingContact::setNome` é uma forma simplificada de escrever uma *lambda expression* como `nome -> existingContact.setNome(nome)`. Ambas executam a mesma operação, mas a versão com `::` torna o código mais enxuto e elegante. O método `ifPresent` da classe `Optional` aceita um argumento do tipo `Consumer<T>`, ou seja, uma função que recebe um parâmetro e retorna `void`. Como `setNome(String nome)` satisfaz essa exigência — recebe uma `String` e não retorna nada — ele pode ser referenciado diretamente com a sintaxe `existingContact::setNome`.

Mesmo que o método `setNome` não esteja declarado explicitamente na classe `Contact`, o uso do Lombok (através de anotações como `@Data`, `@Setter`, ou `@Builder`) garante que esse método seja gerado em tempo de compilação. Isso significa que o compilador reconhece e permite o uso da referência ao método, mesmo que ele não esteja visível no código-fonte.

Ou seja, há duas formas possíveis de aplicar o valor de `dto.getNome()` ao objeto `existingContact`:

**Com lambda:**
```java
dto.getNome().ifPresent(nome -> existingContact.setNome(nome));
```

**Com method reference (forma preferida):**
```java
dto.getNome().ifPresent(existingContact::setNome);
```

A segunda forma é preferida quando possível, pois reduz o boilerplate, melhora a clareza e facilita a leitura do código. Esse tipo de recurso é especialmente útil em operações com `Optional`, `Stream`, ou qualquer outra API funcional introduzida no Java 8.

Capisci? 🤌

Voltando ao fluxo de execução do nosso método, após aplicar as alterações o objeto atualizado é salvo no banco de dados com `contactRepository.save(existingContact)`. Em seguida, o objeto persistido é convertido em um DTO de resposta (`ContactResponseDTO`) por meio do `ModelMapper`, e retornado ao cliente com o status HTTP 200 (OK).

### 🔎 Método 6: `deleteContact(Long id)`

```java
/**
 * * Exclui um contato.
* 
* @param id identificador do contato
* @return resposta sem conteúdo
* @throws ResourceNotFoundException se o contato não for encontrado
*/
@Operation(summary = "Excluir contato", description = "Remove permanentemente um contato do sistema")
@ApiResponses(value = {
    @ApiResponse(responseCode = "204", description = "Contato excluído com sucesso"),
    @ApiResponse(responseCode = "404", description = "Contato não encontrado"),
    @ApiResponse(responseCode = "403", description = "Acesso negado", content = @Content)
})
@DeleteMapping("/{id}")
public ResponseEntity<Void> deleteContact(@PathVariable Long id) {
    if (!contactRepository.existsById(id)) {
        throw new ResourceNotFoundException("Contato não encontrado: " + id);
    }
    contactRepository.deleteById(id);
    return ResponseEntity.noContent().build();
}
```

Outro método que já vimos na seção anterior, mas só para reforçar: este método exclui um contato com base no `id`. A anotação `@DeleteMapping` indica uma operação de deleção. Antes da exclusão, é verificado se o contato realmente existe com `existsById`. Se não existir, uma exceção é lançada. 

Se a exclusão ocorrer com sucesso, o retorno é `ResponseEntity.noContent().build()` com o código de status HTTP 204, que indica sucesso sem conteúdo: `.noContent()` é um método estático da classe ResponseEntity que retorna um `ResponseEntity.BodyBuilder` com o código de status HTTP 204 já configurado e `.build()` é o método que finaliza a construção da resposta sem corpo (ou seja, com null no body).

### 🔎 Método 7: `searchContactsByName(String name, Pageable pageable)`

```java
/**
* * Busca contatos pelo nome.
*
* @param name     nome ou parte do nome a ser pesquisado
* @param pageable informações de paginação
* @return lista paginada de contatos que correspondem ao critério de busca
*/
@Operation(summary = "Buscar contatos por nome", description = "Retorna uma lista paginada de contatos cujo nome contém o termo pesquisado")
@ApiResponses(value = {
    @ApiResponse(responseCode = "200", description = "Busca realizada com sucesso"),
    @ApiResponse(responseCode = "403", description = "Acesso negado", content = @Content)
})
@GetMapping("/search")
public ResponseEntity<Page<ContactResponseDTO>> searchContactsByName(@RequestParam String name, Pageable pageable) {
    Page<Contact> contacts = contactRepository.findByNomeContainingIgnoreCase(name, pageable);
    Page<ContactResponseDTO> responseDTO = contacts
        .map(contact -> modelMapper.map(contact, ContactResponseDTO.class));
    return ResponseEntity.ok(responseDTO);
}
```

O método `searchContactsByName` é responsável por realizar uma busca paginada de contatos cujo nome contenha determinada substring informada como parâmetro da requisição. O endpoint está mapeado para o caminho `/api/contacts/search` e responde a requisições HTTP GET.

Novamente, todos os conceitos abordados abaixo já foram vistos, mas vamos reforçá-los mesmo assim. 😌

A busca funciona a partir do uso da anotação `@RequestParam String name`, que indica que o parâmetro `name` será recebido por meio da **query string** da requisição. Por exemplo, uma chamada à URL `/api/contacts/search?name=ana` resultará em uma busca por todos os contatos cujo nome contenha o texto "ana", independentemente de letras maiúsculas ou minúsculas, graças ao uso do método `findByNomeContainingIgnoreCase` no repositório (visto na aula anterior!).

O segundo parâmetro do método é um objeto `Pageable`, que é injetado automaticamente pelo Spring com base nos parâmetros da requisição. Mais uma vez: a interface `Pageable` encapsula informações sobre a **página atual**, o **tamanho da página** e o **critério de ordenação**, todos fornecidos via query string (ex: `?page=0&size=10&sort=nome,asc`). O Spring cuida de interpretar esses valores e construir um objeto `Pageable` correspondente.

Com o `Pageable` em mãos, o método consulta o repositório usando `findByNomeContainingIgnoreCase`, que retorna um objeto `Page<Contact>`. Essa interface representa não apenas os dados, mas também metadados da consulta, como o número total de páginas, o número da página atual, e o número total de elementos. Isso é extremamente útil em APIs REST que precisam fornecer paginação para o cliente de forma eficiente e padronizada.

Em seguida, cada objeto `Contact` retornado pela consulta é convertido em um `ContactResponseDTO`. No caso, usamos uma **expressão lambda** com o `ModelMapper` para realizar essa conversão: `contact -> modelMapper.map(contact, ContactResponseDTO.class)`.

Por fim, o resultado — agora representado como `Page<ContactResponseDTO>` — é encapsulado em um `ResponseEntity` com código de status 200 (OK), utilizando `ResponseEntity.ok(...)`. O corpo da resposta inclui os dados paginados dos contatos, além de informações úteis para o frontend, como a quantidade total de elementos, total de páginas, tamanho da página, e indicadores se é a primeira ou última página.

Essa abordagem oferece uma implementação aderente aos princípios REST. Além disso, é flexível o suficiente para permitir futuras evoluções, como o uso de HATEOAS (com `PagedModel`) ou a personalização de ordenações e filtros.

---

### 🧠 Considerações Finais sobre a classe `ContactController`

A classe `ContactController`, agora refatorada, apresenta uma estrutura alinhada com as boas práticas do desenvolvimento de APIs REST com Spring Boot. O uso de `Pageable` como parâmetro nos métodos permite o controle eficiente sobre a paginação e ordenação dos dados, tornando a API mais escalável e flexível diante de grandes volumes de registros. O retorno como `Page` carrega, além dos dados em si, informações úteis como número total de elementos, total de páginas, número da página atual, e indicadores de primeira ou última página, facilitando bastante a implementação de interfaces paginadas no cliente.

Outro ponto positivo é a adoção do `ModelMapper`, que fizemos na última aula, evitando mapeamentos manuais repetitivos entre entidades e DTOs, reduzindo o boilerplate e aumentando a legibilidade do código. Ainda assim, a flexibilidade oferecida pelo `ModelMapper` permite ajustes e personalizações pontuais, como no caso da relação entre `Contact` e `Address`, em que foi necessário realizar o “bind reverso” para manter a integridade relacional.

O uso do `ResponseEntity` em todos os métodos também é um ponto de destaque. Essa abordagem confere à API mais controle e clareza, permitindo especificar de forma explícita o código de status HTTP, cabeçalhos adicionais e o corpo da resposta — o que é essencial em cenários mais complexos, como tratamento de erros, redirecionamentos ou retornos com metadados.

A classe também está documentada com o uso de anotações da especificação OpenAPI, como `@Operation` e `@ApiResponses`. Isso facilita a geração de documentação interativa via Swagger UI, tornando a API mais acessível para desenvolvedores e stakeholders.

No entanto, apesar da clareza da implementação atual, ainda há **melhorias que podem ser introduzidas** para fortalecer ainda mais a arquitetura da aplicação. A principal delas é a **separação da lógica de negócio para uma camada de serviço (`@Service`)**, o que promoveria uma melhor organização e reutilização do código. Atualmente, o controller executa operações como a verificação de existência de recursos, salvamento de entidades e manipulação direta de objetos de domínio. Com a introdução de uma camada de serviço, o controller se tornaria mais enxuto e focado apenas na orquestração das requisições e respostas HTTP, enquanto as regras de negócio passariam a ser encapsuladas de forma reutilizável e testável. Para evitar tomar mais tempo nesse projeto, visto que temos ainda que introduzir conceitos de testes e segurança, faremos essa separação de camadas em um projeto posterior.

Além disso, a aplicação pode evoluir para usar **Spring HATEOAS** (Hypermedia as the Engine of Application State), permitindo que as respostas incluam links de navegação e ações relacionadas ao recurso (como "próxima página", "editar", "excluir"). Essa abordagem melhora a **descoberta de recursos** pela API e oferece um ganho significativo na usabilidade e autoexplicação da interface REST. Além disso, há também a possibilidade de definir **DTOs específicos para retorno paginado**, ao invés de expor diretamente a estrutura do `Page<T>`. Essa abordagem melhora a clareza da documentação e permite adicionar informações extras ao corpo da resposta, como mensagens personalizadas ou estatísticas agregadas. Além disso, isso garantiria que nossas respostas estivessem desacopladas da estrutura interna da implementação do `Page<T>`. Ou seja: mesmo que a estrutura do Spring mude, nossa resposta permanece a mesma, garantindo que não quebraríamos nossos clientes.

Por fim, na nossa `AddressController` é necessário aplicar as mesmas melhorias que fizemos na `ContactController`. Para fins pedagógicos, não mostrarei a implementação! Portanto, sua tarefa agora é **fazer as modificações e testar as refatorações na `AddressController`!**

Dito isso, vamos verificar agora como podemos implementar testes na nossa aplicação!
