---
layout: aula
title: "4. Entendendo o conjunto da obra"
parent: Aula 08 - Segurança de APIs com JWT!
nav_order: 5
---

## 4. Entendendo como tudo isso funciona em conjunto 👆🤓

O fluxo de autenticação começa com a chamada ao endpoint `/api/auth/authenticate`, que recebe o nome de usuário e a senha por meio de um objeto `AuthenticationDTO`. O `AuthenticationController` cria uma instância de `UsernamePasswordAuthenticationToken` com esses dados e a envia ao `AuthenticationService`.

O `AuthenticationService`, por sua vez, utiliza o nome de usuário para buscar o usuário correspondente no banco de dados. Caso o usuário exista, ele repassa esse objeto ao `JwtService`, que gera um token JWT contendo informações como o nome do usuário, o identificador (userId), a data de emissão e a data de expiração. Esse token é assinado com a chave privada RSA e retornado ao cliente.

As requisições subsequentes da aplicação cliente devem incluir o JWT no cabeçalho `Authorization`, no formato `Bearer <token>`. Quando essas requisições chegam, o Spring Security, por meio da configuração definida na classe `SecurityConfig`, intercepta os pedidos usando a `SecurityFilterChain`. O token é então validado com a chave pública, utilizando o `JwtDecoder`.

Durante a validação do token, o Spring utiliza o `CustomJwtAuthenticationConverter` para extrair a claim `userId` do JWT. A partir desse valor, a aplicação consulta novamente o banco para obter o objeto `User` correspondente. Esse usuário é encapsulado em uma instância de `UserAuthenticated`, que implementa `UserDetails`, e fornece as authorities necessárias para aplicar as regras de autorização da aplicação.

Com base nas regras definidas em `SecurityConfig`, apenas as rotas públicas (como `/api/auth/**` e `/api/users/register`) estão liberadas. Todas as demais requerem um token válido para acesso.

Esse ciclo garante que apenas usuários autenticados possam acessar os recursos protegidos, e que a verificação do token seja feita de forma segura e independente de sessão, conforme o padrão stateless adotado na aplicação. O uso de chaves RSA garante a integridade do token e impede que ele seja falsificado sem acesso à chave privada.
