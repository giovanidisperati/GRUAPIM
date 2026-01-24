---
layout: aula
title: 1. Fundamentos da Linguagem Java
parent: Aula 01 - Revisão sobre Java e POO
nav_order: 1
---

## **PARTE I - Fundamentos da Linguagem Java**  

### **1. Sintaxe Básica do Java**

### **1.1. Tipos Primitivos e Variáveis**

No Java, os tipos primitivos representam valores simples, não objetos, e são otimizados para performance. Entre eles temos:
- **byte** (8 bits, -128 a 127)
- **short** (16 bits, -32768 a 32767)
- **int** (32 bits, -2.147.483.648 a 2.147.483.647)
- **long** (64 bits)
- **float** (32 bits, ponto flutuante)
- **double** (64 bits, ponto flutuante, maior precisão)
- **char** (16 bits, representa um caractere Unicode)
- **boolean** (pode ser `true` ou `false`)

Ao declarar uma variável, fazemos:

```java
int idade = 25;
double salario = 2500.75;
boolean ativo = true;
```
### **1.2. Operadores**

  - **Aritméticos**: `+`, `-`, `*`, `/`, `%`
  - **De Atribuição**:
      - `=` (Atribuição simples): Atribui o valor da direita à variável da esquerda.
      - `+=`, `-=`, `*=`, `/=`, `%=` (Atribuição composta): Realiza uma operação aritmética e atribui o resultado à variável. Por exemplo, `x += y` é uma forma abreviada de `x = x + y`.
  - **Relacionais**: `==`, `!=`, `>`, `>=`, `<`, `<=`
  - **Lógicos**: `&&` (E), `||` (OU), `!` (NÃO)
  - **Unários**: `++` (incremento), `--` (decremento)
  - **Ternário**: `condicao ? valorSeVerdadeiro : valorSeFalso`

**Exemplo de Operadores de Atribuição:**

```java
int saldo = 100;
saldo += 50; // Equivalente a: saldo = saldo + 100; (saldo agora é 150)
saldo -= 20; // Equivalente a: saldo = saldo - 20; (saldo agora é 130)

int x = (5 > 3) ? 10 : 0; // Exemplo de ternário, que também faz uma atribuição
```

### **1.3. Estruturas de Controle**
Estruturas de controle direcionam o fluxo de execução do programa.

* **Decisão (`if-else`, `switch`)**:

```java
if (idade >= 18) {
    System.out.println("Maior de idade");
} else {
    System.out.println("Menor de idade");
}
```

* **Repetição (`for`, `for-each`, `while`, `do-while`)**:

* **`for` tradicional**: Ideal quando você precisa de acesso ao índice ou tem um controle mais complexo sobre a iteração.

```java
for (int i = 0; i < 5; i++) {
    System.out.println("O valor de i é: " + i);
}
```

* **`for-each` (Enhanced for loop)**: A forma mais simples e segura de percorrer todos os elementos de um array ou coleção, sem se preocupar com índices.

```java
List<String> nomes = Arrays.asList("Ana", "João", "Carlos");

for (String nome : nomes) {
    System.out.println("Olá, " + nome);
}
```

* **`while` e `do-while`**: Executam um bloco de código enquanto uma condição for verdadeira. A diferença é que o `do-while` garante que o bloco seja executado pelo menos uma vez.

```java
int contador = 0;
while (contador < 3) {
    System.out.println("Contador: " + contador);
    contador++;
}
```

### **1.4. Arrays**
Os arrays em Java são estruturas de dados de tamanho fixo, utilizados para armazenar elementos do mesmo tipo. Eles são declarados da seguinte forma:

```java
int[] numeros = new int[5]; // Array de inteiros com 5 posições
String[] nomes = {"João", "Maria", "Pedro"}; // Array inicializado diretamente
```

Os arrays têm a vantagem de serem eficientes em termos de memória e acessíveis via índices, mas sua desvantagem é o tamanho fixo, que não pode ser alterado após a criação.

### **1.5. A Classe `String`**

Diferente dos tipos vistos na seção 1.1, **`String` não é um tipo primitivo, e sim uma classe**. Isso significa que toda variável do tipo `String` é um objeto, com seus próprios métodos.

```java
String nome = "Maria Silva"; // 'nome' é um objeto da classe String

int quantidadeDeLetras = nome.length(); // Chamando um método do objeto
String nomeMaiusculo = nome.toUpperCase(); // Criando um novo objeto String

System.out.println(nomeMaiusculo); // Imprime "MARIA SILVA"
```

### **1.6. Constantes com a Palavra-chave `final`**

Para declarar uma variável cujo valor não pode ser alterado após a inicialização (uma constante), usamos a palavra-chave `final`.

```java
final double PI = 3.14159;
// A linha abaixo causaria um erro de compilação, pois PI é uma constante.
// PI = 3.14; 
```

### **1.7. Conversão de Tipos (Type Casting)**

Converter um valor de um tipo para outro é uma operação comum.

  * **Conversão Implícita (Alargamento):** Ocorre automaticamente quando não há risco de perda de dados (de um tipo menor para um maior).

    ```java
    int meuInt = 100;
    double meuDouble = meuInt; // Conversão automática para 100.0
    ```

  * **Conversão Explícita (Estreitamento / Casting):** Deve ser feita manualmente quando há risco de perda de informação.

    ```java
    double precoProduto = 19.99;
    int precoInteiro = (int) precoProduto; // Forçamos a conversão para int
    // O valor de precoInteiro será 19 (a parte decimal é perdida)
    ```

### **1.8. Entrada de Dados pelo Usuário (Scanner)**

Para criar programas interativos, usamos a classe `Scanner` para ler dados digitados pelo usuário.

```java
import java.util.Scanner; // 1. Precisa importar a classe

public class InteracaoUsuario {
    public static void main(String[] args) {
        // 2. Cria o objeto Scanner para ler da entrada do sistema
        Scanner leitor = new Scanner(System.in);

        System.out.print("Digite seu nome: ");
        String nome = leitor.nextLine(); // Lê uma linha de texto

        System.out.print("Digite sua idade: ");
        int idade = leitor.nextInt(); // Lê um número inteiro

        System.out.printf("Olá, %s! Você tem %d anos.%n", nome, idade);

        // 3. É uma boa prática fechar o leitor quando não for mais usar
        leitor.close(); 
    }
```