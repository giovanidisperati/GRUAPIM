---
layout: aula
title: 1. Introdução à JPA
parent: Aula 04 - Spring Boot - JPA, Exceções e Documentação!
nav_order: 1
---

## 1. Métodos customizados na JPA

No exercício 1 da Aula 03, foi pedida a criação de um novo endpoint GET em `ContactController` que permitisse buscar contatos pelo nome.

Para cumprirmos esse requisito podemos implementar um método **searchContactsByName** no `ContactController`. A ideia dessa implementação é criar um endpoint GET que recebe o nome como um parâmetro de URL e retorne uma lista de contatos cujos nomes correspondem parcial ou totalmente ao termo pesquisado. Isso é útil para implementarmos funcionalidades de busca mais flexíveis e dinâmicas. O método é demonstrado abaixo.

```java
@GetMapping("/search")
public List<Contact> searchContactsByName(@RequestParam String name) {
    return contactRepository.findByNomeContainingIgnoreCase(name);
}
```

Perceba que o método encapsula a chamada à **findByNomeContainingIgnoreCase**, no `ContactRepository`.

O Spring Data JPA fornece a possibilidade de criar métodos personalizados na interface de repositório através de **Convenções de Nomes (Naming Conventions)**. Essa abordagem permite criar consultas complexas apenas definindo métodos que sigam um padrão de nomenclatura específico.

Assim, para implementar o método de busca, podemos adicionar o seguinte código na interface `ContactRepository`:

```java
public interface ContactRepository extends JpaRepository<Contact, Long> {
    List<Contact> findByNomeContainingIgnoreCase(String nome);
}
```

#### 🔍 **1.1 Como funciona essa Convenção de Nomes?**
A criação de métodos customizados se dá pela definição de nomes que indicam ao Spring Data JPA qual consulta deve ser gerada. Isso é feito utilizando palavras-chave específicas que indicam o critério de busca. Alguns exemplos comuns são:

- `findBy`: Indica que o método será usado para realizar uma busca.
- `Nome`: O nome do atributo que será utilizado na busca. Neste caso, `Nome` refere-se ao atributo `nome` da classe `Contact`.
- `Containing`: Indica que a busca deve procurar ocorrências que contenham o termo especificado. É equivalente a um `LIKE %termo%` em SQL.
- `IgnoreCase`: Indica que a busca deve ignorar maiúsculas e minúsculas.

Essa convenção é poderosa e nos permite criar consultas como:
- `findByNome(String nome)` → Busca contatos pelo nome exato.
- `findByNomeAndEmail(String nome, String email)` → Busca contatos que correspondem a ambos os critérios.
- `findByNomeOrEmail(String nome, String email)` → Busca contatos que correspondem a pelo menos um dos critérios.
- `findByNomeStartingWith(String prefix)` → Busca contatos cujos nomes começam com determinado prefixo.
- `findByNomeEndingWith(String sufix)` → Busca contatos cujos nomes terminam com determinado sufixo.

### 📌 **1.2 Métodos Personalizados sem Convenções de Nome**
Caso queiramos implementar um método que não siga essas convenções, podemos usar anotações `@Query`. Isso é útil quando precisamos de consultas mais complexas ou específicas.

Exemplo com HQL:
```java
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;

@Query("SELECT c FROM Contact c WHERE LOWER(c.nome) LIKE LOWER(CONCAT('%', :nome, '%'))")
List<Contact> searchByName(@Param("nome") String nome);
```

Esse método produz o mesmo resultado que `findByNomeContainingIgnoreCase`, mas oferece mais controle sobre a query e permite escrever SQL/HQL diretamente. Também é possível utilizar SQL nativo com `nativeQuery = true`.

Exemplo com SQL nativo:
```java
public interface ContactRepository extends JpaRepository<Contact, Long> {

    @Query(value = "SELECT * FROM Contact WHERE LOWER(nome) LIKE LOWER(CONCAT('%', :nome, '%'))", nativeQuery = true)
    List<Contact> searchByName(@Param("nome") String nome);
}
```

#### 📌 1.3 Como funciona?
- A anotação `@Query` define a consulta SQL nativa usando o parâmetro `nativeQuery = true`.
- Neste exemplo, estamos fazendo um `LIKE` que ignora maiúsculas e minúsculas (`LOWER()`) e busca contatos que contenham o nome informado em qualquer parte do nome.
- É importante usar o nome exato da tabela (`Contact`) como ela existe no banco de dados.

Este método fornece controle total sobre a query e é útil quando precisamos usar recursos específicos do banco de dados que não são suportados diretamente pelo JPA.

#### 📖 **1.4 Quando usar cada abordagem:**
- **Convenção de Nomes:** Para consultas simples e padrão, como busca por atributos específicos (`findByNome`, `findByEmail`).
- **@Query:** Para consultas complexas com relacionamentos entre múltiplas entidades, ou quando precisamos de controle total sobre a query.


#### 📌 **1.5 Exemplo de Requisição:**
```
GET /api/contacts/search?name=joao
```
Isso retornará todos os contatos cujo nome contenha a palavra `joao` em qualquer parte do nome, ignorando maiúsculas e minúsculas.

