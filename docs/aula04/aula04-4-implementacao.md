---
layout: aula
title: 4. Vamos à prática!
parent: Aula 04 - Spring Boot - JPA, Exceções e Documentação!
nav_order: 4
---

## **🔍4. Implementação e Análise dos Códigos-fontes**

Passemos, agora, a análise dos códigos que implementam os Desafios 1 e 2 da Aula 03. Para fins de brevidade e objetividade da explicação, apresentaremos todos em sequência. 

- Caso exista algum erro de código-fonte abaixo, avise ao professor! Eu implemento e testo tudo, claro, mas sempre estamos sujeitos à algum deslize. Todas as correções pertinentes serão incorporadas!

### **4.1 ✅Código da Entidade `Address`**

A classe `Address` foi implementada abaixo com os atributos e validações necessárias, de forma a atender os Desafios 1 e 2. Vamos analisar o código:

```java
@Entity
public class Address {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NotBlank(message = "A rua não pode estar vazia")
    private String rua;

    @NotBlank(message = "A cidade não pode estar vazia")
    private String cidade;

    @NotBlank(message = "O estado não pode estar vazio")
    @Size(min = 2, max = 2, message = "O estado deve ter exatamente 2 caracteres (sigla)")
    @Pattern(regexp = "[A-Z]{2}", message = "O estado deve ser representado por duas letras maiúsculas")
    private String estado;

    @NotBlank(message = "O CEP não pode estar vazio")
    @Pattern(regexp = "\\d{5}-\\d{3}", message = "O CEP deve estar no formato 99999-999")
    private String cep;

    @ManyToOne
    @JoinColumn(name = "contact_id", nullable = false)
    @JsonBackReference
    private Contact contact;
}
```

### **🔍Analisando o código**  
- A classe define os atributos exigidos: `rua`, `cidade`, `estado`, `cep` e a relação com o `Contact`.
- Utiliza anotações de validação (`@NotBlank`, `@Size`, `@Pattern`) para garantir que os dados sejam válidos antes de serem persistidos.
- A anotação `@ManyToOne` define a relação com a classe `Contact`, com a coluna `contact_id` no banco de dados.
- A anotação `@JsonBackReference` é usada para evitar problemas de referência cíclica ao serializar os objetos (`Contact` e `Address`) para JSON. Quando uma entidade possui uma coleção de entidades relacionadas, como acontece em relações `OneToMany`, é necessário indicar que a serialização de JSON deve ignorar a referência de volta para o proprietário da relação. Dessa forma, evitamos erros como `StackOverflowException` quando a conversão para JSON é realizada.

Além disso, percebam que por enquanto vamos transitar diretamente a entidade entre as diferentes camadas da aplicação, inclusive expondo dados como o ID das entidades para o client. É por isso também que temos que usar as anotações para evitar problemas de serialização. A solução para isso seria implementar **DTOs (Data Transfer Objects)**. DTOs são classes criadas para transportar dados entre diferentes camadas da aplicação. Quando implementamos DTOs estamos essencialmente **separando a representação de dados da API (o que é retornado ou recebido pelo cliente) da modelagem interna da nossa aplicação** (as entidades JPA que manipulam o banco de dados). Os DTOs são classes que contêm apenas os dados que você quer expor na API, sem manter relações direcionais ou bidirecionais presentes nas entidades.

Essa abordagem é melhor por alguns motivos:

1. Permite a **Independência da camada de persistência**, onde o que é exposto na API não precisa ter o mesmo formato do banco de dados.
2. **Evita problemas de serialização cíclica:** As entidades JPA podem manter suas relações bidirecionais, mas o Jackson só verá os DTOs.
3. **Segurança:** Permite controlar exatamente o que é exposto ao cliente, sem expor dados sensíveis ou desnecessários.
4. **Flexibilidade:** Facilita a evolução da API sem alterar diretamente a estrutura das entidades no banco de dados.

Na próxima aula faremos essa melhoria no código, mas por enquanto basta que fiquem cientes que a abordagem adotada até aqui é a mais didática, só que não necessariamente a mais adequada.

Evidentemente, também devemos atualizar a classe Contact. A implementação completa pode ser vista a seguir.


### **4.2 ✅Código atualizado da Entidade `Contact`**

```java

package br.ifsp.contacts.model;

@Entity
public class Contact {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @NotBlank(message = "O nome não pode estar vazio")
    private String nome;
    
    @NotBlank(message = "O email não pode estar vazio")
    @Email(message = "Formato de email inválido")
    private String email;
    
    @NotBlank(message = "O telefone não pode estar vazio")
    @Size(min = 8, max = 15, message = "O telefone deve ter entre 8 e 15 caracteres")
    @Pattern(regexp = "\\d+", message = "O telefone deve conter apenas números")
    private String telefone;
    
    @OneToMany(mappedBy = "contact", cascade = CascadeType.ALL, orphanRemoval = true)
    @JsonManagedReference
    @NotEmpty(message = "O contato deve ter pelo menos um endereço")
    private List<Address> addresses = new ArrayList<>();
    
    public Contact() {
    }
    
    public Contact(String nome, String email, String telefone) {
        this.nome = nome;
        this.email = email;
        this.telefone = telefone;
    }
    
    public Long getId() {
        return id;
    }
    
    public String getNome() {
        return nome;
    }
    
    public void setNome(String nome) {
        this.nome = nome;
    }
    
    public String getEmail() {
        return email;
    }
    
    public void setEmail(String email) {
        this.email = email;
    }
    
    public String getTelefone() {
        return telefone;
    }
    
    public void setTelefone(String telefone) {
        this.telefone = telefone;
    }
    
    public List<Address> getAddresses() {
        return addresses;
    }
    
    public void setAddresses(List<Address> addresses) {
        if (addresses != null) {
            addresses.forEach(address -> address.setContact(this)); 
            
            if (this.addresses == null) { 
                this.addresses = new ArrayList<>();
            }
            
            this.addresses.clear(); 
            this.addresses.addAll(addresses);         
        }
    }
}
```
### **🔍Analisando o código**  

A classe `Contact` é uma entidade JPA que representa um **contato** na aplicação e está mapeada para uma tabela no banco de dados por meio da anotação `@Entity`. Esta classe contém diversos atributos, incluindo uma lista de endereços (`List<Address> addresses`) que se relaciona diretamente com a entidade `Address`. A seguir, vamos detalhar cada parte da classe e como ela se relaciona com a classe `Address`.

### 🔍 **Atributos da Classe `Contact`**
- `id`: É o identificador único do contato (`Long`) e é gerado automaticamente pelo banco de dados através da anotação `@GeneratedValue(strategy = GenerationType.IDENTITY)`.
- `nome`: Nome do contato, com validação para garantir que não seja vazio (`@NotBlank`).
- `email`: Endereço de email do contato, validado tanto para não ser vazio (`@NotBlank`) quanto para ter um formato válido (`@Email`).
- `telefone`: Número de telefone do contato, validado para ter entre 8 e 15 caracteres (`@Size`), apenas números (`@Pattern`), e não ser vazio (`@NotBlank`).
- `addresses`: É uma lista de endereços (`Address`) associados a este contato. Essa relação é estabelecida por meio de uma associação **OneToMany**.

O relacionamento entre `Contact` e `Address` é **bidirecional** e configurado com as anotações `@OneToMany` na classe `Contact` e `@ManyToOne` na classe `Address`, como visto na seção anterior. Vamos ver detalhamente o mapeamento, que é feito da seguinte maneira:

#### 📌 **Na Classe `Contact` (Dono do relacionamento):**
```java
@OneToMany(mappedBy = "contact", cascade = CascadeType.ALL, orphanRemoval = true)
@JsonManagedReference
@NotEmpty(message = "O contato deve ter pelo menos um endereço")
private List<Address> addresses = new ArrayList<>();
```

- `@OneToMany`: Define que um contato pode ter vários endereços.
- `mappedBy = "contact"`: Indica que o relacionamento é mapeado pelo atributo `contact` na classe `Address`.
- `cascade = CascadeType.ALL`: Propaga todas as operações (persistência, remoção, atualização) feitas na entidade `Contact` para os seus endereços relacionados.
- `orphanRemoval = true`: Remove automaticamente os endereços que são removidos da lista `addresses`.
- `@JsonManagedReference`: Trabalha junto com `@JsonBackReference` na classe `Address` para evitar problemas de serialização cíclica (loop infinito) quando os objetos são convertidos para JSON.
- `@NotEmpty`: Garante que o contato sempre tenha pelo menos um endereço associado. Esse tipo de validação garante que cada contato deve ser salvo com pelo menos um endereço válido, evitando registros incompletos. 

É importante notar que o método `setAddresses()` é implementado de forma a garantir que todos os endereços associados a este contato estejam sincronizados. Observe o trecho a seguir:

```java
public void setAddresses(List<Address> addresses) {
    if (addresses != null) {
        addresses.forEach(address -> address.setContact(this)); 
        
        if (this.addresses == null) { 
            this.addresses = new ArrayList<>();
        }
        
        this.addresses.clear(); 
        this.addresses.addAll(addresses);         
    }
}
```

Este método faz o seguinte:
1. **Associa o contato atual a todos os endereços recebidos na lista:** Para cada endereço fornecido, o método `address.setContact(this)` é chamado para garantir que o relacionamento bidirecional seja atualizado.
2. **Inicializa a lista de endereços se for nula:** Garante que a lista nunca será manipulada sem ser inicializada.
3. **Limpa a lista de endereços atual:** Evita duplicação de dados e garante que a lista de endereços seja atualizada completamente.
4. **Adiciona todos os endereços novos:** Depois de limpar a lista, os novos endereços são adicionados, mantendo a consistência.

A partir das duas entidades acima, temos o relacionamento completo. Caso tivéssemos mais entidades e regras de negócio mais complexas, a ideia se manteria a mesma: as validaríamos e relacionaríamos conforme a necessidade se apresentasse.

Passemos agora à análise da nossa classe `AddressRepository`.

### **4.3 ✅Repositório `AddressRepository`**

O repositório foi implementado de forma simples. Vamos conferir:

```java
@Repository
public interface AddressRepository extends JpaRepository<Address, Long> {
    List<Address> findByContactId(Long contactId);
}
```

- A interface estende `JpaRepository<Address, Long>`, o que significa que herda todos os métodos básicos de manipulação (`save()`, `delete()`, `findById()`, etc.).
- O método `findByContactId(Long contactId)` é adicionado para recuperar endereços associados a um contato específico.
- A convenção de nomes usada (`findByContactId`) é suficiente para que o Spring Data JPA crie a consulta SQL apropriada automaticamente.

Ou seja: até aqui, nenhuma novidade na criação do Repositório! Vamos verificar agora como ficaram nossos controladores.


### **4.4 ✅Controller `AddressController`**

A classe `AddressController` é um controlador REST responsável por manipular requisições HTTP relacionadas à entidade `Address` na aplicação. Ela é anotada com `@RestController`, o que indica que seus métodos retornarão dados diretamente no corpo da resposta (em formato JSON, por exemplo). Além disso, a anotação `@RequestMapping("/api/addresses")` define o caminho base para todos os endpoints dentro deste controlador. Vejamos o código abaixo.


```java
package br.ifsp.contacts.controller;

@RestController
@RequestMapping("/api/addresses")
public class AddressController {
    
    @Autowired
    private ContactRepository contactRepository;

    @Autowired
    private AddressRepository addressRepository;
    
    @GetMapping("/contacts/{contactId}")
    public List<Address> getAddressesByContact(@PathVariable Long contactId) {
        Contact contact = contactRepository.findById(contactId)
                .orElseThrow(() -> new ResourceNotFoundException("Contato não encontrado: " + contactId));
        
        return contact.getAddresses();
    }
    
    @PostMapping("/contacts/{contactId}")
    @ResponseStatus(HttpStatus.CREATED)
    public Address createAddress(@PathVariable Long contactId, @RequestBody @Valid Address address) {
        Contact contact = contactRepository.findById(contactId)
                .orElseThrow(() -> new ResourceNotFoundException("Contato não encontrado: " + contactId));
        
        address.setContact(contact);
        return addressRepository.save(address);
    }
}

```
### 🔍 **Analisando o código:**  

### 📌 **Injeção de Dependências (`@Autowired`)**
A classe utiliza injeção de dependências para acessar os repositórios `ContactRepository` e `AddressRepository`, que são necessários para buscar contatos existentes e salvar novos endereços. Essa injeção é feita automaticamente pelo Spring por meio da anotação `@Autowired`, como já vimos anteriormente. Nas próximas aulas vamos explorar também a injeção de dependências por meio do método construtor, que é a alternativa mais recomendada pelos desenvolvedores do Spring.

### 📌 **Método `getAddressesByContact()`**
```java
@GetMapping("/contacts/{contactId}")
public List<Address> getAddressesByContact(@PathVariable Long contactId) {
    Contact contact = contactRepository.findById(contactId)
            .orElseThrow(() -> new ResourceNotFoundException("Contato não encontrado: " + contactId));
    
    return contact.getAddresses();
}
```
Esse método é um endpoint `GET` que permite recuperar todos os endereços associados a um contato específico. Ele é acessível por meio da URL:

```
GET /api/addresses/contacts/{contactId}
```

**O que acontece nesse método:**  
1. O método recebe um `contactId` como parâmetro de URL e o busca no banco de dados usando o `contactRepository.findById()` que retorna um `Optional<Contact>`.
2. Se o contato não for encontrado, o método `orElseThrow()` lança uma exceção `ResourceNotFoundException`, que é tratada pelo `GlobalExceptionHandler` e resulta em um retorno com código HTTP 404 (Not Found).
3. Caso o contato seja encontrado, o método retorna a lista de endereços associados ao contato por meio do método `contact.getAddresses()`.  
4. A resposta retornada é uma lista de objetos `Address` convertida automaticamente para JSON pelo Jackson. Esse passo é transparente ao usarmos o Spring Boot.  

### 📌 **Método `createAddress()`**
```java
@PostMapping("/contacts/{contactId}")
@ResponseStatus(HttpStatus.CREATED)
public Address createAddress(@PathVariable Long contactId, @RequestBody @Valid Address address) {
    Contact contact = contactRepository.findById(contactId)
            .orElseThrow(() -> new ResourceNotFoundException("Contato não encontrado: " + contactId));
    
    address.setContact(contact);
    return addressRepository.save(address);
}
```
Esse método é um endpoint `POST` que permite criar um novo endereço e associá-lo a um contato existente. Ele é acessível por meio da URL:

```
POST /api/addresses/contacts/{contactId}
```

**O que acontece nesse método:**  
1. O método recebe um `contactId` como parâmetro de URL e um objeto `Address` como corpo da requisição (`@RequestBody`).
2. A anotação `@Valid` é usada para garantir que o endereço enviado atende a todas as regras de validação definidas na classe `Address`.
3. O controlador busca o contato correspondente ao `contactId` usando `contactRepository.findById()`. Caso não seja encontrado, é lançada uma exceção `ResourceNotFoundException`.
4. Se o contato for encontrado, o endereço é associado ao contato usando o método `address.setContact(contact)`.
5. O endereço é salvo no banco de dados pelo `addressRepository.save(address)`.
6. A anotação `@ResponseStatus(HttpStatus.CREATED)` indica que, se o endereço for salvo com sucesso, o servidor retornará um código HTTP 201 (Created).

### **Em resumo...**

A classe `AddressController` foi implementada utilizando os conceitos já vistos previamente. Vejamos agora como ficou o código atualizado do nosso `ContactController`.

### ✅ **4.4 Controller `ContactController`**

A classe `ContactController` é um **controlador REST**, responsável por fornecer os endpoints para manipulação de recursos `Contact`. Ela é anotada com `@RestController`, o que indica que seus métodos retornam dados diretamente como respostas HTTP, geralmente em formato JSON. A anotação `@RequestMapping("/api/contacts")` define a URL base para todos os endpoints do controlador.

O controlador utiliza injeção de dependência para receber um objeto do tipo `ContactRepository`, que é responsável por realizar operações de acesso a dados relacionadas aos contatos (CRUD). Essa injeção é feita automaticamente pelo Spring por meio da anotação `@Autowired`, assim como na classe `AddressController`.

Vejamos, abaixo, seu código-fonte.

```java
package br.ifsp.contacts.controller;

@RestController
@RequestMapping("/api/contacts")
@Validated
public class ContactController {

    @Autowired
    private ContactRepository contactRepository;

    @GetMapping
    public List<Contact> getAllContacts() {
        return contactRepository.findAll();
    }

    @GetMapping("{id}")
    public Contact getContactById(@PathVariable Long id) {
        return contactRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Contato não encontrado: " + id));
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public Contact createContact(@Valid @RequestBody Contact contact) {
        return contactRepository.save(contact);
    }

    @PutMapping("/{id}")
    public Contact updateContact(@PathVariable Long id, @Valid @RequestBody Contact updatedContact) {
        Contact existingContact = contactRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Contato não encontrado: " + id));

        existingContact.setNome(updatedContact.getNome());
        existingContact.setEmail(updatedContact.getEmail());
        existingContact.setTelefone(updatedContact.getTelefone());
        existingContact.setAddresses(updatedContact.getAddresses());

        return contactRepository.save(existingContact);
    }

    @PatchMapping("/{id}")
    public Contact updateContactPartial(@PathVariable Long id, @RequestBody Map<String, String> updates) {
        Contact contact = contactRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Contato não encontrado: " + id));

        updates.forEach((key, value) -> {
            switch (key) {
                case "nome":
                    contact.setNome(value);
                    break;
                case "telefone":
                    contact.setTelefone(value);
                    break;
                case "email":
                    contact.setEmail(value);
                    break;
            }
        });

        return contactRepository.save(contact);
    }

    @DeleteMapping("/{id}")
    public void deleteContact(@PathVariable Long id) {
        contactRepository.deleteById(id);
    }

    @GetMapping("/search")
    public List<Contact> searchContactsByName(@RequestParam String name) {
        return contactRepository.findByNomeContainingIgnoreCase(name);
    }
}
```

### 📌 **Métodos Implementados no Controller**
O controlador `ContactController` oferece métodos CRUD e um método adicional para busca personalizada. Vamos detalhá-los:


#### ✅ `getAllContacts()`
```java
@GetMapping
public List<Contact> getAllContacts() {
    return contactRepository.findAll();
}
```
Este método retorna todos os contatos presentes no banco de dados. É um endpoint `GET` acessível por:
```
GET /api/contacts
```
A resposta é uma lista de objetos `Contact`.


#### ✅ `getContactById()`
```java
@GetMapping("{id}")
public Contact getContactById(@PathVariable Long id)
```
Esse método busca um contato específico por seu `id`. Ele retorna um erro `404 - Not Found` caso o contato não exista, usando o método `orElseThrow()` que lança a exceção `ResourceNotFoundException`. É acessível por:
```
GET /api/contacts/{id}
```
Se encontrado, retorna o objeto `Contact` correspondente.

#### ✅ `createContact()`
```java
@PostMapping
@ResponseStatus(HttpStatus.CREATED)
public Contact createContact(@Valid @RequestBody Contact contact)
```
Esse método cria um novo contato a partir dos dados enviados no corpo da requisição. A anotação `@Valid` garante que as validações definidas na classe `Contact` sejam respeitadas. É acessível por:
```
POST /api/contacts
```
Se o contato for criado com sucesso, retorna o objeto `Contact` salvo no banco de dados com o código `201 - Created`.

#### ✅ `updateContact()`
```java
@PutMapping("/{id}")
public Contact updateContact(@PathVariable Long id, @Valid @RequestBody Contact updatedContact)
```
Esse método atualiza todos os dados de um contato existente identificado pelo `id`. Caso o contato não seja encontrado, é lançada uma `ResourceNotFoundException`. A atualização é feita de forma completa, substituindo os dados antigos pelos novos. É acessível por:
```
PUT /api/contacts/{id}
```
Retorna o contato atualizado.

#### ✅ `updateContactPartial()`
```java
@PatchMapping("/{id}")
public Contact updateContactPartial(@PathVariable Long id, @RequestBody Map<String, String> updates)
```
Este método permite atualizações **parciais** em um contato. Ele recebe um mapa (`Map<String, String>`) contendo os campos que devem ser atualizados e seus novos valores. Somente os campos presentes no mapa são modificados; os demais permanecem inalterados. É acessível por:
```
PATCH /api/contacts/{id}
```
Caso o contato não seja encontrado, lança uma `ResourceNotFoundException`.


#### ✅ `deleteContact()`
```java
@DeleteMapping("/{id}")
public void deleteContact(@PathVariable Long id)
```
Esse método exclui um contato identificado por seu `id`. A exclusão é feita por meio do método `deleteById()` do `ContactRepository`. Se o contato não existir, o repositório não realiza nenhuma ação. É acessível por:
```
DELETE /api/contacts/{id}
```
Retorna uma resposta sem conteúdo (`204 - No Content`) se a exclusão for bem-sucedida.


#### ✅ `searchContactsByName()`
```java
@GetMapping("/search")
public List<Contact> searchContactsByName(@RequestParam String name)
```
Esse método realiza uma **busca personalizada** por nome. Ele utiliza o método `findByNomeContainingIgnoreCase()` do `ContactRepository` para encontrar contatos cujo nome contenha o termo pesquisado, independentemente de maiúsculas ou minúsculas. É acessível por:
```
GET /api/contacts/search?name=Joao
```
Retorna uma lista de contatos que correspondem ao critério de busca.

### 📌 **Como os métodos interagem com a camada de persistência (`ContactRepository`)?**
O controlador depende diretamente do repositório `ContactRepository` para todas as operações de leitura, escrita, atualização e exclusão. A camada de persistência é configurada para lidar com entidades JPA (`Contact`), o que significa que as operações de banco de dados são tratadas de forma transparente pelo Spring Data JPA. É a mesmíssima coisa que já fizemos anteriormente. Percebam: estamos apenas repetindo os mesmos padrões já vistos anteriormente.


### 📌 **Como os métodos tratam erros e validações?**
- **Validação de Dados:** A anotação `@Valid` garante que os objetos enviados na requisição atendam às regras definidas na classe `Contact`.
- **Tratamento de Erros:** O uso de `ResourceNotFoundException` integrado com o `GlobalExceptionHandler` garante que as requisições inválidas sejam respondidas de forma apropriada (`404 - Not Found`).

É exatamente a mesma lógica que aplicamos no `AddressController`!


### ✅ **4.5 Melhorando o `GlobalExceptionHandler`**

A implementação inicial do `GlobalExceptionHandler` foi projetada para lidar apenas com exceções genéricas (`Exception`) e com uma exceção personalizada específica (`ResourceNotFoundException`). Essa abordagem já proporcionava uma maneira centralizada e padronizada de tratar erros na aplicação. No entanto, ela carecia de um tratamento mais robusto para situações comuns no desenvolvimento de APIs REST, especialmente em relação à validação de dados de entrada.

A implementação inicial era limitada porque:  
- Tratava apenas exceções genéricas (`Exception`) e `ResourceNotFoundException`.
- Não possuía suporte para o tratamento explícito de erros de validação.
- Não fornecia mensagens detalhadas sobre quais campos específicos estavam inválidos, o que poderia dificultar a compreensão por parte do cliente da API.

Podemos melhorar a implementação inicial, mostrada na seção 2.7 com o código abaixo

```java
package br.ifsp.contacts.exception;

import jakarta.validation.ConstraintViolationException;

@RestControllerAdvice
public class GlobalExceptionHandler {
    
    /**
     * Trata erros de validação de entrada (ex: campos inválidos no @Valid)
     */
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ResponseEntity<Map<String, String>> handleValidationExceptions(MethodArgumentNotValidException exception) {
        Map<String, String> errors = new HashMap<>();
        exception.getBindingResult().getAllErrors().forEach((error) -> {
            if (error instanceof FieldError) {
                String fieldName = ((FieldError) error).getField();
                String errorMessage = error.getDefaultMessage();
                errors.put(fieldName, errorMessage);
            }
        });
        return ResponseEntity.badRequest().body(errors);
    }
    
    @ExceptionHandler(ConstraintViolationException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ResponseEntity<Map<String, String>> handleGlobalValidationExceptions(ConstraintViolationException  exception) {
        Map<String, String> errors = new HashMap<>();
        exception.getConstraintViolations().forEach(violation -> 
            errors.put(violation.getPropertyPath().toString(), violation.getMessage())
        );
        return ResponseEntity.badRequest().body(errors);
    }
    
    /**
     * Trata exceções de recurso não encontrado
     */
    @ExceptionHandler(ResourceNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ResponseEntity<Map<String, String>> handleResourceNotFoundException(ResourceNotFoundException exception) {
        Map<String, String> errorResponse = new HashMap<>();
        errorResponse.put("error", exception.getMessage());
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(errorResponse);
    }
    
    /**
     * Trata exceções genéricas que não foram capturadas pelos handlers anteriores.
     */
    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ResponseEntity<Map<String, String>> handleGenericException(Exception exception) {
        Map<String, String> errorResponse = new HashMap<>();
        errorResponse.put("error", "Erro interno no servidor. Entre em contato com o suporte.");
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(errorResponse);
    }
    
}
```

### 📌 **O que foi melhorado na implementação final?**

Essa implementação melhorada do `GlobalExceptionHandler` amplia significativamente o escopo de tratamento de erros ao adicionar suporte para exceções de validação. Isso foi feito adicionando métodos específicos para capturar e processar erros relacionados à entrada de dados inválida. Vamos analisar as melhorias implementadas.

### 📌 **Suporte a `MethodArgumentNotValidException`**  
O método `handleValidationExceptions()` foi adicionado para tratar exceções do tipo `MethodArgumentNotValidException`. Essas exceções são lançadas quando um objeto validado com a anotação `@Valid` não atende às regras definidas nas entidades, como `@NotBlank`, `@Size`, `@Pattern`, etc.

```java
@ExceptionHandler(MethodArgumentNotValidException.class)
@ResponseStatus(HttpStatus.BAD_REQUEST)
public ResponseEntity<Map<String, String>> handleValidationExceptions(MethodArgumentNotValidException exception) {
    Map<String, String> errors = new HashMap<>();
    exception.getBindingResult().getAllErrors().forEach((error) -> {
        if (error instanceof FieldError) {
            String fieldName = ((FieldError) error).getField();
            String errorMessage = error.getDefaultMessage();
            errors.put(fieldName, errorMessage);
        }
    });
    return ResponseEntity.badRequest().body(errors);
}
```

- Este método percorre todos os erros encontrados e os mapeia para um objeto `Map` que associa o nome do campo (`fieldName`) com a mensagem de erro (`errorMessage`).  
- A resposta gerada é uma `ResponseEntity` contendo um `Map` que detalha todos os erros detectados, retornando o status HTTP `400 (Bad Request)`.
- Essa implementação melhora a experiência do usuário, fornecendo feedback claro sobre o que precisa ser corrigido.

### 📌 **Suporte a `ConstraintViolationException`**  
O método `handleGlobalValidationExceptions()` foi adicionado para capturar exceções do tipo `ConstraintViolationException`. Essas exceções geralmente ocorrem quando se tenta validar dados em controladores que não recebem objetos completos, mas parâmetros individuais, por exemplo, ao utilizar `@RequestParam` ou `@PathVariable` com validações `@Valid`.

```java
@ExceptionHandler(ConstraintViolationException.class)
@ResponseStatus(HttpStatus.BAD_REQUEST)
public ResponseEntity<Map<String, String>> handleGlobalValidationExceptions(ConstraintViolationException exception) {
    Map<String, String> errors = new HashMap<>();
    exception.getConstraintViolations().forEach(violation -> 
        errors.put(violation.getPropertyPath().toString(), violation.getMessage())
    );
    return ResponseEntity.badRequest().body(errors);
}
```

- Este método percorre todas as violações detectadas (`constraintViolations`) e as armazena em um `Map`.
- A resposta HTTP retornada é `400 (Bad Request)`, apropriada para requisições inválidas.
- Este tratamento é útil para capturar erros em parâmetros simples, como um telefone inválido passado diretamente na URL.


### 📌 **Tratamento Genérico para Outras Exceções**  
O método `handleGenericException()` é um fallback para qualquer exceção que não foi capturada por métodos mais específicos.

```java
@ExceptionHandler(Exception.class)
@ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
public ResponseEntity<Map<String, String>> handleGenericException(Exception exception) {
    Map<String, String> errorResponse = new HashMap<>();
    errorResponse.put("error", "Erro interno no servidor. Entre em contato com o suporte.");
    return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(errorResponse);
}
```

- A captura genérica de exceções garante que erros inesperados sejam tratados com uma resposta padrão.
- Retorna o status HTTP `500 (Internal Server Error)`, indicando um problema do lado do servidor.

Com isso, temos uma maior cobertura à validação na nossa aplicação, cobrindo todos os casos especificados nos Desafios e Exercícios até o momento!