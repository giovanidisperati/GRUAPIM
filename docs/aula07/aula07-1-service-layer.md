---
layout: aula
title: 1. Para que serve a camada Service?
parent: Aula 07 - Construindo o To-do List
nav_order: 1
---

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
