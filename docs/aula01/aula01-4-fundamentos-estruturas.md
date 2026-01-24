---
layout: aula
title: 4. A API Collections
parent: Aula 01 - Revisão sobre Java e POO
nav_order: 4
---

## **PARTE IV - Explorando a API de Collections do Java**

Enquanto arrays são úteis para armazenar sequências de tamanho fixo, a maioria das aplicações precisa gerenciar grupos de objetos de forma dinâmica. O **Java Collections Framework (JCF)** resolve esse problema, oferecendo um conjunto robusto e eficiente de estruturas de dados prontas para uso. A escolha da `Collection` correta é uma decisão de design crucial que impacta diretamente a performance e a clareza do seu código.

As três interfaces principais que você mais usará são:
* **`List`**: Para sequências ordenadas de elementos.
* **`Set`**: Para conjuntos de elementos únicos.
* **`Map`**: Para associações de chave-valor.

Vamos analisar as implementações mais comuns de cada uma.

### **4.1. A Interface `List`: Sequências Ordenadas**
**Promessa:** Uma `List` garante que os elementos serão mantidos na ordem em que foram inseridos e permite elementos duplicados. O acesso é feito por um índice numérico, assim como nos arrays.

#### **`ArrayList`**
* **Característica Principal:** Uma lista redimensionável que se destaca no acesso rápido a elementos por sua posição.
* **Estrutura Interna:** Utiliza um array (`[]`) por baixo dos panos. Quando o array enche, um novo array maior é alocado e os elementos são copiados, um processo que pode ser custoso.
* **Performance:**
    * **Acesso por índice (`get(i)`):** Excelente, tempo constante - $O(1)$.
    * **Adicionar/Remover no final:** Rápido, tempo constante amortizado - $O(1)$.
    * **Adicionar/Remover no meio ou início:** Lento, pois exige o deslocamento de todos os elementos subsequentes - tempo linear - $O(n)$.
* **Caso de Uso Ideal:** Quando a principal operação é a leitura de dados por índice ou a iteração sobre a lista. É a `List` de propósito geral mais comum.

#### **`LinkedList`**
* **Característica Principal:** Uma lista otimizada para operações de inserção e remoção rápidas em qualquer ponto da lista.
* **Estrutura Interna:** É uma lista duplamente encadeada, onde cada elemento (nó) armazena o valor e ponteiros para o elemento anterior e o próximo.
* **Performance:**
    * **Adicionar/Remover no início ou fim:** Excelente, tempo constante - $O(1)$.
    * **Acesso por índice (`get(i)`):** Lento, pois precisa percorrer a lista desde o início ou o fim até encontrar a posição - tempo linear - $O(n)$.
    * **Inserção/Remoção no meio (se você já tem a referência):** Rápido, tempo constante - $O(1)$.
* **Caso de Uso Ideal:** Quando a aplicação realiza um grande número de inserções e remoções no início ou no meio da lista, como em uma fila de processamento ou ao construir uma estrutura de dados complexa.

#### **`Vector`**
* **Característica Principal:** Uma versão legada e sincronizada (`thread-safe`) do `ArrayList`.
* **Performance:** Similar ao `ArrayList`, mas com uma sobrecarga de performance devido à sincronização em todos os seus métodos públicos.
* **Caso de Uso Ideal:** Raramente é usado em código novo. Para programação concorrente, prefira usar um `ArrayList` com sincronização explícita ou as collections do pacote `java.util.concurrent`.

### **4.2. A Interface `Set`: Conjuntos de Elementos Únicos**
**Promessa:** Um `Set` garante que não haverá elementos duplicados. A tentativa de adicionar um elemento que já existe (verificado pelos métodos `hashCode()` e `equals()`) é simplesmente ignorada.

#### **`HashSet`**
* **Característica Principal:** Armazenar elementos únicos com a máxima velocidade, sem se preocupar com a ordem.
* **Estrutura Interna:** Utiliza uma tabela de dispersão (`HashMap` por baixo dos panos). A posição de cada elemento é determinada pelo seu `hashCode()`.
* **Performance:** Excelente para as operações principais (`add`, `remove`, `contains`), que geralmente são executadas em tempo constante - $O(1)$. A performance de iteração não é previsível.
* **Caso de Uso Ideal:** Verificar rapidamente se um item existe em um grande conjunto de dados, ou simplesmente para garantir a unicidade dos elementos.

#### **`LinkedHashSet`**
* **Característica Principal:** Um `HashSet` que lembra a ordem em que os elementos foram inseridos.
* **Estrutura Interna:** Combina uma tabela de dispersão com uma lista duplamente encadeada.
* **Performance:** Quase tão rápida quanto o `HashSet` (operações em $O(1)$), mas com uma pequena sobrecarga para manter a ordem da lista.
* **Caso de Uso Ideal:** Quando você precisa da velocidade e unicidade de um `HashSet`, mas também da capacidade de iterar sobre os elementos na ordem original de inserção.

#### **`TreeSet`**
* **Característica Principal:** Armazena elementos únicos e os mantém perpetuamente em ordem crescente.
* **Estrutura Interna:** Utiliza uma árvore Rubro-Negra (`Red-Black Tree`).
* **Performance:** Boa, mas mais lenta que `HashSet`. As operações (`add`, `remove`, `contains`) são executadas em tempo de logaritmo - $O(\log n)$.
* **Ordenação:** Os elementos devem implementar a interface `Comparable` (ordem natural) ou um `Comparator` deve ser fornecido no construtor do `TreeSet`.
* **Caso de Uso Ideal:** Quando você precisa manter uma coleção de itens únicos sempre ordenada, como um ranking de pontuações ou uma lista de nomes em ordem alfabética.

### **4.3. A Interface `Map`: Associações Chave-Valor**
**Promessa:** Um `Map` armazena pares de chave-valor. Cada chave é única e mapeia para um único valor. É a estrutura de dados ideal para buscas rápidas baseadas em um identificador único.

*(As implementações `HashMap`, `LinkedHashMap` e `TreeMap` seguem exatamente a mesma lógica de suas contrapartes `Set` em termos de estrutura interna, performance e ordenação, mas aplicadas às **chaves** do mapa.)*

* **`HashMap`**: Máxima velocidade ($O(1)$) para `put`, `get`, `remove`. A ordem não é garantida. Caso de uso mais comum para mapas.
* **`LinkedHashMap`**: Velocidade de `HashMap` com ordem de iteração previsível (ordem de inserção). Ideal para caches ou dados que precisam ser processados na ordem em que chegaram.
* **`TreeMap`**: Chaves mantidas em ordem crescente ($O(\log n)$). Ideal para dicionários ou dados que precisam ser recuperados em um intervalo ordenado.

### **4.4 Interfaces `Queue` e `Deque`: Filas e Pilhas**

#### **`Queue` (Fila)**
* **Característica Principal:** Representa uma estrutura **FIFO** (First-In, First-Out), onde o primeiro elemento a entrar é o primeiro a sair, como uma fila de banco.
* **Implementações Comuns:** `LinkedList` e `PriorityQueue` (uma fila especial que ordena os elementos com base em sua prioridade).
* **Caso de Uso Ideal:** Gerenciar tarefas em uma fila de processamento, algoritmos de busca em largura (BFS), ou qualquer cenário que exija processamento ordenado por chegada.

#### **`Deque` (Fila de Duas Pontas)**
* **Característica Principal:** Uma "double-ended queue" que permite adicionar e remover elementos tanto do início quanto do fim.
* **Uso como Pilha (Stack):** Um `Deque` é a estrutura recomendada atualmente para implementar uma pilha **LIFO** (Last-In, First-Out).
    * Use `push(e)` para adicionar ao início.
    * Use `pop()` para remover do início.
* **Implementação Recomendada:** `ArrayDeque`. É mais eficiente e moderno que a antiga classe `Stack`.
* **Caso de Uso Ideal:** Implementar a funcionalidade de "desfazer" (undo), analisar expressões matemáticas (parsing) ou em algoritmos de busca em profundidade (DFS).

### **4.5. Tabela Resumo: Quando Usar Cada Collection**

| Estrutura | **Preciso de Duplicatas?** | **Preciso de Ordem?** | **Qual é meu foco?** |
| :--- | :--- | :--- | :--- |
| **`ArrayList`** | Sim | Sim, de inserção | Acesso rápido por índice e iteração simples. |
| **`LinkedList`** | Sim | Sim, de inserção | Muitas inserções/remoções no início/fim da lista. |
| **`HashSet`** | **Não** | Não | Máxima velocidade para verificar se um item existe. |
| **`LinkedHashSet`** | **Não** | Sim, de inserção | Velocidade de `HashSet` com ordem de iteração previsível. |
| **`TreeSet`** | **Não** | **Sim, ordenada** | Manter os itens sempre ordenados. |
| **`HashMap`** | **Não** (chaves) | Não | Acesso ultra-rápido a um valor através de uma chave. |
| **`LinkedHashMap`** | **Não** (chaves) | Sim, de inserção | Acesso rápido de `HashMap` com ordem de iteração previsível. |
| **`TreeMap`** | **Não** (chaves) | **Sim, ordenada** | Manter as chaves sempre ordenadas. |
| **`ArrayDeque`** | Sim | Sim | Implementar uma Fila (FIFO) ou uma Pilha (LIFO) de forma eficiente. |

### **4.6. Análise de Performance: Complexidade de Tempo e Espaço**

Para tomar decisões de design informadas, é importante entender a **complexidade computacional** das operações em cada `Collection`. Usamos a notação **Big O** para descrever como a performance de um algoritmo escala conforme o número de elementos (`n`) na coleção aumenta.

* **Complexidade de Tempo:** Mede o tempo de execução.
* **Complexidade de Espaço:** Mede a memória adicional necessária.

**Guia Rápido da Notação Big O:**
* **$O(1)$ (Tempo Constante):** Excelente. A operação leva o mesmo tempo, não importa o tamanho da coleção. É o "santo graal" da performance.
* **$O(\log n)$ (Tempo Logarítmico):** Ótimo. O tempo de execução cresce muito lentamente. Dobrar o número de elementos não dobra o tempo.
* **$O(n)$ (Tempo Linear):** Razoável. O tempo de execução cresce em proporção direta ao número de elementos. Percorrer uma lista inteira é um exemplo clássico.
* **$O(n^2)$ (Tempo Quadrático):** Ruim. O tempo de execução cresce exponencialmente. Deve ser evitado para grandes coleções (ex: laços aninhados que percorrem a mesma coleção).

#### **4.7. Tabela Comparativa de Complexidade de Tempo (Big O)**

A tabela a seguir apresenta a complexidade de tempo para as operações mais comuns nas principais implementações.

| Estrutura         | **add / put** | **remove** | **get / contains** | **Iteração (`next`)** |
| :---------------- | :-----------: | :--------: | :----------------: | :-------------------: |
| `ArrayList`       | $O(1)$ (Amortizado) [^1] | $O(n)$ [^2] | $O(1)$ | $O(1)$ |
| `LinkedList`      | $O(1)$ [^3] | $O(1)$ [^3] | $O(n)$ | $O(1)$ |
| `HashSet`         | $O(1)$ (Médio) [^4] | $O(1)$ (Médio) [^4] | $O(1)$ (Médio) [^4] | $O(1)$ (Médio) [^4] |
| `LinkedHashSet`   | $O(1)$ (Médio) [^4] | $O(1)$ (Médio) [^4] | $O(1)$ (Médio) [^4] | $O(1)$ |
| `TreeSet`         | $O(\log n)$ | $O(\log n)$ | $O(\log n)$ | $O(\log n)$ |
| `HashMap`         | $O(1)$ (Médio) [^4] | $O(1)$ (Médio) [^4] | $O(1)$ (Médio) [^4] | $O(1)$ (Médio) [^4] |
| `LinkedHashMap`   | $O(1)$ (Médio) [^4] | $O(1)$ (Médio) [^4] | $O(1)$ (Médio) [^4] | $O(1)$ |
| `TreeMap`         | $O(\log n)$ | $O(\log n)$ | $O(\log n)$ | $O(\log n)$ |
| `ArrayDeque`      | $O(1)$ (Amortizado) [^1] | $O(1)$ (Amortizado) [^1] | Não aplicável | $O(1)$ |

#### **Legenda e Observações Importantes:**

[^1]: **$O(1)$ Amortizado:** A operação é geralmente muito rápida ($O(1)$), mas ocasionalmente pode ser lenta ($O(n)$). No `ArrayList` e `ArrayDeque`, isso acontece quando a capacidade interna do array se esgota e é preciso alocar um novo array maior e copiar todos os elementos. Na média, ao longo de muitas operações, o custo se "amortiza" para $O(1)$.

[^2]: **$O(n)$ em `ArrayList.remove`:** Este custo se refere a remover um elemento pelo índice no meio da lista. Remover o **último** elemento é $O(1)$.

[^3]: **$O(1)$ em `LinkedList.add/remove`:** Este custo se refere a adicionar ou remover elementos no **início** ou no **fim** da lista. Inserir ou remover no meio é $O(n)$, pois primeiro é preciso navegar até a posição desejada.

[^4]: **$O(1)$ Médio em Estruturas Hash:** `HashSet` e `HashMap` (e suas variantes `Linked`) oferecem performance de tempo constante no cenário médio. No entanto, no **pior caso**, que ocorre quando há muitas colisões de hash (objetos diferentes gerando o mesmo `hashCode`), a performance pode degradar para $O(n)$. Isso é raro se os métodos `hashCode()` e `equals()` forem bem implementados.

#### **Complexidade de Espaço**

Para todas as coleções mencionadas, a **complexidade de espaço é $O(n)$**. Isso significa que a memória utilizada cresce linearmente com o número de elementos (`n`) armazenados na coleção.

* **Observação para `ArrayList`:** Um `ArrayList` tem uma `capacidade` interna que pode ser maior que seu `tamanho` real. Ele cresce em saltos para otimizar o custo de adicionar novos elementos. Isso significa que, por um tempo, ele pode ocupar um pouco mais de memória do que o estritamente necessário para os elementos que contém.
