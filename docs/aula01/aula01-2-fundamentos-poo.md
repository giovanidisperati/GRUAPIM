---
layout: aula
title: 2. O Paradigma da POO
parent: Aula 01 - Revisão sobre Java e POO
nav_order: 2
---

## **PARTE II - O Paradigma da Programação Orientada a Objetos (POO)**

A Programação Orientada a Objetos (POO) é um paradigma que estrutura o software em torno de "objetos" em vez de funções e lógica. Vamos explorar seus pilares fundamentais.

### **2.1. Classes, Objetos e Construtores**

  * **Classe**: É o nosso molde ou planta. Ela define um conjunto de **atributos** (características ou dados) e **métodos** (ações ou comportamentos) que um tipo de objeto terá.
  * **Objeto**: É a instância concreta de uma classe, criada em memória durante a execução do programa. Cada objeto tem seu próprio estado (valores dos seus atributos).
  * **Construtor**: Um método especial, com o mesmo nome da classe, responsável por inicializar um objeto no momento de sua criação (`new`).

**Exemplo Simples:**
A classe `Produto` define o que todo produto no nosso sistema terá: um nome e um preço. O construtor garante que todo produto seja criado com esses valores.

```java
public class Produto {
    // Atributos (estado do objeto)
    private String nome;
    private double preco;

    // Construtor: inicializa o objeto quando ele é criado
    public Produto(String nome, double preco) {
        this.nome = nome;
        this.preco = preco;
    }

    // Métodos (comportamento do objeto)
    public String getNome() { return nome; }
    public double getPreco() { return preco; }

    public double calcularDesconto(double percentual) {
        return this.preco * (1 - percentual / 100);
    }
}
```

### **2.2. Encapsulamento e Modificadores de Acesso**

O encapsulamento é o princípio de proteger os dados internos de um objeto de acessos indevidos. Em Java, isso é feito declarando os atributos como `private` e fornecendo métodos públicos (`getters` e `setters`) para acessá-los de forma controlada.

  * **`public`**: Acessível de qualquer lugar.
  * **`private`**: Acessível apenas de dentro da própria classe.
  * **`protected`**: Acessível pela própria classe, por classes no mesmo pacote e por subclasses.
  * **(default)**: Acessível apenas por classes no mesmo pacote.

**Exemplo Prático:**
Na classe `SalariedEmployee`, o `weeklySalary` é privado. O método `setWeeklySalary` valida o valor antes de atribuí-lo, garantindo que o salário nunca seja negativo.

```java
public class SalariedEmployee extends Employee {
    private double weeklySalary; // Atributo privado e protegido

    // Método público para obter o valor (Getter)
    public double getWeeklySalary() {
        return weeklySalary;
    }

    // Método público para alterar o valor com validação (Setter)
    public void setWeeklySalary(double weeklySalary) {
        if (weeklySalary < 0.0) {
            throw new IllegalArgumentException("Weekly salary must be >= 0.0");
        }
        this.weeklySalary = weeklySalary;
    }
    //...
}
```

### **2.3. Herança e a Palavra-chave `super`**

A herança permite que uma classe (subclasse) herde atributos e métodos de outra (superclasse), promovendo o reuso de código através de uma relação **"é-um"**. A subclasse pode adicionar novos comportamentos ou modificar os herdados.

  * **`extends`**: Palavra-chave usada para definir a herança.
  * **`super`**: Palavra-chave usada para se referir à superclasse, seja para chamar seu construtor (`super(...)`) ou seus métodos (`super.metodo()`).

**Exemplo Prático:**
No nosso sistema de pagamentos, `BasePlusCommissionEmployee` **é um** `CommissionEmployee` que também tem um salário base. Ele herda tudo de `CommissionEmployee` e apenas adiciona o que lhe é específico.

```java
// Superclasse
public class CommissionEmployee {
    // Atributos como firstName, lastName, grossSales, etc.
    public CommissionEmployee(String firstName, ..., double commissionRate) {
        // ... lógica do construtor
    }
    public double earnings() {
        return getCommissionRate() * getGrossSales();
    }
    // ...
}

// Subclasse
public class BasePlusCommissionEmployee extends CommissionEmployee {
    private double baseSalary;

    public BasePlusCommissionEmployee(String firstName, ..., double baseSalary) {
        // 1. Chama o construtor da superclasse para inicializar os atributos herdados
        super(firstName, ..., commissionRate);
        // 2. Inicializa seu próprio atributo
        this.baseSalary = baseSalary;
    }

    // Sobrescreve o método earnings para adicionar sua própria lógica
    @Override
    public double earnings() {
        // 3. Reutiliza o método da superclasse e adiciona o salário base
        return getBaseSalary() + super.earnings();
    }
    // ...
}
```

### **2.4. Polimorfismo e a Palavra-chave `this`**

Polimorfismo (do grego, "muitas formas") é a capacidade de um objeto ser referenciado de múltiplas maneiras. Em termos práticos, permite que tratemos objetos de subclasses diferentes de forma uniforme, através da referência da superclasse.

  * **`@Override`**: Anotação que indica que um método está sobrescrevendo um método da superclasse.
  * **`this`**: Palavra-chave que se refere à **instância atual** do objeto. É usada para desambiguar variáveis de instância de parâmetros locais ou para chamar outro construtor da mesma classe (`this(...)`).

**Exemplo Prático:**
Podemos ter um array do tipo `Employee` (a superclasse) que armazena objetos de vários tipos de funcionários (`SalariedEmployee`, `HourlyEmployee`, etc.). Ao iterar e chamar o método `getPaymentAmount()`, o Java, através da **ligação dinâmica**, executa a versão correta do método para cada objeto específico.

```java
// A interface Payable define o contrato
public interface Payable {
    double getPaymentAmount();
}

// Employee implementa o contrato
public abstract class Employee implements Payable {
    // ...
}

// As subclasses concretas fornecem a implementação
public class SalariedEmployee extends Employee {
    private double weeklySalary;
    // ...
    @Override
    public double getPaymentAmount() { return this.weeklySalary; } // 'this' é opcional aqui
}

public class Invoice implements Payable {
    private int quantity;
    private double pricePerItem;
    // ...
    @Override
    public double getPaymentAmount() { return this.quantity * this.pricePerItem; }
}

// --- Polimorfismo em Ação ---
public class TestePagamentos {
    public static void main(String[] args) {
        // Array do tipo da INTERFACE pode conter qualquer objeto que a implemente
        Payable[] objetosPagaveis = new Payable[2];
        objetosPagaveis[0] = new SalariedEmployee("João", "Silva", "111", 1200.0);
        objetosPagaveis[1] = new Invoice("01234", "Peça de computador", 2, 350.0);

        System.out.println("Processando pagamentos de forma polimórfica:");

        for (Payable pagavel : objetosPagaveis) {
            // Não importa se 'pagavel' é um Employee ou um Invoice,
            // ele responderá à chamada getPaymentAmount() da sua própria maneira.
            System.out.printf("Pagamento devido: $%,.2f%n", pagavel.getPaymentAmount());
        }
    }
}
```

Algo importante a se citar é que o comportamento polimórfico acima é viabilizado pelo mecanismo de **ligação dinâmica** (dynamic binding), como mencionado anteriormente. Esse processo ocorre em duas etapas:

1.  **Em tempo de compilação:** O compilador valida a chamada `pagavel.getPaymentAmount()` apenas com base no tipo da referência (`Payable`), garantindo que o método existe no contrato da interface. Nesta fase, a implementação específica a ser executada ainda é desconhecida.

2.  **Em tempo de execução:** A JVM (Java Virtual Machine) identifica a classe **real** do objeto ao qual a referência `pagavel` aponta a cada iteração (`SalariedEmployee` ou `Invoice`). Somente nesse momento a JVM "liga" a chamada do método à sua implementação (`@Override`) correspondente, encontrada na classe do objeto real.

Dessa forma, a mesma linha de código no laço `for` invoca diferentes blocos de código, o que torna o sistema extensível e flexível.

### **2.5. Classes Abstratas e Interfaces**

Tanto classes abstratas quanto interfaces são usadas para definir contratos e alcançar o polimorfismo, mas elas têm propósitos diferentes.

  * **Classe Abstrata**: Usada para criar uma classe base que compartilha código comum (atributos e métodos concretos) com múltiplas subclasses. Uma classe só pode herdar de **uma** classe abstrata. Use quando as subclasses compartilham uma forte relação **"é-um"** e código.

      * **Exemplo**: `Employee` é uma classe abstrata porque todos os funcionários têm `firstName` e `lastName`, mas o cálculo de `getPaymentAmount()` é específico para cada tipo.

  * **Interface**: Define um contrato puro de comportamentos (métodos) que uma classe deve implementar. Uma classe pode implementar **múltiplas** interfaces. Use para definir uma capacidade ou "papel" que classes não relacionadas podem desempenhar.

      * **Exemplo**: `Payable` é uma interface porque tanto um `Employee` quanto uma `Invoice` podem ser "pagáveis", mas não compartilham nenhuma outra característica em comum.

| Característica         | Classe Abstrata                                   | Interface                                              |
| ---------------------- | ------------------------------------------------- | ------------------------------------------------------ |
| **Herança** | Uma classe pode herdar de apenas UMA classe abstrata. | Uma classe pode implementar MÚLTIPLAS interfaces.      |
| **Atributos** | Pode ter atributos de instância (não `static`).   | Não pode ter atributos de instância (apenas constantes `static final`). |
| **Métodos Concretos** | Pode ter métodos com implementação.               | Pode ter métodos `default` e `static` (desde o Java 8). |
| **Propósito Principal**| Compartilhar código e identidade comum (relação "é-um"). | Definir um contrato de comportamento (relação "é capaz de"). |

### **2.6. Composição sobre Herança: Um Princípio de Design**

Como vimos, a herança cria um forte acoplamento. Muitas vezes, um design mais flexível é alcançado através da **composição**, onde uma classe contém uma instância de outra classe (relação **"tem-um"**).

**Quando usar Herança?**

  * Quando a relação "é-um" é genuína e imutável (`SalariedEmployee` sempre será um `Employee`).
  * Quando a superclasse foi projetada para ser estendida e é estável.

**Quando preferir Composição?**

  * Para reutilizar código de classes não relacionadas.
  * Quando você quer poder alterar o comportamento em tempo de execução.
  * Para criar designs mais flexíveis e com menor acoplamento, favorecendo a injeção de dependências.

**Exemplo de Design Flexível com Composição:**
Um `Personagem` que pode ter diferentes `HabilidadeMovimento`. Em vez de criar `HeroiVoador` e `HeroiNadador`, o `Personagem` **tem uma** `HabilidadeMovimento` que pode ser trocada.

```java
interface HabilidadeMovimento {
    void mover();
}
class Voar implements HabilidadeMovimento { /*...*/ }
class Nadar implements HabilidadeMovimento { /*...*/ }

class Personagem {
    private HabilidadeMovimento habilidade; // Composição

    public Personagem(HabilidadeMovimento habilidadeInicial) {
        this.habilidade = habilidadeInicial;
    }

    public void setHabilidade(HabilidadeMovimento novaHabilidade) {
        this.habilidade = novaHabilidade; // Comportamento pode ser alterado
    }

    public void mover() {
        this.habilidade.mover();
    }
}
```