---
layout: aula
title: 5. Desafio - AcademiaDev
parent: Aula 02 - Lambdas, Streams e Optionals
nav_order: 5
---

## **Concluindo...**

Entender Lambdas, Streams e Optionals eleva seu código Java a um novo patamar de clareza e eficiência. Essas ferramentas representam uma mudança de paradigma para um estilo mais declarativo e funcional.

Com essa base sólida, estamos agora mais preparados para construir os componentes de nossas APIS com Spring Boot. Você verá como esses recursos tornam o código em Controllers, Services e Repositories muito mais enxuto e expressivo.

Mas antes, vamos exercitar esse conceitos com...

---

## **Exercícios**

Elabore os exercícios e o desafio abaixo.

A entrega deverá ser feita individualmente via Moodle.

Crie uma classe `Produto` com os atributos: `nome` (String), `preco` (double) e `categoria` (String). As categorias devem ser, pelo menos, "Eletrônicos" e "Livros". Em uma classe de teste, crie uma `List<Produto>` com pelo menos 8 produtos de diferentes categorias e preços, incluindo alguns com a mesma categoria. Após isso:

- a. Use `forEach` e uma estrutura `if` tradicional para imprimir o nome de todos os produtos da categoria "Eletrônicos". Em seguida, refaça o mesmo exercício usando `stream()` e a operação `filter()`.

- b. Crie uma nova lista contendo apenas os preços de todos os produtos cujo preço seja maior que 500.0. Use as operações `filter()` e `map()`. Imprima a lista de preços.

- c. Calcule o valor total do estoque de produtos da categoria "Livros". Use `filter()` para selecionar os produtos da categoria correta e, em seguida, use `mapToDouble()` e `sum()` para calcular o total.

- d. Escreva um método `buscarProdutoPorNome(List<Produto> produtos, String nome)` que retorna um `Optional<Produto>`. Use a Stream API (`filter` e `findFirst`).

- e. No seu método `main`, chame o `buscarProdutoPorNome`: Primeiro, com um nome de produto que existe. Use `ifPresent()` para imprimir os detalhes do produto; Depois, com um nome que não existe. Use `orElseThrow()` para lançar uma `RuntimeException` com a mensagem "Produto não encontrado!".

- f.  Crie um `stream` a partir da sua lista de produtos e use `.map()` para obter uma `List<String>` contendo apenas os nomes dos produtos. Primeiro, faça isso com uma expressão lambda (`p -> p.getNome()`) e depois refatore para usar uma referência de método (`Produto::getNome`).

## **Desafio - Plataforma de Cursos Online "AcademiaDev"**

A startup de tecnologia educacional **AcademiaDev** está lançando sua nova plataforma de cursos online. Seu modelo de negócio é baseado em um sistema de assinaturas que dá aos alunos acesso a um catálogo de cursos de alta qualidade, focados no desenvolvimento de software.

Para validar sua proposta de negócio, a AcademiaDev contratou sua equipe para desenvolver um protótipo inicial da aplicação. Por um infortúnio do destino, parte de sua equipe foi hospitalizada após a ingestão de dezenas de torresmos no Bar do Bigode ao comemorar mais uma vitória do Corinthians sobre o Palmeiras.

Dessa forma, cabe a você, o(a) único(a) desenvolvedor(a) _geração saúde_ da equipe, trabalhar na implementação desse protótipo inicial utilizando todos os conceitos que foram relembrados na Aula 01 e vistos na Aula 02. Nesse protótipo os requisitos são focados na implementação da lógica de negócio principal, utilizando um conjunto de dados já existente.

Para focar na lógica principal da aplicação, **não será necessário implementar as funcionalidades de CRUD completas**. Em vez disso, sua aplicação deve iniciar com um conjunto de dados pré-cadastrado. Crie uma classe utilitária (ex: `InitialData`) que popule suas estruturas de dados em memória assim que a aplicação iniciar. Ou seja, não é necessário criar um CRUD completo de `Courses` ou `Users` - apenas o suficiente para validar a lógica de negócio.

Nesse sentido, o protótipo deverá implementar funcionalidades para:

* Gerenciamento do catálogo de `Courses` (cursos);
* Gerenciamento de `Users` (usuários) e seus respectivos planos de assinatura;
* Sistema de `Enrollments` (matrículas) e acompanhamento de progresso dos alunos;
* Um sistema de fila para atendimento de `Support Tickets`;
* Geração de relatórios e exportação de dados da plataforma.

A equipe de analistas da sua empresa, a partir de reuniões com os fundadores da AcademiaDev, já havia determinado os requisitos a seguir.

#### **Requisitos Funcionais**

**1) Gerenciamento do Catálogo de `Courses`**
Os cursos da plataforma devem possuir as seguintes características:
* `title` e `description`. O `title` de cada curso deve ser único na plataforma.
* `instructorName`.
* `durationInHours` (carga horária).
* `difficultyLevel`, que pode ser `BEGINNER`, `INTERMEDIATE` ou `ADVANCED`.
* `status`, que pode ser `ACTIVE` ou `INACTIVE`. Um curso com status `INACTIVE` não pode receber novas matrículas.

**2) `Users` e `Subscription Plans`**
Os usuários da plataforma podem pertencer a dois perfis:
* `Admin`: Possui `name` e `email`. Tem permissão para gerenciar o catálogo de cursos e usuários.
* `Student`: Possui `name`, `email` e um `subscriptionPlan`.
* O `email` de cada usuário (seja `Student` ou `Admin`) deve ser único.

Os planos de assinatura (`SubscriptionPlan`) disponíveis para alunos são:
* `BasicPlan`: Permite que o aluno esteja matriculado em, no máximo, **3 cursos ativos** simultaneamente.
* `PremiumPlan`: Permite que o aluno se matricule em um **número ilimitado** de cursos.

**3) Sistema de `Enrollments` e `Progress`**
* Um aluno só pode se matricular em um curso (`Course`) se o seu plano de assinatura permitir e se o curso estiver com o status `ACTIVE`.
* Ao se matricular em um curso, o progresso (`progress`) do aluno é iniciado em 0%.
* O sistema deve permitir que um aluno atualize o seu percentual de progresso (0 a 100) em qualquer curso no qual esteja matriculado.

**4) Fila de Suporte ao `User`**
* Qualquer usuário da plataforma pode abrir um `SupportTicket`, contendo um `title` e uma `message`.
* Os tickets devem ser armazenados em uma fila de atendimento para serem processados pela equipe de administradores. O atendimento deve seguir rigorosamente a **ordem de chegada** (FIFO - First-In, First-Out).

**5) Relatórios e Análises da Plataforma**
O sistema deve ser capaz de gerar as seguintes informações analíticas:
* Uma lista de cursos pertencentes a um determinado `difficultyLevel`, **ordenada alfabeticamente** pelo `title` do curso.
* Uma relação de todos os **instrutores únicos** que ministram cursos ativos na plataforma, sem nomes repetidos.
* Um relatório que agrupe os alunos de acordo com seu `subscriptionPlan`.
* O cálculo da **média geral de `progress`**, considerando todas as matrículas de todos os alunos.
* A identificação do **aluno com o maior número de matrículas** ativas.
* **Exportação de Dados para CSV:** A plataforma precisa de uma funcionalidade de exportação que permita a um administrador gerar um arquivo CSV a partir de qualquer lista de dados (seja de `Courses`, `Students`, etc.). O administrador deve poder **escolher dinamicamente quais colunas (campos) quer no arquivo** no momento da exportação. **Nesse momento, não é necessário gerar um arquivo `.csv` físico: a função deve apenas retornar e exibir a estrutura do CSV formatada como uma `String` no console.**

#### **Funcionalidades da Aplicação (Interface de Linha de Comando)**

A aplicação deve ser desenvolvida como um sistema de linha de comando, com um menu que ofereça, no mínimo, as seguintes funcionalidades:

1.  **Operações de Administrador (`Admin`)**
    * **Gerenciar Status de Cursos:** Ativar/inativar cursos existentes (não precisa implementar CRUD completo).
    * **Gerenciar Planos de Alunos:** Alterar o plano de assinatura de um aluno existente.
    * **Atender Tickets de Suporte:** Processar tickets da fila em ordem FIFO.
    * **Gerar Relatórios e Análises:** Acessar todos os relatórios da plataforma.
    * **Exportar Dados:** Gerar a `String` CSV com colunas selecionáveis dinamicamente.

2.  **Operações do Aluno (`Student`)**
    * **Matricular-se em Curso:** Desde que o plano permita e o curso esteja `ACTIVE`.
    * **Consultar Matrículas:** Ver todos os cursos em que está matriculado e seu progresso.
    * **Atualizar Progresso:** Modificar o percentual de conclusão de um curso.
    * **Cancelar Matrícula:** Remover-se de um curso (libera vaga para planos básicos).

3.  **Operações Gerais (Qualquer Usuário):**
    * **Consultar Catálogo de Cursos:** Listar cursos ativos disponíveis.
    * **Abrir Ticket de Suporte:** Criar um novo ticket para a fila de atendimento.
    * **Autenticação Simples:** Sistema básico de login por email para distinguir entre Admin e Student. Não é necessário que o usuário tenha senha, apenas o e-mail é suficiente para autenticação. Você poderá elaborar duas telas distintas ou simplesmente atribuir papeis aos usuários e fazer a verificação de permissão.

#### **Lógica de Negócio e Regras**

A partir dos requisitos acima, foram destacadas as seguintes regras de negócio que devem ser consideradas no processo de implementação.

**Sistema de Matrículas (`Enrollments`)**
* Um `Student` só pode se matricular em um `Course` se:
  - Seu plano de assinatura permitir (BasicPlan: máximo 3 matrículas ativas).
  - O curso estiver com status `ACTIVE`.
  - Não estiver já matriculado no mesmo curso.
* O progresso inicia em 0% e pode ser atualizado de 0 a 100%.

**Fila de Suporte**
* Tickets são processados em ordem FIFO (First-In, First-Out).
* Qualquer usuário pode abrir tickets, mas apenas `Admin` pode processá-los.

**Relatórios e Análises**
O sistema deve gerar as seguintes informações utilizando **Stream API**:
* Lista de cursos por `difficultyLevel`, ordenada alfabeticamente.
* Relação de instrutores únicos que ministram cursos `ACTIVE`.
* Agrupamento de alunos por `subscriptionPlan`.
* Média geral de progresso de todas as matrículas.
* Aluno com maior número de matrículas ativas (retorno: `Optional<Student>`).

#### **Requisitos de Implementação e Ferramentas**

Para este protótipo, as seguintes ferramentas e abordagens devem ser utilizadas:

1.  **Persistência em Memória:** Toda a persistência de dados deve ser simulada em memória utilizando as **Collections do Java**. Não é necessário usar um banco de dados real.
2.  **Estruturas de Dados Específicas:**
    * Para garantir a unicidade e a busca eficiente de `Courses` por `title` e `Users` por `email`, utilize a interface `Map`.
    * Para a listagem de `instructors` únicos no relatório, utilize a interface `Set`.
    * Para a fila de `Support Tickets`, utilize uma implementação da interface `Queue` (como `LinkedList` ou `ArrayDeque`) para garantir o comportamento FIFO.
3.  **Programação Funcional com Java 8+:**
    * Todos os relatórios e análises descritos devem ser implementados utilizando a **Stream API** e **Expressões Lambda**.
    * Reforçando: a função que busca o aluno com mais matrículas deve obrigatoriamente retornar um `Optional<Student>` para tratar o caso de não haver alunos.
4.  **Reflection e Anotações:**
    * A funcionalidade de **Exportação de Dados para CSV** deve ser implementada de forma genérica. Crie uma classe utilitária `GenericCsvExporter` que utilize **Reflection** para ler os campos de qualquer lista de objetos e gerar uma String no formato CSV.
5. **Tratamento de exceções:**
    * Operações que violem regras de negócio (por exemplo, a tentativa de matricular um `Student` com `BasicPlan` no quarto curso, ou em um curso `INACTIVE`) devem lançar exceções customizadas (ex: `EnrollmentException`). A interface com o usuário deve capturar essas exceções e exibir uma mensagem de erro amigável.
6. **Modelagem implícita:**
    * Você notará que o conceito de "matrícula" (`Enrollment`) é central para o sistema, pois conecta `Student` e `Course` e armazena o `progress`. No entanto, os atributos e a estrutura exata dessa classe não foram detalhados pela sua equipe. Você deverá modelar essa classe de associação, definindo os campos e métodos necessários para que todas as regras de negócio de matrícula, cancelamento e progresso funcionem corretamente. Sinta-se à vontade para criar outras classes de suporte que julgar necessárias para uma boa organização e para cumprir os requisitos. Caso perceba alguma lacuna, você poderá completá-la como julgar melhor, já que o restante de sua equipe está hospitalizada.

#### **Modelagem da Solução**

Para implementar a aplicação, utilize conceitos da Programação Orientada a Objetos (POO) com Java, incluindo:

* **Encapsulamento** - Para garantir que os dados dos objetos sejam acessados de forma segura e controlada.
* **Herança** - Para modelar os diferentes tipos de usuários, se julgar apropriado.
* **Polimorfismo** - Para permitir o tratamento genérico de diferentes entidades, como os planos de assinatura.
* **Classes Abstratas e Interfaces** - Para estruturar a hierarquia de suas classes de forma coesa e flexível.

Além da implementação do código, elabore um **Diagrama de Classes UML** que represente a estrutura do sistema, demonstrando as relações entre as classes (`User`, `Student`, `Admin`, `Course`, `SubscriptionPlan`, `Enrollment`, etc.). Entregue juntamente com o diagrama uma **justificativa para suas escolhas de design** do protótipo.

Esse exercício poderá ser feito em duplas.

**Mãos à obra! ⚒️