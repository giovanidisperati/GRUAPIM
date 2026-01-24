---
layout: aula
title: 2. Tratamento de Exceções e Método Patch
parent: Aula 04 - Spring Boot - JPA, Exceções e Documentação!
nav_order: 2
---

## 2. Implementação do Método PATCH e Tratamento de Exceções

Para darmos continuidade ao conteúdo e abordarmos o tratamento de exceções, relembremos o Exercício 02 da Aula 03, que solicitava a criação de um novo método PATCH que permitisse atualizar parcialmente um contato sem precisar enviar todos os dados - o que é útil quando o cliente  deseja modificar apenas um ou mais atributos específicos de um contato, mantendo os demais inalterados. Caso o ID submetido pelo cliente na requisição não fosse encontrado, entretanto, era necessário lançar uma exceção e retornar o status 404 - não encontrado.

Para cumprir parcialmente esse requisito, podemos podemos prosseguir com a implementação do método PATCH na classe `ContactController`, por meio do método `updateContactPartial`, que recebe um ID e um `Map<String, String>` contendo os campos a serem atualizados.

```java
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
```

### 🔍 **2.1 Explicação do Código:**
1. **Recebimento do ID do Contato:**  
   - A URL da requisição especifica o ID do contato a ser modificado (`/api/contacts/{id}`).
   - Se o contato não for encontrado no banco de dados, é lançada a exceção personalizada `ResourceNotFoundException`, retornando um status HTTP 404. Faremos a implementação dessa exceção personalizada na próxima seção.

2. **Uso do `Map<String, String>`:**  
   - O corpo da requisição é recebido como um `Map`, onde:  
     - A chave (`key`) é o nome do atributo a ser modificado (`nome`, `telefone` ou `email`).  
     - O valor (`value`) é o novo valor que será atribuído ao atributo correspondente.  
   - Esse formato é útil porque permite que a requisição inclua **somente os campos que precisam ser atualizados**, sem exigir o envio do objeto completo.  

3. **Iteração sobre o `Map`:**  
   - A função `updates.forEach()` percorre cada entrada (`key`, `value`) do `Map`.  
   - A estrutura `switch` verifica qual campo deve ser atualizado e o modifica chamando os métodos `setNome()`, `setTelefone()` ou `setEmail()` do objeto `Contact`.  

4. **Persistência dos Dados:**  
   - Após atualizar os campos necessários, o método chama `contactRepository.save(contact)` para salvar as modificações no banco de dados.  
   - O objeto atualizado é retornado como resposta.  



### 📌 **2.2 Exemplo de Requisição:**
Suponha que temos um contato com o ID `1`, e desejamos atualizar apenas o email desse contato. A requisição ficaria assim:

```
PATCH /api/contacts/1
Content-Type: application/json

{
    "email": "novocontato@email.com"
}
```

Também é possível enviar múltiplos campos para serem atualizados ao mesmo tempo:

```
PATCH /api/contacts/1
Content-Type: application/json

{
    "nome": "João Silva",
    "telefone": "99998888"
}
```

## **2.3 🔍 Como o método está relacionado ao tratamento de erros?**
Caso o ID do contato submetido na requisição não seja encontrado, o método lança uma exceção `ResourceNotFoundException`:

```java
Contact contact = contactRepository.findById(id)
        .orElseThrow(() -> new ResourceNotFoundException("Contato não encontrado: " + id));
```

Essa exceção é capturada por outra classe que iremos criar: a `GlobalExceptionHandler`, configurada para retornar uma resposta com código **HTTP 404 (Not Found)** quando ocorrer um `ResourceNotFoundException`, garantindo que o cliente receba uma mensagem clara sobre o problema ocorrido. Antes de passarmos à crição dessas duas classes, entretanto, vamos entender brevemente o uso do `orElseThrow()` com **Lambda Expression**.
  
O método Faz parte da lógica que busca um contato no banco de dados pelo seu ID, como vimos anteriormente quando abordamos as **Naming Conventions**. `findById()` é fornecido pela interface `JpaRepository` (que `ContactRepository` estende) e retorna um objeto do tipo  `Optional<Contact>`.

```java
Optional<Contact> findById(Long id);
```

Nesse caso o uso do `Optional` é importante porque permite que o método retorne um valor presente (o contato encontrado) ou ausente (`Optional.empty()`) se o ID não existir no banco de dados. O Optional é uma classe introduzida no Java 8, que faz parte do pacote `java.util`. Ele foi criado para lidar com o problema comum de `NullPointerException`, proporcionando uma maneira elegante de representar a presença ou ausência de um valor: em vez de retornar um valor potencialmente nulo, métodos podem retornar um Optional, que é um contêiner que pode ou não conter um valor não-nulo. Para saber mais, consulte a [Documentação Oficial do Optional (Java 8)](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html).

Para extrair o valor contido no `Optional`, utilizamos o método `orElseThrow()`, que é uma forma prática e segura de lidar com possíveis ausências de valor.

### **2.4 📌 O que é `orElseThrow()`?**  
O método `orElseThrow()` é um método da classe `Optional<T>` que é utilizado para:

- **Retornar o valor contido no `Optional`** se ele estiver presente.
- **Lançar uma exceção personalizada** caso o valor não esteja presente.

O método `orElseThrow()` recebe um **Supplier** como argumento. Um `Supplier` é uma interface funcional que não recebe parâmetros e retorna um objeto (`T`). Ele é frequentemente implementado usando **Expressões Lambda**, que tornam o código mais enxuto e legível.

A expressão lambda usada aqui é:

```java
() -> new ResourceNotFoundException("Contato não encontrado: " + id)
```

- Os parênteses `()` indicam que a função lambda **não recebe parâmetros**.  
- A seta `->` indica o início da parte executável da expressão lambda.  
- A parte `new ResourceNotFoundException("Contato não encontrado: " + id)` é o código executado quando o lambda é invocado.  
- Neste caso, estamos retornando uma nova instância de `ResourceNotFoundException`.  

#### **2.5 📌 Por que usar uma Expressão Lambda aqui?**  
O método `orElseThrow()` exige um argumento que implemente a interface `Supplier`.  
Se não utilizássemos uma expressão lambda, a declaração seria mais verbosa, algo como:

```java
Contact contact = contactRepository.findById(id)
        .orElseThrow(new Supplier<ResourceNotFoundException>() {
            @Override
            public ResourceNotFoundException get() {
                return new ResourceNotFoundException("Contato não encontrado: " + id);
            }
        });
```

Claramente, a forma lambda é **muito mais compacta e legível**.


### **2.6 📌 Ou seja...**  
```java
Contact contact = contactRepository.findById(id)
        .orElseThrow(() -> new ResourceNotFoundException("Contato não encontrado: " + id));
```

- Tenta buscar um `Contact` pelo `id` fornecido.
- Se encontrado, retorna o objeto `Contact`.
- Se não encontrado, lança uma exceção `ResourceNotFoundException` com a mensagem personalizada.
- A exceção é tratada pelo `GlobalExceptionHandler` e retorna o status HTTP 404 para o cliente.

Agora que entendemos essa linha de código, vamos abordar o uso de Exceptions customizadas e a implementação de nosso `GlobalExceptionHandler`.

### **2.7 Tratamento de Erros Relacionados ao Método PATCH**

O tratamento de exceções na API é feito pela classe `GlobalExceptionHandler`, que lida com diferentes tipos de erros, incluindo a `ResourceNotFoundException`. O tratamento de erros é uma parte essencial de uma API REST bem estruturada. Ele garante que os consumidores da API recebam mensagens claras e adequadas sobre o que ocorreu, além de evitar a exposição de detalhes internos do sistema.

No nosso projeto, utilizamos uma classe chamada `GlobalExceptionHandler` anotada com `@RestControllerAdvice`. Essa classe intercepta exceções lançadas em qualquer parte da aplicação e retorna uma resposta adequada para o cliente. Para manter a organização do código vamos criar um pacote **exception**, a fim de manter uma estrutura clara e direta.

### **2.8 📌 Implementação inicial da Classe `GlobalExceptionHandler`**

```java
package br.ifsp.contacts.exception;

@RestControllerAdvice
public class GlobalExceptionHandler {
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

### **2.9 🔍 Como funciona?**
1. Os métodos são anotados com `@ExceptionHandler` para especificar quais tipos de exceção devem ser capturados. Ou seja, cada método será chamado quando ocorrer o tipo de exceção para o qual ele está anotado (`ResourceNotFoundException.class`, `Exception.class` e assim por diante).
2. A anotação `@RestControllerAdvice` faz com que a classe capture exceções lançadas por qualquer controlador da nossa aplicação.
3. Os métodos retornam um `ResponseEntity`, uma classe do Spring que permite personalizar completamente a resposta HTTP, incluindo o corpo da mensagem, o status HTTP e os cabeçalhos. Isso proporciona maior controle sobre o que é retornado ao cliente, garantindo que respostas adequadas sejam enviadas para diferentes situações, como sucessos, erros ou requisições inválidas.
4. `ResourceNotFoundException` é uma exceção customizada que retorna um código HTTP 404. Vamos definir essa classe a seguir.

### **2.10 📌 Classe `ResourceNotFoundException`**

```java
package br.ifsp.contacts.exception;

public class ResourceNotFoundException extends RuntimeException {

    public ResourceNotFoundException(String message) {
        super(message);
    }
}
```

Essa classe é uma exceção customizada que herda de `RuntimeException`. Ela é utilizada para indicar que um recurso solicitado (por exemplo, um contato que o usuário está tentando acessar) não foi encontrado no banco de dados. A garantia de que essa exceção resultará em uma resposta HTTP com código **404 (Not Found)** ocorre porque o **GlobalExceptionHandler** possui um método anotado com `@ExceptionHandler(ResourceNotFoundException.class)`. Essa anotação instrui o Spring a interceptar qualquer exceção do tipo `ResourceNotFoundException` lançada pela aplicação e tratar essa exceção de maneira centralizada, como explicado acima.

O que acontece aqui é o seguinte:

1. **Interceptação da Exceção:** Sempre que uma `ResourceNotFoundException` é lançada em qualquer parte da aplicação, o método `handleResourceNotFoundException()` é chamado automaticamente.  
2. **Personalização da Resposta:** O método cria um `ResponseEntity` com um corpo JSON contendo a mensagem de erro e define explicitamente o código de status HTTP como `404 (Not Found)`.  
3. **Anotação `@ResponseStatus`:** Embora o código HTTP já seja definido pelo `ResponseEntity` (`HttpStatus.NOT_FOUND`), a anotação `@ResponseStatus(HttpStatus.NOT_FOUND)` fornece uma camada extra de garantia que o status apropriado será retornado.  

Portanto, a garantia se dá pelo mecanismo centralizado do Spring para captura e tratamento de exceções, configurado pelo `@ExceptionHandler` no `GlobalExceptionHandler`.

### **2.10 🎯 Em resumo...**

Quando o método `updateContactPartial` utiliza `contactRepository.findById()`, ele lança a exceção `ResourceNotFoundException` se o ID não existir. Essa exceção é tratada pelo método `handleResourceNotFoundException` no `GlobalExceptionHandler`, retornando uma resposta com o código HTTP **404 - Not Found**.

O tratamento de erros bem estruturado evita que o cliente receba informações internas da aplicação e padroniza as mensagens de erro. A integração do `GlobalExceptionHandler` é essencial para capturar exceções lançadas pelo método PATCH e devolver respostas amigáveis.

Devemos sempre fazer tratamento de erros na nossa aplicação, especialmente em APIs REST que precisam fornecer respostas claras e adequadas para os clientes que as consomem. Podemos citar os seguintes motivadores para isso:

- 1. **Melhoria na Experiência do Usuário:** Um tratamento adequado de erros garante que o cliente receba mensagens úteis e compreensíveis, em vez de mensagens genéricas ou códigos de status que não explicam o problema.

- 2. **Segurança:** Se erros não forem tratados corretamente, informações sensíveis sobre a aplicação podem ser expostas acidentalmente, como detalhes do banco de dados ou da lógica de negócio.

- 3. **Facilidade na Depuração:** Mensagens de erro claras e específicas facilitam a identificação e correção de problemas, tanto durante o desenvolvimento quanto na manutenção da aplicação.

- 4. **Conformidade com os Padrões REST:** APIs bem projetadas devem retornar códigos HTTP apropriados (`404` para recurso não encontrado, `400` para requisição inválida, `500` para erro interno, etc.) e fornecer informações úteis sobre o problema.

- 5. **Evitar Que a Aplicação Quebre:** Tratamento adequado de erros impede que a aplicação seja interrompida inesperadamente por uma exceção não tratada.

#### **2.11 🚩 O que acontece se não tratarmos erros adequadamente?**  
- A aplicação pode retornar mensagens genéricas ou códigos HTTP inadequados, dificultando a depuração.  
- Pode expor informações sensíveis que deveriam permanecer ocultas.  
- Pode quebrar a aplicação se uma exceção crítica não for tratada.  
- Pode resultar em má experiência para os usuários e desenvolvedores que consomem a API.  

Embora o tratamento de erros não seja obrigatório para que a aplicação funcione, ele é essencial para garantir **segurança, clareza, padronização e uma melhor experiência para os usuários e desenvolvedores**. Portanto, é considerado **uma boa prática fundamental em qualquer aplicação**. 

O nosso `GlobalExceptionHandler` entretanto ainda não está pronto. Brevemente vamos voltar a inserir novos tratamentos na classe. Antes disso, entretanto, vamos explorar os Desafios 1 e 2 da Aula 03 e os conceitos que envolvem o relacionamento e validação de Entidades.