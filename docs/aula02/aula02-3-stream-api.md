---
layout: aula
title: 3. Stream API
parent: Aula 02 - Lambdas, Streams e Optionals
nav_order: 3
---

## **3. Stream API: Processando Coleções de Forma Declarativa**

A Stream API é talvez a adição mais impactante do Java 8. Pense nela como uma **linha de montagem para dados:** você coloca uma coleção de itens em uma ponta, e eles passam por várias estações (filtragem, transformação, ordenação) até saírem do outro lado como um resultado final. Ela introduz uma nova forma de processar coleções de dados, focando no **"o que fazer"** em vez de **"como fazer"**.

Uma **Stream** não é uma estrutura de dados, mas sim uma sequência de elementos de uma fonte (como uma `List` ou `Set`) que suporta operações agregadas.

O trabalho com streams geralmente segue um padrão de três etapas:

1.  **Fonte (Source):** Obter um stream a partir de uma coleção.
2.  **Operações Intermediárias (Intermediate Operations):** Transformar o stream (filtrar, mapear, etc.). Essas operações são *lazy* (preguiçosas), ou seja, nada acontece até que uma operação terminal seja chamada.
3.  **Operação Terminal (Terminal Operation):** Produzir um resultado ou um efeito colateral a partir do stream (coletar em uma lista, calcular uma soma, etc.).

### Contexto Prático: Onde a Stream API Brilha?

Imagine que você está trabalhando em uma **API para um e-commerce**. A equipe de marketing pediu um novo endpoint: `GET /api/produtos/destaques`.

**A Regra de Negócio:** Este endpoint deve retornar uma lista contendo apenas os **nomes** dos produtos considerados "premium" (com preço acima de R$ 500), e esses nomes devem estar em **letras maiúsculas** para serem exibidos em um banner promocional no site.

Essa lógica de negócio pertence à **camada de Serviço (`Service`)**. O `Controller` apenas receberá a requisição e chamará o método do serviço.

Vamos ver como as duas abordagens se sairiam dentro de um `ProductService`.

#### **1. Abordagem Imperativa (Com `for`)**

No seu `ProductService`, o método ficaria assim:

```java
@Service
public class ProductService {

    @Autowired
    private ProductRepository productRepository;

    public List<String> findFeaturedProductNamesImperative() {
        // 1. Busca todos os produtos do banco de dados.
        List<Product> allProducts = productRepository.findAll();

        // 2. Prepara uma lista vazia para armazenar os resultados.
        List<String> featuredNames = new ArrayList<>();

        // 3. Itera sobre cada produto para aplicar a regra de negócio (o "COMO").
        for (Product product : allProducts) {
            if (product.getPrice() > 500.0) {
                String upperCaseName = product.getName().toUpperCase();
                featuredNames.add(upperCaseName);
            }
        }

        // 4. Retorna a lista preenchida.
        return featuredNames;
    }
}
```

Este código funciona, mas ele detalha cada passo mecânico necessário para chegar ao resultado.

#### **2. Abordagem Declarativa (Com Stream API)**

Usando a Stream API, o mesmo método fica muito mais direto:

```java
@Service
public class ProductService {

    @Autowired
    private ProductRepository productRepository;

    public List<String> findFeaturedProductNamesFunctional() {
        // 1. Busca todos os produtos do banco.
        List<Product> allProducts = productRepository.findAll();

        // 2. Descreve a transformação desejada (o "O QUÊ").
        return allProducts.stream()
                .filter(product -> product.getPrice() > 500.0)      // Filtra produtos 'premium'
                .map(product -> product.getName().toUpperCase())   // Mapeia para o nome em maiúsculas
                .collect(Collectors.toList());                     // Coleta os resultados em uma nova lista
    }
}
```

Ou seja, ambos os métodos entregam o mesmo resultado. No entanto, em um serviço de uma aplicação real, a **abordagem funcional (declarativa)** é geralmente preferida porque:

* **Expressa melhor a intenção:** O código lê como uma descrição da regra de negócio ("pegue os produtos, filtre por preço, transforme em nome maiúsculo"), não como uma série de instruções de baixo nível.

* **É mais concisa e menos propensa a erros:** Menos código "boilerplate" (como criar e adicionar a uma lista manualmente) significa menos lugares para bugs se esconderem.

* **Facilita a manutenção:** Se a regra mudar (ex: "agora também ordene por nome"), adicionar uma nova operação ao stream (`.sorted()`) é trivial.

Dentre as principais operações da Stream API temos:

* **`filter(Predicate<T>)`**: Retorna um stream com elementos que correspondem a uma condição.
* **`map(Function<T, R>)`**: Transforma cada elemento de um tipo `T` para um tipo `R`.
* **`sorted()`**: Ordena os elementos (usando a ordem natural ou um `Comparator`).
* **`distinct()`**: Remove elementos duplicados.
* **`forEach(Consumer<T>)`**: Executa uma ação para cada elemento (operação terminal).
* **`collect(Collector)`**: Agrupa os resultados em uma coleção, como `List`, `Set` ou `Map` (operação terminal).
* **`findFirst()` / `findAny()`**: Retorna um `Optional` com o primeiro elemento encontrado (operação terminal).

Veremos algumas dessas ao longo do semestre (e nos exercícios!).

---
