---
layout: aula
title: 2. Expressões Lambda
parent: Aula 02 - Lambdas, Streams e Optionals
nav_order: 2
---

## **2. Expressões Lambda: A Revolução da Concisão**

Antes do Java 8, tarefas simples como ordenar uma lista com um critério customizado ou definir um evento de clique exigiam a criação de **classes anônimas internas**, resultando em um código verboso.

**O problema:**

```java
// Ordenando uma lista de Strings por tamanho (antes do Java 8)
Collections.sort(nomes, new Comparator<String>() {
    @Override
    public int compare(String s1, String s2) {
        return Integer.compare(s1.length(), s2.length());
    }
});
```

Toda essa estrutura (`new Comparator...`) existe apenas para passar um único método (`compare`) como lógica.

As **Expressões Lambda** resolvem isso, permitindo que você trate funcionalidades como um argumento de método, ou código como dados.

**A solução com Lambda:**

```java
// A mesma ordenação, agora com uma expressão lambda
Collections.sort(nomes, (s1, s2) -> Integer.compare(s1.length(), s2.length()));
```

O código faz exatamente a mesma coisa, mas de forma muito mais direta e legível. A seguir, vamos detalhar cada parte desse código.

Ele utiliza o método `Collections.sort()`, que aceita dois parâmetros:

#### **1. Primeiro Parâmetro: `nomes`**

  * **O que é:** É a `List<String>` que você deseja ordenar.
  * **Função:** É o alvo da operação de ordenação.

#### **2. Segundo Parâmetro: A Expressão Lambda**

  * **O que é:** É a lógica de comparação, fornecida como uma função. Ela diz ao método `sort` *como* ele deve decidir qual dos dois elementos vem primeiro.

  * **Função:** A lambda `(s1, s2) -> Integer.compare(s1.length(), s2.length())` é uma implementação concisa da interface funcional `Comparator<String>`. Vamos quebrar a lambda em partes:

      * **`(s1, s2)`**: Estes são os parâmetros da função. Em qualquer momento da ordenação, o algoritmo pegará dois elementos da lista `nomes` para comparar, que serão representados por `s1` e `s2` (ambos do tipo `String`).

      * **`->`**: É o operador que separa os parâmetros do corpo (a lógica) da função.

      * **`Integer.compare(s1.length(), s2.length())`**: Esta é a implementação da lógica de comparação.

          * `s1.length()`: Pega o comprimento (número de caracteres) da primeira String.
          * `s2.length()`: Pega o comprimento da segunda String.
          * `Integer.compare(a, b)`: Este método compara dois inteiros (`a` e `b`) e retorna:
              * Um número **negativo** se `a` for menor que `b`.
              * **Zero** se `a` for igual a `b`.
              * Um número **positivo** se `a` for maior que `b`.

O algoritmo de ordenação do `Collections.sort` usa esse retorno para organizar a lista, resultando em uma ordenação baseada no comprimento das strings, da menor para a maior.

Bem prático, né? 👽

### **2.1. Anatomia de uma Expressão Lambda**

Uma lambda é, basicamente, uma função anônima. Como vimos no exemplo acima, sua sintaxe é:
`(parâmetros) -> { corpo da função }`

  * **Parâmetros:** Uma lista de parâmetros de entrada (pode ser vazia). O tipo pode ser omitido se o compilador conseguir inferi-lo.
  * **Seta `->`:** Separa os parâmetros do corpo.
  * **Corpo:** Uma única expressão ou um bloco de código. Se for uma única expressão, o `return` é implícito e as chaves `{}` são opcionais.

### **2.2. Interfaces Funcionais: A Base das Lambdas**

Uma expressão lambda só pode ser usada em um contexto onde um tipo é esperado. Pense nesse tipo como um **contrato** ou um **molde** para a lambda. Em Java, esse molde é sempre uma **Interface Funcional**.

Uma Interface Funcional é definida por uma regra simples: ela deve conter **apenas um método abstrato**. É a assinatura desse único método que define o formato que a lambda deve seguir (quais parâmetros ela recebe e o que ela retorna).

O Java já nos fornece um conjunto de interfaces funcionais prontas no pacote `java.util.function`. Dentre as mais comuns temos:

#### **1. `Predicate<T>`**

  * **Propósito:** Testar uma condição.
  * **Método abstrato:** `boolean test(T t)`
  * **Contrato:** Recebe um objeto do tipo `T` e retorna `true` ou `false`.
  * **Exemplo de Lambda:**
    ```java
    // Testa se um número é par
    Predicate<Integer> isEven = numero -> numero % 2 == 0;
    System.out.println(isEven.test(10)); // Saída: true
    ```
  * **Uso comum:** Ideal para filtros, como em `stream().filter()`.

#### **2. `Function<T, R>`**

  * **Propósito:** Transformar um valor de um tipo em outro.
  * **Método abstrato:** `R apply(T t)`
  * **Contrato:** Recebe um objeto do tipo `T` e retorna um objeto do tipo `R`.
  * **Exemplo de Lambda:**
    ```java
    // Transforma uma String em seu comprimento (Integer)
    Function<String, Integer> getLength = texto -> texto.length();
    System.out.println(getLength.apply("Java")); // Saída: 4
    ```
  * **Uso comum:** Perfeita para mapeamentos, como em `stream().map()`.

#### **3. `Consumer<T>`**

  * **Propósito:** "Consumir" um valor para executar uma ação, sem retornar nada.
  * **Método abstrato:** `void accept(T t)`
  * **Contrato:** Recebe um objeto do tipo `T` e retorna `void`.
  * **Exemplo de Lambda:**
    ```java
    // Ação de imprimir uma String no console
    Consumer<String> printText = texto -> System.out.println(texto);
    printText.accept("Olá, Mundo!"); // Imprime "Olá, Mundo!"
    ```
  * **Uso comum:** Utilizada em operações terminais que executam algo, como `list.forEach()`.

#### **4. `Supplier<T>`**

  * **Propósito:** Fornecer/criar um valor sem receber nenhuma entrada.
  * **Método abstrato:** `T get()`
  * **Contrato:** Não recebe parâmetros e retorna um objeto do tipo `T`.
  * **Exemplo de Lambda:**
    ```java
    // Fornece a data e hora atuais
    Supplier<LocalDateTime> now = () -> LocalDateTime.now();
    System.out.println(now.get()); 
    ```
  * **Uso comum:** Para criação de objetos ou fornecimento de valores padrão, como em `Optional.orElseGet()`.

A anotação `@FunctionalInterface` pode ser usada para garantir que uma interface cumpra o requisito de ter apenas um método abstrato. Ela é opcional, mas funciona como uma **verificação de segurança** para o compilador e uma **dica clara** para outros desenvolvedores de que a interface foi projetada para ser usada com lambdas. Vejamos como isso funciona.

### Criando sua própria Interface Funcional

Você pode facilmente criar suas próprias interfaces funcionais para representar contratos específicos do seu negócio.

**Exemplo: Criando um formatador de texto**

Vamos supor que precisamos de diferentes formas de formatar uma `String`. Podemos criar uma interface para isso:

```java
@FunctionalInterface
public interface TextFormatter {
    String format(String text);
}
```

  * **A Regra:** A interface `TextFormatter` tem apenas um método abstrato, `format(String text)`.
  * **A Anotação:** `@FunctionalInterface` garante que, se alguém tentar adicionar outro método abstrato a esta interface, o código não compilará.

**Como usar:**

Agora, podemos criar diferentes implementações para este contrato usando lambdas:

```java
// Implementação 1: Converte para maiúsculas
TextFormatter toUpperCase = texto -> texto.toUpperCase();

// Implementação 2: Adiciona "!!!" no final
TextFormatter addExclamation = texto -> texto + "!!!";

System.out.println(toUpperCase.format("java"));      // Saída: JAVA
System.out.println(addExclamation.format("Atenção")); // Saída: Atenção!!!
```

**Lembrando: o mais importante aqui é o conceito!** 

`@FunctionalInterface` formaliza a criação de "moldes" para as suas lambdas. Não se preocupe em decorar todas as interfaces prontas do Java agora. O fundamental é entender que, por trás de toda expressão lambda, existe uma interface funcional definindo o contrato que ela cumpre.

---
