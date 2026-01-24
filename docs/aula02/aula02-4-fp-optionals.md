---
layout: aula
title: 4. Optionals e Method References
parent: Aula 02 - Lambdas, Streams e Optionals
nav_order: 4
---

## **4. A Classe `Optional`: Tratando a Ausência de Valor com Elegância**

Um dos erros mais comuns em Java é o `NullPointerException`. Ele ocorre quando tentamos usar um membro de uma referência de objeto que é `null`. A classe `Optional<T>` foi introduzida para fornecer uma solução de tipo seguro para representar a presença ou ausência de um valor.

`Optional` é um **container**: um objeto que pode ou não conter um valor não nulo. Ele nos força a pensar sobre o caso em que o valor não está presente.

### **4.1. Criando um `Optional`**

Existem três maneiras principais de criar uma instância de `Optional`, cada uma adequada a um cenário diferente. Entender qual usar é o primeiro passo para escrever um código mais seguro.

#### **1. `Optional.of(valor)`**

  * **Quando usar:** Quando você tem um valor que, por contrato, **você tem certeza de que não é nulo**.

  * **Comportamento:** Cria um `Optional` que "envolve" o seu objeto. Se você tentar passar `null` para `Optional.of()`, ele falhará imediatamente (*fail-fast*) com um `NullPointerException`. Isso é útil para validar suas próprias premissas no código.

    ```java
    // Cenário: Você acabou de criar um novo objeto. Ele NUNCA será nulo.
    User newUser = new User("Ana");
    Optional<User> userOptional = Optional.of(newUser); // Correto e seguro.

    User userNulo = null;
    // A linha abaixo lançaria um NullPointerException, protegendo o resto do código.
    // Optional<User> userNuloOptional = Optional.of(userNulo);
    ```

#### **2. `Optional.ofNullable(valor)`**

  * **Quando usar:** Este é o método mais seguro e comum. Use-o quando você está lidando com um valor que **pode ser nulo**, como o retorno de um método de busca ou de um mapa.

  * **Comportamento:** Se o valor fornecido não for nulo, ele o envolve em um `Optional`. Se o valor for `null`, ele cria um `Optional` vazio (`Optional.empty()`). Este método **nunca lança `NullPointerException`**.

    ```java
    // Cenário: Buscando um usuário que pode ou não existir em um repositório.
    // O método findById pode retornar um User ou null.
    User foundUser = userRepository.findById(1L);
    Optional<User> userOptional = Optional.ofNullable(foundUser); // Forma segura de lidar com o resultado.
    ```

#### **3. `Optional.empty()`**

  * **Quando usar:** Quando você precisa representar explicitamente a ausência de um valor. É frequentemente usado como valor de retorno em métodos que podem não encontrar o que procuram.

  * **Comportamento:** Retorna uma instância de `Optional` que está garantidamente vazia.

    ```java
    // Cenário: Um método que retorna um Optional<User>, mas uma validação inicial falha.
    public Optional<User> findUser(Long id) {
        if (id <= 0) {
            return Optional.empty(); // Retorna "nada" de forma segura e explícita.
        }
        // ... continua a lógica de busca ...
        return Optional.ofNullable(userRepository.findById(id));
    }
    ```

| Método               | Quando Usar                                     | Comportamento com `null`             |
| -------------------- | ----------------------------------------------- | ------------------------------------ |
| `Optional.of(obj)`   | Quando `obj` **não pode** ser nulo.             | Lança `NullPointerException`.        |
| `Optional.ofNullable(obj)` | Quando `obj` **pode** ser nulo.             | Retorna `Optional.empty()`.          |
| `Optional.empty()`   | Para retornar explicitamente um valor ausente. | N/A (já retorna um `Optional` vazio). |

### **4.2. Usando `Optional` de Forma Segura**

A maneira errada de usar `Optional` é simplesmente chamar `.get()` sem verificar se o valor está presente, pois isso pode lançar uma `NoSuchElementException`.

Para entender o motivo, precisamos pensar na proposta do `Optional`: ele é um **contrato** que diz "este container pode ou não ter um valor". O método `.get()` foi projetado para ser a forma mais direta de extrair o valor, mas ele opera sob uma premissa perigosa: ele **assume que o valor existe**.

Quando o `Optional` está vazio, o contrato do método `.get()` é lançar a exceção `NoSuchElementException`. Este é o seu comportamento esperado.

**O problema:** `NoSuchElementException` é uma `RuntimeException` (uma exceção não checada). Isso significa que o compilador Java **não obriga você a tratá-la** com um bloco `try-catch`. Se você chamar `.get()` em um `Optional` vazio sem uma verificação prévia, e não houver um tratamento de exceção, sua aplicação irá quebrar em tempo de execução.

**Exemplo prático do erro:**

```java
// Cenário: Buscamos um produto que não existe no banco de dados.
Optional<Product> productOptional = productRepository.findById(999L); // Retorna Optional.empty()

// Tentativa errada de extrair o valor
try {
    Product product = productOptional.get(); // <<-- ISTO VAI LANÇAR A EXCEÇÃO
    System.out.println("Produto encontrado: " + product.getName());
} catch (NoSuchElementException e) {
    System.err.println("ERRO: O produto não foi encontrado. A aplicação quebrou aqui!");
    // Em uma API REST, isso resultaria em um erro 500 (Internal Server Error)
    // se não fosse tratado, o que é uma péssima experiência para o usuário.
}
```

Essencialmente, chamar `.get()` sem uma verificação (`.isPresent()`) é trocar um risco (`NullPointerException`) por outro (`NoSuchElementException`), o que não resolve o problema fundamental de lidar com a ausência de valor.

É por isso que as formas idiomáticas (`ifPresent`, `orElse`, `orElseThrow`, etc.) são preferíveis: elas **forçam o desenvolvedor a lidar explicitamente com o caso em que o `Optional` está vazio**, resultando em um código mais seguro, robusto e previsível.

**As formas seguras e idiomáticas de usar `Optional` são as seguintes:**

  * **`ifPresent(Consumer<T>)`**: Executa uma ação somente se o valor estiver presente.

    ```java
    Optional<User> user = userRepository.findById(1L);
    user.ifPresent(u -> System.out.println("Usuário encontrado: " + u.getName()));
    ```

  * **`orElse(T other)`**: Retorna o valor se presente, senão retorna um valor padrão.

    ```java
    String username = user.map(User::getName).orElse("Usuário Padrão");
    ```

  * **`orElseThrow(Supplier<X> exceptionSupplier)`**: Retorna o valor se presente, senão lança uma exceção. **Esta é a forma mais comum em APIs REST.**

    ```java
    Contact contact = contactRepository.findById(id)
        .orElseThrow(() -> new ResourceNotFoundException("Contato não encontrado: " + id));
    ```

### **4.3. Cuidado: `orElse()` vs. `orElseGet()`**

**Existe uma diferença de performance sutil, mas importante, entre `orElse()` e `orElseGet()`.**

  * **`orElse(valorPadrao)`**: O `valorPadrao` é **sempre** calculado, mesmo que o `Optional` contenha um valor.
  * **`orElseGet(supplier)`**: O `supplier` (uma lambda) só é executado se o `Optional` estiver vazio.

**Exemplo:**

```java
// Método custoso que busca um nome padrão no banco de dados
public String getNomePadraoSistema() {
    System.out.println("Executando método custoso...");
    // ...lógica de banco...
    return "Admin";
}

Optional<String> nomeUsuario = Optional.of("Ana");

// Mau uso: o método custoso será executado desnecessariamente.
String nome1 = nomeUsuario.orElse(getNomePadraoSistema()); 
// Console imprime: "Executando método custoso..."

// Bom uso: o método custoso NÃO será executado.
String nome2 = nomeUsuario.orElseGet(this::getNomePadraoSistema);
// Console não imprime nada.
```

**Regra geral:** Se o valor padrão for uma constante simples, use `orElse()`. Se a criação do valor padrão for custosa (envolve I/O, cálculo, etc.), use `orElseGet()`. Vejamos abaixo uma tabela comparativa entre esses dois método.

### Tabela Comparativa: `orElse()` vs. `orElseGet()`

| Característica              | `orElse(T valor)`                                             | `orElseGet(Supplier<T> supplier)`                                    |
| --------------------------- | ------------------------------------------------------------ | -------------------------------------------------------------------- |
| **Argumento que Aceita** | Um valor (`T`) já criado ou uma constante.                   | Uma função/lambda (`Supplier<T>`) que sabe como criar o valor.       |
| **Custo da Operação Padrão** | O valor padrão é calculado **sempre**, mesmo se o `Optional` contiver um valor. | A função para criar o valor padrão é executada **apenas se o `Optional` estiver vazio**. |
| **Cenário Ideal** | Usar com valores constantes ou objetos que já existem e são baratos de obter. | Usar quando a criação do valor padrão exige cálculo, I/O ou chamadas a outros métodos custosos. |

O exemplo abaixo exemplifica novamente os conceitos, para não deixarmos nenhuma dúvida sobre quando usar um ou outro. 🤓

### Mais um exemplo de Código Prático

Imagine que estamos buscando uma configuração em um mapa. Se a configuração não existir, precisamos obter um valor padrão de um método que simula uma operação lenta (como uma consulta ao banco).

```java
import java.util.Map;
import java.util.Optional;

public class OptionalExample {

    // Método que simula uma operação custosa (ex: acesso a banco, chamada de API)
    public static String getExpensiveDefaultValue() {
        System.out.println(">>> Executando método custoso para obter valor padrão...");
        // Simula um delay
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "default_value";
    }

    public static void main(String[] args) {
        Map<String, String> configs = Map.of("key1", "value1");

        System.out.println("--- Teste com um Optional que contém valor ---");
        Optional<String> optionalWithValue = Optional.of("value1");

        // Usando orElse(): O método custoso SERÁ chamado desnecessariamente.
        System.out.println("Usando orElse():");
        String value1 = optionalWithValue.orElse(getExpensiveDefaultValue());
        System.out.println("Resultado: " + value1);

        System.out.println("\n"); // Separador

        // Usando orElseGet(): O método custoso NÃO será chamado.
        System.out.println("Usando orElseGet():");
        String value2 = optionalWithValue.orElseGet(() -> getExpensiveDefaultValue()); // ou OptionalExample::getExpensiveDefaultValue
        System.out.println("Resultado: " + value2);

        System.out.println("\n--- Teste com um Optional vazio ---");
        Optional<String> emptyOptional = Optional.empty();

        // Com Optional vazio, ambos chamarão o método, mas orElseGet() continua sendo a prática mais segura.
        System.out.println("Usando orElse() com Optional vazio:");
        String value3 = emptyOptional.orElse(getExpensiveDefaultValue());
        System.out.println("Resultado: " + value3);
    }
}
```

**Saída do Código:**

```
--- Teste com um Optional que contém valor ---
Usando orElse():
>>> Executando método custoso para obter valor padrão...
Resultado: value1

Usando orElseGet():
Resultado: value1

--- Teste com um Optional vazio ---
Usando orElse() com Optional vazio:
>>> Executando método custoso para obter valor padrão...
Resultado: default_value
```

Como o resultado demonstra, `orElseGet()` evitou a execução desnecessária do método custoso quando o `Optional` já continha um valor, otimizando a performance da aplicação.

Passemos agora a um uso prático do `Optional`.

### **4.4. Exemplo Prático: Usando `Optional` para Conectar Camadas**

Agora que conhecemos as ferramentas, vamos aplicá-las em um cenário realista. Imagine a estrutura de uma aplicação comercial dividida em camadas:

  * **Repository:** Responsável por acessar os dados (simularemos um banco em memória).
  * **Service:** Onde mora a lógica de negócio.
  * **Controller:** A porta de entrada que recebe as requisições (simularemos com uma classe `main`).

Nossa tarefa é criar um método que atualiza o e-mail de um usuário, mas apenas se o usuário existir e se o novo e-mail for válido. Veja como o `Optional` cria um contrato seguro e elegante entre essas camadas (lembrando que o código-fonte abaixo é simplificado para fins didáticos!).

#### **Camada 1: O Repositório (Simulando o acesso a dados)**

Primeiro, nosso `UserRepository` simula a busca no banco. Note como ele já retorna um `Optional`, protegendo o resto da aplicação desde a fonte dos dados.

```java
// Simula uma entidade do banco de dados
class User {
    private Long id;
    private String name;
    private String email;
    // Construtores, Getters e Setters...
    public User(Long id, String name, String email) { this.id = id; this.name = name; this.email = email; }
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
    @Override public String toString() { return "User{id=" + id + ", name='" + name + "', email='" + email + "'}"; }
}

// Simula a classe que acessa o banco de dados
class UserRepository {
    // Um mapa para simular nossa tabela de usuários no banco
    private final Map<Long, User> database = new HashMap<>();

    public UserRepository() {
        // Populando o "banco" com dados iniciais
        database.put(1L, new User(1L, "Ana", "ana@email.com"));
        database.put(2L, new User(2L, "Carlos", "carlos@email.com"));
    }

    // O método de busca já retorna um Optional, como faz o Spring Data JPA
    public Optional<User> findById(Long id) {
        return Optional.ofNullable(database.get(id));
    }

    public User save(User user) {
        database.put(user.id, user);
        return user;
    }
}
```

#### **Camada 2: O Serviço (Onde a lógica de negócio acontece)**

O `UserService` recebe o `UserRepository` e usa o `Optional` para criar um fluxo seguro de validação e atualização.

```java
// Simula a camada de serviço com a lógica de negócio
class UserService {
    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public Optional<User> updateUserEmail(Long userId, String newEmail) {
        // 1. Busca o usuário. O retorno já é um Optional<User>.
        return userRepository.findById(userId)
                .filter(user -> isEmailValid(newEmail)) // 2. Continua apenas se o email for válido.
                .map(user -> {                          // 3. Se tudo estiver ok, transforma o objeto.
                    user.setEmail(newEmail);
                    return userRepository.save(user);   // 4. Salva e retorna o usuário atualizado.
                });
    }

    private boolean isEmailValid(String email) {
        return email != null && email.contains("@");
    }
}
```

* Aqui buscamos o `Optional`, filtramos, e mapeamos para a nova versão salva. Se em qualquer ponto o `Optional` ficar vazio (usuário não encontrado ou e-mail inválido), as operações seguintes são simplesmente ignoradas.

#### **Camada 3: O "Controller" e a Execução (`main`)**

Finalmente, nossa classe `main` simula a "requisição" do cliente e mostra como o `Controller` interpretaria a resposta segura vinda do `Service`.

```java
// Simula a camada que recebe as "requisições"
class UserController {
    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    public String handleUpdateEmailRequest(Long id, String newEmail) {
        // O "Controller" chama o serviço e lida com o Optional retornado.
        return userService.updateUserEmail(id, newEmail)
                .map(user -> "200 OK - Usuário atualizado: " + user.toString()) // Se sucesso, formata a resposta OK.
                .orElse("404 Not Found - Usuário não encontrado ou email inválido."); // Se vazio, formata a resposta de erro.
    }
}

// Ponto de entrada da nossa aplicação simulada
public class MainApplication {
    public static void main(String[] args) {
        // Injeção de dependência manual: criamos e conectamos as camadas.
        UserRepository userRepository = new UserRepository();
        UserService userService = new UserService(userRepository);
        UserController userController = new UserController(userService);

        // --- Simulação das Requisições ---

        // Teste 1: Caso de sucesso
        System.out.println("Tentativa 1: Atualizando usuário com ID 1...");
        String response1 = userController.handleUpdateEmailRequest(1L, "ana.nova@email.com");
        System.out.println("Resposta: " + response1);
        System.out.println("--------------------");

        // Teste 2: Caso de falha (usuário não existe)
        System.out.println("Tentativa 2: Atualizando usuário com ID 99...");
        String response2 = userController.handleUpdateEmailRequest(99L, "fantasma@email.com");
        System.out.println("Resposta: " + response2);
    }
}
```

Este exemplo de ponta a ponta, mesmo sem uso de framework, demonstra o poder do `Optional`: ele atua como um **contrato seguro entre as camadas**, eliminando a necessidade de verificações de nulo e permitindo a criação de fluxos de dados robustos e declarativos, que são fáceis de ler e manter.

---

## **5. Method References: "Sintaxe Açucarada" para Lambdas**

Em muitos casos, uma expressão lambda apenas chama um método existente. As **referências de método (Method References)** são uma sintaxe compacta para esses casos.

| Tipo de Referência | Exemplo Lambda | Exemplo Method Reference |
| :--- | :--- | :--- |
| **Método Estático** | `str -> Integer.parseInt(str)` | `Integer::parseInt` |
| **Método de Instância (objeto específico)** | `() -> expensiveObject.get()` | `expensiveObject::get` |
| **Método de Instância (objeto arbitrário)** | `s -> s.toUpperCase()` | `String::toUpperCase` |
| **Construtor** | `() -> new ArrayList<>()` | `ArrayList::new` |

**Exemplo Prático:**

A atualização parcial de um contato usava `.ifPresent()`:

```java
// Com lambda
dto.getNome().ifPresent(nome -> existingContact.setNome(nome));

// Com method reference (mais limpo e preferível)
dto.getNome().ifPresent(existingContact::setNome);
```

Ambas as linhas fazem a mesma coisa, mas a referência de método é mais expressiva e fácil de ler.