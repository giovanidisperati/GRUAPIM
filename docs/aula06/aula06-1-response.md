---
layout: aula
title: 1. O uso de ResponseEntity
parent: Aula 06 - Service Layers e Testes Automatizados
nav_order: 1
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

Além disso, a documentação gerada pelo Swagger/OpenAPI também pode ser enriquecida com status HTTP mais precisos e podemos alterar a injeção de dependência que vinhamos usando com `@Autowired` para injeção via construtor!fatorações e rever o nosso código completo do controller.