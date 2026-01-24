---
layout: aula
title: 3. Conclusão e exercícios
parent: Aula 03 - Introdução às APIs REST e ao Spring Boot
nav_order: 3
---

## 3. Conclusão e Próximos Passos

Nesta aula vimos que uma **API REST** segue um conjunto de padrões baseados no protocolo HTTP, permitindo a comunicação entre sistemas de maneira **padronizada, previsível e escalável**. Para garantir que uma API seja realmente RESTful, entendemos a importância do **modelo de maturidade de Richardson**, que nos ajuda a avaliar a conformidade de uma API com os princípios REST.  

Além disso, iniciamos a introdução ao **Spring Boot**, uma tecnologia que simplifica o desenvolvimento de aplicações Java, eliminando a necessidade de configurações manuais complexas e permitindo que possamos rapidamente criar e expor endpoints RESTful. Aprendemos a estrutura de um projeto Spring Boot e implementamos os principais componentes necessários para a construção de uma **API de lista de contatos**.

Nas próximas aulas avançaremos na construção de **APIs mais robustas e completas**. Trabalharemos com novos conceitos essenciais para um desenvolvimento profissional, incluindo:

🔹 **Validações e Tratamento de Erros**: Garantindo que nossa API retorne respostas consistentes e evite entradas inválidas.  
🔹 **Persistência e Banco de Dados**: Integração da API com um banco de dados real para armazenar os contatos de forma permanente.  
🔹 **Autenticação e Segurança**: Protegendo a API com mecanismos de autenticação, como OAuth e JWT.  
🔹 **Boas Práticas e Documentação**: Como tornar nossa API mais compreensível e fácil de usar para outros desenvolvedores.  

Antes disso, entretanto, é necessário solidificar os conteúdos vistos até aqui. 

## **4. Exercícios**  

Os exercícios a seguir têm como objetivo reforçar os conceitos apresentados na aula e permitir que você pratique o desenvolvimento de APIs REST com **Spring Boot**.  

> 📌 **Instruções:**  
> - Os exercícios devem ser entregues no **Moodle** dentro do prazo estipulado.  
> - Sempre **comente o código** explicando as principais partes da implementação.  
> - Utilize **Postman, Insomnia ou cURL** para testar os endpoints criados.  
> - O código-fonte deve ser enviado em um repositório no GitHub (ou em anexo no Moodle, se preferir).  

---

### **1️⃣ Exercício 1 - Criando um Novo Endpoint GET**  

Crie um novo endpoint **GET** em `ContactController` que permita buscar **contatos pelo nome**.  

📌 **Requisitos:**  
- O método deve receber o nome como um **parâmetro de URL** (`/api/contacts/search?name=João`).
- O método deve retornar uma lista de contatos que correspondam ao nome fornecido.
- Caso nenhum contato seja encontrado, retorne uma **lista vazia**.  

📌 **Dicas:**  
- Você pode precisar modificar a interface `ContactRepository` para adicionar um método de busca personalizada.

📝 **Saída esperada (JSON)**:
```json
[
  {
    "id": 1,
    "nome": "João Silva",
    "telefone": "9999-9999",
    "email": "joao@email.com"
  }
]
```

### **2️⃣ Exercício 2 - Implementando um Método PATCH**  

Adicione um novo método **PATCH** à API, permitindo que o usuário **atualize apenas um ou mais campos** de um contato, sem precisar enviar todos os dados.  

📌 **Requisitos:**  
- O método deve permitir alterar apenas os campos enviados na requisição.  
- Se o campo **não for enviado**, o valor original deve ser mantido.  
- Retorne o contato atualizado após a alteração.  
- Caso o ID fornecido não exista, retorne um **erro 404**.  

📌 **Exemplo de chamada:**  
```json
PATCH /api/contacts/1
{
  "email": "novoemail@email.com"
}
```

📝 **Saída esperada (JSON)**:
```json
{
  "id": 1,
  "nome": "João Silva",
  "telefone": "9999-9999",
  "email": "novoemail@email.com"
}
```

### **3️⃣ Exercício 3 - Comparando REST e SOAP**  

Agora que você já conhece APIs REST, faça uma **pesquisa sobre APIs SOAP** e responda às perguntas abaixo com suas próprias palavras.  

📌 **Questões:**  
1. Qual a principal diferença entre **REST e SOAP**?  
2. Em quais **cenários SOAP ainda é amplamente utilizado**?  
3. Quais são as vantagens e desvantagens de **usar REST ao invés de SOAP**?  
4. O que é **WS-Security** e como ele se compara à segurança em APIs REST?  
5. Explique o modelo de **maturidade de Richardson** e em que nível SOAP se encaixa.  

---

### 1️⃣ Desafio 1 - Criando um Novo Modelo de Dados**  

Atualmente, nossa API gerencia apenas **contatos**. Agora, queremos adicionar um novo recurso: **endereços**.  

📌 **Tarefas:**  
1. **Crie uma nova entidade `Address`**, com os seguintes atributos:  
   - `id` (Long, auto-increment)  
   - `rua` (String)  
   - `cidade` (String)  
   - `estado` (String)  
   - `cep` (String)  
   - `contactId` (Long) - Chave estrangeira referenciando um contato.  
2. **Crie uma relação bidirecional entre contatos e endereços.**
3. **Crie um repositório (`AddressRepository`)** para gerenciar os endereços.  
4. **Implemente um novo `AddressController`** para adicionar e recuperar endereços.  
5. **Crie uma rota GET `/api/contacts/{id}/addresses`** para listar todos os endereços de um contato específico.  

---

### **2️⃣ Desafio 2 - Melhorando a Validação dos Dados**  

Atualmente, nossa API aceita qualquer valor no cadastro de contatos. Precisamos garantir que os dados sejam válidos antes de inseri-los no banco de dados.  

📌 **Requisitos:**  
- **Adicione validações** à entidade `Contact`, utilizando a anotação `@Valid`.  
- Implemente regras como:  
  - O campo `nome` **não pode estar vazio**.  
  - O campo `email` deve ter um formato válido (`@Email`).  
  - O campo `telefone` deve ter **no mínimo 8 e no máximo 15 caracteres**.  

📌 **Exemplo de resposta para entrada inválida:**  
Se tentarmos criar um contato com um telefone inválido:
```json
{
  "nome": "Maria",
  "telefone": "123",
  "email": "maria@email.com"
}
```
A API deve retornar **HTTP 400** e uma mensagem de erro:
```json
{
  "erro": "O telefone deve ter entre 8 e 15 caracteres"
}
```

### **📌 Instruções Finais**  

- ✅ Para os exercícios **práticos (1 e 2)** a entrega esperada é o código das novas rotas e prints das requisições no Postman ou Insomnia.   **Envie um link do GitHub** ou um arquivo **.zip** com o código-fonte.
- ✅ Para o **exercício teórico (3)**, envie um arquivo **.pdf ou .txt** com as respostas.
- ✅ A **entrega dos desafios terá um prazo estendido** até o Domingo posterior à data de entrega dos exercícios, também devendo ser entregue o código de implementação e os prints dos testes.  
- ✅Teste todas as funcionalidades antes de enviar e garanta que o código está funcionando.  

## **Bons estudos e mãos à obra! 🛠🔥**