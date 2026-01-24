---
layout: aula
title: 6. Exercícios
parent: Aula 01 - Revisão sobre Java e POO
nav_order: 6
---

## **Exercícios ⚒️**

Os exercícios abaixo ajudarão a fixar os conceitos abordados. Elabore-os individualmente. 

#### **Bloco 1: Fundamentos e Estruturas de Controle (Parte I)**

1.  **Calculadora de Média:** Escreva um programa que utiliza a classe `Scanner` para ler 3 notas de um aluno. Calcule e exiba a média aritmética das notas. Em seguida, usando uma estrutura `if-else`, informe se o aluno foi "Aprovado" (média >= 7), "Recuperação" (média >= 5 e < 7) ou "Reprovado" (média < 5).

2.  **Tabuada com `for`:** Peça ao usuário um número inteiro. Use um laço `for` tradicional para calcular e exibir a tabuada de multiplicação desse número, do 1 ao 10. (Ex: "5 x 1 = 5", "5 x 2 = 10", ...).

3.  **Adivinhe o Número:** Gere um número aleatório entre 1 e 100. Peça ao usuário para adivinhar o número. Use um laço `while` para continuar pedindo um número até que o usuário acerte. A cada tentativa, dê uma dica se o palpite foi "muito alto" ou "muito baixo". No final, informe o número de tentativas.

4.  **Soma de Ímpares em um Array:** Crie um array de inteiros com números pré-definidos. Utilize um laço `for-each` para percorrer o array e somar todos os números que forem ímpares. Exiba o resultado final.

#### **Bloco 2: Programação Orientada a Objetos (Parte II)**

5.  **Classe `Carro`:** Crie uma classe `Carro` com os seguintes atributos privados: `marca` (String), `modelo` (String) e `ano` (int). Implemente um construtor para inicializar esses atributos e métodos públicos `getters` para cada um deles. Adicione um método `exibirInfo()` que imprime os detalhes do carro.

6.  **Classe `Circulo` com Encapsulamento:** Crie uma classe `Circulo` com um atributo privado `raio` (double). Crie um construtor e os métodos `getRaio` e `setRaio`. No `setRaio`, adicione uma validação para garantir que o raio nunca seja um valor negativo ou zero (lance uma `IllegalArgumentException` se a condição não for atendida). Crie também um método `calcularArea()` que retorna a área do círculo ($\pi \times raio^2$).

7.  **Herança de `Veiculo`:** Crie uma classe `Veiculo` com atributos `marca` e `modelo`. Em seguida, crie duas subclasses: `Carro` (que adiciona `numeroDePortas`) e `Moto` (que adiciona `cilindradas`). Sobrescreva o método `toString()` em todas as classes para exibir suas informações de forma completa.

8.  **Exceção Personalizada `SaldoInsuficienteException`:** Reutilizando a ideia da `ContaBancaria` da aula, crie sua própria exceção checada `SaldoInsuficienteException`. Modifique o método `sacar` para que, em vez de retornar `false`, ele lance essa exceção quando o saldo for insuficiente. Crie uma classe de teste para tratar essa exceção com um bloco `try-catch`.

#### **Bloco 3: API de Collections - `List` (Parte IV)**

9.  **Lista de Tarefas (`ArrayList`):** Crie um programa que gerencia uma lista de tarefas (Strings). Permita ao usuário: adicionar uma tarefa, remover uma tarefa pelo seu índice e listar todas as tarefas. Use um `ArrayList`.

10. **Ordenando Números:** Crie um `ArrayList` de `Integer`. Adicione 10 números inteiros aleatórios ou definidos por você. Utilize a classe `Collections` e seu método `sort()` para ordenar a lista em ordem crescente e, em seguida, exiba o resultado.

11. **Manipulando o Início e o Fim (`LinkedList`):** Crie uma `LinkedList` para simular uma fila de atendimento. Adicione 5 nomes de clientes no final da fila. Em seguida, "atenda" os 2 primeiros clientes (removendo-os do início da lista). Por fim, adicione 2 novos clientes "prioritários" no início da fila. Exiba a ordem final da fila.

12. **Busca por Elemento:** Crie um `ArrayList` de Strings com nomes de cidades. Peça ao usuário para digitar o nome de uma cidade. Verifique se a cidade está presente na lista usando o método `contains()`. Se estiver, informe o índice da sua primeira ocorrência usando o método `indexOf()`.

#### **Bloco 4: API de Collections - `Set` (Parte IV)**

13. **Removendo Duplicatas:** Crie um `ArrayList` de `Integer` que contenha números duplicados. Escreva um código que receba esta lista e retorne uma nova coleção sem os elementos duplicados. (Dica: a forma mais fácil é usar um `HashSet`).

14. **Unicidade de E-mails (`HashSet`):** Crie um `HashSet` para armazenar endereços de e-mail (Strings). Tente adicionar alguns e-mails, incluindo um que seja duplicado. Imprima o tamanho do `Set` para confirmar que o e-mail duplicado não foi adicionado.

15. **Ordem de Inserção (`LinkedHashSet`):** Crie um `LinkedHashSet` e adicione os nomes dos dias da semana fora de ordem (ex: "Quarta", "Segunda", "Sexta"). Itere sobre o `Set` e imprima os elementos para verificar que eles são exibidos na ordem exata em que foram inseridos.

16. **Nomes em Ordem Alfabética (`TreeSet`):** Crie um `TreeSet` de Strings e adicione 5 nomes de pessoas fora da ordem alfabética. Itere sobre o `Set` e observe que os nomes são impressos em ordem alfabética natural.

17. **Objetos Personalizados em um `TreeSet`:** Crie uma classe `Produto` com `nome` (String) e `preco` (double). Faça com que a classe `Produto` implemente a interface `Comparable` para que os produtos sejam ordenados pelo preço (do menor para o maior). Crie um `TreeSet<Produto>` e adicione alguns produtos para testar a ordenação.

#### **Bloco 5: API de Collections - `Map` (Parte IV)**

18. **Dicionário Simples (`HashMap`):** Crie um `HashMap` para funcionar como um dicionário de tradução simples (Português -> Inglês). Adicione 5 palavras e suas traduções. Peça ao usuário uma palavra em português e, se ela existir no mapa, exiba sua tradução.

19. **Contador de Frequência de Palavras:** Crie uma String contendo um parágrafo de texto. Use um `HashMap<String, Integer>` para contar a frequência de cada palavra no texto. Ao final, itere sobre o mapa e exiba cada palavra e sua contagem.

20. **Agenda de Contatos:** Use um `HashMap` para criar uma agenda onde a chave é o nome do contato (String) e o valor é o número de telefone (String). Permita ao usuário: adicionar um novo contato, buscar um telefone pelo nome e listar todos os contatos (nome e telefone).

21. **Mantendo a Ordem de Cadastro (`LinkedHashMap`):** Crie um `LinkedHashMap` para armazenar produtos e seus respectivos códigos (ex: `Integer` como chave, `String` como valor). Adicione 5 produtos. Itere sobre o mapa e mostre que a ordem de exibição é a mesma da ordem de inserção.

22. **Listagem Ordenada (`TreeMap`):** Crie um `TreeMap` para armazenar as notas de alunos em uma prova, onde a chave é o nome do aluno (String) e o valor é a nota (Double). Adicione 5 alunos fora de ordem alfabética. Ao listar os alunos e suas notas, observe que o `TreeMap` os exibe em ordem alfabética pelo nome.

23. **Verificando a Existência de Chave e Valor:** Usando o `HashMap` do exercício da agenda, escreva um código que verifique se um determinado nome (`containsKey`) e se um determinado telefone (`containsValue`) já existem na agenda.

#### **Bloco 6: API de Collections - `Queue` e `Deque` (Parte IV)**

24. **Fila de Impressão (`Queue`):** Simule uma fila de impressão. Crie uma `Queue` (usando `LinkedList` como implementação) e adicione 5 documentos (Strings com nomes como "Documento1.pdf", "Foto.png", etc.). Em seguida, processe a fila, "imprimindo" (removendo) cada documento e exibindo seu nome na ordem em que entraram.

25. **Pilha de Livros (`Deque` como Stack):** Use um `ArrayDeque` para simular uma pilha de livros. Permita ao usuário "empilhar" 3 livros (`push`). Depois, "desempilhe" um livro (`pop`) e veja qual foi removido (o último que entrou). Por fim, use `peek` para "espiar" o livro que está no topo da pilha sem removê-lo.

#### **Bloco 7: Exercícios Integrados**

26. **Catálogo de Produtos por Categoria:** Crie uma estrutura de dados para um catálogo de produtos. Use um `Map<String, List<Produto>>`, onde a chave é o nome da categoria (ex: "Eletrônicos") e o valor é uma lista de objetos da classe `Produto` pertencentes àquela categoria. Popule a estrutura com alguns dados e depois escreva um código para listar todos os produtos de uma categoria específica.

27. **Sorteio de Ganhadores Únicos:** Crie uma lista (`ArrayList`) com nomes de participantes, permitindo que alguns nomes se repitam. Escreva um método que realize um sorteio: ele deve primeiro garantir que cada participante seja considerado apenas uma vez (mesmo que seu nome apareça várias vezes) e depois sortear aleatoriamente 3 nomes únicos para serem os ganhadores.

28. **Invertendo uma Frase:** Peça ao usuário uma frase. Use um `Deque` (como uma pilha) para armazenar cada palavra da frase. Em seguida, desempilhe as palavras uma a uma para formar e exibir a frase na ordem inversa.

29. **Histórico de Navegação:** Use uma `LinkedList` para simular o histórico de um navegador. Crie métodos `visitar(String url)`, `voltar()` e `avancar()`. O método `voltar` deve navegar para a URL anterior no histórico, e o `avancar` para a próxima, gerenciando um "ponteiro" ou índice da página atual.

30. **Agrupando Alunos por Nota:** Tendo uma `List<Aluno>` (onde `Aluno` tem `nome` e `nota`), crie um `Map<String, List<Aluno>>` que agrupe os alunos por faixa de nota: "Aprovados" (nota >= 7), "Recuperação" (nota >= 5 e < 7) e "Reprovados" (nota < 5).


#### **Bloco 8: Desafios - Reflection**

Os exercícios a seguir têm como objetivo praticar o uso da API de Reflexão do Java para inspecionar e manipular objetos dinamicamente.

31. **Inspetor de Classe com Reflection**

O primeiro exemplo da aula mostra como listar os métodos de uma classe. Vamos expandir essa ideia para criar um inspetor universal.

  * **Objetivo:** Crie uma classe `AnalisadorDeClasse` com um método estático `public static void inspecionar(Object obj)`. Este método deve receber qualquer objeto Java e imprimir no console:

    1.  O nome completo da classe do objeto.
    2.  O nome de todos os seus atributos (campos), incluindo os privados.
    3.  O nome de todos os seus métodos, incluindo os privados.

  * **Dicas:**

      * Use `obj.getClass()` para obter o objeto `Class`.
      * Use `getDeclaredFields()` para obter os atributos.
      * Use `getDeclaredMethods()` para obter os métodos.

  * **Classe para Teste:**

    ```java
    class Produto {
        private int codigo;
        public String nome;
        protected double preco;

        public Produto(int codigo, String nome, double preco) {
            this.codigo = codigo;
            this.nome = nome;
            this.preco = preco;
        }

        private double calcularImposto() {
            return preco * 0.1;
        }
    }

    // No seu método main:
    // Produto p = new Produto(101, "Notebook Gamer", 8500.0);
    // AnalisadorDeClasse.inspecionar(p);
    ```

32. **Modificador de Atributos Privados**

A aula demonstra como a reflexão pode quebrar o encapsulamento para modificar atributos privados, uma técnica essencial para frameworks de injeção de dependência e ORM.

  * **Objetivo:** Crie uma classe `Configuracao` com um atributo `private String urlConexao = "localhost:5432";`. Em outra classe, crie um método `main` que, **sem usar getters ou setters**, utilize reflection para alterar o valor deste atributo privado para `"db.producao.com:5432"`. Ao final, imprima o valor para confirmar a alteração.

  * **Dicas:**

    1.  Crie uma instância de `Configuracao`.
    2.  Obtenha o `Field` (campo) correspondente a `urlConexao` usando `getDeclaredField("urlConexao")`.
    3.  Torne o campo acessível com `field.setAccessible(true)`.
    4.  Altere seu valor usando `field.set(objetoInstanciado, "novoValor")`.
    5.  Para verificar, use `field.get(objetoInstanciado)` para ler o novo valor e imprimi-lo.

33. **Framework de Testes Simulado com Anotações**

Este exercício simula como frameworks (JUnit, TestNG) usam reflection para encontrar e executar métodos de teste automaticamente.

  * **Objetivo:** Desenvolver um pequeno executor de testes que executa métodos marcados com uma anotação personalizada.

  * **Passos:**

    1.  **Crie uma anotação:**
        ```java
        import java.lang.annotation.Retention;
        import java.lang.annotation.RetentionPolicy;
        import java.lang.annotation.Target;
        import java.lang.annotation.ElementType;

        @Retention(RetentionPolicy.RUNTIME) // Essencial para que a anotação esteja disponível via reflection
        @Target(ElementType.METHOD) // A anotação só pode ser aplicada a métodos
        public @interface Teste {
        }
        ```
    2.  **Crie uma classe com métodos de "teste":**
        ```java
        public class MinhaClasseDeTeste {
            @Teste
            public void testeSoma() {
                System.out.println("Executando testeSoma: SUCESSO");
            }

            public void metodoComum() {
                System.out.println("Este não é um teste.");
            }

            @Teste
            public void testeLogin() {
                System.out.println("Executando testeLogin: SUCESSO");
            }
        }
        ```
    3.  **Crie a classe `ExecutorDeTestes`:**
          * Ela deve ter um método `public static void executarTestes(Object obj)`.
          * Dentro deste método, use reflection para obter todos os métodos da classe do objeto recebido.
          * Itere sobre os métodos e verifique, para cada um, se ele possui a anotação `@Teste` usando `method.isAnnotationPresent(Teste.class)`.
          * Se um método tiver a anotação, invoque-o dinamicamente usando `method.invoke(obj)`.

#### **Bloco 9: VcRiquinho e Lanchonete Quase Três Lanches**

34. Os exercícios VcRiquinho e Lanchonete Quase Três Lanches estão disponíveis no Moodle. Leia o enunciado e elabore as tarefas pedidas.

### Todos os exercícios deverão ser entregues no Moodle!

## Bom trabalho! ⚒️