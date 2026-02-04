---
layout: aula
title: "2. Conceitos Essenciais"
parent: Aula 09 - Introdução aos Microsserviços
nav_order: 2
---

# **2. Conceitos Essenciais dos Microsserviços**

Ok! Agora que falamos sobre a diferença entre SOA e Microsserviços, fizemos uma apresentação inicial das ideias e conceitos. Contudo, vamos aprofundá-las e entendê-las de forma mais coerente e abrangente: existem algumas ideias centrais que precisam ser compreendidas ao explorar microsserviços! E dado que alguns aspectos são frequentemente negligenciados, vamos explorá-los para garantir que vocês entendam o que realmente faz os microsserviços funcionarem.

## 2.1. Implantação Independente: A Pedra Angular

A **implantação independente** é a ideia de que podemos fazer uma alteração em um microsserviço, implantá-lo e liberar essa alteração para nossos usuários, **sem ter que implantar nenhum outro microsserviço**. Mais importante, não é apenas o fato de *podermos* fazer isso, mas que essa é a forma como você *gerencia* as implantações no seu sistema; é uma disciplina que você adota como sua abordagem de liberação padrão. Essa é uma ideia simples, mas complexa na execução.

Nesse âmbito, Sam Newman chega a apontar que "se você tirar uma coisa do livro e do conceito de microsserviços em geral", que seja esta: **certifique-se de abraçar o conceito de implantação independente dos seus microsserviços**. Crie o hábito de implantar e liberar alterações em um único microsserviço em produção sem precisar implantar mais nada. A partir disso, muitas coisas boas surgirão.

Para garantir a implantação independente, precisamos garantir que os microsserviços sejam **fracamente acoplados**: devemos ser capazes de mudar um serviço sem ter que mudar mais nada. Isso significa que precisamos de contratos explícitos, bem definidos e estáveis entre os serviços. Algumas escolhas de implementação dificultam isso: compartilhamento de bancos de dados, por exemplo, é especialmente problemático.

## 2.2. Modelados em Torno de um Domínio de Negócio (Domain-Driven Design - DDD)

Técnicas como o **Domain-Driven Design (DDD)** permitem que você estruture seu código para representar melhor o domínio do mundo real em que o software opera. Com arquiteturas de microsserviços, usamos essa mesma ideia para definir nossos limites de serviço. Ao modelar serviços em torno de domínios de negócio — os chamados *bounded contexts* — podemos facilitar o lançamento de novas funcionalidades e recombinar microsserviços de maneiras diferentes para entregar valor ao usuário.

Na próxima aula faremos uma abordagem aprofundada dos conceitos de DDD e como eles se relacionam com a arquitetura de microsserviços, mas de forma geral, a ideia é que implementar uma feature que exige alterações em mais de um microsserviço costuma ser caro: é necessário coordenar times, sincronizar versões e garantir ordem correta nos deploys. Por isso, buscamos formas de organizar nossos serviços para **minimizar mudanças que cruzem fronteiras de contexto**.

Em arquiteturas tradicionais em camadas, como o clássico modelo MVC (apresentação, lógica de negócios e dados), cada camada representa uma separação técnica. Isso facilita alterações locais — por exemplo, só na camada de apresentação —, mas **dificulta mudanças que envolvem regras de negócio**, já que elas frequentemente atravessam múltiplas camadas. Com DDD aplicado a microsserviços, a proposta é **organizar os serviços como "fatias verticais"**, cada uma encapsulando toda a funcionalidade relacionada a um domínio específico. Assim, priorizamos a **coesão da lógica de negócio** dentro de cada serviço, mesmo que isso implique alguma duplicação técnica entre eles.

Assim, ao usar DDD e tornar nossos serviços fatias de ponta a ponta da funcionalidade de negócio, garantimos que nossa arquitetura esteja organizada para tornar as alterações na funcionalidade de negócio o mais eficientes possível. Assim, argumenta-se que, com microsserviços, tomamos a decisão de priorizar a **alta coesão da funcionalidade de negócio** em detrimento da mais alta coesão da funcionalidade técnica.

## 2.3. Donos de Seu Próprio Estado

Uma das ideias que mais geram dúvidas ao trabalhar com microsserviços é a regra de que **cada microsserviço deve ter seu próprio banco de dados**. Isso significa que **não devemos permitir que vários serviços acessem diretamente o mesmo banco de dados**. Em vez disso, se um serviço A precisar de uma informação que pertence ao serviço B, ele deve **fazer uma requisição diretamente ao serviço B** (por exemplo, via API REST), e não acessar o banco de dados de B.

Vamos pensar em um exemplo: imagine um sistema de pedidos com dois microsserviços — `Serviço de Pedidos` e `Serviço de Clientes`. Se o `Serviço de Pedidos` quiser saber o nome de um cliente, ele deve **consultar o `Serviço de Clientes`**, e não tentar acessar diretamente a tabela de clientes no banco de dados de outro serviço. Dessa forma, o `Serviço de Clientes` controla o que está sendo exposto e mantém a liberdade de modificar sua estrutura interna (por exemplo, alterar colunas ou regras de negócio) **sem impactar diretamente outros serviços**, desde que mantenha o contrato externo (a API) estável.

Esse isolamento é importante para que os microsserviços sejam **implantados de forma independente**. Se um serviço compartilha diretamente seus dados com outros, qualquer pequena mudança pode **quebrar funcionalidades** que dependem dele, forçando atualizações em cadeia. Por isso, devemos separar o que é **implementação interna** (que pode mudar à vontade) do que é **contrato público** (que deve mudar com cuidado e previsibilidade).

Esse princípio é muito parecido com o **encapsulamento na programação orientada a objetos (OOP)**. Assim como não deixamos outros objetos acessarem diretamente os atributos internos de uma classe — preferindo métodos públicos controlados —, também não devemos deixar outros serviços acessarem nosso banco de dados diretamente. Expor dados internos é como deixar que outras partes do sistema mexam por dentro da sua classe: o risco de quebrar tudo é grande!

Por isso, a recomendação é clara: **não compartilhe bancos de dados entre microsserviços, exceto em situações extremamente justificadas — e mesmo assim, com muito cuidado**. Cada serviço deve ser uma fatia completa da funcionalidade de negócio, contendo a interface com o usuário (se necessário), a lógica de negócio e os dados. Isso nos dá **alta coesão** dentro de cada serviço e **baixo acoplamento** entre eles, facilitando tanto a manutenção quanto a evolução do sistema como um todo.

## 2.4. Qual o Tamanho Ideal para um microsserviço?

"Quão grande deve ser um microsserviço?" é uma das perguntas mais comuns. Considerando que a palavra "micro" está ali no nome, isso não surpreende. No entanto, quando se trata do que faz os microsserviços funcionarem como um tipo de arquitetura, o conceito de tamanho é, na verdade, um dos aspectos menos interessantes.

Como medir o tamanho? Contando linhas de código? Isso não faz muito sentido. Algo que pode exigir 25 linhas de código em Java pode ser escrito em 10 linhas em Clojure; algumas linguagens são simplesmente mais expressivas que outras. Precisamos, portanto, de outra abordagem.

James Lewis, diretor técnico da Thoughtworks, costuma dizer que "um microsserviço deve ser tão grande quanto a minha cabeça". À primeira vista, isso não parece muito útil. A lógica por trás dessa afirmação é que um microsserviço deve ser mantido em um tamanho que possa ser **facilmente compreendido**. O desafio, claro, é que a capacidade de diferentes pessoas entenderem algo não é sempre a mesma, e você precisará fazer seu próprio julgamento sobre qual tamanho funciona para você. Uma equipe experiente pode gerenciar melhor uma base de código maior do que outra equipe. Talvez seja melhor ler a citação de James como "um microsserviço deve ser tão grande quanto *sua* cabeça".

* **Comentário do professor**: talvez seja ainda melhor ler como "um microsserviço deve ser tão grande quanto a capacidade coletiva da equipe que o mantém". Se um microsserviço começa a se tornar um "nanosserviço", a fragmentação e dispersão de conhecimento é evidente. Se ele é grande suficiente para precisar de manutenção quando _features_ descorrelacionadas mudam, seu contexto foi mal delimitado e ele é grande demais. 🧑‍💻

Nesse sentido, Sam Newman apontar que o mais próximo que chegamos de "tamanho" ter algum significado em termos de microsserviços é algo que Chris Richardson, autor de *Microservice Patterns*, disse uma vez: o objetivo dos microsserviços é ter "uma interface tão pequena quanto possível". Isso se alinha novamente com o conceito de ocultação de informações. Ou seja, em última análise, o conceito de tamanho é altamente contextual. Para quem está começando, é muito mais importante focar em duas coisas principais: primeiro, **quantos microsserviços você consegue lidar?** À medida que você tem mais serviços, a complexidade do seu sistema aumentará. Segundo, **como definir os limites dos microsserviços** para obter o máximo deles, sem que tudo se torne uma bagunça horrivelmente acoplada? É aí que está a resposta.

## 2.5. Flexibilidade: Comprando Opções

James Lewis costuma dizer que "microsserviços compram opções". A escolha do verbo comprar não é por acaso: ela nos lembra que essa flexibilidade tem um custo. Ao adotar uma arquitetura de microsserviços, você passa a ter mais liberdade para lidar com mudanças futuras – como trocar uma tecnologia por outra, escalar partes específicas do sistema, distribuir responsabilidades entre times menores ou aumentar a resiliência de um serviço sem afetar os demais. Mas tudo isso vem acompanhado de mais complexidade, necessidade de monitoramento, orquestração e infraestrutura.

Pense nos microsserviços como uma forma de adquirir *opções futuras*. Por exemplo, imagine que seu sistema de e-commerce cresceu bastante. Se ele for monolítico, escalar apenas o módulo de pagamentos pode ser difícil. Mas, com microsserviços, você poderia rodar múltiplas instâncias só do serviço de pagamentos, reduzindo gargalos. A questão é: você precisa disso agora? Vale a pena pagar esse custo agora ou seria melhor esperar?

É por isso que **Lewis propõe que a adoção de microsserviços não seja vista como um interruptor que você liga de uma vez, mas como um botão de volume que você gira aos poucos**. Você começa com poucos microsserviços, talvez separando apenas os módulos que mais mudam ou que mais precisam escalar. À medida que ganha experiência e percebe benefícios reais, pode ir "aumentando o volume", isto é, modularizando mais partes do sistema. E se perceber que os custos superam os ganhos, pode parar por ali.

Essa abordagem incremental ajuda a evitar surpresas. Se você for direto para uma arquitetura com dezenas de microsserviços, pode acabar enfrentando problemas de comunicação entre serviços, falhas em cascata, dificuldades de testes e deploys mais complexos – tudo isso sem ter estrutura ou equipe preparadas para lidar com essa nova realidade.

## 2.6. Alinhamento entre Arquitetura e Organização (Lei de Conway)

Vamos imaginar a **MusicCorp**, uma loja virtual que vende CDs e que serviu de exemplo no livro que estamos nos baseando (Criando Microsserviços, segunda edição).

O sistema da MusicCorp é construído em uma arquitetura bem tradicional: três camadas — uma interface web para os usuários (UI), um backend com toda a lógica do sistema (monolítico) e um banco de dados relacional para guardar as informações. Até aí, nada muito diferente do que vemos em muitas empresas. O detalhe importante é que **cada uma dessas camadas é gerenciada por uma equipe diferente**: os designers e desenvolvedores de frontend cuidam da UI, os programadores do backend cuidam da lógica de negócio, e os DBAs administram o banco de dados.

Agora pense que a empresa quer adicionar um campo no cadastro para que o cliente possa escolher seu gênero musical favorito. Parece uma mudança simples, certo? Mas, nesse modelo, essa alteração vai exigir trabalho das **três equipes**: a equipe de UI precisa criar o novo campo visual, a equipe do backend tem que tratar o dado e enviá-lo ao banco, e a equipe de banco de dados precisa ajustar a estrutura para armazenar a nova informação. Além disso, tudo isso precisa ser coordenado na ordem certa e implantado junto, senão o sistema quebra. O que era uma pequena melhoria se transforma em uma operação complexa.

Essa forma de organizar os sistemas reflete algo mais profundo: **como organizamos as nossas equipes**. Isso é o que a famosa **Lei de Conway** nos ensina: *"as organizações tendem a criar sistemas que são cópias das suas próprias estruturas de comunicação"*. Ou seja, se temos uma equipe para cada camada (UI, backend, banco), nosso sistema também tende a ser dividido assim.

No passado, esse modelo fazia sentido. As empresas de TI agrupavam pessoas por especialidade: todos os DBAs juntos, todos os devs Java juntos, e assim por diante. Era natural que os sistemas também fossem construídos por camadas, refletindo essa divisão. É por isso que a arquitetura em três camadas se tornou tão comum.

Mas **os tempos mudaram**. Hoje queremos entregar software mais rápido, com menos dependência entre equipes. Começamos a formar **equipes polivalentes**, compostas por pessoas de diferentes áreas que conseguem, juntas, cuidar de uma funcionalidade do começo ao fim — do banco à interface. O objetivo é reduzir a quantidade de passagens de tarefa entre times e agilizar o desenvolvimento.

A maioria das mudanças que fazemos em um sistema tem a ver com **funcionalidade de negócio**. Só que, na arquitetura em camadas, essa funcionalidade está **espalhada** por todo o sistema: uma parte na UI, outra no backend, outra no banco. Isso aumenta as chances de qualquer mudança pequena gerar impacto em várias partes, exigindo coordenação entre vários times.

Para resolver isso, é melhor organizarmos nosso código (e nossas equipes) em torno de **funcionalidades de negócio** e não de tecnologias. Isso significa que cada equipe cuida de uma parte específica do domínio da empresa — por exemplo, uma equipe só para o perfil do cliente. Essa equipe teria total autonomia para mudar o que for necessário no cadastro de clientes, incluindo, por exemplo, adicionar o campo de gênero musical favorito.

Essa forma de organização é chamada de **arquitetura vertical por domínio de negócio**. Em vez de separarmos o sistema em camadas horizontais (UI, backend, banco), nós o dividimos em **linhas de negócio verticais**, onde cada time cuida de tudo relacionado a uma parte do sistema. Nesse exemplo, a equipe de perfil do cliente poderia até manter um microsserviço próprio, com a lógica, a UI e o banco de dados necessários apenas para aquilo.

Esse modelo traz mais agilidade e reduz o atrito entre times. O livro *Team Topologies* chama isso de **equipe alinhada ao fluxo** (stream-aligned team): times focados em um fluxo de trabalho específico, com autonomia para entregar valor ao usuário **de ponta a ponta**, sem precisar ficar dependendo de outros grupos para cada mudança.
