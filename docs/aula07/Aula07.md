---
layout: aula
title: Aula 07 - Construindo o To-do List
nav_order: 9
has_children: true
permalink: /aula07
---

# Aula 07 – Construção da API de Gerenciamento de Tarefas (To-Do List)

Na Aula 06, evoluímos a nossa API aplicando boas práticas como o uso de `ResponseEntity`, a organização da documentação com Swagger/OpenAPI, a injeção de dependência via construtor e a introdução dos primeiros testes automatizados. Agora, nesta aula, desenvolveremos a nossa **API de Gerenciamento de Tarefas** (To-Do List) seguindo os princípios RESTful e aplicando todos os conceitos que vimos até aqui. Vamos, a partir dessa aula, dar continuidade nos conceitos vistos anteriormente por meio da resolução do exercício do To-Do List.

Daremos ênfase na correta separação de responsabilidades em camadas, na implementação de boas práticas de validação, na construção de DTOs e no tratamento adequado de exceções.

A proposta é desenvolver uma API REST que permita que usuários criem, consultem, atualizem e excluam tarefas pessoais. Cada tarefa possuirá atributos como título, descrição, prioridade, categoria, data limite, entre outros.

As funcionalidades obrigatórias incluem:
- CRUD completo para tarefas.
- Pesquisa de tarefas por categoria.
- Marcação de tarefas como concluídas.
- Paginação e ordenação dos resultados.
- Validação de campos obrigatórios e regras de negócio.
- Tratamento global de exceções.
- Testes unitários e funcionais dos endpoints.

Neste exercício, poderíamos ter seguido a mesma abordagem adotada na Aula 06, onde organizamos nosso código apenas com Controller → Repository → Model → DTO. No entanto, para tornar nossa arquitetura ainda mais robusta e alinhada às boas práticas, incluiremos também uma camada de serviço (@Service). Essa nova camada será responsável por isolar a lógica de negócio, promovendo melhor organização, reutilização de código e maior facilidade na manutenção e nos testes.

## 1. A Importância da Camada Service em Arquiteturas em Camadas

### Propósito e Objetivos da Camada Service 
A **camada de serviço** (Service Layer) tem como propósito principal definir os limites da aplicação e concentrar a lógica de negócios em operações coesas, expondo essas operações de forma organizada para as camadas clientes (como a interface de usuário ou APIs externas), tal como descrito por Martin Fowler ao dissertar sobre a ([Service Layer](https://martinfowler.com/eaaCatalog/serviceLayer.html#:~:text=A%20Service%20Layer%20defines%20an,the%20implementation%20of%20its%20operations)). Em uma arquitetura em camadas clássica, a camada Service atua como intermediária entre os controladores (interface com o usuário ou APIs) e a camada de acesso a dados (repositórios ou DAOs). Martin Fowler define Service Layer como a camada que **“estabelece um conjunto de operações disponíveis e coordena a resposta da aplicação em cada operação”**, encapsulando a lógica de negócio e controlando transações. Essa definição ressalta dois objetivos centrais: (1) prover **uma interface uniforme** das funcionalidades de negócio da aplicação, evitando que cada ponto de entrada (por exemplo, diferentes *endpoints* REST ou componentes de UI) tenha de implementar logicamente essas funcionalidades de forma redundante, e (2) **orquestrar as interações** necessárias para cumprir cada caso de uso, possivelmente envolvendo várias operações de dados e regras de negócio em sequência.

Sem uma camada de serviços, diversos componentes da aplicação poderiam precisar repetir operações semelhantes. Fowler observa, por exemplo, que interfaces distintas (interfaces gráficas, carregadores de dados, gateways de integração, etc.) frequentemente precisam realizar interações complexas para manipular os dados e invocar a lógica de negócio; se cada interface implementa por conta própria essas interações, isso leva a **duplicação de código e lógica** através do sistema ([Service Layer](https://martinfowler.com/eaaCatalog/serviceLayer.html#:~:text=integration%20gateways%2C%20and%20others,causes%20a%20lot%20of%20duplication)). A camada Service soluciona esse problema centralizando tais interações comuns em um único local. Assim, **reduz-se a duplicação** e assegura-se que regras de negócio importantes sejam implementadas de forma consistente.

Por exemplo, imagine uma aplicação de uma **loja online**, que possui:

- Uma **interface web** para os clientes realizarem pedidos;
- Um **aplicativo móvel** que também permite aos clientes fazerem compras;
- Um **sistema de integração com marketplaces externos** (por exemplo, para vender os produtos também via Amazon ou Mercado Livre).

Todos esses canais (Web, Mobile, Integração com marketplaces) precisam realizar uma operação comum: **finalizar um pedido de compra**.

Agora, sem uma camada Service centralizada, cada canal implementaria sua própria versão da lógica para "finalizar pedido". O que poderia envolver:

1. Validar o estoque dos produtos;
2. Calcular descontos promocionais aplicáveis;
3. Atualizar o status do pedido para "Em processamento";
4. Reduzir a quantidade dos produtos em estoque;
5. Gerar uma fatura de pagamento;
6. Enviar um e-mail de confirmação para o cliente.

Assim:

- O **Controller Web** teria que implementar todos esses passos.
- O **Controller Mobile** teria que reimplementar todos esses mesmos passos.
- O **Gateway de integração** teria que repetir essa sequência para pedidos vindos de marketplaces.

Resultado? **Código duplicado em vários lugares** da aplicação — e com um alto risco de inconsistências: se, por exemplo, um novo requisito surgir (como aplicar um novo tipo de desconto), o desenvolvedor teria que lembrar de atualizar a lógica **em todas as interfaces**. Se esquecesse de um deles, bugs e inconsistências surgiriam.

**Com uma camada Service**, o cenário seria muito mais organizado:

- Criamos uma classe `OrderService` contendo o método `finalizarPedido(Pedido pedido)`.
- Toda a lógica complexa (validar estoque, calcular descontos, alterar status, debitar estoque, gerar fatura, enviar e-mail) fica **centralizada** no `OrderService`.
- A interface Web apenas chama `orderService.finalizarPedido(pedido)`.
- A interface Mobile também chama `orderService.finalizarPedido(pedido)`.
- O Gateway de integração igualmente chama `orderService.finalizarPedido(pedido)`.

Assim, a regra de negócio para finalizar pedidos **existe em apenas um lugar**. Qualquer modificação futura será feita **uma única vez**, de forma consistente para todos os canais.

Resumindo o exemplo:
| Sem Service Layer                  | Com Service Layer                     |
|-------------------------------------|---------------------------------------|
| Código duplicado em várias interfaces | Código único, centralizado no serviço |
| Alto risco de inconsistência        | Regra de negócio consistente          |
| Alta manutenção manual              | Manutenção facilitada                 |
| Testar cada canal individualmente   | Testar o serviço uma única vez         |

Ou seja: como descreve Fowler, a criação de uma camada de serviço é uma boa ideia em muitos casos! 😊

Já Eric Evans, em *Domain-Driven Design (2004)*, também conceitua uma camada de aplicação (análoga à camada de serviço) com objetivos semelhantes. Segundo Evans, essa camada aplica casos de uso do sistema coordenando as operações de domínio, porém **“mantida fina, sem conter regras de negócio próprias, apenas delegando tarefas às entidades de domínio”** ([Anemic Domain Model](https://martinfowler.com/bliki/AnemicDomainModel.html#:~:text=,for%20the%20user%20or%20the)). Ou seja, sua responsabilidade está em saber *quando* e *em que ordem* acionar as operações de negócio, mas o *como* (as regras em si) permanece nas camadas de domínio mais baixas. Essa perspectiva enfatiza que a Service Layer **não deve reinventar a lógica de negócio**, mas sim servir como um **orquestrador**, chamando métodos das entidades de domínio ou repositórios conforme necessário para atender a uma solicitação do sistema.

Em resumo, os objetivos primários da camada Service numa arquitetura em camadas são: **centralizar e expor a lógica de negócio de forma consistente**, **estabelecer uma separação clara de responsabilidades** entre a lógica de aplicação e os detalhes de interface ou persistência, e **coordenar transações e fluxos** complexos envolvendo múltiplos componentes. Essa organização resulta em uma fronteira bem definida da aplicação, na qual a camada de serviço atua como *fachada* das operações de negócio disponíveis, aumentando a clareza arquitetural sobre “o que” a aplicação faz (casos de uso) independentemente de “como” a interface ou a base de dados funcionam ([Service Layer](https://martinfowler.com/eaaCatalog/serviceLayer.html#:~:text=A%20Service%20Layer%20defines%20an,the%20implementation%20of%20its%20operations)).

Essa concepção da camada Service, proposta por Eric Evans, se conecta de maneira profunda aos princípios da **Clean Architecture**, descrita por Robert C. Martin (Uncle Bob) em *Clean Architecture: A Craftsman's Guide to Software Structure and Design* (2017). Em Clean Architecture, a lógica de negócio (casos de uso) deve ser **independente** das camadas externas (como banco de dados, frameworks ou interfaces gráficas), sendo organizada em torno de *use cases* (regras de aplicação) e *entities* (regras de negócio de domínio).

Dentro desse contexto, a camada Service que estamos implementando hoje cumpre o papel de **orquestrar os casos de uso**, isolando-os das preocupações de infraestrutura. Ela se torna a linha divisória entre o mundo interno da aplicação (domínio) e o mundo externo (interface, banco de dados, APIs).

Quando evoluirmos nossa arquitetura para **microsserviços**, aplicaremos ainda mais rigor a esse modelo: cada microsserviço será orientado a um **bounded context** do DDD (Domain-Driven Design), no qual a camada Service será responsável por coordenar as operações de um domínio bem definido, preservando a autonomia, a coesão e a consistência internas do serviço. Ou seja, no cenário de microsserviços, a camada de serviço continuará sendo o ponto de articulação central, mas agora focada em atuar dentro dos limites de um contexto de negócio específico, promovendo **baixo acoplamento** e **alta coesão** entre os serviços! 🤠

Portanto, ao adotarmos a camada Service desde já, estamos pavimentando o caminho para aplicar os mesmos princípios de arquitetura limpa, escalável e robusta em projetos futuros mais complexos — sejam eles monolíticos, como agora, ou baseados em microsserviços, como veremos posteriormente.

### Manutenibilidade e testabilidade ao adotar uma camada de Service

A adoção de uma camada Service também traz benefícios significativos à **manutenibilidade** do sistema. Ao separar a lógica de negócio da camada de apresentação e da de acesso a dados, promove-se o princípio da **separação de interesses** (*Separation of Concerns*), tornando cada parte do sistema mais isolada e com responsabilidades bem definidas. Isso significa que alterações em regras de negócio tendem a se concentrar nos serviços, sem exigir modificações na interface (desde que a interface dos serviços se mantenha estável) ou nos repositórios (desde que as operações de dados básicas não mudem). Essa modularização facilita a evolução do sistema: é possível **alterar ou estender funcionalidades** na camada Service sem quebrar diretamente outras partes, contanto que os contratos entre as camadas (por exemplo, os métodos dos serviços) sejam respeitados. Autores clássicos enfatizam que tal desacoplamento aumenta a *flexibilidade* para modificar componentes individuais da aplicação sem efeitos colaterais indesejados. Em particular, a camada de serviço permite trocar implementações de repositório ou detalhes de infraestrutura sem alterar a lógica de negócio, ou adaptar a interface de entrada (por exemplo, migrar de uma interface REST para outra tecnologia) sem reescrever as regras de negócio, desde que esta continue consumindo os mesmos serviços.

A manutenibilidade também se manifesta na facilidade de **localizar e corrigir defeitos** ou ajustar comportamentos: sabendo-se que a lógica de negócio reside nos serviços, um desenvolvedor pode concentrar sua busca por bugs de regra de negócio nessa camada, sem precisar vasculhar código de UI ou de persistência. Além disso, vários desenvolvedores podem trabalhar em paralelo em um mesmo projeto focando em camadas distintas (por exemplo, um engenheiro de *frontend* consumindo a API e um engenheiro de backend implementando a lógica de negócio nos serviços), graças ao contrato claro que a Service Layer fornece entre o front-end e o domínio. Essa divisão de trabalho melhora a **coesão** de cada parte e diminui o risco de conflitos, contribuindo para a produtividade e qualidade do código.

Outro ponto importante é que a camada Service aumenta a **coerência das regras de negócio**. Como todas as funcionalidades críticas passam por ela, garante-se que as mesmas validações e operações sejam aplicadas independentemente de qual interface acionou a funcionalidade. Por exemplo, se tanto uma interface web quanto uma tarefa em lote precisam aplicar a mesma regra, implementá-la no serviço (ao invés de duplicá-la em cada chamador) assegura consistência. Essa centralização reduz erros e facilita futuras mudanças (basta alterar a regra no serviço para que todas as interfaces passem a usar a nova lógica). Em suma, do ponto de vista de Engenharia de Software, a camada de serviço **melhora a manutenibilidade ao promover baixo acoplamento entre apresentação/infraestrutura e negócio, e alta coesão da lógica de negócio**, o que está alinhado com princípios fundamentais como o *Single Responsibility Principle* de Robert Martin (cada camada tendo responsabilidade única) e o *Open/Closed Principle* (podemos estender a lógica adicionando novos serviços ou métodos sem modificar controladores já estáveis, por exemplo).

A discussão a seguir apresenta alguns pontos interessantes nesse sentido: ([design pattern - Por que separar camadas? Quais os benefícios de uma arquitetura multicamada? - Stack Overflow em Português](https://pt.stackoverflow.com/questions/22403/por-que-separar-camadas-quais-os-benef%C3%ADcios-de-uma-arquitetura-multicamada#:~:text=,parte%20sem%20afetar%20as%20demais)).

Já do ponto de vista da **testabilidade**, a camada Service também se mostra vantajosa. Como os serviços concentram a lógica de negócios, pode-se escrever **testes unitários** direcionados exclusivamente a essa camada, instanciando os serviços em memória e simulando (via *mocks* ou *stubs*) as dependências externas, como repositórios ou APIs externas. Isso permite verificar as regras e fluxos de negócio de forma isolada, rápida e determinística, sem necessidade de carregar toda a infraestrutura da aplicação. Por exemplo, é possível simular diferentes respostas dos repositórios (dados existentes, inexistentes, exceções) e verificar se o serviço toma as ações adequadas (retorna erros significativos, lança exceções de negócio, realiza cálculos corretamente, etc.). Essa isolação de testes decorre diretamente do design em camadas: **cada camada pode ser testada independentemente**. De fato, uma arquitetura bem estratificada torna o sistema “mais fácil de testar”, pois podemos testar os controladores separadamente (simulando requisições e verificando se delegam corretamente aos serviços) e testar os repositórios separadamente (interagindo com uma base de dados de teste), enquanto a camada Service é testada em separado verificando a lógica de negócio.

### Vantagens Práticas em Projetos Spring Boot (e Similares!) 
Em frameworks como o **Spring Boot**, o uso de uma camada Service é uma prática consagrada na estruturação de aplicações. O próprio *framework* fornece estereótipos (@Service) que indicam classes de serviço, integrando-as ao mecanismo de injeção de dependências e possibilitando gerenciamento transacional automático. Na perspectiva prática, a camada Service em um projeto Spring Boot traz todos os benefícios gerais já mencionados e agrega alguns pontos específicos do ecossistema Spring:

- **Demarcação de transações:** É comum definir métodos transacionais na camada de serviço usando anotações como `@Transactional`. Isto estabelece um escopo de transação abrangendo toda a lógica de negócio daquela operação, mesmo que envolva múltiplos acessos a repositórios diferentes. Conforme as recomendações, *“o principal objetivo da camada de serviço é definir os limites transacionais de uma dada unidade de trabalho”*. Por exemplo, em uma operação de venda que debita o estoque e credita o saldo do cliente, ambos os acessos ao banco (estoque e clientes) devem ocorrer numa única transação. Colocar essa lógica na camada Service, anotada como transacional, assegura que ou **tudo ocorra com sucesso, ou em caso de falha tudo seja revertido** (rollback), mantendo a consistência dos dados. Sem a camada Service, se um controlador chamasse dois repositórios separadamente, poderia ser mais difícil coordenar a transação – cada chamada poderia abrir sua própria transação isolada, resultando em inconsistências caso uma parte falhasse no meio do fluxo. O artigo a seguir apresenta em mais detalhes essas características: [Spring Transaction Best Practices - Vlad Mihalcea](https://vladmihalcea.com/spring-transaction-best-practices/#:~:text=,the%20entire%20unit%20of%20work).

- **Orquestração de múltiplos componentes:** Em aplicações reais, um caso de uso raramente consiste em apenas uma operação CRUD simples. Muitas vezes a camada Service combina **chamadas a vários repositórios e/ou serviços externos**, aplica regras condicionais, faz conversões de formatos (por exemplo, de DTOs para entidades), e decide como tratar erros. No Spring, a Service Layer serve como o lugar apropriado para implementar essa *orquestração*. Isso mantém os controladores REST **enxutos**, limitados a receber a requisição, acionar o serviço adequado e retornar a resposta (transformando objetos de domínio em DTOs de resposta se necessário), seguindo o princípio de *controllers thin, services thick*. A consequência é um código mais **legível e organizado**, onde a lógica de negócio não fica misturada com lógica de protocolo HTTP ou detalhes de JSON, por exemplo. Aqui temos um bom aprofundamento dessa aplicação: [Skinny Models, Skinny Controllers, Fat Services - Ryan Rebo](https://medium.com/marmolabs/skinny-models-skinny-controllers-fat-services-e04cfe2d6ae).

- **Facilidade de injeção de dependências e mockagem:** Ao seguir a convenção de criar interfaces de repositório (@Repository) e classes de serviço (@Service), o Spring Boot permite injetar facilmente essas dependências. Os serviços podem depender de interfaces de repositório, e controladores dependem de interfaces de serviço. Essa inversão de dependências facilita a substituição de implementações (por exemplo, usar um repositório fake ou uma implementação alternativa de serviço em testes de integração). Em tempo de execução, o container do Spring resolve as dependências reais, mas em testes podemos fornecer *doubles* (mocks) para verificar isoladamente o comportamento do serviço. Assim, a arquitetura em camadas casa bem com o **modelo de injeção de dependência do Spring**, aumentando a testabilidade e flexibilidade.

- **Aplicação de aspectos transversais (*cross-cutting*)**: A camada Service é também um ponto ideal para aplicar funcionalidades transversais como logging, segurança ou cache. No Spring, pode-se, por exemplo, colocar anotações de segurança (@PreAuthorize) ou de cache (@Cacheable) sobre métodos de serviço. Dessa forma, garante-se que tais preocupações sejam manejadas de forma consistente em torno da lógica de negócio, sem poluir o código do controlador ou do repositório com essas responsabilidades. Esse modelo está alinhado com a ideia de manter cada camada focada em seu propósito primário (controle de fluxo no controller, regras de negócio no serviço, acesso a dados no repositório).

Em projetos **Spring Boot** (ou em muitos frameworks similares!), a camada Service se mostra vantajosa inclusive em contextos de APIs REST, que geralmente correspondem a casos de uso relativamente pequenos. Mesmo em **microserviços**, onde o serviço em si (microserviço) já é uma unidade de implantação focada em uma funcionalidade, costuma-se manter uma separação interna em camadas para preservar a organização: o *controller* trata da interface HTTP, enquanto a lógica de negócio do microserviço fica nos serviços. Isso evita duplicação de lógica caso o microserviço exponha múltiplos *endpoints* relacionados (todos podem reutilizar métodos da camada de serviço comum). Em suma, frameworks modernos dão suporte nativo a esse padrão de camadas porque ele provou fornecer um equilíbrio saudável entre **simplicidade na interação** (cada camada trata de um aspecto) e **robustez na implementação** (transações e regras consistentes).

### Possíveis Críticas e Visões Contrárias 
Apesar das claras vantagens, a introdução de uma camada Service não é isenta de críticas. Alguns especialistas e desenvolvedores argumentam que, em certos contextos, essa camada pode representar uma complexidade desnecessária. A principal objeção surge em **projetos de menor porte ou baixa complexidade**, onde adicionar classes e interfaces de serviço que meramente repassam chamadas pode ser visto como *overengineering*. De fato, é aconselhável analisar caso a caso se os benefícios compensam o custo adicional em complexidade. Conforme discutido pela comunidade, *“quanto menos camadas, melhor”* em termos de simplicidade, pois cada nova camada adiciona indireção e potencialmente dificulta o rastreamento do fluxo de execução. Em sistemas muito simples – por exemplo, uma aplicação interna mantida por um único desenvolvedor, com escopo bem delimitado e poucos casos de uso – separar controladores, serviços e repositórios rigorosamente pode ser excessivo. Nesses cenários, um design mais enxuto (talvez controladores acessando repositórios diretamente) pode atender aos requisitos sem problemas de manutenibilidade, já que a escala e o escopo são reduzidos. Tentar aplicar uma arquitetura pesada “só porque um livro disse que deve fazer isso” é questionável quando **não há uma motivação clara no problema em questão**. Em outras palavras, violaria o princípio YAGNI (“You Aren’t Gonna Need It”), acrescentando trabalho e código para lidar com complexidades que talvez nunca se manifestem naquele sistema específico.

Outra crítica técnica relevante diz respeito ao risco de se cair no chamado **Anemic Domain Model** (Modelo de Domínio Anêmico). Esse termo, cunhado por Martin Fowler, descreve uma situação em que as entidades de domínio ficam “anêmicas”, ou seja, sem comportamento algum, atuando apenas como estruturas de dados, enquanto toda a lógica de negócio reside em serviços e procedimentos. Fowler e Evans apontaram este estilo como um anti-padrão que fere os princípios de Orientação a Objetos, por essencialmente separar dados e comportamento – algo que a orientação a objetos procura unir. Em um domínio anêmico, frequentemente existem **“um conjunto de objetos de serviço que capturam toda a lógica de domínio, realizando todos os cálculos e atualizando os objetos do modelo com os resultados”**, ao passo que os objetos de domínio viram recipientes passivos de dados (com campos e *getters/setters*). O problema, segundo Fowler, é que isso **transforma o design em procedural** disfarçado, perdendo-se os benefícios de um verdadeiro modelo de objetos rico em comportamento.

É importante notar, entretanto, que *criticar o modelo anêmico não significa descartar a camada de serviço*. Os próprios defensores de Domain-Driven Design recomendam uma camada de serviços **em conjunto com um domínio rico**, e não em substituição a este!

Por fim, em arquiteturas altamente desacopladas ou alternativas (como a **arquitetura hexagonal** ou a **Clean Architecture** de Robert C. Martin), a camada Service pode aparecer com outro nome ou forma. Na Clean Architecture, por exemplo, temos a camada de **casos de uso** (ou **Interactor**), que cumpre papel similar ao serviço de aplicação – orquestrando entidades de domínio e gerenciando as regras de aplicação. O link a seguir mostra uma aplicação interessante nesse sentido: [Building Your First Use Case With Clean Architecture](https://www.milanjovanovic.tech/blog/building-your-first-use-case-with-clean-architecture#:~:text=The%20Application%20layer%20contains%20application,entities%20to%20achieve%20its%20goals). Nessas abordagens, todos os acessos externos (interfaces, bancos, dispositivos) dependem da camada de casos de uso, mantendo o núcleo de negócio isolado. A diferença é mais conceitual do que prática: a ideia de separar *o que o software faz* (regras de negócio, casos de uso) do *como ele interage com o mundo externo* é mantida. Assim, mesmo em projetos “desacoplados” segundo padrões modernos, existe alguma forma de Service Layer, ainda que os autores enfatizem a inversão total de dependências (interfaces definidas no núcleo e implementações fora). As críticas, portanto, geralmente não defendem eliminar qualquer separação, mas sim **evitar separações desnecessárias** ou mal definidas.

Por exemplo, se fossemos estruturar nosso To-Do List por meio de uma estrutura parecida com a Clean Architecture, teríamos algo tal como:

```
src/main/java/br/ifsp/edu/todo/
├── application/
│   ├── service/
│   │   └── TaskService.java
│   └── usecase/
│       ├── CreateTaskUseCase.java
│       ├── UpdateTaskUseCase.java
│       ├── DeleteTaskUseCase.java
│       ├── CompleteTaskUseCase.java
│       └── ListTasksUseCase.java
├── domain/
│   ├── model/
│   │   ├── Task.java
│   │   ├── Category.java
│   │   └── Priority.java
│   └── repository/
│       └── TaskRepository.java (interface abstrata, sem Spring aqui)
├── infrastructure/
│   ├── repository/
│   │   └── JpaTaskRepository.java (implementação concreta usando Spring Data JPA)
│   ├── persistence/
│   │   └── HibernateConfig.java
│   └── external/
│       └── EmailNotificationService.java (exemplo de serviço externo)
├── interfaces/
│   ├── controller/
│   │   └── TaskController.java
│   ├── dto/
│   │   ├── task/
│   │   │   ├── TaskRequestDTO.java
│   │   │   ├── TaskResponseDTO.java
│   │   │   └── TaskPatchDTO.java
│   │   └── page/
│   │       └── PagedResponse.java
│   └── mapper/
│       └── TaskMapper.java
├── config/
│   ├── ModelMapperConfig.java
│   ├── SecurityConfig.java
│   └── ApplicationConfig.java (injeções de dependência)
└── TodoApplication.java
```

Nesse caso:

- `domain/` **não** depende de nada externo (nem de Spring, nem de JPA). É puro Java.
- `application/` define a **lógica de orquestração** dos fluxos (Use Cases), chamando métodos do domínio e das portas (repositories).
- `infrastructure/` implementa **tecnologias concretas** (como JPA, envio de e-mails, integrações externas).
- `interfaces/` é a camada que se comunica com o "mundo externo" — REST APIs, Web Controllers, DTOs.
- `config/` agrupa configurações específicas do projeto (Spring Beans, Security, etc).

Ou seja, fazemos o uso da camada *Service* mas não nomeamos tal como fazemos em outros padrões arquiteturais. Futuramente implementaremos essa arquitetura, por ora fica apenas como um exemplo de maneiras alternativas de estruturar nossa aplicação. 🤓

### Resumindo!

A camada Service desempenha um papel importante em arquiteturas em camadas ao **clarificar e centralizar a lógica de negócio da aplicação**, atuando como a fronteira que delimita *o que* a aplicação faz internamente. Sua importância se manifesta em sistemas de médio e grande porte, nos quais a complexidade das regras de negócio e a necessidade de evolução contínua exigem uma estrutura modular. Com apoio de autores clássicos, observa-se um forte embasamento teórico para sua utilização. Em projetos reais, especialmente no desenvolvimento de **APIs REST** com frameworks como Spring Boot, esses conceitos se traduzem em práticas concretas que melhoram a qualidade do código e facilitam manutenção, testes e colaboração da equipe.

Entretanto, fica evidente que a decisão de introduzir ou não a camada Service deve ser **contextualizada**. Em muitos cenários ela traz claras vantagens de organização e robustez, mas em outros pode ser vista como sobrecarga. Arquitetos e desenvolvedores devem avaliar fatores como o tamanho do projeto, a probabilidade de requisitos mudarem, o número de integrações necessárias e até a familiaridade da equipe com o paradigma. O **equilíbrio arquitetural** é desejável: utilizar camadas de serviço quando elas de fato agregam valor (evitando duplicação de lógica, permitindo transações abrangentes, isolando regras complexas) e reconhecer quando um design mais simples poderia bastar (em aplicações de escopo restrito e invariável, por exemplo). Como discutimos em aulas anteriores, não basta apenas dominar a implementação técnica — é fundamental compreender o contexto, as motivações e as consequências de cada decisão que tomamos no projeto!

Dito isso e entendendo a importância dessa camada, vamos passar ao código da solução do exercício!

---

## 2. Estrutura de Pastas e Pacotes

Nossa aplicação será organizada da seguinte forma:

```
src/main/java/br/ifsp/edu/todo/
├── config
│   └── ModelMapperConfig.java
├── controller
│   └── TaskController.java
├── dto
│   ├── page
│   │   └── PagedResponse.java
│   └── task
│       ├── TaskRequestDTO.java
│       └── TaskResponseDTO.java
├── exception
│   ├── ErrorResponse.java
│   ├── GlobalExceptionHandler.java
│   ├── InvalidTaskStateException.java
│   └── ResourceNotFoundException.java
├── mapper
│   └── PagedResponseMapper.java
├── model
│   ├── Category.java
│   ├── Priority.java
│   └── Task.java
├── repository
│   └── TaskRepository.java
├── service
│   └── TaskService.java
└── TodoApplication.java
```

Essa estrutura promove a separação clara de responsabilidades, facilita a manutenção, melhora a escalabilidade do projeto e segue a divisão em camadas que já vínhamos adotando nas aulas anteriores.

A camada `model` é responsável pelas entidades JPA que representam nossas tabelas no banco de dados.

- As entidades possuem anotações do pacote `jakarta.persistence`, como `@Entity`, `@Id`, `@GeneratedValue`, `@Enumerated`, entre outras.
- O mapeamento segue o padrão ORM (Object-Relational Mapping) para garantir a persistência correta dos dados.
- Utilizamos enums para representar valores fixos, como as prioridades das tarefas.

A camada `repository` é responsável pela interação direta com o banco de dados.

- Utilizamos a interface `JpaRepository` para herdar métodos padrão de CRUD.
- Métodos personalizados foram adicionados para suportar funcionalidades como busca por categoria.

A camada `dto` define objetos de transferência de dados, separando o modelo de domínio da representação utilizada nos endpoints.

- `TaskRequestDTO`: encapsula os dados enviados pelo cliente para criação ou atualização de tarefas.
- `TaskResponseDTO`: representa a resposta da API para o cliente.
- `PagedResponse`: padroniza respostas paginadas.

Todos os DTOs contam com anotações de validação (`@NotBlank`, `@Size`, `@Future`, etc.) para garantir a integridade dos dados no momento da entrada.

A camada `mapper` contém classes utilitárias para conversão entre entidades e DTOs, facilitando a adaptação dos modelos de domínio para exposições externas.

- Utilizamos o `ModelMapper` (configurado na pasta `config`) para automatizar os mapeamentos.

A camada `exception` é responsável pelo tratamento centralizado de erros.

- Utilizamos um `GlobalExceptionHandler` anotado com `@ControllerAdvice` para capturar e tratar exceções de forma padronizada.
- Definimos exceções customizadas para representar erros de domínio, como `ResourceNotFoundException` (recurso não encontrado) e `InvalidTaskStateException` (tentativa de operação inválida em tarefas concluídas).
- As respostas de erro são estruturadas utilizando o DTO `ErrorResponse`.

A camada `service` centraliza a lógica de negócio da aplicação, atuando como uma ponte entre o controller e o repositório.

- A classe `TaskService`, anotada com `@Service`, concentra toda a lógica de manipulação de tarefas: criação, atualização, conclusão e exclusão, bem como validações adicionais que não podem ser garantidas apenas com Bean Validation.
- A lógica de verificação de regras de negócio, como **impedir a modificação ou exclusão de tarefas já concluídas**, é implementada aqui.
- Essa abordagem evita a duplicação de código e garante que regras de negócio sejam mantidas de maneira **consistente e centralizada** em um único ponto do sistema.
- A injeção de dependência é feita via construtor, permitindo que os atributos sejam `final`, promovendo imutabilidade e facilitando a criação de mocks para testes unitários.
- A camada de serviço orquestra as operações necessárias: chama o repositório para acessar os dados e, quando necessário, utiliza o ModelMapper para transformar entidades em DTOs ou vice-versa.

**Importante:**  
A camada `service` **não implementa lógica de persistência** (não executa diretamente operações no banco) e **não implementa lógica de interface** (não formata diretamente respostas HTTP). Ela simplesmente **coordena o fluxo de dados e a execução das regras de negócio**.

---

## 3. Código-fonte do To-Do List

Para deixar a explicação mais clara e facilitar o entendimento da estrutura do projeto, organizamos a apresentação das classes seguindo a ordem em que naturalmente elas se conectam dentro da aplicação: primeiro vamos ver o modelo de domínio, que representa os dados que manipulamos; depois os DTOs, que são usados para trocar informações com quem consome a API; na sequência os repositórios, que salvam e recuperam os dados no banco; logo depois a camada de serviços, onde reunimos e coordenamos as regras de negócio; e, por fim, os controllers, que expõem tudo isso através dos endpoints da API. Essa sequência ajuda a construir o raciocínio de dentro para fora — do coração da aplicação até a porta de entrada! 🤩

## 3.1. Modelos de Domínio (`model`)

Vamos começar nossa análise pela **camada de modelo**, onde definimos as **entidades** que representam o núcleo de informação da nossa aplicação. Tudo no sistema — criação de tarefas, atualizações, conclusões, consultas — gira em torno desses modelos. Entender suas estruturas é essencial para compreendermos o comportamento do sistema como um todo.

#### `Task.java`

```java
package br.ifsp.edu.todo.model;

@NoArgsConstructor
@Data
@Entity
public class Task {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NotBlank
    @Size(min = 10, max = 100)
    private String title;

    @Size(max = 255)
    private String description;

    @NotNull
    @Enumerated(EnumType.STRING)
    private Priority priority;

    @NotNull
    private LocalDateTime dueDate;

    private boolean completed;

    @NotNull
    @Enumerated(EnumType.STRING)
    private Category category;

    private LocalDateTime createdAt;
}
```

A classe `Task` representa a entidade principal da nossa aplicação de gerenciamento de tarefas. Ela está anotada com `@Entity`, o que indica que será mapeada para uma tabela no banco de dados pela JPA (Jakarta Persistence API). Cada instância de `Task` corresponde a uma linha na tabela.

Entre seus atributos, temos:

- `id`: a chave primária da tarefa, gerada automaticamente com estratégia de incremento (`IDENTITY`).
- `title`: o título da tarefa, obrigatório (`@NotBlank`) e limitado entre 10 e 100 caracteres para garantir descrições concisas e claras.
- `description`: um campo opcional para detalhes adicionais, limitado a 255 caracteres.
- `priority`: a prioridade da tarefa, obrigatoriamente preenchida, armazenada como texto (`EnumType.STRING`) para facilitar a leitura no banco de dados.
- `dueDate`: a data limite para conclusão da tarefa, também obrigatória.
- `completed`: um booleano que indica se a tarefa foi concluída ou não.
- `category`: a categoria à qual a tarefa pertence (ex: trabalho, estudo, pessoal), também obrigatoriamente preenchida e mapeada como texto no banco.
- `createdAt`: o timestamp de criação da tarefa, preenchido no momento em que a tarefa é criada.

A classe utiliza ainda o `Lombok` para gerar automaticamente métodos como getters, setters e o construtor padrão (`@NoArgsConstructor` e `@Data`), reduzindo o código repetitivo (boilerplate) e deixando a implementação mais limpa e focada apenas nas informações essenciais da entidade.

#### `Category.java`

```java
package br.ifsp.edu.todo.model;

public enum Category {
    STUDY, WORK, LEISURE, HEALTH, FAMILY, FRIENDS, PERSONAL, OTHER;
    
    public static Category fromString(String value) {
        return Arrays.stream(values()).filter(c -> c.name().equalsIgnoreCase(value)).findFirst()
                .orElseThrow(() -> new IllegalArgumentException("Invalid category: " + value));
    }
}
```

A classe `Category` define uma enumeração que representa as categorias possíveis para uma tarefa no nosso sistema. As opções incluem valores como `STUDY`, `WORK`, `LEISURE`, `HEALTH`, entre outros, possibilitando que o usuário classifique melhor suas tarefas de acordo com áreas da vida. 

Além disso, a enumeração implementa o método utilitário `fromString(String value)`, que permite converter uma string recebida — como em entradas de usuários ou dados de APIs — em um valor válido da enumeração. Esse método percorre todas as categorias existentes e realiza uma comparação `case-insensitive` para encontrar o valor correspondente. Caso a string informada não corresponda a nenhuma categoria conhecida, é lançada uma `IllegalArgumentException`, garantindo que apenas categorias válidas sejam aceitas. Essa abordagem reforça a robustez da nossa aplicação ao evitar erros silenciosos e inconsistências nos dados. Isso será bastante útil na nossa `TaskService`, como veremos mais adiante. 

#### `Priority.java`

```java
package br.ifsp.edu.todo.model;

public enum Priority {
    HIGH,
    MEDIUM,
    LOW
}
```

A classe `Priority` define uma enumeração simples que representa os níveis de prioridade que uma tarefa pode ter no sistema. As opções disponíveis são `HIGH` (alta), `MEDIUM` (média) e `LOW` (baixa). Essa enumeração é usada para classificar a importância relativa de cada tarefa, permitindo que o usuário ou a aplicação deem tratamento diferenciado de acordo com a prioridade definida. Por ser uma enumeração básica sem métodos adicionais, seu papel principal é fornecer um conjunto fixo e seguro de valores que podem ser atribuídos às tarefas, garantindo consistência e evitando a utilização de valores inválidos no sistema.

Concluindo esta seção, percebemos que as entidades `Task`, `Priority` e `Category` formam a espinha dorsal do nosso domínio, definindo tanto os dados persistidos quanto as restrições de valores possíveis. A clareza e a correção nesta camada são fundamentais, pois qualquer erro aqui propaga-se para todas as demais camadas. Agora que compreendemos como o núcleo da nossa aplicação — o modelo de domínio — está estruturado, é hora de avançarmos para uma camada igualmente fundamental: a dos DTOs. São eles que estabelecerão a comunicação segura entre o mundo externo e os nossos modelos internos.

## 3.2. Data Transfer Objects (`dto`)

Agora que conhecemos os modelos de domínio, vamos analisar como estruturamos a comunicação de dados entre o cliente e a nossa aplicação: os **DTOs**. Os DTOs nos permitem expor apenas as informações necessárias de forma controlada, além de validar a entrada de dados de maneira eficiente e desacoplada do modelo de domínio.

#### `TaskRequestDTO.java`

```java
package br.ifsp.edu.todo.dto.task;

@Data
public class TaskRequestDTO {
 @NotBlank
    @Size(min = 10, max = 100)
    private String title;

    @Size(max = 255)
    private String description;

    @NotNull
    private Priority priority;

    @NotNull
    @FutureOrPresent
    private LocalDateTime dueDate;

    private boolean completed;

    @NotNull
    private Category category;
}
```
A classe `TaskRequestDTO` representa o modelo de **entrada de dados** que a aplicação espera receber do cliente quando ele quiser criar ou atualizar uma tarefa. Ou seja, sempre que um usuário submete uma requisição para cadastrar ou alterar uma tarefa, os dados são mapeados para este DTO.

Principais características:
- **Validações com Bean Validation**:
  - `@NotBlank` e `@Size(min = 10, max = 100)` no `title`, garantindo que o título seja obrigatório e esteja entre 10 e 100 caracteres.
  - `@Size(max = 255)` para limitar o tamanho da `description`.
  - `@NotNull` no `priority`, `dueDate` e `category`, assegurando que esses campos sejam sempre informados.
  - `@FutureOrPresent` na `dueDate`, impedindo a criação de tarefas com data limite no passado.
- **Campos**:
  - `title`, `description`, `priority`, `dueDate`, `completed`, `category`.
  - Note que **não há campo `id` nem `createdAt`** no `TaskRequestDTO`, pois essas informações são geradas e controladas internamente pela aplicação e não devem ser manipuladas pelo usuário. 🙂

Em resumo, o `TaskRequestDTO` protege o modelo de domínio de dados inválidos e padroniza a estrutura de entrada de dados para as operações de criação e atualização.

#### `TaskResponseDTO.java`

```java
package br.ifsp.edu.todo.dto.task;

@Data
public class TaskResponseDTO {
    private Long id;
    private String title;
    private String description;
    private Priority priority;
    private LocalDateTime dueDate;
    private boolean completed;
    private Category category;
    private LocalDateTime createdAt;
}
```

A classe `TaskResponseDTO` define o modelo de **saída de dados** enviado de volta ao cliente quando uma tarefa é consultada, criada ou atualizada. É a estrutura que encapsula todas as informações necessárias para o cliente visualizar ou tratar a resposta da API.

Principais características:
- **Campos retornados**:
  - `id`: identificador único da tarefa.
  - `title`: título da tarefa.
  - `description`: descrição da tarefa.
  - `priority`: prioridade da tarefa.
  - `dueDate`: data limite para conclusão.
  - `completed`: status de conclusão.
  - `category`: categoria associada.
  - `createdAt`: data em que a tarefa foi criada.

Aqui, diferente do `TaskRequestDTO`, incluímos informações **geradas pela aplicação**, como `id` e `createdAt`, que não fazem sentido serem enviados pelo usuário, mas são muito relevantes para o consumidor da API!

#### 📄 `PagedResponse.java`

```java
package br.ifsp.edu.todo.dto.page;

@Data
@AllArgsConstructor
public class PagedResponse<T> {
    private List<T> content;
    private int page;
    private int size;
    private long totalElements;
    private int totalPages;
    private boolean last;
}
```

Aqui temos a primeira diferença em relação ao que fizemos anteriormente: a classe `PagedResponse<T>` é um DTO genérico criado para padronizar as respostas paginadas da API. Dessa forma, conseguimos garantir que todas as respostas paginadas da nossa aplicação sigam um formato consistente, facilitando o consumo por parte de front-ends ou integrações. Em vez de retornarmos diretamente um `Page<T>`, que é uma estrutura interna do Spring, adaptamos os dados para esse DTO mais controlado e amigável para o cliente.

Principais características:
- **Campos**:
  - `content`: lista de elementos da página (do tipo genérico `T`).
  - `pageNumber`, `pageSize`, `totalElements`, `totalPages`: metadados sobre a paginação.
  - `first`, `last`: flags indicando se estamos na primeira ou na última página.

Assim, os DTOs servem como **fronteira de segurança** e **contrato de comunicação** da nossa API. Eles reforçam a separação entre o que acontece internamente na aplicação e o que é exposto externamente, promovendo segurança, clareza e controle sobre a evolução da interface pública do sistema. Com os DTOs estruturando as entradas e saídas de dados, precisamos garantir que essas informações sejam corretamente persistidas no banco de dados. Vamos então explorar a camada de repositórios, que isola e facilita essa comunicação com o armazenamento permanente.

## 3.3. Repositórios (`repository`)

Com os dados e suas representações de entrada e saída definidos, precisamos agora garantir a **persistência** dessas informações. Para isso, utilizamos os **repositórios**, que fornecem uma abstração poderosa para comunicação com o banco de dados, reduzindo significativamente o esforço necessário para operações CRUD e permitindo métodos de consulta personalizados.

#### `TaskRepository.java`

```java
package br.ifsp.edu.todo.repository;
public interface TaskRepository extends JpaRepository<Task, Long> {

    Page<Task> findByCategory(Category category, Pageable pageable);
}
```

A interface `TaskRepository` representa a camada de acesso a dados da nossa aplicação, sendo responsável pela comunicação direta com o banco de dados. Ela estende `JpaRepository<Task, Long>`, o que nos permite herdar diversos métodos prontos para realizar operações CRUD (`findAll`, `save`, `deleteById`, `findById`, etc.) sem precisar implementá-los manualmente. Lembrem-se que vimos isso nas aulas anteriores!

Além disso, a `TaskRepository` define um método personalizado:

```java
Page<Task> findByCategory(Category category, Pageable pageable);
```

Esse método permite buscar tarefas filtrando por uma determinada categoria, já com suporte a paginação. O Spring Data JPA interpreta o nome do método (pela convenção *query method naming*, lembram-se?!) e gera automaticamente a consulta necessária para o banco de dados.

Com isso, conseguimos facilmente construir consultas específicas apenas declarando métodos na interface, sem a necessidade de escrever JPQL ou SQL manualmente — o que torna o desenvolvimento mais ágil e o código mais limpo.

Essa abordagem é especialmente útil para aplicações como a nossa, onde queremos que o repositório atue como um componente **focado apenas em persistência de dados**, enquanto toda a lógica de negócio permanece na camada de serviço.

Dessa forma, a camada de repositórios isola o acesso à base de dados e libera as demais camadas — principalmente a de serviço — da preocupação com detalhes técnicos de persistência. Esse isolamento é crucial para garantir a flexibilidade e testabilidade da nossa aplicação. Sabendo como armazenar e recuperar informações, surge agora uma pergunta importante: quem será o responsável por coordenar a lógica que conecta tudo isso? Para responder a essa questão, vamos mergulhar na camada de serviço, onde reside a inteligência operacional da aplicação.

## 3.4. Serviços (`service`)

A seguir, vamos explorar a **camada de serviço**, que é responsável por coordenar os casos de uso da aplicação, centralizar regras de negócio e garantir a integridade das operações. Nesta camada implementamos toda a lógica que regula as operações do sistema, sempre respeitando o princípio de separação de responsabilidades que vimos anteriormente.

#### `TaskService.java`

```java
package br.ifsp.edu.todo.service;

@Service
public class TaskService {
    private final TaskRepository taskRepository;
    private final ModelMapper modelMapper;
    private final PagedResponseMapper pagedResponseMapper;
    
    public TaskService(TaskRepository taskRepository, ModelMapper modelMapper, PagedResponseMapper pagedResponseMapper) {
        this.taskRepository = taskRepository;
        this.modelMapper = modelMapper;
        this.pagedResponseMapper = pagedResponseMapper;
    }
    
    public TaskResponseDTO createTask(TaskRequestDTO taskDto) {
        if (taskDto.getDueDate().isBefore(LocalDateTime.now()))
            throw new ValidationException("Due date cannot be in the past.");
        
        Task task = modelMapper.map(taskDto, Task.class);
        task.setCreatedAt(LocalDateTime.now());
        task.setCompleted(false);
        Task createdTask = taskRepository.save(task);
        return modelMapper.map(createdTask, TaskResponseDTO.class);
    }
    
    public PagedResponse<TaskResponseDTO> getAllTasks(Pageable pageable) {
        Page<Task> tasksPage = taskRepository.findAll(pageable);
        return pagedResponseMapper.toPagedResponse(tasksPage, TaskResponseDTO.class);
    }
    
    public TaskResponseDTO getTaskById(Long id) {
        Task task = taskRepository.findById(id).orElseThrow(() -> new ResourceNotFoundException("Task not found"));
        return modelMapper.map(task, TaskResponseDTO.class);
    }
    
    public PagedResponse<TaskResponseDTO> searchByCategory(String category, Pageable pageable) {
        Category categoryEnum = Category.fromString(category);
        Page<Task> tasks = taskRepository.findByCategory(categoryEnum, pageable);
        return pagedResponseMapper.toPagedResponse(tasks, TaskResponseDTO.class);
    }
    
    public TaskResponseDTO concludeTask(Long id) {
        Task task = taskRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Task not found with ID: " + id));
        
        if (task.isCompleted()) {
            throw new InvalidTaskStateException("Task is already completed.");
        }
        
        task.setCompleted(true);
        Task updatedTask = taskRepository.save(task);
        return modelMapper.map(updatedTask, TaskResponseDTO.class);
    }
    
    public TaskResponseDTO updateTask(Long id, TaskRequestDTO taskDto) {
        if (taskDto.getDueDate().isBefore(LocalDateTime.now()))
            throw new ValidationException("Due date cannot be in the past");
        
        Task existingTask = taskRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Task not found with ID: " + id));
        
        if (existingTask.isCompleted())
            throw new InvalidTaskStateException("Completed tasks cannot be updated");
        
        modelMapper.map(taskDto, existingTask);
        existingTask.setId(id);
        existingTask.setCreatedAt(existingTask.getCreatedAt()); // preserva a data original de criação!
        Task updatedTask = taskRepository.save(existingTask);
        return modelMapper.map(updatedTask, TaskResponseDTO.class);
    }
    
    public void deleteTask(Long id) {
        Task task = taskRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Task not found with ID: " + id));
        
        if (task.isCompleted()) {
            throw new InvalidTaskStateException("Cannot delete a completed task");
        }
        
        taskRepository.delete(task);
    }
}
```

A classe `TaskService` é o **centro da nossa lógica de negócio** no projeto de gerenciamento de tarefas. Ela é a responsável por coordenar as operações sobre as entidades do sistema, garantindo que as regras sejam aplicadas de maneira consistente e que o controller não fique sobrecarregado com decisões que não lhe competem.

A classe é anotada com `@Service`, o que faz com que o Spring reconheça automaticamente essa classe como um *componente de serviço*, permitindo sua injeção em outros pontos do sistema (como nos controllers) de forma automática. Essa anotação é importante porque ajuda a manter a **organização por responsabilidades** dentro do projeto.

As dependências (`TaskRepository`, `ModelMapper`, `PagedResponseMapper`) são injetadas via **construtor**. Essa abordagem de injeção por construtor, ao invés do uso de `@Autowired` nos atributos, traz vários benefícios:
- Permite que os campos sejam `final`, garantindo **imutabilidade** — ou seja, que o serviço não troque de dependência depois de criado.
- Facilita a **testabilidade** — podemos facilmente criar instâncias da `TaskService` passando mocks dessas dependências em testes unitários.
- Deixa a classe mais **autoexplicativa**, pois o construtor mostra claramente quais são suas necessidades para funcionar.

Vamos explicar os métodos da `TaskService`:

- **`createTask(TaskRequestDTO dto)`**  
  - Primeiro, valida se a `dueDate` da tarefa (data limite) é futura. Se for uma data no passado, lançamos uma `ValidationException`.  
  - Depois, o `ModelMapper` é usado para converter o DTO recebido (`TaskRequestDTO`) para uma entidade real (`Task`).  
  - A tarefa recém-criada é inicializada com a data atual (`createdAt = LocalDateTime.now()`) e marcada como não concluída (`completed = false`).  
  - Por fim, a tarefa é salva no banco de dados através do `taskRepository`, e retornamos a resposta no formato `TaskResponseDTO` — ou seja, padronizamos sempre a saída para o cliente.

- **`getAllTasks(Pageable pageable)`**  
  - Faz a consulta paginada no banco através do `taskRepository.findAll(pageable)`.
  - Usa o `PagedResponseMapper` para transformar o `Page<Task>` em um `PagedResponse<TaskResponseDTO>`, que é um DTO nosso mais amigável e controlado (evitando expor detalhes internos do Spring como o objeto `Page`).  
  - Essa separação entre entidades internas e o que expomos para fora é fundamental para garantir que **mudanças internas** (por exemplo, trocarmos a biblioteca de paginação) não quebrem contratos da API.

- **`getTaskById(Long id)`**  
  - Busca uma tarefa pelo ID.
  - Se a tarefa não existir, lança uma `ResourceNotFoundException` com uma mensagem específica.
  - Retorna o DTO da tarefa encontrada.
  
  Esse padrão (`findById().orElseThrow()`) é uma forma moderna e segura de trabalhar com valores opcionais no Java usando `Optional`, evitando `null` e melhorando a legibilidade.

- **`searchByCategory(String category, Pageable pageable)`**  
  - Converte o nome da categoria (`String`) para o `enum Category` usando o método `Category.fromString(category)`.
  - Consulta no repositório todas as tarefas com a categoria correspondente.
  - Usa novamente o `PagedResponseMapper` para devolver o resultado no formato consistente.

- **`concludeTask(Long id)`**  
  - Busca a tarefa pelo ID.
  - Verifica se a tarefa já está marcada como concluída. Se sim, lança uma `InvalidTaskStateException` (evitando que a mesma tarefa seja "reconcluída" várias vezes).  
  - Se não estiver concluída, marca como concluída (`completed = true`) e salva a atualização.

- **`updateTask(Long id, TaskRequestDTO dto)`**  
  - Busca a tarefa existente.
  - Verifica se ela já foi concluída. Se estiver concluída, não é possível alterar — então lançamos uma `InvalidTaskStateException`.
  - Valida se a nova `dueDate` fornecida é futura (não aceitamos alterações que "voltem no tempo").
  - Usa o `ModelMapper` para copiar os dados do DTO para a entidade existente.
  - **Importante**: preservamos campos que não devem ser alterados manualmente, como `id`, `completed` e `createdAt`. Isso é feito explicitamente logo após o mapeamento para garantir que o cliente não consiga sobrescrever essas informações.  
    ```java
    updatedTask.setId(existingTask.getId());
    updatedTask.setCompleted(existingTask.isCompleted());
    updatedTask.setCreatedAt(existingTask.getCreatedAt());
    ```
  - Salva a tarefa atualizada e retorna o DTO.

- **`deleteTask(Long id)`**  
  - Busca a tarefa pelo ID.
  - Verifica se ela foi concluída. Se já estiver concluída, não permitimos a exclusão — para evitar perda de histórico de tarefas finalizadas (uma regra de negócio típica em muitos sistemas de tarefas).  
  - Se não estiver concluída, apagamos do banco usando `deleteById(id)`.

⚠️⚠️ Destaques Importantes:

- A `TaskService` **coordena todos os casos de uso** da aplicação, sem misturar responsabilidade de interação com a web ou com o banco — essas tarefas ficam para o controller e o repository, respectivamente.
- **Regra de negócio** (ex: não alterar tarefas concluídas) é aplicada de forma centralizada, consistente e previsível.
- Uso cuidadoso de **exceções específicas** torna os erros mais compreensíveis e o tratamento no controller mais fácil.
- **Mapeamento com ModelMapper** facilita a conversão entre entidades e DTOs, mas com cuidados manuais em campos sensíveis.
- Separação entre resposta para cliente (`PagedResponse`, `TaskResponseDTO`) e estruturas internas do Spring (`Page`) aumenta a robustez e liberdade evolutiva da API.

Assim, o `TaskService` cumpre seu papel essencial: proteger o domínio da aplicação e fornecer um ponto de entrada claro, seguro e reutilizável para todas as operações relacionadas às tarefas.

Encerrando esta seção, fica claro que a camada de serviço não apenas organiza o fluxo das operações como também **evita duplicação de lógica**, **facilita a manutenção** e **torna a aplicação mais testável**. Ela é o verdadeiro cérebro da aplicação, conectando modelos, DTOs e repositórios em fluxos consistentes de negócio. Com a lógica de negócio centralizada e bem organizada na camada de serviços, resta apenas um elo para completarmos nosso fluxo de aplicação: a exposição dessa lógica para o mundo exterior. Vamos então conhecer a camada de controllers, responsável por receber as requisições dos usuários e orquestrar as operações do sistema.

## 3.5. Controladores (`controller`)

Vamos analisar os **controllers**, que são responsáveis por receber as requisições HTTP, validar entradas e delegar as operações para os serviços. Os controllers atuam como **portões de entrada** da aplicação, traduzindo o mundo externo (requisições REST) para chamadas internas de negócio.

#### `TaskController.java`

```java
package br.ifsp.edu.todo.controller;

@RestController
@RequestMapping("/api/tasks")
public class TaskController {
    private final TaskService taskService;
    
    public TaskController(TaskService taskService) {
        this.taskService = taskService;
    }
    
    @PostMapping
    public ResponseEntity<TaskResponseDTO> createTask(@Valid @RequestBody TaskRequestDTO task) {
        TaskResponseDTO taskResponseDTO = taskService.createTask(task);
        return ResponseEntity.status(HttpStatus.CREATED).body(taskResponseDTO);
    }
    
    @GetMapping
    public ResponseEntity<PagedResponse<TaskResponseDTO>> getAllTasks(Pageable pageable) {
        return ResponseEntity.ok(taskService.getAllTasks(pageable));
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<TaskResponseDTO> getTaskById(@PathVariable Long id) {
        return ResponseEntity.ok(taskService.getTaskById(id));
    }
    
    @GetMapping("/search")
    public ResponseEntity<PagedResponse<TaskResponseDTO>> searchByCategory(@RequestParam String category, Pageable pageable) {
        return ResponseEntity.ok(taskService.searchByCategory(category, pageable));
    }
    
    @PatchMapping("/{id}/finish")
    public ResponseEntity<TaskResponseDTO> concludeTask(@PathVariable Long id) {
        TaskResponseDTO response = taskService.concludeTask(id);
        return ResponseEntity.ok(response);
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<TaskResponseDTO> updateTask(@PathVariable Long id,
            @Valid @RequestBody TaskRequestDTO taskDto) {
        TaskResponseDTO updatedTask = taskService.updateTask(id, taskDto);
        return ResponseEntity.ok(updatedTask);
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteTask(@PathVariable Long id) {
        taskService.deleteTask(id);
        return ResponseEntity.noContent().build();
    }
}
```

A classe `TaskController` é o ponto de entrada da nossa API para manipulação de tarefas. Ela define os **endpoints REST** que os clientes podem utilizar para interagir com o sistema, como criar, buscar, atualizar, concluir e excluir tarefas.

Logo de início, vemos que a classe é anotada com `@RestController` e `@RequestMapping("/api/tasks")`, o que significa que:

- Ela será responsável por tratar requisições HTTP.
- Todos os endpoints definidos aqui terão como prefixo `/api/tasks` (por exemplo: `/api/tasks/1`, `/api/tasks/search`, etc).

Internamente, o `TaskController` injeta a dependência do `TaskService` via construtor. Como vimos anteriormente, isso é uma boa prática, pois favorece a imutabilidade dos atributos (`final`) e facilita a testabilidade da classe.

Agora vamos entender cada um dos métodos:

##### Endpoint de criação de tarefas

```java
@PostMapping
public ResponseEntity<TaskResponseDTO> createTask(@Valid @RequestBody TaskRequestDTO task) {
    TaskResponseDTO taskResponseDTO = taskService.createTask(task);
    return ResponseEntity.status(HttpStatus.CREATED).body(taskResponseDTO);
}
```

- **`@PostMapping`** indica que este método responde a requisições HTTP POST.
- Ele recebe um `TaskRequestDTO` no corpo da requisição (`@RequestBody`) e valida os dados automaticamente com `@Valid`.
- A chamada é repassada para o `taskService.createTask()`, onde a lógica de criação acontece.
- A resposta é encapsulada em um `ResponseEntity` com o status `201 CREATED`, indicando que um novo recurso foi criado com sucesso.

##### Endpoint de listagem paginada de tarefas

```java
@GetMapping
public ResponseEntity<PagedResponse<TaskResponseDTO>> getAllTasks(Pageable pageable) {
    return ResponseEntity.ok(taskService.getAllTasks(pageable));
}
```

- **`@GetMapping`** mapeia este método para requisições HTTP GET em `/api/tasks`.
- Utiliza o `Pageable`, que é injetado automaticamente pelo Spring para suportar paginação e ordenação.
- Retorna uma resposta padronizada usando nosso `PagedResponse`, encapsulada com `ResponseEntity.ok()` para indicar sucesso.

##### Endpoint de busca por ID

```java
@GetMapping("/{id}")
public ResponseEntity<TaskResponseDTO> getTaskById(@PathVariable Long id) {
    return ResponseEntity.ok(taskService.getTaskById(id));
}
```

- Busca uma tarefa específica pelo seu identificador (`id`) passado na URL.
- Se o ID for encontrado, retorna o DTO da tarefa com `200 OK`.
- Se não, o `TaskService` irá lançar uma exceção que será tratada pelo `GlobalExceptionHandler`.

##### Endpoint de busca por categoria

```java
@GetMapping("/search")
public ResponseEntity<PagedResponse<TaskResponseDTO>> searchByCategory(@RequestParam String category, Pageable pageable) {
    return ResponseEntity.ok(taskService.searchByCategory(category, pageable));
}
```

- Permite buscar tarefas que pertencem a uma determinada categoria.
- O parâmetro `category` é passado pela **query string** (ex: `/api/tasks/search?category=STUDY`).
- Também suporta paginação (`Pageable`).

##### Endpoint para concluir tarefa

```java
@PatchMapping("/{id}/finish")
public ResponseEntity<TaskResponseDTO> concludeTask(@PathVariable Long id) {
    TaskResponseDTO response = taskService.concludeTask(id);
    return ResponseEntity.ok(response);
}
```

- Este método permite marcar uma tarefa como concluída.
- Utiliza `@PatchMapping`, que é o verbo apropriado para **atualizações parciais** (alterar apenas o status da tarefa).
- O status de resposta será `200 OK` com os dados da tarefa atualizada.

##### Endpoint para atualização completa de tarefa

```java
@PutMapping("/{id}")
public ResponseEntity<TaskResponseDTO> updateTask(@PathVariable Long id,
        @Valid @RequestBody TaskRequestDTO taskDto) {
    TaskResponseDTO updatedTask = taskService.updateTask(id, taskDto);
    return ResponseEntity.ok(updatedTask);
}
```

- Permite **atualizar todos os dados** de uma tarefa existente (não apenas um campo específico).
- Utiliza `@PutMapping`, que, de acordo com a semântica REST, representa **substituição integral** do recurso.
- O corpo da requisição também é validado automaticamente.

##### Endpoint para excluir tarefa

```java
@DeleteMapping("/{id}")
public ResponseEntity<Void> deleteTask(@PathVariable Long id) {
    taskService.deleteTask(id);
    return ResponseEntity.noContent().build();
}
```

- Exclui uma tarefa com base no ID informado.
- Utiliza `@DeleteMapping`, que mapeia para operações de deleção.
- Retorna um `204 No Content`, indicando que a operação foi bem-sucedida mas que **não há corpo na resposta**.

##### 🌟 Considerações Gerais

Perceba que nessa implementação seguimos a ideia mencionada ao dissertarmos sobre a importância da camada service:

- **Responsabilidade clara:** O controller não possui lógica de negócio, apenas orquestra chamadas ao `TaskService`.
- **Validação:** Usa `@Valid` nos métodos que recebem dados para garantir que as informações estejam corretas antes de tentar persistir no banco.
- **Uso correto de status HTTP:** Cada operação retorna um status condizente com o seu objetivo (201, 200, 204).
- **Separa controle de fluxo da lógica de negócio:** Delega tudo que é mais complexo para a camada de serviço.
- **Padronização:** Utiliza `ResponseEntity` em todos os métodos, garantindo que o formato das respostas seja consistente.

Concluindo a análise dos controllers, percebemos que eles permanecerem **leves e orquestradores**, lidando apenas com aspectos de roteamento, validação inicial e formatação de resposta. Toda a complexidade do sistema já foi devidamente isolada nas camadas anteriores — exatamente como propõe uma boa arquitetura em camadas. 🚀

Ainda falta, entretanto, explorarmos o tratamento de erros, as configurações de mapeamento entre Entidades e os Testes — aspectos transversais que permeiam toda a aplicação e que são fundamentais para garantir robustez e qualidade ao nosso sistema.

## 3.6. Aspectos Transversais da Aplicação

Até agora exploramos as principais camadas estruturais do projeto — **modelos**, **DTOs**, **repositórios**, **serviços** e **controllers** — seguindo uma separação de responsabilidades bem definida. No entanto, existem alguns aspectos que, embora não pertençam diretamente a uma única camada, são essenciais para o funcionamento fluido e profissional da aplicação como um todo.  

Esses aspectos são chamados de **transversais** porque impactam várias partes do sistema simultaneamente, oferecendo suporte e robustez tanto no desenvolvimento quanto na execução da API. São eles:

- **Configuração de mapeamento de objetos** (ModelMapper);
- **Tratamento global de exceções**;
- **Testes automatizados**.

A ordem de apresentação seguirá uma lógica de dependência e compreensão: primeiro veremos **como os dados são transformados** internamente, depois **como lidamos com comportamentos inesperados** no fluxo de execução, e, finalmente, **como garantimos a qualidade e a estabilidade** da aplicação com testes.

Nao teremos nenhuma novidade em relação ao que vimos, na prática, anteriormente: a apresentação é para fins didáticos e conceituais, reforçando a aprendizagem que temos tido nas últimas semanas!

## 3.6.1. Mapeamento de Entidades

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

## 3.6.2. Tratamento Global de Exceções: GlobalExceptionHandler

Após entender como os dados são mapeados dentro da aplicação, precisamos nos preocupar com **o que acontece quando algo sai do esperado**.  

Nem sempre uma requisição será válida, nem todo recurso solicitado existirá, e nem todas as operações serão permitidas — e nossa API precisa reagir a essas situações **de forma padronizada e amigável**. É o que já fizemos anteriormente, mas vamos repetir aqui para fins didáticos.

#### `GlobalExceptionHandler.java`

```java
package br.ifsp.edu.todo.exception;

@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFoundException(ResourceNotFoundException ex) {
        ErrorResponse errorResponse = new ErrorResponse(HttpStatus.NOT_FOUND.value(), ex.getMessage());
        return new ResponseEntity<>(errorResponse, HttpStatus.NOT_FOUND);
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleMethodArgumentNotValid(MethodArgumentNotValidException ex) {
        String errorMessage = ex.getBindingResult().getFieldErrors().stream()
                .map(err -> err.getField() + ": " + err.getDefaultMessage()).collect(Collectors.joining(", "));
        
        ErrorResponse errorResponse = new ErrorResponse(HttpStatus.BAD_REQUEST.value(), errorMessage);
        return new ResponseEntity<>(errorResponse, HttpStatus.BAD_REQUEST);
    }

    @ExceptionHandler(ValidationException.class)
    public ResponseEntity<ErrorResponse> handleValidationException(ValidationException ex) {
        ErrorResponse errorResponse = new ErrorResponse(HttpStatus.BAD_REQUEST.value(), ex.getMessage());
        return new ResponseEntity<>(errorResponse, HttpStatus.BAD_REQUEST);
    }
    
    @ExceptionHandler(InvalidTaskStateException.class)
    public ResponseEntity<ErrorResponse> handleInvalidTaskStateException(InvalidTaskStateException ex) {
        ErrorResponse errorResponse = new ErrorResponse(HttpStatus.CONFLICT.value(), ex.getMessage());
        return new ResponseEntity<>(errorResponse, HttpStatus.CONFLICT);
    }
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGenericException(Exception ex) {
        ErrorResponse errorResponse = new ErrorResponse(HttpStatus.INTERNAL_SERVER_ERROR.value(),
                "An unexpected error occurred");
        return new ResponseEntity<>(errorResponse, HttpStatus.INTERNAL_SERVER_ERROR);
    }

    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<ErrorResponse> handleIllegalArgumentException(IllegalArgumentException ex) {
        ErrorResponse errorResponse = new ErrorResponse(HttpStatus.BAD_REQUEST.value(), ex.getMessage());
        return new ResponseEntity<>(errorResponse, HttpStatus.BAD_REQUEST);
    }
}
```

A `GlobalExceptionHandler` é a classe que efetivamente realiza o tratamento centralizado das exceções em nossa aplicação.
Ela é anotada com `@RestControllerAdvice`, uma especialização do `@ControllerAdvice` combinada com `@ResponseBody`, que intercepta exceções lançadas em toda a aplicação e gera respostas HTTP amigáveis e padronizadas.

Essa classe define métodos como:

Tratamento de `ResourceNotFoundException` → retorna HTTP 404.

Tratamento de `InvalidTaskStateException` → retorna HTTP 409.

Tratamento genérico de outras exceções → retorna HTTP 500.

Cada método constrói um ErrorResponse, define o status correto e retorna uma ResponseEntity<ErrorResponse>. Isso nos permite separar totalmente a lógica de negócio da lógica de tratamento de erro, promovendo a limpeza e a organização da aplicação.

Além disso, ter uma camada de tratamento global facilita a adição de tratamentos personalizados no futuro, como logs de exceções, métricas de falhas ou alertas de erro.

Com isso, garantimos que os clientes da nossa API recebam:

- Códigos de status HTTP adequados (`400`, `404`, `409`, `500`, etc.);
- Mensagens de erro claras e compreensíveis;
- Estruturas de resposta consistentes.

Até aqui, nada de novo!

#### `ResourceNotFoundException.java`

```java
package br.ifsp.edu.todo.exception;

public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }

    public ResourceNotFoundException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

Essa classe representa uma exceção específica para o cenário em que um recurso solicitado (por exemplo, uma tarefa) não é encontrado no sistema. 

Ela estende `RuntimeException`, o que significa que é uma exceção não verificada (unchecked), e por isso não exige tratamento obrigatório no momento de sua propagação.

Sua implementação é bastante simples e elegante: possui apenas um construtor que recebe a mensagem de erro. Essa mensagem será utilizada mais adiante pela camada de tratamento global (`GlobalExceptionHandler`) para gerar a resposta HTTP apropriada.

Essa exceção melhora a legibilidade e a semântica da aplicação, pois ao lançarmos explicitamente uma `ResourceNotFoundException`, deixamos claro qual é o problema ocorrido, em vez de depender de mensagens genéricas.

#### `InvalidTaskStateException.java`

```java
package br.ifsp.edu.todo.exception;

public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }

    public ResourceNotFoundException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

De maneira semelhante, a classe `InvalidTaskStateException` representa situações em que a operação solicitada não é permitida dado o estado atual da tarefa — por exemplo, tentar editar ou excluir uma tarefa que já foi concluída.

Ela também estende `RuntimeException` e possui dois construtores:

- Um que recebe apenas a mensagem de erro.

- Outro que recebe a mensagem de erro e a causa (outra exceção que possa ter originado o erro), permitindo o encadeamento de exceções se necessário.

A criação de exceções específicas como esta é fundamental para a clareza do código: elas tornam o fluxo de erro mais explícito e a manutenção mais fácil, além de favorecer o tratamento diferenciado de casos de erro específicos na camada de exceções globais.

#### `ErrorResponse.java`

```java
package br.ifsp.edu.todo.exception;

@Data
@AllArgsConstructor
public class ErrorResponse {
    private int status;
    private String message;
    private LocalDateTime timestamp;

    public ErrorResponse(int status, String message) {
        this.status = status;
        this.message = message;
        this.timestamp = LocalDateTime.now();
    }
}
```

A classe ErrorResponse é um DTO de erro, criado para padronizar o corpo das respostas de erro que nossa API envia para os clientes. Ela possui três atributos:

- status: o código de status HTTP associado ao erro (ex: 404, 409, 400).

- message: a mensagem de erro detalhada.

- timestamp: o momento exato em que o erro ocorreu.

O uso desse DTO traz vários benefícios:

- Consistência: todas as respostas de erro seguem o mesmo formato.

- Facilidade de análise: tanto humanos quanto sistemas automatizados (como front-ends) podem interpretar e exibir as mensagens de erro de maneira uniforme.

- Rastreamento: o timestamp facilita a investigação de problemas e a correlação de eventos em logs.

Além disso, a classe utiliza anotações Lombok (`@Data` e `@AllArgsConstructor`) para reduzir o boilerplate, mas também oferece um construtor personalizado que define o timestamp automaticamente como `LocalDateTime.now()`, caso ele não seja informado.

Repare que essa classe, apesar de ser um DTO, está no pacote `exception`. Ao posicioná-la no pacote `exception`, estamos reforçando uma decisão **semântica**: o ErrorResponse não é um DTO comum usado em fluxos de sucesso, mas sim um DTO especializado no tratamento de erros.

No nosso projeto de To-Do List, dado que o escopo é controlado e estamos prezando por clareza semântica acima da ortodoxia estrutural, deixar ErrorResponse no pacote exception é uma escolha válida e até recomendada. Mas, para projetos muito grandes e de múltiplas equipes, seria prudente repensar e possivelmente consolidar todos os DTOs no mesmo pacote!

Entendida a camada de tratamento de erros, passemos, finalmente, aos testes!

## 3.6.3. Testes Automatizados

Com o mapeamento de objetos bem resolvido e o tratamento de erros devidamente padronizado, estamos prontos para consolidar a robustez da aplicação: através dos **testes automatizados**.

Testes são a nossa principal ferramenta para **garantir que o sistema se comporte como o esperado** hoje — e continue se comportando assim no futuro, mesmo diante de evoluções ou refatorações.  

No projeto, estruturamos nossos testes de forma a cobrir tanto aspectos **unitários** (camada a camada) quanto aspectos **funcionais** (comportamento da API como um todo).

#### `TaskServiceTest.java`

```java
package br.ifsp.edu.todo.task;

@ExtendWith(MockitoExtension.class)
public class TaskServiceTest {
    
    @Mock
    private TaskRepository taskRepository;

    @Mock
    private ModelMapper modelMapper;

    @Mock
    private PagedResponseMapper pagedResponseMapper;

    @InjectMocks
    private TaskService taskService;
    
    @Test
    void shouldCreateTaskWithValidData() {
        TaskRequestDTO dto = new TaskRequestDTO();
        dto.setTitle("Valid Task");
        dto.setPriority(Priority.MEDIUM);
        dto.setDueDate(LocalDateTime.now().plusDays(2));
        dto.setCategory(Category.WORK);
        
        Task taskEntity = new Task();
        Task savedTask = new Task();
        savedTask.setId(1L);
        savedTask.setTitle("Valid Task");
        
        when(modelMapper.map(dto, Task.class)).thenReturn(taskEntity);
        when(taskRepository.save(any())).thenReturn(savedTask);
        when(modelMapper.map(savedTask, TaskResponseDTO.class)).thenReturn(new TaskResponseDTO());
        
        TaskResponseDTO response = taskService.createTask(dto);
        assertNotNull(response);
    }
    
    @Test
    void shouldThrowValidationExceptionWhenDueDateIsPast() {
        TaskRequestDTO dto = new TaskRequestDTO();
        dto.setTitle("Invalid Task");
        dto.setDueDate(LocalDateTime.now().minusDays(1));
        
        assertThrows(ValidationException.class, () -> taskService.createTask(dto));
    }
    
    @Test
    void shouldFetchTaskById() {
        Task task = new Task();
        task.setId(1L);
        
        when(taskRepository.findById(1L)).thenReturn(Optional.of(task));
        when(modelMapper.map(any(), eq(TaskResponseDTO.class))).thenReturn(new TaskResponseDTO());
        
        TaskResponseDTO response = taskService.getTaskById(1L);
        assertNotNull(response);
    }
    
    @Test
    void shouldThrowErrorWhenDeletingCompletedTask() {
        Task completedTask = new Task();
        completedTask.setId(1L);
        completedTask.setCompleted(true);
        
        when(taskRepository.findById(1L)).thenReturn(Optional.of(completedTask));
        
        assertThrows(InvalidTaskStateException.class, () -> taskService.deleteTask(1L));
    }
    
}
```

Esses testes são **testes unitários**, focados **exclusivamente na lógica da camada de serviço**, usando mocks para isolar o comportamento do repositório (`TaskRepository`), do `ModelMapper`, e do `PagedResponseMapper`. Isso garante que estamos testando apenas a lógica da classe `TaskService`, sem dependências externas.

- **Mock e Injeção**
  - Usamos `@Mock` para criar versões simuladas das dependências.
  - Usamos `@InjectMocks` para injetar essas dependências no `TaskService` real que estamos testando.

Métodos testados:

- **shouldCreateTaskWithValidData()**
  - Testa a criação de uma tarefa válida.
  - Verifica se conseguimos criar uma tarefa quando todos os dados são corretos.
  - Usa `when(...).thenReturn(...)` para simular o comportamento do repositório e do mapeador.

- **shouldThrowValidationExceptionWhenDueDateIsPast()**
  - Testa o cenário em que a data limite (`dueDate`) é passada.
  - Espera que uma `ValidationException` seja lançada.
  
- **shouldFetchTaskById()**
  - Testa a busca de uma tarefa pelo seu ID.
  - Garante que o método de busca retorna corretamente a resposta mapeada.

- **shouldThrowErrorWhenDeletingCompletedTask()**
  - Testa a tentativa de deletar uma tarefa já concluída.
  - Espera que seja lançada uma `InvalidTaskStateException`.


É importante que façamos algumas considerações: todos os testes isolam a lógica do `TaskService`, e testamos tanto fluxos positivos quanto negativos (ex: validação de data e deleção de tarefas concluídas).

Esse teste não fornece cobertura completa de nossa aplicação, mas consegue já cumprir o proposto no exercício e fazer uso dos conceitos que vimos anteriormente sobre testes.

Passemos, agora, aos testes funcionais.

#### `TaskControllerTest.java`

```java
package br.ifsp.edu.todo.task;

@SpringBootTest
@AutoConfigureMockMvc
public class TaskControllerTest {
    @Autowired
    private MockMvc mockMvc;
    
    @Autowired
    private ObjectMapper objectMapper;
    
    @Autowired
    private TaskRepository taskRepository;
    
    @BeforeEach
    void cleanDb() {
        taskRepository.deleteAll();
    }
    
    @Test
    void shouldCreateTask() throws Exception {
        TaskRequestDTO dto = new TaskRequestDTO();
        dto.setTitle("My Task");
        dto.setPriority(Priority.HIGH);
        dto.setDueDate(LocalDateTime.now().plusDays(1));
        dto.setCategory(Category.STUDY);
        
        mockMvc.perform(post("/api/tasks").contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(dto))).andExpect(status().isCreated())
                .andExpect(jsonPath("$.title").value("My Task"));
    }
    
    @Test
    void shouldFailWithPastDueDate() throws Exception {
        TaskRequestDTO dto = new TaskRequestDTO();
        dto.setTitle("Invalid Task");
        dto.setPriority(Priority.MEDIUM);
        dto.setDueDate(LocalDateTime.now().minusDays(1));
        dto.setCategory(Category.WORK);
        
        mockMvc.perform(post("/api/tasks").contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(dto))).andExpect(status().isBadRequest());
    }
    
    @Test
    void shouldGetTaskById() throws Exception {
        Task task = new Task();
        task.setTitle("Search Me");
        task.setPriority(Priority.LOW);
        task.setDueDate(LocalDateTime.now().plusDays(2));
        task.setCategory(Category.OTHER);
        task.setCreatedAt(LocalDateTime.now());
        Task saved = taskRepository.save(task);
        
        mockMvc.perform(get("/api/tasks/" + saved.getId())).andExpect(status().isOk())
                .andExpect(jsonPath("$.title").value("Search Me"));
    }
    
    @Test
    void shouldNotDeleteCompletedTask() throws Exception {
        Task task = new Task();
        task.setTitle("Done Task");
        task.setCompleted(true);
        task.setCategory(Category.PERSONAL);
        task.setDueDate(LocalDateTime.now().plusDays(2));
        task.setCreatedAt(LocalDateTime.now());
        Task saved = taskRepository.save(task);
        
        mockMvc.perform(delete("/api/tasks/" + saved.getId())).andExpect(status().isConflict());
    }
    
    @Test
    void shouldListTasksWithPagination() throws Exception {
        for (int i = 0; i < 10; i++) {
            Task task = new Task();
            task.setTitle("Task " + i);
            task.setCategory(Category.WORK);
            task.setDueDate(LocalDateTime.now().plusDays(3));
            task.setCreatedAt(LocalDateTime.now());
            taskRepository.save(task);
        }
        
        mockMvc.perform(get("/api/tasks?page=0&size=5")).andExpect(status().isOk())
                .andExpect(jsonPath("$.content.length()").value(5));
    }
    
    @Test
    void shouldSearchByCategory() throws Exception {
        Task task = new Task();
        task.setTitle("Work Task");
        task.setCategory(Category.WORK);
        task.setDueDate(LocalDateTime.now().plusDays(2));
        task.setCreatedAt(LocalDateTime.now());
        taskRepository.save(task);
        
        mockMvc.perform(get("/api/tasks/search").param("category", "WORK")).andExpect(status().isOk())
                .andExpect(jsonPath("$.content[0].category").value("WORK"));
    }
}
```

Esses testes são **testes de integração funcional**, realizados com **`MockMvc`**. Aqui simulamos requisições HTTP reais, sem subir o servidor, mas envolvendo de fato toda a stack Spring Boot configurada.

- **Configurações**
  - `@SpringBootTest` indica que queremos inicializar o contexto do Spring.
  - `@AutoConfigureMockMvc` habilita o uso do `MockMvc`.
  - `@Autowired` injeta dependências reais como o `MockMvc`, o `ObjectMapper`, e o `TaskRepository`.

- **`@BeforeEach cleanDb()`**
  - Antes de cada teste, limpamos o banco de dados (H2 em memória) para garantir que os testes sejam independentes.

Métodos testados:

- **shouldCreateTask()**
  - Testa se conseguimos criar uma tarefa válida via `POST /api/tasks`.
  - Verifica se a resposta contém o título esperado.

- **shouldFailWithPastDueDate()**
  - Testa se o sistema rejeita a criação de tarefas com data vencida.

- **shouldGetTaskById()**
  - Testa a recuperação de uma tarefa existente por `GET /api/tasks/{id}`.

- **shouldNotDeleteCompletedTask()**
  - Testa a tentativa de excluir uma tarefa concluída, que deve resultar em erro de conflito (`409 Conflict`).

- **shouldListTasksWithPagination()**
  - Testa a paginação criando múltiplas tarefas e garantindo que apenas 5 tarefas sejam retornadas na página solicitada.

- **shouldSearchByCategory()**
  - Testa a busca de tarefas filtradas pela categoria.


É importante considerar que aqui testamos o fluxo **end-to-end** (entrada HTTP → validação → serviço → resposta HTTP). Além disso, há uso de um **banco de dados real:** o `TaskRepository` salva de fato no banco de dados H2 para os testes funcionarem. Por fim, também temos a **validação de respostas:** por meio da utilização do `jsonPath` para validar atributos específicos da resposta JSON.

#### Resumo dos Testes

Podemos resumir as características gerais dos nossos testes tal como mostrado a seguir:

| Tipo de Teste  | Arquivo                 | Foco Principal                          | Dependências Real/Mocadas | Observações                  |
|----------------|--------------------------|-----------------------------------------|----------------------------|-------------------------------|
| Unitário       | `TaskServiceTest.java`    | Lógica isolada do `TaskService`          | Tudo mockado (Mockito)     | Não interage com banco real.  |
| Funcional      | `TaskControllerTest.java` | Fluxo HTTP completo (`MockMvc`)          | Banco de dados real (H2)    | Simula chamadas reais.        |

Ou seja, concluímos a implementação dos testes da nossa aplicação To-Do List com duas abordagens complementares: testes **unitários** na camada de serviço e testes **funcionais** na camada de controle. Essa estratégia permite que tenhamos confiança **tanto no comportamento interno da aplicação quanto no seu comportamento externo** diante dos usuários.

Ao isolar as responsabilidades (usando mocks nos testes de serviço) e ao validar fluxos reais (simulando requisições HTTP com MockMvc), conseguimos cobrir cenários tanto positivos quanto negativos — desde o simples cadastro de uma nova tarefa até situações de erro como tentativas inválidas de exclusão.

É importante reforçar que boas práticas de testes, como vimos aqui, não são apenas uma exigência burocrática dos projetos, mas também uma grande aliada dos desenvolvedores no dia a dia. Ao construir uma base sólida de testes, reduzimos o medo de refatorar, ganhamos agilidade em novas implementações e entregamos um software com qualidade consistente.

Por fim, vale lembrar: **testar não é apenas encontrar defeitos**, mas também **documentar o comportamento esperado** do sistema. Nosso conjunto de testes torna explícito — e validável — como cada funcionalidade deve operar! 🚀

---

## 4. Conclusão

Chegamos ao final da nossa Aula 07 — e, com ela, estruturamos e implementamos uma **API RESTful completa** para gerenciamento de tarefas, aplicando conceitos fundamentais que temos explorado nas últimas semanas!

Nesta aula, você viu na prática:

- Como organizar um projeto em **camadas bem definidas** (model, repository, service, controller, dto, exception, config);
- A **importância da camada Service** para centralizar a lógica de negócio e promover manutenibilidade e testabilidade;
- A construção e utilização de **DTOs** para proteger e padronizar as entradas e saídas da nossa API;
- O uso de **ModelMapper** para facilitar a conversão entre objetos internos e externos;
- A implementação de um **tratamento global de exceções** robusto e padronizado;
- E a criação de **testes automatizados** (unitários e funcionais) para garantir a qualidade e a confiabilidade da aplicação.

Mais do que apenas codar, exercitamos um olhar **crítico e consciente** sobre a arquitetura da aplicação — refletindo sobre *por que* certas escolhas são feitas (como a inclusão da camada de serviço) e *quando* elas de fato agregam valor.

Fica cada vez mais evidente que dominar desenvolvimento de APIs não se resume a conhecer frameworks ou decorar anotações — é sobre **saber estruturar soluções que sejam limpas, compreensíveis, escaláveis e evolutivas**.

É isso que queremos deixar claro: não se trata de aprender a fazer APIs com uso do Spring Boot, se trata de entender os fundamentos. O Java e o Spring são meios, não fim. Todos os conceitos e discussões que temos trazido são transferíveis para outras linguagens e frameworks. 

E tenha sempre em mente: **o melhor código é aquele que é fácil de entender, de testar e de melhorar**. E todos os princípios aplicados nesta aula caminham nesse sentido.

Vamos explorar, na próxima aula, os conceitos autenticação, autorização e segurança de APIs — levando nossas aplicações para um nível mais próximo do mundo real. 🚀

E claro... já sabem o que vamos ter, né? 

---

## Desafios 🏋️‍♂️

Para consolidar ainda mais os conceitos vistos e introduzir **novas práticas**, você deverá implementar as seguintes melhorias no projeto atual como exercício:

### 1. Implementar Autenticação de Usuários

- **Crie uma funcionalidade de registro de novos usuários** (`UserRegistrationDTO`, `UserController`, etc.).
- **Implemente autenticação** via **JWT (JSON Web Token)**:
  - Crie endpoints para login e geração de token (por exemplo, `/api/auth/login`).
  - Configure o projeto para aceitar apenas requisições autenticadas em nossos endpoints de tarefas.
  
Com isso, nossa API se tornará mais realista, pois será necessário fazer login para manipular as tarefas!

☀️ **DICA**
- Assista o vídeo a seguir, da Giuliana Bezerra, para ter uma explicação inicial de como fazer essa implementação. Vídeo **muito** didático e com uso de ferramentas modernas para facilitar nossa vida: [https://www.youtube.com/watch?v=kEJ8a1w4a2Q](https://www.youtube.com/watch?v=kEJ8a1w4a2Q)

### 2. Relacionar Tarefas a Usuários

- **Associe cada tarefa a um usuário específico** no momento da criação.
- **Garanta que apenas o dono da tarefa** possa visualizar, atualizar, concluir ou excluir suas próprias tarefas.
  
Esse exercício reforçará a prática de regras de negócio e de segurança de acesso aos dados, que é fundamental no desenvolvimento de APIs RESTful seguras.

### 3. Implementar Controle de Acesso com Papéis (Roles)

- **Crie roles de usuário**: `USER` e `ADMIN`.
- **Permita que apenas ADMINs possam consultar todas as tarefas** (endpoint `/api/tasks`).
- **Usuários comuns (`USER`) só devem poder gerenciar suas próprias tarefas**.
  
Esse controle de papéis é essencial para implementar autorização baseada em responsabilidades — uma prática indispensável em aplicações reais.