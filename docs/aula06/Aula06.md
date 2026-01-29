# Aula 06 – Refatoração dos Controllers, camada de serviço e Introdução aos Testes Automatizados

Na Aula 05, estruturamos nossa API com uso de DTOs, paginamos os endpoints, substituímos o banco em memória por um banco relacional real e documentamos todos os nossos recursos com Swagger. Agora, avançaremos um passo além na maturidade da nossa aplicação: vamos realizar uma **refatoração completa dos nossos controllers**, otimizar o uso do `ModelMapper` para reduzir mapeamentos manuais, separar a lógica da controller na camada de serviço e, ainda, iniciaremos a **introdução aos testes automatizados** da aplicação.

Como sempre, seguiremos um processo prático dando início de onde paramos na aula anterior: nosso exercício pedia. Começaremos com a **refatoração dos endpoints**, adotando o padrão `ResponseEntity` — um dos mais recomendados para retornos HTTP em APIs Spring Boot.

---

## 1. Refatorando nossos Controllers com `ResponseEntity`

Até aqui, utilizamos retornos diretos como `ContactResponseDTO` ou `Page<ContactResponseDTO>` nos métodos de nossos controllers. Embora essa abordagem funcione bem, ela **não oferece controle detalhado sobre o que está sendo retornado na resposta HTTP**, como cabeçalhos, status específicos ou metadados adicionais - para isso precisamos usar anotações como `@ResponseStatus(HttpStatus.OK)` e outros artifícios de implementação.

O uso da classe `ResponseEntity<T>` resolve essa limitação, fornecendo uma interface simplificada para uso. Essa classe encapsula:
- o **corpo da resposta** (um DTO, por exemplo),
- o **status HTTP** (200 OK, 201 Created, 204 No Content, 404 Not Found, etc),
- e opcionalmente **cabeçalhos adicionais**.

### 1.1 Por que usar `ResponseEntity`?

Adotar o `ResponseEntity` proporciona mais **clareza, flexibilidade e aderência aos padrões HTTP**, além de preparar a aplicação para requisitos mais avançados, como:
- inclusão de cabeçalhos de autenticação ou cache,
- retorno de localizações (`Location`) após criação de recursos,
- respostas com conteúdos customizados ou sem corpo (`204 No Content`),
- e suporte facilitado a testes e a respostas específicas em diferentes cenários.

Além disso, ele torna explícito para quem lê o código qual é o status retornado pela requisição, o que melhora a manutenção e a legibilidade da aplicação, sendo o uso de `ResponseEntity` a implementação preferida pela comunidade Spring.

### ✨ Vantagens do uso de `ResponseEntity`

| Recurso                             | Benefício                                                   |
|-------------------------------------|-------------------------------------------------------------|
| Controle explícito do status        | Podemos retornar 200, 201, 204, 400, 404, 500, etc.         |
| Headers customizados                | Podemos adicionar cabeçalhos HTTP (como `Location`)        |
| Corpo da resposta opcional          | Podemos retornar apenas status, sem corpo (`noContent()`)  |
| Adequação a testes e versionamento  | Facilita asserções em testes e torna o contrato mais claro |

### 1.2 Sintaxe básica de `ResponseEntity`

O `ResponseEntity` possui uma forma mais verbosa de uso, como mostrado abaixo

```java
return ResponseEntity
        .status(HttpStatus.CREATED)
        .body(contactResponseDTO); 
```

E uma forma mais enxuta com métodos auxiliares:

```java
return ResponseEntity.ok(dto); // 200 OK com corpo
return ResponseEntity.noContent().build(); // 204 No Content
```

### 1.3 Refatorando um exemplo de endpoint

Antes da refatoração:

```java
@DeleteMapping("/{id}")
@ResponseStatus(HttpStatus.NO_CONTENT)
public void deleteContact(@PathVariable Long id) {
    contactRepository.deleteById(id);
}
```

Após refatoração:

```java
@DeleteMapping("/{id}")
public ResponseEntity<Void> deleteContact(@PathVariable Long id) {
    if (!contactRepository.existsById(id)) {
        throw new ResourceNotFoundException("Contato não encontrado: " + id);
    }
    contactRepository.deleteById(id);
    return ResponseEntity.noContent().build();
}
```

Observe que com `ResponseEntity` ganhamos **clareza semântica**. Além disso, adicionamos a cláusula de verificação para checar se o id passado como argumento existe em nossa base de dados, já que por algum motivo desconhecido havia me esquecido de implementar isso anteriormente. 🫠

### 1.4 Padrão a ser adotado nos métodos

A seguir estão os padrões que iremos aplicar na refatoração dos métodos da controller:

| Tipo de operação     | Código HTTP | Exemplo Spring                                 |
|----------------------|-------------|-----------------------------------------------|
| Buscar recurso       | 200 OK      | `ResponseEntity.ok(resource)`                 |
| Criar recurso        | 201 Created | `ResponseEntity.status(CREATED).body(resource)`|
| Atualizar recurso    | 200 OK      | `ResponseEntity.ok(resource)`                 |
| Atualizar parcial    | 200 OK      | `ResponseEntity.ok(resource)`                 |
| Deletar recurso      | 204 NoContent | `ResponseEntity.noContent().build()`         |
| Recurso não encontrado | 404 Not Found | Lançar `ResourceNotFoundException` para ser tratada globalmente |


### 1.5 Refatorando os métodos do Controller

Além da refatoração do endpoint de deleção de um contato, vejamos a refatoração de mais alguns dos métodos do `ContactController` e `AddressController`, substituindo os retornos diretos pelos retornos com `ResponseEntity`. O objetivo é tornar os endpoints mais claros e preparados para evoluções, como headers, cache, redirecionamentos ou alterações no corpo da resposta.

Para manter a aula leve e compreensível, mostraremos alguns métodos como exemplo a seguir. A versão final de todos os métodos será apresentada ao final da seção, como fizemos na Aula 05.

### 🧱 Exemplo 1: `createContact()`

Na aula anterior nosso método estava da seguinte forma:

```java
@PostMapping
@ResponseStatus(HttpStatus.CREATED)
public ContactResponseDTO createContact(@Valid @RequestBody ContactRequestDTO dto) {
    // Mapeia os campos simples
    Contact contact = new Contact(dto.getNome(), dto.getEmail(), dto.getTelefone());
            
    // Mapeia os endereços manualmente
    var addresses = dto.getAddresses().stream()
        .map(addrDto -> {
            Address address = new Address();
            address.setRua(addrDto.getRua());
            address.setCidade(addrDto.getCidade());
            address.setEstado(addrDto.getEstado());
            address.setCep(addrDto.getCep());
            address.setContact(contact); 
            return address;
        }).toList();
            
    contact.setAddresses(addresses);
            
    Contact saved = contactRepository.save(contact);
    return modelMapper.map(saved, ContactResponseDTO.class);
}
```

Agora, vamos refatorar nosso método e deixá-lo como mostrado a seguir:

```java
@PostMapping
public ResponseEntity<ContactResponseDTO> createContact(@Valid @RequestBody ContactRequestDTO dto) {
    Contact contact = modelMapper.map(dto, Contact.class);
    contact.getAddresses().forEach(address -> address.setContact(contact));
    Contact saved = contactRepository.save(contact);
    ContactResponseDTO responseDTO = modelMapper.map(saved, ContactResponseDTO.class);
    return ResponseEntity.status(HttpStatus.CREATED).body(responseDTO);
}
```

No primeiro método a criação de um novo contato é realizada de maneira bastante manual. Inicialmente, os campos do objeto `Contact` são populados diretamente a partir dos valores do DTO, utilizando um construtor explícito. Em seguida, os endereços são mapeados um a um através de um `stream`, onde cada `AddressRequestDTO` é convertido manualmente em uma instância da entidade `Address`. Durante esse processo, também é feita a associação entre cada endereço e o contato recém-criado, garantindo o vínculo bidirecional necessário para persistência correta com JPA. Após o mapeamento, a lista de endereços é atribuída ao contato e, por fim, o contato é salvo no banco e convertido em um `ContactResponseDTO` utilizando o `ModelMapper`.

Essa abordagem, embora funcional, gera um acúmulo de responsabilidades no controller e uma repetição considerável de código, o que pode dificultar a manutenção à medida que a aplicação cresce.

Já o segundo método apresenta uma versão mais enxuta e elegante da mesma funcionalidade. Nessa versão refatorada, o `ModelMapper` é utilizado diretamente para mapear o `ContactRequestDTO` para a entidade `Contact`, eliminando a necessidade de instanciar o objeto manualmente e escrever código repetitivo para setar os atributos. O único passo que permanece manual é a associação entre os endereços e o contato — e isso é feito de forma simples e clara, com um `forEach`. Após o salvamento da entidade no banco, o resultado é novamente convertido para o DTO de resposta com o `ModelMapper`.

Outro ponto de melhoria é o uso do `ResponseEntity` para retornar a resposta da API. Com isso, temos controle explícito sobre o status HTTP (201 Created), o que torna a resposta mais alinhada às práticas RESTful e facilita o envio de cabeçalhos adicionais, se necessário.

Essa refatoração traz diversos benefícios: o código fica mais limpo, a responsabilidade de conversão entre DTOs e entidades é centralizada no `ModelMapper`, e o controller passa a ser responsável apenas por orquestrar as chamadas — o que é exatamente seu papel. Além disso, a nova versão favorece a legibilidade, testabilidade e manutenção do código, características essenciais em projetos profissionais e de médio a longo prazo.

Uma melhoria posterior seria a implementação de uma camada de serviço, por meio da extração da lógica e simplificação ainda maior do nosso Controller. Abordaremos esse padrão organizacional posteriormente na aula.

### 🧱 Exemplo 2: `getAllContacts()`

Na aula anterior nosso método estava da seguinte forma:

```java
@GetMapping
public Page<ContactResponseDTO> getAllContacts(Pageable pageable) {
    Page<Contact> contacts = contactRepository.findAll(pageable);
    return contacts.map(contact -> modelMapper.map(contact, ContactResponseDTO.class));
}            
```

Agora, vamos refatorar nosso método e deixá-lo como mostrado a seguir:

```java
@GetMapping
public ResponseEntity<Page<ContactResponseDTO>> getAllContacts(Pageable pageable) {
    Page<Contact> contacts = contactRepository.findAll(pageable);
    Page<ContactResponseDTO> responseDTO = contacts
        .map(contact -> modelMapper.map(contact, ContactResponseDTO.class));
    return ResponseEntity.ok(responseDTO);
}
```

Como percebem, seguimos a mesma linha de refatoração que fizemos anteriormente, ou seja, a diferença entre os dois métodos está na forma como a resposta é construída e retornada para o cliente. Ambos realizam a mesma tarefa essencial: recuperar todos os contatos de forma paginada e convertê-los para um Page<ContactResponseDTO>, utilizando o ModelMapper para transformar os objetos da entidade Contact em objetos de transferência de dados (DTO). No entanto, a segunda versão aplica uma refatoração importante ao envolver o retorno em um ResponseEntity, enquanto a primeira retorna diretamente o Page resultante.

Na primeira versão a resposta da requisição é retornada de maneira direta. O Spring Boot, por padrão, interpreta o tipo de retorno e aplica um status HTTP 200 (OK) automaticamente. Esse estilo é válido e perfeitamente funcional, principalmente em projetos menores ou em endpoints que não exigem personalizações adicionais no cabeçalho da resposta ou no status. No entanto, essa abordagem oferece menos controle sobre o que está sendo retornado, já que não há flexibilidade para modificar status HTTP, cabeçalhos ou outras configurações da resposta de forma explícita.

Já a versão refatorada segue uma prática mais robusta e alinhada às boas práticas em APIs RESTful: utiliza o ResponseEntity, para representar toda a resposta HTTP, incluindo corpo, status e cabeçalhos. Ao encapsular a resposta em um ResponseEntity.ok(...), o método deixa claro e explícito que está retornando uma resposta com status HTTP 200 (OK), além de permitir, se necessário, o uso de outros métodos como .status(), .headers(), ou até mesmo .noContent() para outros cenários.

### 🤠 Resumo 

Refatorar os métodos do controller para retornar `ResponseEntity` é um pequeno ajuste com **impacto positivo** na legibilidade, testabilidade e padronização da API. Essa abordagem permite que, futuramente, adicionemos headers, links, status alternativos ou tratamentos especiais sem precisar alterar a assinatura do método.

Além disso, a documentação gerada pelo Swagger/OpenAPI também pode ser enriquecida com status HTTP mais precisos e podemos alterar a injeção de dependência que vinhamos usando com `@Autowired` para injeção via construtor!

Vamos implementar essas refatorações e rever o nosso código completo do controller.

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

Nesta etapa, vimos alguns testes automatizados para o `ContactController`, cobrindo tanto os casos esperados quanto os de erro. Com isso, nossa API começa a se aproximar de um projeto profissional: funcional, testável e documentada.

É importante ressaltar, entretanto, que estamos loooonge de completar todos os testes da aplicação. Por isso é importante não negligenciar o desenvolvimento dos testes ao implementarmos nossas features. 🤓

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