---
layout: aula
title: 3. Tópicos essenciais do Java
parent: Aula 01 - Revisão sobre Java e POO
nav_order: 3
---

## **PARTE III - Tópicos Essenciais do Ecossistema Java**

Com os fundamentos da POO estabelecidos, vamos agora explorar recursos cruciais da plataforma Java que nos permitem escrever código mais seguro, robusto e flexível.

### **3.1. Tratamento de Exceções**

O **tratamento de exceções** é o mecanismo do Java para lidar com erros que ocorrem durante a execução do programa de forma controlada, evitando que a aplicação pare abruptamente. As exceções são tratadas utilizando os blocos `try`, `catch` e `finally`.

#### **Exemplo de Tratamento de Exceção**

```java
try {
    // Bloco de código onde um erro pode ocorrer
    int resultado = 10 / 0;
} catch (ArithmeticException e) {
    // Bloco executado se a exceção do tipo especificado ocorrer
    System.out.println("Erro: Tentativa de divisão por zero.");
} finally {
    // Bloco opcional que SEMPRE será executado, com ou sem exceção
    System.out.println("Finalizando a operação.");
}
```

#### **Tipos de Exceções**

  * **Exceções Checadas (Checked Exceptions)**: São exceções que o compilador obriga o programador a tratar (`try-catch`) ou declarar (`throws`). Geralmente representam condições externas recuperáveis (ex: `IOException`, `SQLException`).

    ```java
    try {
        FileReader file = new FileReader("arquivo_inexistente.txt");
    } catch (FileNotFoundException e) {
        System.out.println("Arquivo não pôde ser encontrado.");
    }
    ```

  * **Exceções Não Checadas (Unchecked Exceptions)**: São exceções que ocorrem em tempo de execução, geralmente devido a erros de programação, e não precisam ser obrigatoriamente tratadas (ex: `NullPointerException`, `ArrayIndexOutOfBoundsException`).

    ```java
    String texto = null;
    // A linha abaixo lançará uma NullPointerException em tempo de execução
    // System.out.println(texto.length());
    ```

#### **Criando Exceções Personalizadas**

Podemos criar nossas próprias exceções para representar erros específicos do nosso sistema, herdando de `Exception` (checada) ou `RuntimeException` (não checada).

```java
class SaldoInsuficienteException extends Exception {
    public SaldoInsuficienteException(String mensagem) {
        super(mensagem);
    }
}
```

### **3.2. Generics**

Os **Generics** permitem definir **tipos parametrizados** para classes, interfaces e métodos. O principal benefício é aumentar a segurança de tipos em tempo de compilação, eliminando a necessidade de *casts* e evitando erros em tempo de execução.

#### **Conceito de Generics**

Antes dos Generics (Java 5), coleções armazenavam `Object`, o que permitia adicionar qualquer tipo de dado a uma mesma lista, gerando potenciais `ClassCastException` em tempo de execução. Com Generics, especificamos o tipo de dado que a coleção irá armazenar.

#### **Exemplo com `List`**

```java
// O uso de <String> garante que esta lista só aceitará Strings
List<String> nomes = new ArrayList<>();
nomes.add("Ana");
nomes.add("Carlos");

// A linha abaixo causaria um ERRO DE COMPILAÇÃO, garantindo a segurança de tipos.
// nomes.add(10); 

String primeiroNome = nomes.get(0); // Não é necessário fazer cast: (String) nomes.get(0)
```

#### **Benefícios dos Generics**

  * **Segurança de tipos**: Detecta erros de tipo em tempo de compilação.
  * **Reutilização de código**: Permite criar componentes genéricos que funcionam com qualquer tipo.
  * **Legibilidade**: O código se torna mais claro, pois as intenções de tipo são explícitas.

### **3.3. Reflexão (Reflection)**

A **Reflexão** é um mecanismo avançado da API do Java que permite a um programa inspecionar e manipular suas próprias estruturas (classes, métodos, atributos) em tempo de execução.

É uma ferramenta poderosa, sendo a base para o funcionamento de muitos frameworks modernos como Spring (injeção de dependências) e Hibernate/JPA (mapeamento objeto-relacional).

#### **Exemplo de Inspeção de Classe**

Podemos, por exemplo, listar todos os métodos de uma classe dinamicamente.

```java
Class<?> classe = String.class; // Obtém a representação da classe String
Method[] metodos = classe.getDeclaredMethods();

System.out.println("Métodos da classe String:");
for (Method metodo : metodos) {
    System.out.println("- " + metodo.getName());
}
```

#### **Exemplo de Modificação de Atributos Privados**

A reflexão pode até mesmo quebrar o encapsulamento para acessar e modificar atributos privados (algo que deve ser feito com extremo cuidado, geralmente em testes ou frameworks).

```java
class Pessoa {
    private String nome = "João";
}

//...
Pessoa p = new Pessoa();
Field campo = p.getClass().getDeclaredField("nome");
campo.setAccessible(true); // Permite o acesso ao campo privado
campo.set(p, "Maria"); // Altera o valor do atributo 'nome' no objeto 'p'
System.out.println(campo.get(p)); // Imprime "Maria"
```