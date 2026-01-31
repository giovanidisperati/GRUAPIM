---
layout: aula
title: 6. Aspectos Tranversais e Entidades
parent: Aula 07 - Construindo o To-do List
nav_order: 6
---

### 3.6. Aspectos Transversais da Aplicação

Até agora exploramos as principais camadas estruturais do projeto — **modelos**, **DTOs**, **repositórios**, **serviços** e **controllers** — seguindo uma separação de responsabilidades bem definida. No entanto, existem alguns aspectos que, embora não pertençam diretamente a uma única camada, são essenciais para o funcionamento fluido e profissional da aplicação como um todo.  

Esses aspectos são chamados de **transversais** porque impactam várias partes do sistema simultaneamente, oferecendo suporte e robustez tanto no desenvolvimento quanto na execução da API. São eles:

- **Configuração de mapeamento de objetos** (ModelMapper);
- **Tratamento global de exceções**;
- **Testes automatizados**.

A ordem de apresentação seguirá uma lógica de dependência e compreensão: primeiro veremos **como os dados são transformados** internamente, depois **como lidamos com comportamentos inesperados** no fluxo de execução, e, finalmente, **como garantimos a qualidade e a estabilidade** da aplicação com testes.

Nao teremos nenhuma novidade em relação ao que vimos, na prática, anteriormente: a apresentação é para fins didáticos e conceituais, reforçando a aprendizagem que temos tido nas últimas semanas!

### 3.6.1. Mapeamento de Entidades

A primeira etapa dos nossos aspectos transversais é entender como o projeto realiza a conversão entre objetos de diferentes camadas — **Entidades**, **DTOs de requisição** e **DTOs de resposta**.  

Para isso, utilizamos o **ModelMapper**, uma biblioteca que automatiza a tarefa de copiar dados entre objetos de maneira simples e segura.

Em APIs bem estruturadas, **não expomos entidades JPA diretamente para o cliente** — usamos DTOs para proteger, filtrar e organizar as informações que trafegam entre o servidor e o consumidor. Entretanto, converter manualmente cada objeto seria repetitivo, sujeito a erros e de difícil manutenção. O **ModelMapperConfig** centraliza essa configuração, permitindo que o mapeamento seja feito de maneira **automática**, **personalizável** e **padronizada** em todo o projeto. De novo: já vimos isso anteriormente, mas nunca é demais relembrar, né?

#### `ModelMapperConfig.java`

```java
package br.ifsp.edu.todo.config;

@Configuration
public class ModelMapperConfig {

    @Bean
    public ModelMapper modelMapper() {
        ModelMapper modelMapper = new ModelMapper();
        modelMapper.getConfiguration().setMatchingStrategy(MatchingStrategies.STRICT);
        return modelMapper;
    }
    
}
```

A classe `ModelMapperConfig` é responsável por **configurar e expor** uma instância de `ModelMapper` para toda a aplicação de forma centralizada e controlada.

Anotada com `@Configuration`, essa classe indica ao Spring que ela contém **definições de beans** — ou seja, objetos que devem ser gerenciados pelo próprio Spring no ciclo de vida da aplicação.

O método `modelMapper()` está anotado com `@Bean`, o que significa que a instância retornada será registrada no contexto do Spring Boot. Assim, sempre que uma outra classe precisar de um `ModelMapper`, ela poderá receber esse bean automaticamente, seja via injeção no construtor ou com `@Autowired`.

Dentro do método, configuramos o `ModelMapper` para usar a estratégia de correspondência `MatchingStrategies.STRICT`. Essa estratégia define que:

- Um mapeamento só será considerado válido se **os nomes dos atributos forem exatamente iguais** entre a origem (entidade) e o destino (DTO).
- Além disso, o `ModelMapper` será mais rigoroso em relação à estrutura dos objetos mapeados, evitando mapeamentos implícitos que poderiam causar problemas silenciosos.

Essa escolha aumenta a **segurança** e a **previsibilidade** dos mapeamentos, garantindo que apenas atributos compatíveis sejam copiados entre os objetos — o que é especialmente importante em aplicações onde a estrutura dos DTOs precisa ser confiável e consistente.

Essa é a diferença entre a configuração atual e as configurações anteriores que fizemos. Nós precisamos disso NESSA aplicação? Não, mas a ideia aqui é mostrar que nós podemos configurar o `ModelMapper` de acordo com as necessidades que possam surgir no projeto, não ficando "travados" em uma única abordagem. 👨‍🏭  

#### `PagedResponseMapper.java`

```java
package br.ifsp.edu.todo.mapper;

@Component
public class PagedResponseMapper {
    private final ModelMapper modelMapper;

    public PagedResponseMapper(ModelMapper modelMapper) {
        this.modelMapper = modelMapper;
    }

    public <S, T> PagedResponse<T> toPagedResponse (Page<S> sourcePage, Class<T> targetClass) {
        List<T> mappedContent = sourcePage.getContent()
                .stream()
                .map(source -> modelMapper.map(source, targetClass))
                .toList();

        return new PagedResponse<>(
                mappedContent,
                sourcePage.getNumber(),
                sourcePage.getSize(),
                sourcePage.getTotalElements(),
                sourcePage.getTotalPages(),
                sourcePage.isLast()
        );
    }
}
```

A classe `PagedResponseMapper` foi criada para resolver uma necessidade importante em APIs REST modernas: **transformar respostas paginadas do Spring Data** (`Page<S>`) em uma **estrutura padronizada e amigável** (`PagedResponse<T>`), que possa ser facilmente consumida por frontends ou outros sistemas integradores.

Podemos apenas retornar `Page<S>` para nossos clientes, como fizemos anteriormente, mas o que acontece se essa estrutura for alterada? Nossos clientes inviariavelmente irão _quebrar_. Nesse sentido, é importante definirmos o limite claro de acoplamento entre o modelo de dados que servimos e o modelo que é fornecido pelo framework. Isso nos fornece:

- **Padronização de Respostas**: O cliente da API sempre receberá um formato previsível, independentemente de mudanças internas no Spring ou no JPA.
- **Desacoplamento**: Não expomos entidades internas da aplicação diretamente ao mundo externo.
- **Controle sobre a Resposta**: Podemos, se necessário, adicionar outros metadados (mensagens, códigos de erro, etc.) ao `PagedResponse` futuramente sem quebrar contratos.

Antes de entrarmos no funcionamento do método, entretanto, vale lembrar que estamos lidando com tipos **genéricos** (`S` e `T`).

**Generics** permitem que classes, interfaces e métodos no Java sejam **parametrizados por tipos**. Em vez de fixar um tipo específico, como `String` ou `Integer`, podemos deixar o tipo como um parâmetro — `T`, `S`, `U`, etc. — tornando o código **mais flexível, reutilizável e seguro** em tempo de compilação.

No caso do nosso método:

- `S` representa o **tipo de objeto original** retornado pelo repositório (por exemplo, `Task`).
- `T` representa o **tipo de objeto que será entregue ao cliente** (por exemplo, `TaskResponseDTO`).

Assim, o método `toPagedResponse` pode ser usado para transformar qualquer tipo de entidade em qualquer tipo de DTO, bastando informar os tipos no momento da chamada. Isso evita duplicação de código e favorece a reusabilidade.

Vamos analisar o fluxo passo a passo:

```java
public <S, T> PagedResponse<T> toPagedResponse(Page<S> sourcePage, Class<T> targetClass)
```

1. **Entrada**:
   - `sourcePage`: uma instância de `Page<S>`, ou seja, uma página de entidades vindas do banco de dados.
   - `targetClass`: a classe que queremos gerar a partir dos objetos (`T`), normalmente um DTO.

2. **Abertura de Stream**:
   ```java
   sourcePage.getContent().stream()
   ```
   - `sourcePage.getContent()` retorna a lista de objetos (`List<S>`) contida na página (por exemplo, várias `Task`).
   - `.stream()` cria um fluxo de dados sobre essa lista, permitindo processamento funcional (map, filter, collect etc.).

3. **Mapeamento de cada elemento**:
   ```java
   .map(source -> modelMapper.map(source, targetClass))
   ```
   - Para cada objeto `source` no fluxo, usamos o `modelMapper` para transformá-lo de `S` para `T`.
   - Essa operação converte, por exemplo, uma entidade `Task` em um `TaskResponseDTO`, respeitando o mapeamento de campos.

4. **Coleta dos objetos mapeados**:
   ```java
   .toList();
   ```
   - Após o mapeamento, a `Stream<T>` gerada é transformada em uma lista (`List<T>`) com o método `toList()`.
   - Ou seja, temos agora uma lista de DTOs prontos para serem enviados como resposta.

5. **Construção do `PagedResponse`**:
   ```java
   return new PagedResponse<>(
       mappedContent,
       sourcePage.getNumber(),
       sourcePage.getSize(),
       sourcePage.getTotalElements(),
       sourcePage.getTotalPages(),
       sourcePage.isLast()
   );
   ```
   - Agora criamos um novo objeto `PagedResponse<T>`, passando:
     - `mappedContent`: a lista de DTOs.
     - `sourcePage.getNumber()`: número da página atual.
     - `sourcePage.getSize()`: quantidade de elementos por página.
     - `sourcePage.getTotalElements()`: total de registros na base.
     - `sourcePage.getTotalPages()`: número total de páginas.
     - `sourcePage.isLast()`: se esta é a última página.

Assim, toda a estrutura de paginação é preservada, mas o conteúdo foi adaptado para o formato de DTO.

O fluxo portanto, é algo como:

```
Page<S> (entidades do banco)
     ↓ (stream)
List<S> (conteúdo da página)
     ↓ (mapeamento com ModelMapper)
List<T> (DTOs)
     ↓
PagedResponse<T> (resposta final padronizada)
```

Viram só? Uso interessante dos Generics, né? 🤓

Passemos agora à próxima etapa de explicação das nossas classes tranversais: o tratamento de exceções!
