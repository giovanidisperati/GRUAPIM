---
layout: aula
title: "3. E o Monólito?"
parent: Aula 09 - Introdução aos Microsserviços
nav_order: 3
---

# **3. E o Monólito?**

Microsserviços são frequentemente discutidos como uma alternativa à arquitetura monolítica. Para distinguir mais claramente a arquitetura de microsserviços, é importante discutir o que se entende por monólitos. Um **monólito** é primariamente definido como uma **unidade de implantação**: quando toda a funcionalidade de um sistema deve ser implantada em conjunto, é considerado um monólito.

## 3.1. O Monólito de Processo Único

O exemplo mais comum de monólito é um sistema onde todo o código é implantado como um **único processo**. Você pode ter múltiplas instâncias desse processo por razões de robustez ou escalabilidade, mas fundamentalmente todo o código está empacotado em um único processo. Na realidade, esses sistemas de processo único quase sempre acabam lendo ou armazenando dados em um banco de dados, ou apresentando informações para aplicações web ou móveis, tornando-os sistemas distribuídos simples por si sós. Um monólito clássico de processo único pode fazer sentido para muitas organizações, especialmente as menores.

## 3.2. O Monólito Modular

Dentro do universo das arquiteturas monolíticas, existe uma variação chamada **monólito modular**. O **monólito modular** é uma variação da arquitetura monolítica tradicional que aplica, de forma deliberada, os princípios de **modularidade** propostos por **David Parnas** no artigo clássico *"On the Criteria To Be Used in Decomposing Systems into Modules"*. Em vez de dividir o sistema com base nas etapas de execução (como "entrada, processamento, saída"), Parnas defendeu que deveríamos organizar os módulos com base em **decisões que podem mudar no futuro**, ocultando essas decisões dentro dos próprios módulos. Esse princípio ficou conhecido como **ocultamento da informação (information hiding)**.

Dessa forma, ao invés de termos um único bloco de código todo misturado, a aplicação é dividida em **módulos separados** — como se fosse um prédio com vários apartamentos independentes, mas todos ainda fazendo parte do mesmo edifício. Na prática, isso significa que diferentes partes do sistema (por exemplo, módulo de pagamentos, módulo de usuários, módulo de catálogo) são desenvolvidas separadamente, com regras e responsabilidades bem definidas, mas **ainda são empacotadas e implantadas juntas** como um único sistema.

Essa abordagem pode funcionar muito bem para muitas empresas, especialmente as que ainda não têm maturidade ou necessidade de investir na complexidade dos microsserviços. Se os **limites entre os módulos** forem bem definidos e respeitados, é possível que várias equipes trabalhem **em paralelo** em partes diferentes do sistema, sem se atrapalhar, o que traz muitos dos benefícios da modularidade sem os custos operacionais da arquitetura distribuída.

Um bom exemplo é o **Shopify**, que por muito tempo escolheu não adotar microsserviços em larga escala. Em vez disso, estruturou seu sistema como um grande monólito com módulos bem organizados, mantendo a simplicidade na hora de fazer deploys e testes — tudo rodando num único processo, mas com divisão interna clara.

Porém, essa abordagem também tem seus desafios. Um dos principais é que, embora o código esteja modularizado, o **banco de dados costuma continuar como uma estrutura única** e compartilhada. Ou seja, mesmo que você tenha um módulo só de pedidos, ele ainda acessa tabelas que também são usadas por outros módulos. Isso cria um forte acoplamento entre as partes, dificultando futuras tentativas de extrair os módulos para transformá-los em microsserviços independentes. Algumas equipes tentam resolver esse problema **modularizando também o banco de dados**, ou seja, criando conjuntos de tabelas separados por módulo, cada um com sua própria lógica de negócio. Isso ajuda a reduzir o acoplamento e deixa o caminho mais aberto para, no futuro, migrar cada módulo para um microsserviço de forma mais tranquila. Mas vale lembrar que essa abordagem exige muito mais disciplina técnica e governança de dados.

O **monólito modular**, portanto, é como um "meio-termo" entre o monólito tradicional e os microsserviços. Ele pode ser uma escolha estratégica muito boa, especialmente para organizações que querem escalar o desenvolvimento sem lidar desde já com toda a complexidade da arquitetura distribuída.

## 3.3. O Monólito Distribuído

O que chamamos de **monólito distribuído** é um sistema que, à primeira vista, parece *moderno* por ser composto por vários serviços independentes. No entanto, na prática, **todos esses serviços precisam ser implantados juntos**, como se fossem partes inseparáveis de um único sistema. Ou seja, mesmo que você tenha dividido seu código em vários projetos ou repositórios, ainda precisa subir tudo de uma vez — o que elimina boa parte das vantagens da arquitetura distribuída.

Esse tipo de sistema costuma surgir quando as equipes tentam migrar de um monólito tradicional para microsserviços, mas **não mudam de fato a forma como os serviços se relacionam**. Por exemplo: imagine um sistema com serviços como `PedidoService`, `EstoqueService` e `PagamentoService`. Em vez de funcionarem de forma independente, eles estão tão interligados que uma pequena alteração no serviço de estoque obriga a reimplantação dos outros dois, porque tudo está sincronizado de forma rígida — talvez com chamadas diretas, bancos de dados compartilhados ou dependências circulares.

O resultado é um sistema com **todas as dificuldades de uma arquitetura distribuída** (como complexidade de comunicação, monitoramento, tolerância a falhas) **sem colher os benefícios** (como deploys isolados, escalabilidade independente ou autonomia entre times).

Além disso, as **dependências entre os serviços são tão fortes** que qualquer mudança em um pedaço do sistema **acaba afetando outros módulos** inesperadamente. Um ajuste simples — como mudar o cálculo do frete — pode quebrar a lógica de emissão de nota fiscal, porque os dois serviços compartilham regras, dados ou chamadas internas de forma pouco controlada.

Esse acoplamento excessivo impede que as equipes trabalhem de forma independente e aumenta o risco de erro. Em vez de facilitar a evolução do sistema, a arquitetura distribuída **passa a gerar medo de mudanças**, tornando o sistema mais lento e difícil de manter.

Para evitar cair nesse cenário, é importante não apenas dividir o sistema em serviços, mas garantir que **cada serviço seja realmente autônomo**. Isso inclui:

* Definir claramente o que cada serviço faz.
* Evitar que serviços compartilhem banco de dados diretamente.
* Criar APIs bem definidas para comunicação entre serviços.
* Minimizar as dependências ocultas (como regras de negócio duplicadas ou implícitas).

Em resumo, o **monólito distribuído parece moderno, mas funciona como um monólito disfarçado**. Ele traz o custo da complexidade sem entregar os benefícios da separação. Por isso, ao adotar uma arquitetura distribuída, é essencial garantir que os serviços sejam verdadeiramente independentes — tanto na lógica quanto na operação.


## Em resumo...

Podemos sintetizar os diferentes tipos de monólitos da seguinte forma:

| **Característica**                    | **Monólito Tradicional**                                          | **Monólito Modular**                                                                  | **Monólito Distribuído**                                                                       |
| ------------------------------------- | ----------------------------------------------------------------- | ------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| **Organização do código**             | Mistura de responsabilidades, sem separação clara entre módulos   | Dividido em módulos coesos com limites definidos                                      | Dividido em serviços aparentes, mas com forte acoplamento entre eles                           |
| **Implantação**                       | Um único artefato implantado junto                                | Um único artefato implantado junto (apesar da separação interna)                      | Vários serviços, mas que precisam ser implantados juntos                                       |
| **Autonomia entre partes do sistema** | Baixa                                                             | Moderada – os módulos têm autonomia lógica, mas não de runtime                        | Baixa – os serviços dependem uns dos outros para funcionar e evoluir                           |
| **Facilidade de manutenção**          | Baixa – mudanças em uma parte podem afetar todo o sistema         | Boa – se os módulos forem bem desenhados, mudanças locais têm impacto mais controlado | Ruim – mudanças locais podem ter impacto inesperado em outros serviços                         |
| **Escalabilidade**                    | Escala como um todo (não é possível escalar partes isoladas)      | Escala como um todo, mas é possível otimizar desempenho internamente por módulo       | Em teoria escalável por serviço, mas na prática limitado pelo acoplamento                      |
| **Complexidade de desenvolvimento**   | Baixa – fácil de começar                                          | Moderada – exige organização e disciplina modular                                     | Alta – complexidade de sistemas distribuídos, mas sem os benefícios reais                      |
| **Complexidade operacional**          | Baixa – simples de testar, implantar e monitorar                  | Baixa a moderada – depende da modularidade interna                                    | Alta – exige monitoramento, coordenação de versões e tolerância a falhas entre serviços        |
| **Coesão funcional**                  | Baixa – responsabilidades frequentemente espalhadas entre camadas | Alta – cada módulo tende a se concentrar em uma única responsabilidade                | Baixa – serviços podem se cruzar em funcionalidades, criando dependências implícitas           |
| **Acoplamento entre partes**          | Alto                                                              | Baixo a moderado, se bem projetado                                                    | Alto – dependência forte entre serviços, com comunicação acoplada e compartilhamento de estado |
| **Exemplo típico**                    | Aplicações legadas com separação por camadas (UI, lógica, banco)  | Shopify, projetos modernos que optam por monólito bem estruturado                     | Tentativas falhas de adotar microsserviços sem autonomia real de serviços                      |

## 3.4. Monólitos e a Contenção na Entrega 
Quando muitas pessoas trabalham no mesmo sistema, é comum que comecem a **atrapalhar o trabalho umas das outras**. Por exemplo: dois desenvolvedores podem querer alterar o mesmo trecho de código ao mesmo tempo, ou equipes diferentes podem precisar fazer deploys em momentos distintos — uma pronta para liberar uma nova funcionalidade, outra querendo adiar uma entrega por ainda estar testando algo. Além disso, pode surgir confusão sobre **quem é responsável por cada parte do sistema**, dificultando a tomada de decisões e a coordenação do trabalho.

Esse tipo de situação é conhecido como **contenção na entrega (delivery contention)** — quando múltiplas equipes ou pessoas competem por modificar, testar ou implantar partes do mesmo sistema ao mesmo tempo. Isso gera atrasos, conflitos e, muitas vezes, retrabalho.

É importante destacar que **esse problema pode acontecer em qualquer tipo de arquitetura**. Ter um monólito **não significa automaticamente** que você terá esse problema, assim como usar microsserviços **não garante** que ele desaparecerá. No entanto, os **microsserviços oferecem uma estrutura mais clara de separação entre as partes do sistema**, com limites técnicos e organizacionais bem definidos. Cada equipe pode ser dona de um serviço específico, com mais autonomia para decidir quando e como fazer alterações ou implantar novas versões, sem depender tanto dos outros times.

Essa separação ajuda a reduzir a contenção na entrega, pois **limita o número de pessoas que atuam diretamente no mesmo código** e **permite que as equipes operem de forma mais independente**. Em resumo, os microsserviços não eliminam o problema, mas **criam condições melhores para lidar com ele** à medida que o sistema e as equipes crescem.

## 3.5. Vantagens dos Monólitos 

Embora os microsserviços estejam em alta, **monólitos bem estruturados** — como os de **processo único** ou os chamados **monólitos modulares** — ainda têm **muitas vantagens relevantes**, especialmente para equipes menores ou projetos em estágio inicial. Uma das maiores vantagens é a **simplicidade na implantação**: todo o sistema é empacotado e executado como uma única aplicação. Isso evita vários problemas típicos de sistemas distribuídos, como falhas na comunicação entre serviços, necessidade de orquestração ou complexidades no versionamento de APIs internas.

Na prática, isso significa que **o fluxo de trabalho do desenvolvedor pode ser muito mais simples**. Por exemplo, para testar uma nova funcionalidade, o desenvolvedor só precisa rodar um único serviço localmente. Já em uma arquitetura distribuída, ele possivelmente teria que subir múltiplos serviços, configurar dependências e simular integrações. Além disso, **atividades como monitoramento, depuração (debug) e testes de ponta a ponta** tendem a ser mais diretas e fáceis em um monólito, porque tudo está no mesmo lugar — inclusive os logs, os dados e o contexto da execução.

Outra vantagem importante é a **facilidade na reutilização de código**. Dentro de um monólito, é comum que múltiplos módulos compartilhem funções, classes ou bibliotecas sem precisar publicar pacotes separados, criar contratos entre serviços ou lidar com compatibilidade de versões.

Apesar disso, **muita gente passou a enxergar o monólito como algo ultrapassado**, como se fosse sinônimo de "código legado" ou "erro de projeto". Esse preconceito pode levar equipes a adotar microsserviços antes da hora, enfrentando toda a complexidade de sistemas distribuídos sem necessidade.

Mas, na verdade, **usar um monólito é uma escolha técnica válida e, em muitos casos, a melhor escolha**. O autor inclusive afirma que, em sua opinião, a **arquitetura monolítica deveria ser o ponto de partida padrão** para a maioria dos projetos. Ou seja, ele parte do princípio de que o monólito é o caminho mais sensato, e só consideraria microsserviços **se houver motivos concretos para isso** — como escala organizacional, autonomia de equipes ou demandas técnicas específicas.

Essa visão ajuda a evitar o erro comum de "usar microsserviços por moda" e reforça a ideia de que **a arquitetura deve servir às necessidades do sistema e da equipe — e não o contrário**.


