---
layout: aula
title: 3. Mapeamento objeto relacional
parent: Aula 05 - Aprimorando nossas APIs Rest
nav_order: 3
---
## 3. 📖 Implementação de Banco de Dados

Até este ponto da aplicação, utilizamos um banco de dados em memória, geralmente o H2, que é configurado automaticamente pelo Spring Boot durante o desenvolvimento. Essa abordagem é útil para testes rápidos, pois dispensa instalação e configuração de servidores de banco, mas os dados são perdidos ao reiniciar a aplicação, e não é adequada para ambientes de produção ou para testes mais realistas.

Agora, vamos configurar a aplicação para utilizar um banco de dados relacional real, como o MySQL ou o PostgreSQL, garantindo persistência de dados entre execuções, maior controle sobre o ambiente e maior proximidade com cenários do mundo real.

### 🔧 Passo 1 – Adicionando a dependência no `pom.xml`

A primeira etapa é incluir a dependência do driver JDBC correspondente ao banco de dados escolhido. Exemplo com **MySQL**:

```xml
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.33</version>
</dependency>
```

Se preferir usar **PostgreSQL**, a dependência seria:

```xml
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>42.5.1</version>
</dependency>
```

### ⚙️ Passo 2 – Configurando a conexão no `application.properties`

Em seguida, precisamos fornecer ao Spring os dados de conexão do banco de dados real. Supondo o uso de MySQL, configure o arquivo `src/main/resources/application.properties` com:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/contacts_db
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# Configuração do JPA
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect
```

Vamos entender essa configuração acima linha por linha.

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/contacts_db
```
Essa linha define a **URL de conexão JDBC** com o banco de dados MySQL. No exemplo, estamos nos conectando a um banco chamado `contacts_db` hospedado no próprio computador (`localhost`) na porta padrão do MySQL (`3306`).

```properties
spring.datasource.username=root
spring.datasource.password=123456
```
Essas duas linhas especificam as **credenciais de acesso** ao banco de dados — no caso, o usuário `root` e a senha `123456`. Esses dados devem ser ajustados conforme a configuração do seu ambiente.

```properties
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```
Aqui, indicamos explicitamente a **classe do driver JDBC** que será usada para se comunicar com o banco MySQL. Essa linha nem sempre é obrigatória, pois o Spring Boot costuma inferi-la automaticamente a partir da URL de conexão, mas é uma boa prática incluí-la para evitar ambiguidades.


```properties
spring.jpa.hibernate.ddl-auto=update
```
Essa opção instrui o **Hibernate** (a implementação de JPA utilizada pelo Spring Boot) a **atualizar automaticamente o esquema do banco** com base nas entidades da aplicação. Ou seja, se você adicionar um campo ou uma nova entidade, o banco será modificado para refletir isso na próxima inicialização. Para ambientes de produção, recomenda-se evitar `update` e usar `validate`, `none` ou controlar via ferramentas de versionamento de schema como **Flyway** ou **Liquibase**.

```properties
spring.jpa.show-sql=true
```
Essa linha ativa o **log das instruções SQL** executadas pelo Hibernate no console. Isso ajuda bastante durante o desenvolvimento, pois permite verificar o que está sendo enviado ao banco de dados.

```properties
spring.jpa.properties.hibernate.format_sql=true
```
Aqui, ativamos a **formatação das queries SQL** no console, deixando-as mais legíveis (com quebras de linha e indentação).

```properties
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect
```
Por fim, essa linha define o **dialeto SQL** que o Hibernate deve usar. O dialeto adapta a geração de queries de acordo com as particularidades do banco (sintaxe, funções, tipos de dados etc). No caso, estamos dizendo para o Hibernate usar o **dialeto específico do MySQL 8**.

Essa configuração completa conecta o Spring Boot a um banco de dados relacional real, garante persistência de dados e permite que o Hibernate cuide do mapeamento entre as classes Java e as tabelas no banco.

**Atenção! Essa configuração pode mudar dependendo da versão de seu banco de dados e do SO de sua máquina‼️**

No meu notebook (OS X) a configuração ficou da seguinte forma:

```
# MySQL Database Configuration
spring.datasource.url=jdbc:mysql://localhost:3306/contacts_db?useSSL=false&serverTimezone=UTC
spring.datasource.username=user
spring.datasource.password=password

# Hibernate JPA Configuration
spring.jpa.hibernate.ddl-auto=create
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect
```

Sem a flag `useSSL=false&serverTimezone=UTC` o mariadb não permitia a conexão 🫠

Contudo, testando no PC com Linux via WSL a configuração mostrada acima deu certo. Ou seja: podem existir pequenas diferenças. Utilizem esses eventuais problemas como oportunidade de pesquisa e aprendizagem!

Feita essa ressalva, vamos falar um pouco sobre um tema que mencionamos acima: migrations!

#### 🔄 O que são **Migrations**?

Quando utilizamos a configuração `spring.jpa.hibernate.ddl-auto=update`, como visto anteriormente, o Hibernate assume a responsabilidade de **criar e alterar automaticamente o esquema do banco de dados** com base nas entidades da aplicação. Essa abordagem é conveniente durante o desenvolvimento, mas pode ser **arriscada em ambientes de produção**, onde uma pequena alteração no código pode gerar mudanças indesejadas (e até destrutivas) no banco.

É nesse contexto que entram as **migrations** — ou **migrações de banco de dados**.

Uma *migration* é, basicamente, um **registro controlado e versionado de alterações estruturais no banco de dados**, como criação de tabelas, adição de colunas, criação de índices, entre outros. Essas alterações são descritas em arquivos e aplicadas **de forma previsível e segura** a cada nova versão do sistema.

Em vez de deixar o Hibernate alterar o banco automaticamente, com migrations a equipe de desenvolvimento define **exatamente o que deve ser alterado**, em qual ordem, e com possibilidade de rollback se necessário. Isso traz muito mais **controle, rastreabilidade e segurança** para o processo de evolução do schema.

🛠️ Ferramentas populares para migrations:

- **Flyway**: baseia-se em scripts SQL ou Java e é amplamente utilizado em projetos Spring Boot. Integra-se facilmente ao ciclo de vida da aplicação.
- **Liquibase**: usa XML, YAML, JSON ou SQL para definir mudanças e também oferece ferramentas avançadas de auditoria e rollback.

Por hora nos manteremos com a implementação mais simples, mas em aulas posteriores abordaremos a configuração e integração dessas ferramentas em nossa aplicação 🤩

Agora basta seguir para o passo 3...

### 🧪 Passo 3 – Criando o banco de dados

Antes de executar a aplicação, certifique-se de que o banco de dados **`contacts_db`** já está criado no servidor MySQL (ou PostgreSQL) local. Você pode criá-lo via terminal ou usando um cliente gráfico como **MySQL Workbench** ou **pgAdmin**.

```sql
CREATE DATABASE contacts_db;
```

### ✅ Resultado

Ao iniciar a aplicação, o Spring Boot utilizará a nova configuração, conectando-se ao banco de dados relacional, criando as tabelas com base nas entidades mapeadas com `@Entity`, e persistindo os dados de forma permanente.

Esse processo aproxima a aplicação do cenário de produção, permite maior controle sobre o ambiente de dados, e possibilita o uso de recursos avançados como índices, constraints e consultas otimizadas — fundamentais em aplicações reais. A configuração acima é suficiente para fazer a migração do banco H2 em memória, que estávamos utilizando, para o MySQL ou Postgres.

Isso foi possível por nos valermos do poder do ORM fornecido pelo Spring. 

### 💪 Reforçando os conceitos de ORM

Esse é um bom momento para relembrarmos um conceito importante: a sigla **ORM** significa *Object-Relational Mapping* (Mapeamento Objeto-Relacional) e trata-se de uma técnica que permite que desenvolvedores interajam com bancos de dados relacionais usando objetos da linguagem de programação, em vez de escrever diretamente comandos SQL. ORMs abstraem a complexidade do mundo relacional e nos permitem trabalhar no mundo orientado a objetos. Em Java, o ORM é frequentemente realizado por meio da especificação **JPA (Java Persistence API)**, sendo o **Hibernate** a implementação mais popular dessa especificação.  

O principal objetivo do ORM é reduzir a complexidade do acesso a dados e evitar o acoplamento direto entre o código da aplicação e o banco de dados. Com um ORM, classes Java são mapeadas para tabelas do banco, e os atributos dessas classes representam as colunas. O desenvolvedor pode persistir, atualizar, consultar e remover objetos com comandos simples e legíveis — como `repository.save(objeto)` ou `repository.findAll()` — em vez de escrever instruções SQL completas.

Esse modelo oferece diversos benefícios: melhora a produtividade, favorece o reuso de código, centraliza as regras de negócio na aplicação e, sobretudo, **torna muito simples mudar de banco de dados sem alterar a lógica da aplicação** — como acabamos de fazer.

A migração do banco de dados H2 (em memória) para o MySQL foi um excelente exemplo prático da flexibilidade que o ORM proporciona. No nosso projeto, bastou alterar algumas configurações no arquivo `application.properties` — como a URL de conexão, o usuário e a senha — e adicionar a dependência do driver do MySQL no `pom.xml`. A estrutura do código, as entidades JPA, os repositórios e os controladores permaneceram absolutamente inalterados.

Essa transição transparente só é possível porque o Hibernate é responsável por gerar e executar os comandos SQL apropriados para o banco configurado, a partir das anotações nas entidades. Isso demonstra na prática o **desacoplamento entre a aplicação e o banco de dados**, o que facilita muito a portabilidade, a manutenção e a evolução do sistema. Em ambientes reais, essa capacidade é extremamente valiosa: permite começar o desenvolvimento com um banco mais simples (como o H2), e migrar para uma solução mais robusta (como MySQL, PostgreSQL, etc.) sem grandes retrabalhos.

Se quisermos, podemos até mesmo trocar o banco novamente — por PostgreSQL, MariaDB, Oracle — e, desde que os dialetos e drivers estejam corretamente configurados, o restante da aplicação continuará funcionando da mesma forma. Isso é o poder do ORM aliado à padronização do JPA.

Agora falta apenas documentar nossa API com Swagger!