---
layout: aula
title: 4. Swagger
parent: Aula 05 - Aprimorando nossas APIs Rest
nav_order: 4
---
## 4. Documentação com Swagger/OpenAPI

### 🔍 **O que é Swagger?**

O Swagger (atualmente conhecido como Swagger UI ou Springdoc OpenAPI em projetos Spring modernos) é uma ferramenta que permite documentar automaticamente uma API REST com base nas anotações feitas no código. Ele gera uma interface interativa no navegador onde qualquer pessoa pode consultar os endpoints, visualizar os parâmetros, tipos de retorno e até realizar chamadas HTTP diretamente pela interface web.

A grande vantagem é que ele **aumenta a transparência e a usabilidade da API**, além de **reduzir a necessidade de documentações manuais**.

### 🛠️ Como integrar o Swagger com Spring Boot?

**Adicione a dependência no `pom.xml`**:

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.7.0</version>
</dependency>
```

**Configuração Básica**
Anote a classe principal da aplicação, `ContactsApiApplication`, com `@OpenAPIDefinition`.

```java
package br.ifsp.contacts;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

import io.swagger.v3.oas.annotations.OpenAPIDefinition;
import io.swagger.v3.oas.annotations.info.Info;

@OpenAPIDefinition(info = @Info(title = "Contacts API", version = "1.0", description = "Documentação da API de Contatos"))
@SpringBootApplication
public class ContactsApiApplication {

	public static void main(String[] args) {
		SpringApplication.run(ContactsApiApplication.class, args);
	}
}
```

Essa anotação faz parte da especificação OpenAPI/Swagger. Ela é usada para documentar a API de forma automática, com suporte à interface gráfica do Swagger UI.

Com isso, quando a aplicação estiver rodando, o Swagger UI será carregado com as seguintes informações:
- Título: "Contacts API"
- Versão: 1.0
- Descrição: "Documentação da API de Contatos"

Esses dados aparecem na interface do Swagger (/swagger-ui.html ou /swagger-ui/index.html), facilitando o uso da API por desenvolvedores e testadores.

**Tudo pronto! 🤠**

Com a configuração feita, o próximo passo é refatorar nossos controllers adicionando anotações descritivas nos endpoints — essas descrições serão utilizadas pelo Swagger para gerar uma documentação rica, interativa e fácil de entender.

Vamos ver o estado atual do nosso sistema de gestão de contatos!