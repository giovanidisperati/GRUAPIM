---
layout: aula
title: 2. Paginação
parent: Aula 05 - Aprimorando nossas APIs Rest
nav_order: 2
---
## 2. 📖 Paginação e Ordenação

A paginação é uma prática importante no desenvolvimento de APIs que lidam com grandes volumes de dados, pois permite que esses dados sejam entregues em partes menores, conhecidas como páginas. Ao invés de retornar todos os registros de uma só vez, a API responde com uma fração controlada deles, o que melhora o desempenho da aplicação, reduz o uso de memória e otimiza o tempo de resposta — tanto no servidor quanto para o usuário final. Isso tem impacto direto na experiência do usuário, pois garante que as informações sejam carregadas de forma mais rápida e fluida, mesmo em dispositivos com recursos limitados ou conexões de rede lentas.

Um dos conceitos associados a esse comportamento é o *lazy loading*, ou carregamento sob demanda. Em vez de carregar todos os dados logo de início, o sistema busca apenas os dados necessários naquele momento (por exemplo, os 10 primeiros itens) e carrega os próximos à medida que o usuário interage com a aplicação, como ao rolar a página. Isso evita sobrecarga no frontend e garante uma melhor percepção de velocidade por parte do usuário.

No ecossistema Spring, a paginação é implementada através das interfaces `Pageable` e `Page`. A interface `Pageable` representa a requisição de uma página específica e carrega as informações sobre qual página foi solicitada, qual o tamanho da página e quais os critérios de ordenação. O Spring monta automaticamente esse objeto com base em parâmetros da URL, como `page=0&size=10&sort=nome,asc`. Já a interface `Page<T>` é a resposta retornada quando utilizamos um método paginado no repositório. Ela contém tanto a lista de dados quanto metadados relevantes, como o número total de elementos, total de páginas, número da página atual e o tamanho da página.

Do ponto de vista do backend e dos acessos ao banco de dados, a paginação também é extremamente benéfica. Consultas paginadas geram instruções SQL otimizadas com `LIMIT` e `OFFSET`, o que significa que o banco só precisa retornar exatamente a quantidade solicitada de dados — e não a tabela inteira. Isso diminui significativamente a pressão sobre o banco de dados, melhora a escalabilidade da aplicação e torna os sistemas mais preparados para lidar com múltiplos acessos simultâneos. Ou seja, a paginação é uma estratégia técnica e de usabilidade que traz vantagens tanto para a infraestrutura quanto para a experiência do usuário. Ela reduz o tráfego de rede, o tempo de carregamento das informações e o uso de recursos computacionais, enquanto aumenta a escalabilidade e a clareza da navegação em APIs REST.

O Spring Data JPA, por padrão, utiliza a contagem zero-based (ou seja, começa na página 0). No entanto, você pode manipular o valor do parâmetro `page` recebido na controller, subtraindo 1 antes de construir o `Pageable`. Por exemplo, eventualmente em nossa implementação poderíamos fazer algo como:

```java
@GetMapping
public ResponseEntity<Page<ContactResponseDTO>> getPaginatedContacts(
        @RequestParam(defaultValue = "1") @Min(1) int page,
        @RequestParam(defaultValue = "10") int size,
        @RequestParam(defaultValue = "nome") String sort,
        @RequestParam(defaultValue = "asc") String direction
) {
    Sort.Direction sortDirection = Sort.Direction.fromString(direction);
    Pageable pageable = PageRequest.of(page - 1, size, Sort.by(sortDirection, sort));

    Page<ContactResponseDTO> resultPage = contactService.getAllPaginated(pageable);

    return ResponseEntity
            .status(HttpStatus.OK)
            .body(resultPage);
}
```

Essa adaptação mantém a semântica de uso mais amigável para o usuário (paginação a partir de 1) sem alterar a lógica da infraestrutura do Spring (que continua baseada no zero). É uma prática recomendada para sistemas com UI paginada, como painéis administrativos, apps ou sites públicos com listagens.

O método acima é um exemplo interessante de onde podemos eventualmente chegar em nosso sistema. Por hora, entretanto, vamos começar da maneira mais simples 

### 2.1 ✅ **Implementação da paginação**

Como vimos acima, o Spring Data JPA facilita bastante a implementação de paginação por meio das interfaces `Page` e `Pageable`. Vamos implementá-las alterando, primeiro, nossos repositórios e fazendo com que nossas queries retornem `Page`.

#### Modificando os Repositórios
Para paginar os resultados, basta que os métodos dos repositórios retornem um objeto `Page<T>`. O Spring cuida do resto! Vamos atualizar ambos repositórios de nossa aplicação.

**`AddressRepository.java`**

```java
@Repository
public interface AddressRepository extends JpaRepository<Address, Long> {
    Page<Address> findByContactId(Long contactId, Pageable pageable);
}
```

👉 Esse método agora retorna uma `Page` de endereços, filtrando por `contactId` e aplicando paginação via o parâmetro `Pageable`, que será montado no controller a partir dos parâmetros da URL.

**`ContactRepository.java`**
```java
@Repository
public interface ContactRepository extends JpaRepository<Contact, Long> {
    Page<Contact> findByNomeContainingIgnoreCase(String nome, Pageable pageable);
}
```

👉 Aqui, o método busca contatos que contenham o nome informado (ignorando maiúsculas e minúsculas), retornando os dados em formato paginado também. A interface `Pageable` pode incluir, além da página e quantidade de itens, a ordenação.

#### Atualizando os Controllers

**`ContactController.java`**

Método `searchContactsByName`

```java
    @GetMapping("/search")
    public Page<ContactResponseDTO> searchContactsByName(@RequestParam String name, Pageable pageable) {
        return contactRepository.findByNomeContainingIgnoreCase(name, pageable)
                .map(contact -> modelMapper.map(contact, ContactResponseDTO.class));
    }
```

🧠 **Explicação**:  
Esse endpoint recebe dois parâmetros:
- `name` → termo de busca (obrigatório)
- `pageable` → automaticamente preenchido pelo Spring com base nos parâmetros da URL (`page`, `size`, `sort`, etc.)

O método retorna uma `Page` de `ContactResponseDTO`, convertendo os objetos `Contact` usando o `ModelMapper`.

📌 **Exemplo de requisição no Postman**:

```
GET http://localhost:8080/api/contacts/search?name=ana&page=0&size=5&sort=nome,asc
```

Método `getAllContacts`

```java
@GetMapping
public Page<ContactDTO> getAllContacts(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int size,
        @RequestParam(defaultValue = "nome") String sort) {
    
    Pageable pageable = PageRequest.of(page, size, Sort.by(sort));
    Page<Contact> contacts = contactRepository.findAll(pageable);
    return contacts.map(contact -> modelMapper.map(contact, ContactDTO.class));
}
```

🧠 **Explicação**:  
Aqui criamos manualmente o `Pageable` com:
- `page` → número da página (inicia em 0)
- `size` → número de registros por página
- `sort` → campo para ordenação (ex: nome, email)

Usamos o método `findAll()` do `JpaRepository`, que aceita um `Pageable` e retorna um `Page`. Depois, mapeamos os objetos da entidade para DTOs.

📌 **Exemplo de requisição no Postman**:

```
GET http://localhost:8080/api/contacts?page=1&size=5&sort=email
```

**`AddressController.java`**

Método `getAddressesByContact`

```java
    @GetMapping("/contacts/{contactId}")
    public Page<AddressResponseDTO> getAddressesByContact(@PathVariable Long contactId, Pageable pageable) {
        return addressRepository.findByContactId(contactId, pageable)
                .map(address -> modelMapper.map(address, AddressResponseDTO.class));
    }
```

🧠 **Explicação**:  
Esse endpoint retorna os endereços de um contato específico, também de forma paginada.

📌 **Exemplo no Postman**:

```
GET http://localhost:8080/api/addresses/contacts/2?page=0&size=3&sort=cidade,desc
```

Aqui, buscamos os endereços do contato com ID 2, na **primeira página**, com **3 endereços por página**, ordenados por **cidade (em ordem decrescente)**.

### ✅ Resumo prático:

- Use `Page<T>` como tipo de retorno nos repositórios.
- Construa ou injete `Pageable` nos controladores.
- O método `.map()` de `Page` facilita a conversão de entidades para DTOs.
- Parâmetros de paginação são passados via URL e entendidos automaticamente pelo Spring.


Até que não foi tão difícil implementar a paginação, né? O Spring realmente nos ajuda bastante com essa infraestrutura de requisitos que dão suporte às funcionalidades de nossa aplicação. Essa é a vantagem de utilizar um framework robusto ao invés de fazer tudo "na mão". Por outro lado, é importante não ficar muito confortável e dependente das ferramentas sem saber como elas funcionam por baixo dos panos. Tente imaginar: como poderíamos implementar uma versão rudimentar de paginação sem uso das interfaces já fornecidas pelo Spring? 💭

Reflita um pouco antes de passar à próxima seção.