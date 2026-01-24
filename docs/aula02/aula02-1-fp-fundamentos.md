---
layout: aula
title: 1. Introdução à Programação Funcional
parent: Aula 02 - Lambdas, Streams e Optionals
nav_order: 1
---

## **1. Mudando a Perspectiva: Uma Introdução à Programação Funcional**

Até agora, nossa visão sobre Java foi moldada pela **Programação Orientada a Objetos (POO)**. Na POO, modelamos o mundo como um conjunto de **objetos** que possuem *estado* (atributos) e *comportamento* (métodos). O foco está nos substantivos: um `Carro`, um `Cliente`, uma `NotaFiscal`. Nosso código opera modificando o estado desses objetos ao longo do tempo.

Agora, vamos introduzir uma perspectiva diferente, mas complementar: a **Programação Funcional (PF)**.

Na Programação Funcional, em vez de focar em objetos que mudam de estado, pensamos em termos de **transformação de dados**. O software é visto como uma série de funções matemáticas, onde cada função recebe uma entrada, processa-a e produz uma saída, **sem alterar nada fora de seu escopo**. O foco aqui está nos verbos: `calcular`, `filtrar`, `transformar`.

### **1.1 Os Pilares da Programação Funcional**

Para entender essa abordagem, precisamos conhecer três conceitos-chave que a sustentam:

#### **1. Funções como Cidadãos de Primeira Classe (First-Class Citizens)**

Este é o pilar central. Significa que as funções são tratadas como qualquer outro valor no programa. Em Java, isso é alcançado através das **Interfaces Funcionais** e **Expressões Lambda**. Nesse paradigma uma função pode ser:

  * **Atribuída a uma variável:**

    ```java
    // A função (x -> x * 2) é atribuída à variável 'dobrar'.
    // A variável 'dobrar' é do tipo Function<Integer, Integer>, uma interface funcional.
    Function<Integer, Integer> dobrar = x -> x * 2;

    // Agora, podemos usar a variável para executar a função.
    int resultado = dobrar.apply(5); // resultado = 10
    ```

  * **Passada como argumento para outra função:**

    ```java
    // Este método recebe uma lista e uma função como argumentos.
    public static List<Integer> aplicarFuncaoNaLista(List<Integer> lista, Function<Integer, Integer> funcao) {
        List<Integer> novaLista = new ArrayList<>();
        for (Integer item : lista) {
            novaLista.add(funcao.apply(item));
        }
        return novaLista;
    }

    // No código principal, passamos a função 'dobrar' como argumento.
    List<Integer> numeros = Arrays.asList(1, 2, 3);
    List<Integer> numerosDobrados = aplicarFuncaoNaLista(numeros, dobrar); // [2, 4, 6]
    ```

  * **Retornada como resultado de outra função:**

    ```java
    // Este método retorna uma função que multiplica por um valor específico.
    public static Function<Integer, Integer> criarMultiplicador(int multiplicador) {
        return numero -> numero * multiplicador;
    }

    // Criamos e armazenamos novas funções dinamicamente.
    Function<Integer, Integer> triplicar = criarMultiplicador(3);
    Function<Integer, Integer> quintuplicar = criarMultiplicador(5);

    int resultadoTriplo = triplicar.apply(10);   // resultadoTriplo = 30
    int resultadoQuintuplo = quintuplicar.apply(10); // resultadoQuintuplo = 50
    ```

Essa capacidade é o que abre as portas para as **Expressões Lambda**, que nada mais são do que uma forma de escrever essas funções "portáteis".

#### **2. Imutabilidade e a Ausência de Efeitos Colaterais (Side Effects)**

Este é talvez o maior contraste com a POO tradicional.

  * **Efeito Colateral (Side Effect):** É qualquer alteração de estado fora do escopo da função. Modificar um atributo de um objeto que foi passado como parâmetro, alterar uma variável global, escrever em um arquivo ou no console são todos exemplos de efeitos colaterais.
  * **Imutabilidade:** Significa que os dados, uma vez criados, não podem ser alterados. Em vez de modificar uma lista existente, uma função funcional cria e retorna uma *nova* lista com as mudanças desejadas.

Vamos ver a diferença na prática.

**Exemplo com Efeito Colateral (abordagem mutável):**

```java
public void dobrarValores(List<Integer> numeros) {
    // EFEITO COLATERAL: A lista original passada como parâmetro é modificada.
    for (int i = 0; i < numeros.size(); i++) {
        numeros.set(i, numeros.get(i) * 2);
    }
}

List<Integer> minhaLista = new ArrayList<>(Arrays.asList(1, 2, 3));
dobrarValores(minhaLista);
// Agora 'minhaLista' foi alterada e contém [2, 4, 6].
// Isso pode ser inesperado e causar bugs em outras partes do código que usam 'minhaLista'.
```

**Exemplo Sem Efeitos Colaterais (abordagem imutável e funcional):**

```java
public List<Integer> dobrarValores(List<Integer> numeros) {
    // SEM EFEITO COLATERAL: A lista original não é modificada.
    // Uma nova lista é criada e retornada.
    return numeros.stream()
                  .map(n -> n * 2)
                  .collect(Collectors.toList());
}

List<Integer> minhaLista = Arrays.asList(1, 2, 3);
List<Integer> listaDobrada = dobrarValores(minhaLista);

// 'minhaLista' permanece intacta: [1, 2, 3]
// 'listaDobrada' contém o novo resultado: [2, 4, 6]
```

E isso é valioso porque código sem efeitos colaterais é mais previsível. Uma função, dadas as mesmas entradas, **sempre** produzirá as mesmas saídas. Isso torna o código mais fácil de testar, depurar e, especialmente, de paralelizar, pois elimina as condições de corrida (quando múltiplas threads tentam modificar o mesmo dado ao mesmo tempo).

#### **3. Programação Declarativa vs. Imperativa**

Este pilar define o estilo de escrita do código.

  * **Estilo Imperativo (o "como"):** Você descreve passo a passo *como* o computador deve executar uma tarefa. Laços `for` e `while` são a marca registrada desse estilo. Você controla cada detalhe do fluxo.
  * **Estilo Declarativo (o "o quê"):** Você descreve *o que* você quer como resultado, e a linguagem ou API se encarrega dos detalhes da execução.

Vejamos um exemplo simples: obter os nomes dos produtos com preço acima de R$100.

**Abordagem Imperativa (foco no "COMO"):**

```java
List<String> nomesProdutosCaros = new ArrayList<>();
for (Produto produto : listaProdutos) {
    if (produto.getPreco() > 100.0) {
        nomesProdutosCaros.add(produto.getNome());
    }
}
```

Aqui, nós instruímos cada passo: crie uma lista vazia, itere sobre a lista original, verifique a condição, adicione à nova lista.

**Abordagem Declarativa (foco no "O QUÊ"):**

```java
List<String> nomesProdutosCaros = listaProdutos.stream()
    .filter(p -> p.getPreco() > 100.0)
    .map(p -> p.getNome())
    .collect(Collectors.toList());
```

Aqui, nós simplesmente declaramos o que queremos: "A partir da lista de produtos, filtre aqueles com preço maior que 100, mapeie o resultado para seus nomes e colete em uma nova lista". A **Stream API** cuida do "como" por baixo dos panos.

### **1.2 Conectando com o que Você Já Sabe: Recursão**

Se você já usou **recursão**, já teve um contato com o pensamento funcional. A recursão é uma técnica clássica da Programação Funcional para realizar repetições sem usar laços que dependem de variáveis de controle mutáveis (como o `int i = 0` do `for`).

### **Por que Estamos Aprendendo Isso?**

O Java, a partir da versão 8, abraçou fortemente os conceitos da programação funcional. O motivo é prático: esse paradigma ajuda a escrever código mais **conciso, expressivo e seguro**, especialmente ao lidar com coleções de dados e programação concorrente. No desenvolvimento de APIs com Spring Boot, você usará esses conceitos o tempo todo para manipular dados, tratar respostas assíncronas e escrever lógicas de negócio de forma mais limpa.

Agora que entendemos a *filosofia* da programação funcional, vamos analisar as *ferramentas* que o Java nos oferece para aplicá-la na prática. Começaremos pelas **Expressões Lambda**.