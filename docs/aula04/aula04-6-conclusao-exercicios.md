---
layout: aula
title: 6. Conclusão e exercícios
parent: Aula 04 - Spring Boot - JPA, Exceções e Documentação!
nav_order: 6
---

## **6. Recapitulando!**

Nesta aula, aprimoramos a nossa API REST desenvolvida na aula anterior, implementando novos recursos e adotando boas práticas essenciais para garantir a robustez e a consistência da aplicação. Abordamos conceitos como **consultas customizadas no JPA** utilizando convenções de nomes e queries personalizadas por meio da anotação `@Query`. Implementamos métodos PATCH para atualizações parciais de recursos, utilizando o tipo `Map<String, String>` e garantindo que apenas os campos especificados sejam modificados, mantendo os demais intactos.

O tratamento de erros foi aprimorado com a implementação de um `GlobalExceptionHandler`, que captura e trata exceções específicas, como `ResourceNotFoundException`, garantindo respostas HTTP adequadas e amigáveis. Exploramos a importância do tratamento de erros para garantir segurança, clareza, padronização e uma melhor experiência do usuário.

Além disso, trabalhamos com **relacionamentos de entidades usando JPA**, implementando uma relação bidirecional `OneToMany / ManyToOne` entre `Contact` e `Address`. Também introduzimos conceitos de **validação de dados** utilizando a `Jakarta Bean Validation` para garantir que os dados fornecidos pelos clientes sejam válidos antes de serem processados pela aplicação.

Por fim, implementamos controladores REST (`ContactController` e `AddressController`) para manipulação dessas entidades, expondo endpoints que permitem criar, recuperar, atualizar e excluir contatos e seus respectivos endereços. 

Além disso, mencionamos que a forma implementada atualmente é didática, porém pode ser aprimorada. Fizemos uma breve introdução ao conceito de DTOs, o que nos leva aos...

---

## **7. Exercícios 🤓**

**1️⃣ - Implementação de DTOs (Data Transfer Objects)**  
Atualmente, a API utiliza entidades diretamente na comunicação entre o cliente e o servidor. Para melhorar a segurança, controle dos dados expostos e evitar problemas de serialização cíclica, implemente DTOs para `Contact` e `Address`. Substitua os objetos retornados pelos controladores por DTOs e modifique os controladores para aceitar DTOs como entrada. 

**2️⃣ - Persistência em Banco de Dados Relacional**  
Até agora, estamos utilizando o banco de dados em memória configurado pelo Spring Data JPA. Altere a aplicação para utilizar um banco de dados relacional real, como MySQL ou PostgreSQL. Modifique o arquivo `application.properties` configurando as propriedades de conexão, e adicione dependências adequadas no `pom.xml`.

**3️⃣ - Paginação e Ordenação**  
Implemente paginação e ordenação nos métodos que retornam listas de contatos e endereços. Utilize a interface `Pageable` do Spring Data JPA e crie endpoints que aceitem parâmetros de paginação e ordenação na URL. Garanta que o resultado seja retornado de forma paginada, e não como uma lista completa.

**4️⃣ - Implementação de Documentação da API com Swagger**  
Agora que já implementamos funcionalidades importantes na nossa API REST, é hora de garantir que os usuários da API possam entendê-la e utilizá-la de forma adequada. Para isso, integre o Swagger, que permitirá a geração automática de uma documentação interativa e amigável. Adicione as dependências adequadas no `pom.xml`, crie uma Classe de Configuração, documente os Endpoints e Teste a Documentação.

### **📌 Instruções Finais**  

- ✅ Para os exercícios **práticos (1 a 4)** a entrega esperada é o código das novas rotas e prints das requisições no Postman ou Insomnia. **Envie um link do GitHub** ou um arquivo **.zip** com o código-fonte por meio do Moodle da disciplina.
- ✅ Para o exercício **4** entregue também prints mostrando o teste da documentação gerada pelo swagger. Adicione os prints ao repositório GIT ou no arquivo .zip juntamente com o código-fonte.
- ✅Teste todas as funcionalidades antes de enviar e garanta que o código está funcionando.  

## **Bons estudos e mãos à obra! 🛠**