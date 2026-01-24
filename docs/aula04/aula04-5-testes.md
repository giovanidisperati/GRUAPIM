---
layout: aula
title: 5. Testando a API!
parent: Aula 04 - Spring Boot - JPA, Exceções e Documentação!
nav_order: 5
---

## 5. Estrutura do Projeto e testes após a implementação dos exercícios e desafios

Após a implementação das classes acima, nossa estrutura de pacotes ficou da seguinte forma:

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
│   │   │               │   └── AddressController.java
│   │   │               │   └── ContactController.java
│   │   │               ├── model
│   │   │               │   └── Address.java
│   │   │               │   └── Contact.java
│   │   │               ├── repository
│   │   │               │   └── AddressRepository.java
│   │   │               │   └── ContactRepository.java
│   │   │               └── exception
│   │   │                   └── GlobalExceptionHandler.java
│   │   │                   └── ResourceNotFoundException.java
│   │   └── resources
│   │       └── application.properties
│   └── test
│       └── java
└── pom.xml (ou build.gradle)
```

Vamos ver agora como testar nossa API para garantir que todas as funcionalidades implementadas estão funcionando corretamente! 


## 📌 **Teste 1: Criar um Contato (`POST /api/contacts`)**

No Postman ou Insomnia:

1. **Método:** `POST`
2. **URL:** `http://localhost:8080/api/contacts`
3. **Body (JSON):**
```json
{
  "nome": "João Silva",
  "email": "joao.silva@email.com",
  "telefone": "11999999999",
  "addresses": [
    {
      "rua": "Rua A",
      "cidade": "São Paulo",
      "estado": "SP",
      "cep": "12345-678"
    }
  ]
}
```
4. **Resultado Esperado:** 
   - Código HTTP `201 - Created`
   - Resposta com o contato criado, incluindo o endereço associado.


## 📌 **Teste 2: Listar Todos os Contatos (`GET /api/contacts`)**

No Postman ou Insomnia:

1. **Método:** `GET`
2. **URL:** `http://localhost:8080/api/contacts`
3. **Resultado Esperado:** 
   - Código HTTP `200 - OK`
   - Resposta com a lista de contatos criados, incluindo seus endereços.


## 📌 **Teste 3: Buscar Contato por ID (`GET /api/contacts/{id}`)**

No Postman ou Insomnia:

1. **Método:** `GET`
2. **URL:** `http://localhost:8080/api/contacts/1` (Substitua o `1` pelo ID do contato que você deseja consultar)
3. **Resultado Esperado:** 
   - Código HTTP `200 - OK`
   - Retorno do contato correspondente ao ID especificado.


## 📌 **Teste 4: Atualização Parcial de Contato (`PATCH /api/contacts/{id}`)**

No Postman ou Insomnia:

1. **Método:** `PATCH`
2. **URL:** `http://localhost:8080/api/contacts/1` (Substitua o `1` pelo ID do contato que você deseja atualizar)
3. **Body (JSON):**
```json
{
  "nome": "João Silva Jr.",
  "telefone": "11988888888"
}
```
4. **Resultado Esperado:** 
   - Código HTTP `200 - OK`
   - Retorno do contato atualizado, refletindo as alterações enviadas.


## 📌 **Teste 5: Criar um Endereço para um Contato (`POST /api/addresses/contacts/{contactId}`)**

No Postman ou Insomnia:

1. **Método:** `POST`
2. **URL:** `http://localhost:8080/api/addresses/contacts/1` (Substitua o `1` pelo ID do contato que você deseja associar o endereço)
3. **Body (JSON):**
```json
{
  "rua": "Rua B",
  "cidade": "Rio de Janeiro",
  "estado": "RJ",
  "cep": "22222-222"
}
```
4. **Resultado Esperado:** 
   - Código HTTP `201 - Created`
   - O endereço deve ser salvo e associado ao contato especificado.


## 📌 **Teste 6: Listar Endereços de um Contato (`GET /api/addresses/contacts/{contactId}`)**

No Postman ou Insomnia:

1. **Método:** `GET`
2. **URL:** `http://localhost:8080/api/addresses/contacts/1`
3. **Resultado Esperado:** 
   - Código HTTP `200 - OK`
   - Lista dos endereços associados ao contato especificado.


## 📌 **Teste 7: Buscar Contatos por Nome (`GET /api/contacts/search?name=Joao`)**

No Postman ou Insomnia:

1. **Método:** `GET`
2. **URL:** `http://localhost:8080/api/contacts/search?name=Joao`
3. **Resultado Esperado:** 
   - Código HTTP `200 - OK`
   - Retorna uma lista de contatos cujos nomes correspondem parcial ou totalmente ao termo pesquisado.


## 📌 **Teste 8: Exclusão de Contato (`DELETE /api/contacts/{id}`)**

No Postman ou Insomnia:

1. **Método:** `DELETE`
2. **URL:** `http://localhost:8080/api/contacts/1`
3. **Resultado Esperado:** 
   - Código HTTP `204 - No Content`
   - O contato é excluído permanentemente do banco de dados.


## 📌 **Teste 9: Teste de Validação (`POST /api/contacts`)**

Vamos verificar se as validações da entidade `Contact` funcionam corretamente. 

1. **Método:** `POST`
2. **URL:** `http://localhost:8080/api/contacts`
3. **Body (JSON):**
```json
{
  "nome": "",
  "email": "invalidEmail",
  "telefone": "123",
  "addresses": []
}
```
4. **Resultado Esperado:** 
   - Código HTTP `400 - Bad Request`
   - Mensagens de erro indicando o que está inválido:
```json
{
  "nome": "O nome não pode estar vazio",
  "email": "Formato de email inválido",
  "telefone": "O telefone deve ter entre 8 e 15 caracteres",
  "addresses": "O contato deve ter pelo menos um endereço"
}
```