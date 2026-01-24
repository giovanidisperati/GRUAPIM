---
layout: aula
title: 1. Introdução ao REST
parent: Aula 03 - Introdução às APIs REST e ao Spring Boot
nav_order: 1
---

## **1. O que é uma API?**

Uma **API (Application Programming Interface)** é um conjunto de regras e definições que permitem que diferentes sistemas de software se comuniquem entre si. Elas atuam como intermediárias que possibilitam que um sistema solicite e envie dados para outro, de maneira estruturada e padronizada.  

Dentre os diversos estilos de arquitetura para APIs, uma das mais populares e amplamente utilizadas hoje em dia é a **API REST** (Representational State Transfer). Esse padrão se baseia nos princípios da web, utilizando o protocolo HTTP para a comunicação entre sistemas distribuídos.  

---

## **1.1 O que é uma API REST?**  

O termo **REST (Representational State Transfer)** foi criado por **Roy Fielding** em sua tese de doutorado em 2000. Ele descreveu um conjunto de princípios arquiteturais que utilizam os padrões já estabelecidos na web para criar sistemas escaláveis e eficientes.  

Uma API REST segue esses princípios, permitindo que os dados sejam manipulados através de **recursos (resources)**, representados por **URLs (Uniform Resource Locators)** e acessados usando os métodos HTTP padrão:

- **GET** → Obtém um recurso (ex.: buscar lista de usuários).  
- **POST** → Cria um novo recurso (ex.: cadastrar um novo usuário).  
- **PUT** → Atualiza um recurso existente (ex.: editar os dados de um usuário).  
- **PATCH** → Atualiza parcialmente um recurso (ex.: modificar apenas um campo específico de um usuário).
- **DELETE** → Remove um recurso (ex.: excluir um usuário).  

Esses métodos garantem que a API siga uma **estrutura previsível e intuitiva** para desenvolvedores que a utilizam. Isso acontece pois o uso adequado dos métodos HTTP em APIs REST garante **padronização**, **previsibilidade** e **intuitividade** para os desenvolvedores que consomem a API. Ou seja, ao interagir com qualquer API REST bem projetada, um desenvolvedor já pode inferir **como fazer requisições** apenas conhecendo os métodos e os recursos expostos.

Se uma API segue corretamente os padrões REST, um desenvolvedor não precisa consultar a documentação sempre que for fazer uma chamada, pois os métodos HTTP seguem uma **convenção universal**.  

### **1.2 Exemplo de Convenção em APIs REST**  

Vamos imaginar uma API para gerenciar **usuários** (`users`). Seguindo a convenção REST, os endpoints e métodos HTTP devem funcionar assim:

| Ação Desejada | Método HTTP | Caminho (URI) | Corpo da Requisição | Resposta Esperada |
|--------------|------------|---------------|---------------------|------------------|
| Obter todos os usuários | **GET** | `/users` | ❌ (sem corpo) | Lista de usuários (JSON) |
| Obter um usuário pelo ID | **GET** | `/users/{id}` | ❌ (sem corpo) | JSON do usuário |
| Criar um novo usuário | **POST** | `/users` | ✅ JSON com os dados do usuário | JSON do usuário criado com ID gerado |
| Atualizar **todo** um usuário existente | **PUT** | `/users/{id}` | ✅ JSON com todos os campos do usuário | JSON do usuário atualizado |
| Atualizar **parcialmente** um usuário | **PATCH** | `/users/{id}` | ✅ JSON com apenas os campos alterados | JSON do usuário atualizado |
| Excluir um usuário | **DELETE** | `/users/{id}` | ❌ (sem corpo) | Resposta vazia (status HTTP 204) |

### **Por que isso faz sentido?**  

Se um desenvolvedor encontrar uma API com o recurso `products` (produtos), ele já pode inferir como a API funciona sem ler a documentação, pois seguirá o mesmo padrão:

| Ação Desejada | Método HTTP | Caminho (URI) |
|--------------|------------|---------------|
| Obter todos os produtos | **GET** | `/products` |
| Obter um produto pelo ID | **GET** | `/products/{id}` |
| Criar um novo produto | **POST** | `/products` |
| Atualizar um produto | **PUT** | `/products/{id}` |
| Atualizar um campo do produto | **PATCH** | `/products/{id}` |
| Excluir um produto | **DELETE** | `/products/{id}` |

Isso garante que, se um desenvolvedor aprender a consumir uma API REST, ele poderá consumir qualquer outra API REST bem estruturada **sem precisar reaprender** cada API do zero. Na seção 2.5 é apresentado o código fonte da implementação de um Controller de uma aplicação Spring Boot que segue a convenção descrita acima.

Ainda em relação à convenção, é importante o uso adequado dos verbos PUT e PATCH.

### **1.3 Diferença entre PUT e PATCH**  

Tanto o **PUT** quanto o **PATCH** são usados para atualizar recursos, mas com diferenças importantes:

| Método | Propósito | Tipo de Atualização | Exemplo de Uso |
|--------|----------|--------------------|----------------|
| **PUT** | Atualiza um recurso inteiro | Substitui completamente os dados | Atualizar todas as informações de um usuário |
| **PATCH** | Atualiza parcialmente um recurso | Modifica apenas os campos enviados na requisição | Atualizar apenas o e-mail de um usuário |

#### **Exemplo de Requisição PUT**  
Se temos um usuário cadastrado assim:  
```json
{
  "id": 1,
  "nome": "João Silva",
  "email": "[email protected]",
  "telefone": "9999-9999"
}
```
E enviamos um **PUT** com este corpo:
```json
{
  "id": 1,
  "nome": "Maria Silva",
  "email": "[email protected]",
  "telefone": "8888-8888"
}
```
O registro original será completamente substituído, mesmo que não tenha havido mudanças no ID.

#### **Exemplo de Requisição PATCH**  
Se enviarmos um **PATCH** apenas com:
```json
{
  "email": "[email protected]"
}
```
Apenas o campo `email` será alterado, e os outros campos permanecerão inalterados.

---

## **1.4 Utilização de APIs REST**  

Em relação à aplicabilidade, as APIs REST são amplamente utilizadas em diversos cenários do desenvolvimento de software, incluindo:

1. **Aplicações Web e Mobile**:  
   - APIs REST são a base para **aplicativos móveis** e **front-ends modernos**, permitindo que o cliente (navegador ou aplicativo) se comunique com servidores back-end.  
   - Exemplo: O aplicativo de um banco acessa os dados do usuário através de uma API REST.  

2. **Integração entre Sistemas**:  
   - Diferentes sistemas podem se comunicar via APIs REST, eliminando a necessidade de compartilhamento de banco de dados.  
   - Exemplo: Um ERP de uma empresa pode se integrar ao sistema de contabilidade por meio de uma API.  

3. **Serviços em Nuvem e IoT**:  
   - Dispositivos IoT (Internet das Coisas) frequentemente usam APIs REST para enviar dados para servidores.  
   - Exemplo: Um smartwatch pode enviar informações de batimentos cardíacos para uma API REST na nuvem.  

4. **Plataformas de Terceiros**:  
   - Muitas empresas oferecem APIs REST para que terceiros utilizem seus serviços de forma programática.  
   - Exemplo: O Google Maps fornece APIs REST para que desenvolvedores integrem mapas em seus aplicativos.  

---

## **1.5 Vantagens das APIs REST**  

1. **Simplicidade e Facilidade de Uso**  
   - As APIs REST utilizam o protocolo HTTP, que é amplamente conhecido e usado na web.  
   - O uso de JSON como formato de dados facilita a leitura e escrita, tornando a comunicação leve e eficiente.  

2. **Escalabilidade**  
   - Por serem **stateless** (não armazenam estado entre requisições), as APIs REST facilitam a escalabilidade horizontal, permitindo múltiplos servidores processando requisições simultaneamente.  

3. **Flexibilidade e Independência**  
   - REST permite que clientes em diferentes tecnologias (React, Angular, iOS, Android) consumam a mesma API sem alterações no servidor.  

4. **Padronização e Interoperabilidade**  
   - Como segue padrões bem definidos, uma API REST pode ser usada por qualquer cliente que compreenda HTTP.  

5. **Cacheável**  
   - APIs REST permitem o uso eficiente de cache, reduzindo a carga no servidor e melhorando a performance.  

É importante notar, entretanto, que não basta fazer as chamadas dos métodos HTTP para que se tenha uma API REST. 

## **1.6 Modelo de Maturidade de Richardson**  

Nesse sentido, o **Modelo de Maturidade de Richardson (RMM - Richardson Maturity Model)** foi criado pelo cientista da computação **Leonard Richardson** e apresentado em 2008 durante uma palestra na QCon, uma conferência sobre desenvolvimento de software. Esse modelo propõe uma forma de **avaliar o nível de conformidade** de uma API com os princípios REST, ajudando a medir quão bem uma API segue a filosofia RESTful descrita por **Roy Fielding** em sua tese de doutorado em 2000.

A motivação principal desse modelo é a observação de que **nem todas as APIs chamadas de "REST" realmente seguem os princípios REST**. Muitas APIs usam apenas HTTP como meio de transporte, mas continuam operando como sistemas antigos, sem aproveitar os benefícios reais do REST. O modelo de Richardson classifica APIs em **quatro níveis de maturidade**, destacando aspectos como a organização de recursos, o uso correto dos métodos HTTP e a adoção de hipermídia (HATEOAS).

---

## **Os Quatro Níveis do Modelo de Richardson**

O modelo define **quatro níveis de maturidade**, numerados de 0 a 3, onde cada nível indica uma progressão rumo a uma API verdadeiramente RESTful.

### **Nível 0 - "O Ponto de Entrada Único" (The Swamp of POX - Plain Old XML)**

Neste nível, a API **não utiliza os conceitos REST**. Em vez disso, ela usa HTTP apenas como **meio de transporte** para mensagens genéricas, muitas vezes enviando e recebendo dados em formatos como XML ou JSON, sem aproveitar a estrutura do protocolo HTTP.

#### **Exemplo Histórico: APIs Baseadas em RPC ou SOAP**
- Antes da adoção de REST, muitas APIs eram construídas utilizando **SOAP (Simple Object Access Protocol)**, que encapsula mensagens XML dentro de requisições HTTP.
- As APIs SOAP normalmente usavam apenas **um único endpoint**, como `/service`, e todas as operações eram diferenciadas pelo **conteúdo do corpo da requisição**.
- Como consequência, **HTTP era tratado apenas como um canal de transporte**, sem o uso adequado de métodos como GET, POST, PUT e DELETE. Ainda há usos para SOAP, que serão discutidos posteriormente, mas de forma geral sua utilização é atualmente restrita à casos específicos.
- **Exemplo real:** Serviços SOAP do governo dos EUA nos anos 2000 (exemplo: **APIs SOAP do IRS** para comunicação entre sistemas fiscais).

---

### **Nível 1 - "Recursos" (Resources)**

No **Nível 1**, a API começa a **organizar seus dados em recursos individuais** e cada recurso recebe uma **URL única**. No entanto, os métodos HTTP ainda não são utilizados corretamente, e a API continua tratando HTTP apenas como um meio de transporte.

#### **Exemplo Histórico: APIs Baseadas em XML-RPC**
- No início dos anos 2000, APIs baseadas em **XML-RPC (Remote Procedure Call sobre XML)** começaram a organizar os dados em URLs específicas, mas ainda usavam **um único método HTTP (normalmente POST)** para todas as operações.
- Um exemplo clássico foi o **Movable Type API** (2001), uma das primeiras APIs para blogs, que permitia a publicação de posts usando XML-RPC.
- Todas as requisições eram enviadas para um único endpoint `/api`, com o corpo da requisição contendo instruções como:
  ```xml
  <methodCall>
      <methodName>getPost</methodName>
      <params>
          <param>
              <value><string>123</string></value>
          </param>
      </params>
  </methodCall>
  ```

---

### **Nível 2 - "Uso Correto de Verbos HTTP" (HTTP Verbs)**

Aqui, a API começa a **utilizar corretamente os métodos HTTP** (GET, POST, PUT, DELETE, PATCH). Isso significa que as operações são realizadas de maneira semântica:

- **GET** → Para leitura de dados
- **POST** → Para criação de dados
- **PUT** → Para substituição completa de um recurso
- **PATCH** → Para atualização parcial de um recurso
- **DELETE** → Para remoção de um recurso

A partir deste nível, a API se torna **mais intuitiva e previsível**, pois os clientes podem deduzir como interagir com ela sem precisar consultar documentação detalhada.

#### **Exemplo Histórico: APIs RESTful do Twitter e Facebook**
- Em **2006**, o Twitter lançou uma **API RESTful** que adotava corretamente os verbos HTTP.
- Exemplo real:
  - Para obter um tweet: `GET /tweets/12345`
  - Para deletar um tweet: `DELETE /tweets/12345`
  - Para postar um novo tweet: `POST /tweets`
- Essa API foi um marco na adoção de REST, e influenciou APIs de redes sociais como Facebook e Instagram.
- **Comparação:** Enquanto a API REST do Twitter era **nível 2**, a API SOAP de serviços bancários ainda operava no **nível 0 ou 1**, sem uso correto dos verbos HTTP.

---

### **Nível 3 - "HATEOAS" (Hypermedia as the Engine of Application State)**

O nível mais avançado da maturidade REST adiciona um conceito chamado **HATEOAS (Hypermedia as the Engine of Application State)**. Isso significa que a API **não apenas expõe recursos, mas também fornece informações sobre como interagir com esses recursos dinamicamente**.

- Em um sistema HATEOAS, cada resposta da API contém **links para ações relacionadas**.
- Isso permite que os clientes descubram funcionalidades **sem depender de documentações rígidas**.

#### **Exemplo Histórico: API REST da Amazon (2011)**
- Em **2011**, a Amazon implementou uma versão avançada de sua API de e-commerce usando **HATEOAS**.
- Exemplo de resposta JSON:
  ```json
  {
    "id": 12345,
    "nome": "Produto X",
    "preco": 99.99,
    "_links": {
      "comprar": { "href": "/carrinho/12345", "method": "POST" },
      "avaliacoes": { "href": "/produtos/12345/avaliacoes", "method": "GET" }
    }
  }
  ```
- Aqui, o cliente não precisa saber previamente que `/carrinho/12345` é o endpoint para adicionar um item ao carrinho. O próprio servidor fornece essa informação.
- **Impacto:** Essa abordagem começou a ser adotada por APIs de grandes plataformas como **PayPal e Stripe**, permitindo **descoberta dinâmica de ações** e reduzindo a necessidade de hardcoding nos clientes.

---

## **Ou seja...**

O **Modelo de Maturidade de Richardson** nos ajuda a avaliar o quão RESTful uma API realmente é. Ele permite entender como **a evolução das APIs foi impulsionada pela necessidade de escalabilidade, flexibilidade e simplicidade**.

| Nível | Característica | Exemplo Histórico |
|------|--------------|----------------|
| **0** | Apenas HTTP como transporte (SOAP/XML-RPC) | APIs SOAP do governo dos EUA (anos 2000) |
| **1** | Recursos identificáveis por URL, mas sem uso adequado de métodos HTTP | XML-RPC do Movable Type (2001) |
| **2** | Uso correto dos métodos HTTP (GET, POST, PUT, DELETE, PATCH) | API REST do Twitter (2006) |
| **3** | HATEOAS, permitindo descoberta dinâmica | API REST da Amazon (2011) |

Hoje, a maioria das **APIs modernas** opera no **nível 2**, mas algumas empresas estão adotando **nível 3 com HATEOAS**, especialmente em ambientes complexos, como **microserviços**.

---

## **1.7 Comparação: REST vs SOAP vs RPC**  

### **REST vs SOAP (Simple Object Access Protocol)**  

| Característica | REST | SOAP |
|--------------|------|------|
| **Protocolo** | HTTP | HTTP, SMTP, TCP |
| **Formato de Dados** | JSON, XML | Apenas XML |
| **Facilidade de Uso** | Simples e flexível | Estruturado e verboso |
| **Desempenho** | Alto (leve) | Baixo (mais pesado) |
| **Escalabilidade** | Fácil de escalar | Difícil de escalar |
| **Cache** | Sim | Não |
| **Segurança** | Depende da implementação (HTTPS, JWT, OAuth) | Segurança robusta integrada (WS-Security) |
| **Utilização** | Aplicações web modernas, microserviços | Sistemas bancários, integração empresarial |

- **SOAP** é usado em aplicações críticas (bancos, pagamentos) e sistemas legados.  
- **REST** é mais leve e flexível, sendo mais popular para aplicações web e microsserviços.  

### **REST vs RPC (Remote Procedure Call)**  

| Característica | REST | RPC |
|--------------|------|------|
| **Conceito** | Manipulação de recursos | Chamada direta de funções remotas |
| **Formato de Comunicação** | JSON, XML | Pode ser binário (gRPC, Thrift) ou JSON |
| **Independência de Plataforma** | Alta | Média (pode exigir bibliotecas específicas) |
| **Simplicidade** | Simples, segue HTTP | Pode ser complexo |
| **Utilização** | Aplicações web, microsserviços | Comunicação de alto desempenho entre serviços |

- **RPC** é usado quando a comunicação precisa ser rápida e eficiente (ex.: gRPC no Google).  
- **REST** é mais intuitivo e fácil de usar para integração entre sistemas independentes.  


---

## **1.8 Boas Práticas ao Criar APIs REST 📌**

Criar uma API REST eficiente, intuitiva e escalável vai além de simplesmente expor endpoints HTTP. Seguir boas práticas melhora a **usabilidade**, **manutenção**, **segurança** e **desempenho** da API, tornando-a mais fácil de integrar com outras aplicações. A seguir, apresentamos algumas diretrizes fundamentais para a construção de APIs REST profissionais:

#### **1️. Use Substantivos nos Endpoints e Evite Verbos**
Os endpoints representam **recursos**, portanto, devem ser **nomes de substantivos no plural**, e não **ações ou verbos**.

✅ **Certo**:
```http
GET /contacts        → Obtém todos os contatos  
GET /contacts/{id}   → Obtém um contato específico  
POST /contacts       → Cria um novo contato  
DELETE /contacts/{id} → Remove um contato  
```

❌ **Errado**:
```http
GET /getContacts      → Não precisa do verbo "get", pois o método HTTP já indica a ação  
POST /createContact   → O verbo "create" é desnecessário, pois o método POST já sugere criação  
DELETE /removeContact → O verbo "remove" também é desnecessário  
```

💡 **Regra geral**: O método HTTP já indica a ação (GET para buscar, POST para criar, DELETE para remover, etc.), então o endpoint deve apenas representar o **recurso**.


#### **2️. Use os Códigos de Status HTTP Corretamente**
Os **códigos de status HTTP** ajudam o cliente da API a entender o resultado da requisição. Usar códigos corretos torna a API mais intuitiva e facilita a depuração.

✅ **Principais códigos de status HTTP:**

| Código | Significado | Quando Usar? |
|--------|------------|--------------|
| **200 OK** | Sucesso | Quando uma requisição GET, PUT ou DELETE for bem-sucedida |
| **201 Created** | Recurso Criado | Quando um novo recurso é criado via POST |
| **204 No Content** | Sem Conteúdo | Quando um recurso é excluído com sucesso |
| **400 Bad Request** | Requisição Inválida | Quando os dados enviados são inválidos |
| **401 Unauthorized** | Não Autenticado | Quando o usuário não está autenticado |
| **403 Forbidden** | Acesso Negado | Quando o usuário não tem permissão para acessar o recurso |
| **404 Not Found** | Recurso Não Encontrado | Quando o recurso solicitado não existe |
| **409 Conflict** | Conflito | Quando há um conflito de dados (ex.: tentativa de criar um registro duplicado) |
| **500 Internal Server Error** | Erro Interno | Quando ocorre um erro inesperado no servidor |

💡 **Exemplo Prático:**
Se um usuário tenta buscar um contato que não existe:

❌ **Errado**:
```json
{
  "message": "Contato não encontrado"
}
```
✅ **Certo** (Retorna o código 404):
```http
HTTP/1.1 404 Not Found
{
  "error": "Contato não encontrado"
}
```

#### **3️. Evite Expor Detalhes Internos da API**
Nunca retorne informações sensíveis sobre a API ou stack traces detalhadas em respostas de erro. Isso pode expor vulnerabilidades para possíveis ataques.

❌ **Errado** (Expondo detalhes internos):
```json
{
  "error": "java.lang.NullPointerException at ContactService.java:34"
}
```

✅ **Certo** (Mensagem amigável e segura):
```json
{
  "error": "Erro ao processar a requisição. Tente novamente mais tarde."
}
```
💡 **Dica**: Sempre **trate exceções** e retorne mensagens amigáveis para o cliente, sem expor detalhes da implementação.

#### **4️. Implemente Paginação em Grandes Listas**
Quando a API retorna uma grande quantidade de dados, a **paginação** evita sobrecarregar o servidor e melhora o desempenho.

✅ **Exemplo de Paginação:**
```http
GET /contacts?page=1&size=10
```

✅ **Resposta JSON com metadados de paginação:**
```json
{
  "data": [
    { "id": 1, "nome": "João" },
    { "id": 2, "nome": "Maria" }
  ],
  "page": 1,
  "size": 10,
  "totalPages": 5,
  "totalItems": 50
}
```
💡 **Dica**: Utilize frameworks como o **Spring Data Pageable** para implementar paginação de forma eficiente.

#### **5️. Mantenha a API Intuitiva e Consistente**
Uma API bem projetada deve ser **fácil de usar**, **padronizada** e **previsível**, permitindo que os desenvolvedores consigam integrá-la sem precisar consultar constantemente a documentação.

✅ **Boas práticas para manter a API intuitiva:**
- **Use convenções de nomenclatura consistentes** em todos os endpoints.
- **Padronize os formatos de resposta JSON**, garantindo que todas as respostas sigam a mesma estrutura.
- **Retorne mensagens de erro claras**, explicando o problema e como resolvê-lo.
- **Forneça documentação** da API com exemplos de uso.

#### **6️. Use Versionamento na API**
Com o tempo, APIs evoluem e podem quebrar compatibilidade com versões antigas. Para evitar problemas, sempre versione a API.

✅ **Exemplo de versionamento:**
```http
GET /v1/contacts   → Versão 1  
GET /v2/contacts   → Versão 2  
```

💡 **Dica**: Se a API passar por mudanças grandes, **mantenha versões anteriores** disponíveis para evitar que clientes antigos quebrem.

#### **7️. Documente sua API de Forma Clara**
A documentação é essencial para que outros desenvolvedores entendam como usar sua API. Utilize ferramentas como:

- **Swagger/OpenAPI** → Gera documentação interativa automaticamente.  
- **Postman** → Permite criar coleções de requisições de API documentadas.  
- **Redoc** → Gera documentação HTML a partir do OpenAPI.  

✅ **Exemplo de documentação com Swagger:**
```java
import io.swagger.v3.oas.annotations.Operation;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/contacts")
public class ContactController {

    @Operation(summary = "Lista todos os contatos", description = "Retorna uma lista paginada de contatos")
    @GetMapping
    public List<Contact> getAllContacts() {
        return contactRepository.findAll();
    }
}
```

Seguir essas boas práticas ao criar APIs REST **melhora a usabilidade, segurança e desempenho** da aplicação. Implementar um design padronizado facilita a adoção da API por outros desenvolvedores e reduz a necessidade de documentação extensiva.

**Checklist de Boas Práticas:**
✅ **Nomes de endpoints no plural e sem verbos**  
✅ **Uso correto dos métodos HTTP e códigos de status**  
✅ **Mensagens de erro claras e sem detalhes internos**  
✅ **Paginação para grandes listas**  
✅ **API intuitiva e fácil de entender**  
✅ **Versionamento para evitar quebra de compatibilidade**  
✅ **Documentação clara e bem estruturada**  

Aplique essas práticas para construir APIS REST mais **robustas, escaláveis e fáceis de usar**! 🚀

## **1.9 Conclusão**  

As **APIs REST** se tornaram o padrão para integração de sistemas modernos devido à sua simplicidade, flexibilidade e eficiência. Elas permitem que diferentes aplicações, independentemente da tecnologia utilizada, possam se comunicar utilizando os princípios da web.  

Vamos colocar as mãos na massa e construir uma **API REST com Spring Boot**, explorando como criar, expor e manipular recursos de forma eficiente. Nosso objetivo é compreender os fundamentos e boas práticas do desenvolvimento de APIs REST para aplicações reais.