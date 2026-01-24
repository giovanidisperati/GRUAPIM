---
layout: aula
title: 5. Modelagem UML
parent: Aula 01 - Revisão sobre Java e POO
nav_order: 5
---

## **PARTE V - Modelagem com UML**

### **5.1 O que é e por que usar a UML?**

A **UML (Unified Modeling Language)** não é apenas um "conjunto de diagramas", mas sim uma **linguagem de modelagem visual padronizada** para a engenharia de software. Pense nela como a **planta de uma casa**: antes de construir, você desenha a planta para planejar, comunicar ideias e garantir que todos os envolvidos (engenheiros, eletricistas, proprietário) entendam o projeto da mesma forma.

Na engenharia de software, a UML serve para:

  * **Visualizar:** Dar uma forma concreta às ideias abstratas do sistema.
  * **Especificar:** Descrever o sistema de forma precisa, sem ambiguidades.
  * **Documentar:** Criar um registro da arquitetura e das decisões de design.
  * **Comunicar:** Servir como uma ponte de comunicação clara entre analistas, desenvolvedores, testadores e até mesmo clientes.

Vamos focar nos dois diagramas mais fundamentais para o início de um projeto.

### **1. Diagrama de Caso de Uso (Use Case Diagram)**

Este diagrama responde à pergunta: **"O que o sistema faz do ponto de vista do usuário?"**. Ele descreve as funcionalidades principais do sistema e como os usuários (ou outros sistemas) interagem com ele.

#### **Componentes Principais:**

  * **Ator (Actor):** Representa um papel desempenhado por um usuário, outro sistema ou até mesmo o tempo, que interage com o sistema. É desenhado como um "boneco palito".
      * *Exemplo:* `Cliente`, `Administrador`, `Sistema de Pagamento Externo`.
  * **Caso de Uso (Use Case):** Representa uma funcionalidade ou um objetivo que um ator deseja alcançar com o sistema. É desenhado como uma elipse.
      * *Exemplo:* `Realizar Login`, `Cadastrar Produto`, `Gerar Relatório de Vendas`.
  * **Fronteira do Sistema (System Boundary):** Uma caixa que delimita o escopo do sistema, separando os casos de uso (dentro) dos atores (fora).

#### **Relações Comuns:**

  * **Associação:** Uma linha contínua ligando um Ator a um Caso de Uso, indicando que o ator participa daquela funcionalidade.
  * **`<<include>>` (Inclusão):** Uma seta pontilhada que indica que um caso de uso **obrigatoriamente** inclui a funcionalidade de outro. É usado para reutilizar comportamento comum.
      * *Exemplo:* Os casos de uso `Consultar Saldo` e `Realizar Transferência` **ambos incluem** o caso de uso `Validar Credenciais`.
  * **`<<extend>>` (Extensão):** Uma seta pontilhada que indica um comportamento **opcional** ou alternativo que pode estender um caso de uso base, sob certas condições.
      * *Exemplo:* O caso de uso `Realizar Empréstimo de Livro` pode ser **estendido** pelo caso `Calcular Multa por Atraso`, mas apenas se o usuário tiver livros atrasados.

### **2. Diagrama de Classes (Class Diagram)**

Este diagrama responde à pergunta: **"Qual é a estrutura estática do meu sistema?"**. Ele é o mapa dos "tijolos" de um sistema orientado a objetos: as classes, seus atributos, métodos e como elas se relacionam umas com as outras.

#### **A Caixa da Classe**

Uma classe é representada por um retângulo dividido em três partes:

1.  **Nome da Classe:** No topo.
2.  **Atributos (Campos):** No meio.
3.  **Métodos (Operações):** Na base.

**Notação de Visibilidade:**

  * `+` : `public`
  * `-` : `private`
  * `#` : `protected`

```
+---------------------------+
|         ContaBancaria     |  <-- Nome da Classe
+---------------------------+
| - titular: String         |  <-- Atributos com visibilidade
| - saldo: double           |
+---------------------------+
| + depositar(valor: double): void |  <-- Métodos com visibilidade e parâmetros
| + sacar(valor: double): boolean  |
| + getSaldo(): double      |
+---------------------------+
```

#### **Relacionamentos Fundamentais:**

  * **Associação:** Uma linha contínua que representa uma relação estrutural entre classes (um objeto "conhece" ou "usa" o outro). Pode ter **multiplicidade**, que indica quantos objetos estão envolvidos.

      * `1` : Exatamente um.
      * `*` : Zero ou mais.
      * `1..*` : Um ou mais.
      * `0..1` : Zero ou um.

    ```
    +-----------+ 1      1..* +-----------+
    | Professor |<>----------|   Turma   |
    +-----------+            +-----------+
    (Um Professor ensina em uma ou mais Turmas)
    ```

  * **Agregação:** Um tipo especial de associação (relação "tem-um") onde as classes têm um ciclo de vida independente. É representada por um **losango vazio**.

      * *Exemplo:* Um `Time` de futebol **tem** `Jogadores`. Se o time for desfeito, os jogadores continuam a existir.

    ```
    +------+ <>----* +---------+
    | Time |          | Jogador |
    +------+          +---------+
    ```

  * **Composição:** Uma forma forte de agregação ("parte-de") onde o ciclo de vida das partes depende do todo. Se o todo é destruído, as partes também são. É representada por um **losango preenchido**.

      * *Exemplo:* Uma `NotaFiscal` **é composta por** `ItensDaNota`. Se a nota fiscal for excluída, seus itens não fazem mais sentido e são excluídos também.

    ```
    +------------+ <*>----1..* +------------+
    | NotaFiscal |             | ItemDaNota |
    +------------+             +------------+
    ```

  * **Generalização (Herança):** Representa a relação "é-um" (`extends` em Java). É representada por uma seta com uma **ponta de triângulo vazia** apontando para a superclasse.

    ```
          +---------------+
          | ContaBancaria |
          +---------------+
                 ^
                 |
        ---------'---------
        |                 |  
    ---------------+   +---------------+
    | ContaCorrente |   | ContaPoupanca |
    ---------------+   +---------------+
    ```

  * **Realização (Implementação):** Representa a relação entre uma classe e uma `interface` que ela implementa. É representada por uma **linha pontilhada** com uma ponta de triângulo vazia.

    ```
        +-------------+
        |   Payable   |  (<<interface>>)
        +-------------+
               ^
               | (linha pontilhada)
        +-------------+
        |   Invoice   |
        +-------------+
    ```