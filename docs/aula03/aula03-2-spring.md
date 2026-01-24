---
layout: aula
title: 2. Introdução ao Spring
parent: Aula 03 - Introdução às APIs REST e ao Spring Boot
nav_order: 2
---

## **2. Introdução ao Spring Boot**

O **Spring Boot** é um framework que facilita a criação de aplicativos em Java, fornecendo uma estrutura de projeto pré-configurada que agiliza o desenvolvimento. 

O Spring Boot segue o princípio de **Convention Over Configuration** (Convenção sobre Configuração), que é um princípio de desenvolvimento que reduz a necessidade de configurações explícitas, fornecendo valores e comportamentos padrão sensíveis. O desenvolvedor só precisa configurar explicitamente algo quando deseja modificar o comportamento padrão. 

Isso significa que ele vem com configurações padrão inteligentes que permitem aos desenvolvedores começarem rapidamente sem precisar definir manualmente cada detalhe da aplicação, o que elimina grande parte da configuração manual tradicional do Spring, permitindo que o desenvolvedor foque mais na lógica de negócio.

Sim, a explicação está clara e bem estruturada! Ela destaca os principais pontos sobre o **Spring Boot** e seu uso do princípio **Convention Over Configuration** de forma objetiva. No entanto, se quiser torná-la ainda mais acessível, você pode adicionar um exemplo rápido para ilustrar a ideia.

📌 **Exemplo prático:**  
No Spring tradicional, para configurar um banco de dados, seria necessário definir manualmente um **DataSource**, gerenciar transações e configurar o Hibernate.  

Com o **Spring Boot**, basta adicionar as dependências necessárias e incluir algumas linhas no `application.properties`:
> ```properties
> spring.datasource.url=jdbc:mysql://localhost:3306/meubanco
> spring.datasource.username=root
> spring.datasource.password=senha
> ```

Ou seja, o Spring Boot detecta automaticamente o banco de dados e configura o **Hibernate** sem necessidade de configurações adicionais.

### **Criando o Projeto Spring Boot**

Antes de iniciarmos a implementação da nossa API REST, precisamos criar o projeto Spring Boot. Podemos fazer isso de diversas maneiras, mas utilizaremos a abordagem mais prática: o **Spring Initializr**.

#### **🔹 Criando o Projeto com Spring Initializr**
O **Spring Initializr** é uma ferramenta online que permite configurar e gerar rapidamente um projeto Spring Boot com as dependências necessárias.

#### **Passo a passo para criar o projeto:**
1. **Acesse o Spring Initializr**  
   - Acesse [https://start.spring.io/](https://start.spring.io/)
   
2. **Configure o projeto**  
   - **Project:** Maven (ou Gradle, se preferir)  
   - **Language:** Java  
   - **Spring Boot Version:** Escolha a versão mais recente estável  
   - **Group:** `br.ifsp`  
   - **Artifact:** `contacts-api`  
   - **Name:** `contacts-api`  
   - **Description:** `API para gerenciamento de contatos`  
   - **Package Name:** `br.ifsp.contacts`  
   - **Packaging:** Jar  
   - **Java Version:** 11 ou superior (recomendado)

3. **Adicione as dependências essenciais**  
   - **Spring Web** → Para criar a API REST  
   - **Spring Boot DevTools** → Para recarregamento automático durante o desenvolvimento  
   - **Spring Data JPA** → Para interação com banco de dados  
   - **H2 Database** → Banco de dados em memória para testes

4. **Gerar e baixar o projeto**  
   - Clique em **"Generate"** para baixar o projeto `.zip`  
   - Extraia o `.zip` em uma pasta de sua preferência.

#### **🔹 Importando o Projeto na IDE**
Após baixar e extrair o projeto, é hora de importá-lo para uma IDE (como IntelliJ IDEA, Eclipse ou VS Code com extensão Java).

**Se estiver usando IntelliJ IDEA:**
1. Abra o IntelliJ IDEA.
2. Clique em **File** > **Open**.
3. Selecione a pasta do projeto (`contacts-api`) e clique em **Open**.
4. Aguarde o Maven ou Gradle baixar as dependências automaticamente.

**Se estiver usando Eclipse:**
1. Vá até **File** > **Import**.
2. Escolha **Existing Maven Projects**.
3. Selecione a pasta onde extraiu o projeto.
4. Clique em **Finish** e aguarde a configuração.

#### **🔹 Rodando o Projeto pela Primeira Vez**
Agora que temos o projeto configurado, podemos rodá-lo pela primeira vez.

1. **Abra o terminal na pasta do projeto.**
2. Se estiver usando **Maven**, execute:
   ```
   mvn spring-boot:run
   ```
   Se estiver usando **Gradle**, execute:
   ```
   ./gradlew bootRun
   ```
3. Aguarde até que a aplicação inicie. Se tudo estiver correto, você verá algo como:
   ```
   Tomcat started on port(s): 8080 (http)
   ```

Agora sua API está rodando localmente na porta **8080** e pronta para receber requisições! 🎉

Caso queira testar se está funcionando, abra o navegador e acesse:
```
http://localhost:8080
```
A princípio, receberá uma resposta `Whitelabel Error Page`, pois ainda não criamos nenhuma rota. Nas próximas etapas, adicionaremos os **endpoints REST**.


## 2.1 Estrutura do Projeto

Ao criar um projeto com o **Spring Boot**, é comum utilizar o padrão de pastas do Maven, mesmo que o seu build tool seja o Maven ou Gradle. Essa estrutura fica geralmente assim:

```
.
├── src
│   ├── main
│   │   ├── java
│   │   │   └── br
│   │   │       └── ifsp
│   │   │           └── contacts
│   │   │               ├── ContactsApplication.java
│   │   │               ├── controller
│   │   │               │   └── ContactController.java
│   │   │               ├── model
│   │   │               │   └── Contact.java
│   │   │               └── repository
│   │   │                   └── ContactRepository.java
│   │   └── resources
│   │       └── application.properties
│   └── test
│       └── java
└── pom.xml (ou build.gradle)
```

A seguir, veremos cada arquivo principal e as explicações associadas.

---

## 2.2 Arquivo de Inicialização: `ContactsApplication.java`

Este é o ponto de entrada da aplicação Spring Boot. É nele que colocamos a anotação `@SpringBootApplication`, que habilita diversas configurações automáticas:

```java
package br.ifsp.contacts;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * Classe principal da nossa aplicação Spring Boot.
 * 
 * A anotação @SpringBootApplication habilita as configurações
 * automáticas do Spring (auto-configuration) e também indica 
 * que esta é a classe que deve ser executada para iniciar 
 * a aplicação.
 */
@SpringBootApplication
public class ContactsApplication {

    public static void main(String[] args) {
        // Método main: ponto de entrada de uma aplicação Java.
        // SpringApplication.run() inicia a aplicação Spring Boot.
        SpringApplication.run(ContactsApplication.class, args);
    }

}
```

**Explicação**:
- `@SpringBootApplication` é uma anotação que combina diversas funcionalidades, como:
  - `@Configuration`: para permitir configurações na aplicação.
  - `@EnableAutoConfiguration`: faz com que o Spring Boot configure automaticamente componentes relevantes, de acordo com o que encontra no classpath do projeto.
  - `@ComponentScan`: faz com que o Spring procure automaticamente classes anotadas como componentes dentro do mesmo pacote ou em pacotes filhos.

Quando executamos o método `main`, o **Spring Boot** levanta um servidor embutido (por padrão, o Tomcat) e passa a escutar as requisições HTTP.

---

## 2.3 Modelo de Dados: `Contact.java`

Para representar cada contato, precisamos de uma classe que contenha seus atributos. Chamamos essa classe de **modelo**, ou **entidade**, quando falamos de persistência de dados.

```java
package br.ifsp.contacts.model;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

/**
 * Classe que representa o modelo de dados para um Contato.
 * 
 * @Entity indica que este objeto será mapeado para uma tabela
 * no banco de dados (em cenários de persistência com JPA).
 */
@Entity
public class Contact {

    /**
     * @Id indica que este campo é a chave primária (primary key) da entidade.
     * @GeneratedValue permite que o banco de dados (ou o provedor JPA) 
     * gere automaticamente um valor único para cada novo registro.
     */
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String nome;
    private String telefone;
    private String email;

    // Construtor vazio exigido pelo JPA
    public Contact() {}

    // Construtor para facilitar a criação de objetos
    public Contact(String nome, String telefone, String email) {
        this.nome = nome;
        this.telefone = telefone;
        this.email = email;
    }

    // Getters e Setters
    public Long getId() {
        return id;
    }

    public String getNome() {
        return nome;
    }
    public void setNome(String nome) {
        this.nome = nome;
    }

    public String getTelefone() {
        return telefone;
    }
    public void setTelefone(String telefone) {
        this.telefone = telefone;
    }

    public String getEmail() {
        return email;
    }
    public void setEmail(String email) {
        this.email = email;
    }
}
```

**Explicação**:
- `@Entity` diz ao **Spring/Data JPA** (ou outro provedor de JPA) que esta classe é uma entidade que será mapeada para uma tabela no banco de dados.
- `@Id` define o identificador único do registro.
- `@GeneratedValue(strategy = GenerationType.IDENTITY)` faz com que o valor do ID seja gerado automaticamente (por exemplo, auto-increment em bancos relacionais).
- `Long id`, `String nome`, `String telefone` e `String email` são os campos que vamos manipular.

Mesmo que ainda não estejamos aprofundando no uso de banco de dados real, este exemplo já está preparado para persistência caso seja configurado um banco H2 ou outro SGBD.

---

## 2.4 Repositório de Dados: `ContactRepository.java`

O repositório é responsável por realizar as operações de acesso aos dados (criar, ler, atualizar, excluir). Vamos utilizar a **interface** `JpaRepository` do Spring Data JPA, que já oferece as implementações básicas de CRUD, bastando nós definirmos a interface corretamente.

```java
package br.ifsp.contacts.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import br.ifsp.contacts.model.Contact;

/**
 * Esta interface extende JpaRepository, que nos fornece métodos prontos 
 * para acessar e manipular dados no banco de dados. Basta especificar 
 * a classe de entidade (Contact) e o tipo da chave primária (Long).
 */
public interface ContactRepository extends JpaRepository<Contact, Long> {
    // Podemos adicionar métodos personalizados se necessário.
}
```

**Explicação**:
- Ao estender `JpaRepository<Contact, Long>`, nós ganhamos de forma automática diversos métodos como:
  - `save()` - para inserir ou atualizar um contato
  - `findById()` - para buscar contato por ID
  - `findAll()` - para buscar todos os contatos
  - `deleteById()` - para deletar contato por ID
- O `ContactRepository` já está pronto para ser injetado em outras camadas da aplicação (como na Controller).

---

## 2.5 Controlador (Controller): `ContactController.java`

O **controlador** é a classe que “escuta” as requisições HTTP e define como o sistema vai responder. Nele, utilizamos anotações como `@RestController` (que indica um controller REST) e `@RequestMapping` (para definir o caminho base dos endpoints, também chamados de “URIs” ou “rotas”).

```java
package br.ifsp.contacts.controller;

import br.ifsp.contacts.model.Contact;
import br.ifsp.contacts.repository.ContactRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;

/**
 * Classe responsável por mapear as rotas/endpoints relacionados
 * aos contatos. Cada método abaixo corresponde a uma operação
 * em nosso sistema.
 * 
 * @RestController: Indica que esta classe é um controlador 
 *                  responsável por responder requisições REST.
 * @RequestMapping("/api/contacts"): Indica o caminho base.
 */
@RestController
@RequestMapping("/api/contacts")
public class ContactController {

    /**
     * @Autowired permite que o Spring "injete" automaticamente
     * uma instância de ContactRepository aqui, 
     * sem que precisemos criar manualmente.
     */
    @Autowired
    private ContactRepository contactRepository;

    /**
     * Método para obter todos os contatos.
     * 
     * @GetMapping indica que este método vai responder a chamadas HTTP GET.
     * Exemplo de acesso: GET /api/contacts
     */
    @GetMapping
    public List<Contact> getAllContacts() {
        return contactRepository.findAll();
    }

    /**
     * Método para obter um contato específico pelo seu ID.
     * 
     * @PathVariable "amarra" a variável {id} da URL 
     * ao parâmetro do método.
     * Exemplo de acesso: GET /api/contacts/1
     */
    @GetMapping("/{id}")
    public Contact getContactById(@PathVariable Long id) {
        // findById retorna um Optional, então usamos orElseThrow
        return contactRepository.findById(id)
                .orElseThrow(() -> new RuntimeException("Contato não encontrado: " + id));
    }

    /**
     * Método para criar um novo contato.
     * 
     * @PostMapping indica que este método responde a chamadas HTTP POST.
     * @RequestBody indica que o objeto Contact será preenchido 
     * com os dados JSON enviados no corpo da requisição.
     */
    @PostMapping
    public Contact createContact(@RequestBody Contact contact) {
        return contactRepository.save(contact);
    }

    /**
     * Método para atualizar um contato existente.
     * 
     * @PutMapping indica que este método responde a chamadas HTTP PUT.
     * Exemplo de acesso: PUT /api/contacts/1
     */
    @PutMapping("/{id}")
    public Contact updateContact(@PathVariable Long id, @RequestBody Contact updatedContact) {
        // Buscar o contato existente
        Contact existingContact = contactRepository.findById(id)
                .orElseThrow(() -> new RuntimeException("Contato não encontrado: " + id));

        // Atualizar os campos
        existingContact.setNome(updatedContact.getNome());
        existingContact.setTelefone(updatedContact.getTelefone());
        existingContact.setEmail(updatedContact.getEmail());

        // Salvar alterações
        return contactRepository.save(existingContact);
    }

    /**
     * Método para excluir um contato pelo ID.
     * 
     * @DeleteMapping indica que este método responde a chamadas HTTP DELETE.
     * Exemplo de acesso: DELETE /api/contacts/1
     */
    @DeleteMapping("/{id}")
    public void deleteContact(@PathVariable Long id) {
        contactRepository.deleteById(id);
    }
}
```

**Explicação**:

1. **`@RestController`**: 
   - Indica que esta classe processa requisições REST (ou seja, retorna dados em JSON ou outro formato) e não retorna diretamente páginas HTML.

2. **`@RequestMapping("/api/contacts")`**:
   - Define que todos os métodos dentro deste controller estarão acessíveis a partir de URIs que começam com `/api/contacts`. Por exemplo, se o método for anotado com `@GetMapping`, isso virará `GET /api/contacts`.

3. **Métodos HTTP**:
   - `@GetMapping` → Método GET do protocolo HTTP. Para obter (ler) dados.
   - `@PostMapping` → Método POST do protocolo HTTP. Para criar (inserir) dados.
   - `@PutMapping` → Método PUT do protocolo HTTP. Para atualizar (modificar) dados.
   - `@DeleteMapping` → Método DELETE do protocolo HTTP. Para excluir dados.

4. **`@PathVariable`**:
   - Indica que o valor do parâmetro `id` virá diretamente do segmento correspondente na URL. Por exemplo, ao acessar `GET /api/contacts/10`, o valor `10` será atribuído ao parâmetro `id`.

5. **`@RequestBody`**:
   - Indica que o parâmetro do método (`Contact contact`) deve ser populado com o corpo da requisição. Em uma chamada POST com corpo JSON, os campos `nome`, `telefone`, `email` (por exemplo) serão convertidos em um objeto `Contact`.

6. **Injeção de Dependência (`@Autowired`)**:
   - O Spring cria e gerencia o objeto `ContactRepository`. Ao colocar `@Autowired`, estamos dizendo ao Spring para injetar nesse campo a instância do nosso repositório, economizando a tarefa de instanciá-lo manualmente.

---

## 2.6 Arquivo de Configuração: `application.properties`

Caso quiséssemos usar o banco H2 em memória para testes ou configurações adicionais, poderíamos adicionar alguns parâmetros no `application.properties`. Por exemplo:

```properties
spring.datasource.url=jdbc:h2:mem:contacts_db
spring.datasource.driverClassName=org.h2.Driver
spring.jpa.hibernate.ddl-auto=update
spring.h2.console.enabled=true
```

**Explicação**:
- `spring.datasource.url` e `spring.datasource.driverClassName` definem o local do banco e o driver de conexão.
- `spring.jpa.hibernate.ddl-auto=update` faz com que a tabela seja criada/atualizada automaticamente ao iniciar a aplicação, conforme as entidades definidas.
- `spring.h2.console.enabled=true` habilita um console web para ver e manipular dados do banco (acessível em `/h2-console` quando a aplicação está rodando).

Se não quisermos usar H2, ainda assim podemos manter o projeto como está, pois o Spring Boot, por padrão, tentará criar um banco em memória se não for configurado outro banco real. 

---

## 2.7 Build e Execução

### Usando Maven

Se estivermos utilizando Maven, teremos um arquivo `pom.xml` na raiz do projeto. Ele conterá, entre outras coisas, as dependências:

```xml
<dependencies>
    <!-- Dependência do Spring Web, para criar a API REST -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- Dependência do Spring Data JPA, para acesso a dados -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <!-- Dependência para banco H2 em memória (opcional) -->
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
    </dependency>
</dependencies>
```

Para rodar, podemos simplesmente executar:
```
mvn spring-boot:run
```
ou, se gerarmos um `.jar`, podemos rodar:
```
java -jar target/contacts-0.0.1-SNAPSHOT.jar
```

### Usando Gradle

Caso nosso projeto fosse em **Gradle**, teríamos um arquivo `build.gradle`. O conteúdo principal para as dependências seria algo como:

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    runtimeOnly 'com.h2database:h2'
}
```

Para iniciar a aplicação:
```
./gradlew bootRun
```

### **Maven vs. Gradle: Comparação e Diferenças**  

Ao desenvolver aplicações com **Spring Boot**, é necessário um **gerenciador de dependências e build** para automatizar tarefas como **compilação, empacotamento, testes e execução do projeto**. Os dois principais gerenciadores no ecossistema Java são **Maven** e **Gradle**.  

#### **Estrutura e Configuração**  
- **Maven** utiliza um arquivo **XML (`pom.xml`)**, que define as dependências e configurações do projeto de maneira declarativa.  
- **Gradle** usa arquivos baseados em **Groovy ou Kotlin (`build.gradle` ou `build.gradle.kts`)**.  

#### **Desempenho e Eficiência**  
- **Gradle** é mais rápido que Maven, pois utiliza **builds incrementais** e **cache de tarefas**, evitando a recompilação de código desnecessário.  
- **Maven** sempre executa as tarefas do zero, tornando o build mais previsível, porém mais lento.  

#### **Sintaxe e Facilidade de Uso**  
- **Maven** tem uma sintaxe rígida e baseada em XML, o que pode gerar arquivos longos e verbosos.  
- **Gradle** tem uma sintaxe mais concisa e expressiva, facilitando a configuração de builds complexos.  

#### **Popularidade e Comunidade**  
- **Maven** tem uma comunidade maior e uma adoção mais ampla, sendo o padrão em muitos projetos Java.  
- **Gradle** tem ganhado espaço, especialmente em projetos modernos, e é amplamente usado no desenvolvimento Android.  

#### **Ou seja...**  
- Se você busca **padronização e estabilidade**, **Maven** é uma excelente escolha.  
- Se precisa de **builds mais rápidos e configuráveis**, **Gradle** pode ser a melhor opção.  

Ambas as ferramentas são compatíveis com **Spring Boot**, então a escolha depende das necessidades do projeto e do conhecimento prévio da equipe. 

Ao longo da disciplina utilizaremos o **Maven** como ferramenta padrão, dada sua maior popularidade.

---

## 2.8 Testando a API

Ao desenvolver uma **API REST**, é essencial testar suas funcionalidades para verificar se os endpoints estão respondendo corretamente. Para isso, utilizamos **clients de API**, que permitem enviar requisições HTTP e visualizar as respostas do servidor. Dentre os mais populares, temos as seguintes opções.

### **Postman** 📨  
O **Postman** é um dos **clientes de API mais populares** e oferece uma interface gráfica para:  
✅ Enviar requisições **GET, POST, PUT, PATCH, DELETE** com facilidade.  
✅ Adicionar **headers, parâmetros e autenticação** às requisições.  
✅ Salvar **coleções de requisições** para testes automatizados.  
✅ Simular **corpos JSON e XML** no envio de dados.  

👉 **Exemplo:**  
1. Abra o Postman e crie uma nova requisição.  
2. Escolha **GET** e insira `http://localhost:8080/api/contacts`.  
3. Clique em **Send** e visualize a resposta JSON da API.  

💡 O Postman também suporta **testes automatizados** e integração com CI/CD.  

### **Insomnia** 🌙  
O **Insomnia** é uma alternativa ao Postman, mais leve e focada na simplicidade. Ele permite:  
✅ Criar e organizar requisições de forma intuitiva.  
✅ Testar APIs REST, GraphQL e WebSockets.  
✅ Gerenciar variáveis de ambiente para diferentes contextos (desenvolvimento, produção).  

💡 É recomendado para quem busca uma experiência mais fluida e rápida, sem tantas funcionalidades avançadas.  

### **cURL** 🖥️  
O **cURL** é uma ferramenta de linha de comando para testar APIs diretamente no terminal. É útil para desenvolvedores que preferem **scripts e automação**.  

👉 **Exemplo de requisição GET com cURL:**  
```sh
curl -X GET http://localhost:8080/api/contacts
```
👉 **Exemplo de requisição POST com JSON:**  
```sh
curl -X POST http://localhost:8080/api/contacts \
     -H "Content-Type: application/json" \
     -d '{"nome": "Maria", "telefone": "9999-9999", "email": "maria@email.com"}'
```

💡 cURL é embutido no Linux e macOS e pode ser usado em scripts para **testes automatizados**.  

### **Alternativas e Ferramentas Adicionais**  
- **Hoppscotch** → Cliente de API online e gratuito, semelhante ao Postman.  
- **Thunder Client** → Extensão do VS Code para testar APIs REST sem sair do editor.  
- **HTTPie** → Alternativa ao cURL com saída mais legível.  


Independentemente da ferramenta, testar a API é fundamental para garantir que os endpoints funcionam conforme esperado! 🚀

### Com o client em mãos, vamos ao teste da API!

Depois que a aplicação estiver em execução (por padrão, na porta 8080), podemos testar as rotas usando alguma das ferramentas descritas acima. Ao longo da disciplina utilizaremos principalmente o **Postman**.

1. **Criar contato (POST)**  
   ```
   POST /api/contacts
   Body (JSON):
   {
     "nome": "João da Silva",
     "telefone": "9999-9999",
     "email": "[email protected]"
   }
   ```
   - **Resposta**: Objeto JSON do contato criado, incluindo o `id` atribuído.

2. **Obter todos os contatos (GET)**  
   ```
   GET /api/contacts
   ```
   - **Resposta**: Lista de contatos em formato JSON.

3. **Obter contato específico (GET)**  
   ```
   GET /api/contacts/1
   ```
   - **Resposta**: Objeto JSON com o contato de ID 1 (caso exista).

4. **Atualizar contato (PUT)**  
   ```
   PUT /api/contacts/1
   Body (JSON):
   {
     "nome": "Maria da Silva",
     "telefone": "8888-8888",
     "email": "[email protected]"
   }
   ```
   - **Resposta**: Objeto JSON com o contato atualizado.

5. **Excluir contato (DELETE)**  
   ```
   DELETE /api/contacts/1
   ```
   - **Resposta**: Sem corpo (status HTTP 200 ou 204).

---

## 2.9 Recapitulando

Neste projeto, apresentamos:
1. **O que é uma API REST** e como o Spring Boot facilita sua criação.
2. **Os principais verbos HTTP (GET, POST, PUT, DELETE)** e como eles se relacionam com operações de leitura, criação, atualização e exclusão de dados.
3. **Como estruturar um projeto Spring Boot** com:
   - Classe principal (`ContactsApplication`)
   - Entidade de modelo (`Contact`)
   - Repositório de dados (`ContactRepository`)
   - Controlador REST (`ContactController`)
   - Configurações no `application.properties` e dependências no `pom.xml` ou `build.gradle`.

Essa base serve para, gradualmente, adicionar mais funcionalidades, como:
- Validações nos campos de contato.
- Tratamento de exceções customizadas.
- Autenticação e autorização (segurança).
- Integração com bancos de dados reais ou outros serviços.

### **O que aprendemos até aqui?**  

✅ **O que são APIs REST e seus princípios fundamentais.**  
✅ **Os métodos HTTP (GET, POST, PUT, PATCH, DELETE) e seu uso correto.**  
✅ **A importância da padronização e do modelo de maturidade de Richardson.**  
✅ **A estrutura de um projeto Spring Boot e como criar uma API básica.**  
✅ **Como testar APIs usando ferramentas como Postman, Insomnia e cURL.**  


Por enquanto, nossa aplicação já exemplifica o essencial do desenvolvimento de uma **API REST** com o **Spring Boot**. 

Sigam este passo-a-passo e executem o código-fonte como ponto de partida para os exercícios posteriores.