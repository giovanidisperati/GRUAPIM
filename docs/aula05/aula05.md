---
layout: aula
title: Aula 05 - Aprimorando nossas APIs Rest
nav_order: 7
has_children: true
permalink: /aula05
---

# **Aula 04 - REST II: Refinando e Aprimorando a API REST com Spring Boot**

Nesta aula, continuaremos aprimorando nossa API REST com Spring Boot, abordando tópicos como DTOs (Data Transfer Objects), paginação, versionamento de API e documentação automática com Swagger. Vamos garantir que nossa aplicação esteja mais robusta e fácil de consumir e, para isso, vamos seguir a mesma lógica que já usamos previamente: introduzir os conceitos dessa aula por meio da resolução dos exercícios da aula anterior.

Para tanto, vamos relembrar o que havia sido pedido:

- O primeiro exercício consiste na implementação de DTOs (Data Transfer Objects), substituindo o uso direto das entidades “Contact” e “Address” por objetos específicos para transporte de dados. Essa alteração evita a exposição desnecessária de informações internas, aumenta a segurança e resolve problemas de serialização cíclica. 

- O segundo se refere à persistência em um banco de dados relacional: em vez de utilizar o banco H2 em memória, a aplicação deve ser configurada para usar um banco real como MySQL ou PostgreSQL, o que exige alterar o arquivo application.properties com as configurações adequadas de conexão, além de adicionar dependências correspondentes no pom.xml. 

- O terceiro propõe a adoção de paginação e ordenação para os endpoints que retornam listas de contatos e endereços, implementando a interface Pageable do Spring Data JPA e permitindo que o cliente especifique parâmetros (exemplo: “page”, “size” e “sort”) para controlar como os resultados são exibidos. 

- Finalmente, o quarto exercício requer a documentação da API com Swagger, adicionando as dependências do springdoc-openapi ao pom.xml, criando uma classe de configuração e anotando os endpoints para que a documentação seja gerada de modo automático e interativo, facilitando o entendimento e o uso da API.

Vamos resolver esses exercícios de forma sequencial e abordar os conceitos envolvidos em cada um, assim como fizemos na aula anterior.