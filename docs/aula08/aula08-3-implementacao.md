---
layout: aula
title: "3. Hora de implementar!"
parent: Aula 08 - Segurança de APIs com JWT!
nav_order: 4
---


## 3. Códigos-fontes 🧑‍💻

Feita essa introdução aos conceitos envolvidos na segurança da aplicação, vamos agora ver o código-fonte envolvido na implementação do JWT!

### 3.1. `SecurityConfig.java`

```java
package br.ifsp.edu.todo.config;

import java.security.interfaces.RSAPrivateKey;
import java.security.interfaces.RSAPublicKey;

import br.ifsp.edu.todo.security.CustomJwtAuthenticationConverter;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.oauth2.jwt.JwtDecoder;
import org.springframework.security.oauth2.jwt.JwtEncoder;
import org.springframework.security.oauth2.jwt.NimbusJwtDecoder;
import org.springframework.security.oauth2.jwt.NimbusJwtEncoder;
import org.springframework.security.web.SecurityFilterChain;

import com.nimbusds.jose.jwk.JWKSet;
import com.nimbusds.jose.jwk.RSAKey;
import com.nimbusds.jose.jwk.source.ImmutableJWKSet;

@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Value("${jwt.public.key}")
    private RSAPublicKey key;
    @Value("${jwt.private.key}")
    private RSAPrivateKey priv;
    
    @Bean
    public CustomJwtAuthenticationConverter customJwtAuthenticationConverter() {
        return new CustomJwtAuthenticationConverter();
    }
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http,
            CustomJwtAuthenticationConverter customJwtAuthenticationConverter) throws Exception {
        http.csrf(csrf -> csrf.disable())
                .authorizeHttpRequests(auth -> auth.requestMatchers("/api/auth/**").permitAll()
                        .requestMatchers("/api/users/register").permitAll().anyRequest().authenticated())
                .oauth2ResourceServer(
                        conf -> conf.jwt(jwt -> jwt.jwtAuthenticationConverter(customJwtAuthenticationConverter)))
                .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS));
        return http.build();
    }
    
    @Bean
    PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
    
    @Bean
    JwtDecoder jwtDecoder() {
        return NimbusJwtDecoder.withPublicKey(this.key).build();
    }
    
    @Bean
    JwtEncoder jwtEncoder() {
        var jwk = new RSAKey.Builder(this.key).privateKey(this.priv).build();
        var jwks = new ImmutableJWKSet<>(new JWKSet(jwk));
        return new NimbusJwtEncoder(jwks);
    }
}
```

A classe `SecurityConfig` é responsável por configurar as regras de segurança da aplicação, definindo como as requisições HTTP serão protegidas e como os tokens JWT serão gerados, validados e utilizados para autenticação. Essa configuração estabelece a aplicação como um Resource Server OAuth2, utilizando criptografia assimétrica com chaves RSA para assinar e validar tokens, e especifica o comportamento de autenticação, autorização e controle de sessão nas rotas expostas pela API.

Essa classe é responsável por:

- Gerar e validar tokens JWT com criptografia assimétrica (RSA).
- Configurar o Spring Security como um Resource Server com proteção de endpoints.
- Integrar uma conversão personalizada das claims JWT.
- Garantir segurança com senhas encriptadas usando BCrypt.

Vamos entendê-la passo-a-passo! 🤓

#### 📦 **Anotações e Estrutura Geral**

- `@Configuration`: Indica que esta classe contém definições de beans para o contexto do Spring.
- `@EnableWebSecurity`: Ativa o suporte à segurança via Spring Security para aplicações web (Servlet).

#### 🔑 **Injeção de Chaves RSA**

```java
@Value("${jwt.public.key}")
private RSAPublicKey key;

@Value("${jwt.private.key}")
private RSAPrivateKey priv;
```

- As chaves **RSA pública e privada** são injetadas a partir do `application.properties`.
- A **chave pública** é usada para **validar** tokens recebidos (no `JwtDecoder`).
- A **chave privada** é usada para **assinar** tokens JWT (no `JwtEncoder`).

⚠️ No momento nossa chave pública está sendo meramente adicionada ao projeto, tendo sido "copiada e colada" (`app.pub` e `app.key`). 

Nosso `app.key`

```
-----BEGIN PRIVATE KEY-----
MIIEvwIBADANBgkqhkiG9w0BAQEFAASCBKkwggSlAgEAAoIBAQDcWWomvlNGyQhA
iB0TcN3sP2VuhZ1xNRPxr58lHswC9Cbtdc2hiSbe/sxAvU1i0O8vaXwICdzRZ1JM
g1TohG9zkqqjZDhyw1f1Ic6YR/OhE6NCpqERy97WMFeW6gJd1i5inHj/W19GAbqK
LhSHGHqIjyo0wlBf58t+qFt9h/EFBVE/LAGQBsg/jHUQCxsLoVI2aSELGIw2oSDF
oiljwLaQl0n9khX5ZbiegN3OkqodzCYHwWyu6aVVj8M1W9RIMiKmKr09s/gf31Nc
3WjvjqhFo1rTuurWGgKAxJLL7zlJqAKjGWbIT4P6h/1Kwxjw6X23St3OmhsG6HIn
+jl1++MrAgMBAAECggEBAMf820wop3pyUOwI3aLcaH7YFx5VZMzvqJdNlvpg1jbE
E2Sn66b1zPLNfOIxLcBG8x8r9Ody1Bi2Vsqc0/5o3KKfdgHvnxAB3Z3dPh2WCDek
lCOVClEVoLzziTuuTdGO5/CWJXdWHcVzIjPxmK34eJXioiLaTYqN3XKqKMdpD0ZG
mtNTGvGf+9fQ4i94t0WqIxpMpGt7NM4RHy3+Onggev0zLiDANC23mWrTsUgect/7
62TYg8g1bKwLAb9wCBT+BiOuCc2wrArRLOJgUkj/F4/gtrR9ima34SvWUyoUaKA0
bi4YBX9l8oJwFGHbU9uFGEMnH0T/V0KtIB7qetReywkCgYEA9cFyfBIQrYISV/OA
+Z0bo3vh2aL0QgKrSXZ924cLt7itQAHNZ2ya+e3JRlTczi5mnWfjPWZ6eJB/8MlH
Gpn12o/POEkU+XjZZSPe1RWGt5g0S3lWqyx9toCS9ACXcN9tGbaqcFSVI73zVTRA
8J9grR0fbGn7jaTlTX2tnlOTQ60CgYEA5YjYpEq4L8UUMFkuj+BsS3u0oEBnzuHd
I9LEHmN+CMPosvabQu5wkJXLuqo2TxRnAznsA8R3pCLkdPGoWMCiWRAsCn979TdY
QbqO2qvBAD2Q19GtY7lIu6C35/enQWzJUMQE3WW0OvjLzZ0l/9mA2FBRR+3F9A1d
rBdnmv0c3TcCgYEAi2i+ggVZcqPbtgrLOk5WVGo9F1GqUBvlgNn30WWNTx4zIaEk
HSxtyaOLTxtq2odV7Kr3LGiKxwPpn/T+Ief+oIp92YcTn+VfJVGw4Z3BezqbR8lA
Uf/+HF5ZfpMrVXtZD4Igs3I33Duv4sCuqhEvLWTc44pHifVloozNxYfRfU0CgYBN
HXa7a6cJ1Yp829l62QlJKtx6Ymj95oAnQu5Ez2ROiZMqXRO4nucOjGUP55Orac1a
FiGm+mC/skFS0MWgW8evaHGDbWU180wheQ35hW6oKAb7myRHtr4q20ouEtQMdQIF
snV39G1iyqeeAsf7dxWElydXpRi2b68i3BIgzhzebQKBgQCdUQuTsqV9y/JFpu6H
c5TVvhG/ubfBspI5DhQqIGijnVBzFT//UfIYMSKJo75qqBEyP2EJSmCsunWsAFsM
TszuiGTkrKcZy9G0wJqPztZZl2F2+bJgnA6nBEV7g5PA4Af+QSmaIhRwqGDAuROR
47jndeyIaMTNETEmOnms+as17g==
-----END PRIVATE KEY-----
```

E abaixo nosso `app.pub`
```
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA3FlqJr5TRskIQIgdE3Dd
7D9lboWdcTUT8a+fJR7MAvQm7XXNoYkm3v7MQL1NYtDvL2l8CAnc0WdSTINU6IRv
c5Kqo2Q4csNX9SHOmEfzoROjQqahEcve1jBXluoCXdYuYpx4/1tfRgG6ii4Uhxh6
iI8qNMJQX+fLfqhbfYfxBQVRPywBkAbIP4x1EAsbC6FSNmkhCxiMNqEgxaIpY8C2
kJdJ/ZIV+WW4noDdzpKqHcwmB8FsrumlVY/DNVvUSDIipiq9PbP4H99TXN1o746o
RaNa07rq1hoCgMSSy+85SagCoxlmyE+D+of9SsMY8Ol9t0rdzpobBuhyJ/o5dfvj
KwIDAQAB
-----END PUBLIC KEY-----
```

ATENÇÃO! **NUNCA USEM ESSA COMBINAÇÃO DE CHAVE PÚBLICA/PRIVADA EM QUALQUER APLICAÇÃO EM PRODUÇÃO!**. Para aplicações reais, gere a chave por meios seguros e nunca disponibilize sua chave privada, que deve ficar fora do repositório de sua aplicação.

#### 🔐 **PasswordEncoder com BCrypt**

```java
@Bean
PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

- Define o algoritmo BCrypt para encriptar senhas dos usuários.
- É uma boa prática recomendada pela Spring Security, por ser resistente a ataques de força bruta.

#### 🔒 **JwtDecoder e JwtEncoder**

```java
@Bean
JwtDecoder jwtDecoder() {
    return NimbusJwtDecoder.withPublicKey(this.key).build();
}

@Bean
JwtEncoder jwtEncoder() {
    var jwk = new RSAKey.Builder(this.key).privateKey(this.priv).build();
    var jwks = new ImmutableJWKSet<>(new JWKSet(jwk));
    return new NimbusJwtEncoder(jwks);
}
```

- Usa a biblioteca **Nimbus JOSE + JWT**, integrada ao Spring Security por meio do pacote `spring-security-oauth2-jose`.
- `JwtDecoder`: valida a assinatura do token recebido usando a **chave pública**.
- `JwtEncoder`: assina o token com a **chave privada**, gerando o JWT para o cliente.
- `JWKSet`: conjunto de chaves (aqui, com uma única RSAKey contendo pública e privada).

#### 🔄 **SecurityFilterChain: o Coração da Configuração**

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http,
        CustomJwtAuthenticationConverter customJwtAuthenticationConverter) throws Exception {
    http.csrf(csrf -> csrf.disable())
        .authorizeHttpRequests(auth -> auth
            .requestMatchers("/api/auth/**").permitAll()
            .requestMatchers("/api/users/register").permitAll()
            .anyRequest().authenticated())
        .oauth2ResourceServer(conf -> conf
            .jwt(jwt -> jwt.jwtAuthenticationConverter(customJwtAuthenticationConverter)))
        .sessionManagement(session -> session
            .sessionCreationPolicy(SessionCreationPolicy.STATELESS));
    return http.build();
}
```

✅ Explicações por partes:

1. **`csrf.disable()`**:
   - Desativa a proteção contra CSRF (Cross-Site Request Forgery).
   - Justificável porque a aplicação é **stateless** (sem sessão) e expõe apenas APIs REST.

2. **`authorizeHttpRequests(...)`**:
   - Define as regras de autorização por padrão de URL:
     - Libera `/api/auth/**` (login, emissão de tokens).
     - Libera `/api/users/register` (registro de novos usuários).
     - Todas as demais rotas exigem autenticação JWT.

3. **`oauth2ResourceServer(...).jwt(...)`**:
   - Configura o projeto como um **Resource Server** OAuth2.
   - Habilita o suporte a tokens JWT para autenticação de requisições.

4. **`jwtAuthenticationConverter(...)`**:
   - Usa o bean `CustomJwtAuthenticationConverter` para converter as claims do JWT em uma `Authentication` do Spring (geralmente contendo authorities/roles). Logo mais veremos como foi implementado nosso conversor custom.

5. **`sessionManagement(...).stateless`**:
   - Garante que **nenhuma sessão HTTP** será criada ou usada.
   - Alinha-se ao paradigma REST, que deve ser **stateless** por definição.

### 3.2. Classe `CustomJwtAuthenticationConverter.java`

A classe CustomJwtAuthenticationConverter é responsável por transformar um token JWT válido em uma instância de AbstractAuthenticationToken, objeto utilizado pelo Spring Security para representar o usuário autenticado em uma requisição. Essa conversão é fundamental para que o framework consiga aplicar corretamente as regras de autorização com base nas informações contidas no token.

```java
package br.ifsp.edu.todo.security;

import br.ifsp.edu.todo.model.User;
import br.ifsp.edu.todo.model.UserAuthenticated;
import br.ifsp.edu.todo.repository.UserRepository;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.convert.converter.Converter;
import org.springframework.security.authentication.AbstractAuthenticationToken;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.oauth2.jwt.Jwt;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.userdetails.UsernameNotFoundException;

import java.util.List;

public class CustomJwtAuthenticationConverter implements Converter<Jwt, AbstractAuthenticationToken> {
    
    @Autowired
    private UserRepository userRepository;
    
    @Override
    public AbstractAuthenticationToken convert(Jwt jwt) {
        UserAuthenticated userAuthenticated = extractUser(jwt);
        List<GrantedAuthority> authorities = List.copyOf(userAuthenticated.getAuthorities());
        return new UsernamePasswordAuthenticationToken(userAuthenticated, null, authorities);
    }
    
    private UserAuthenticated extractUser(Jwt jwt) {
        Long userId = jwt.getClaim("userId");
        User user = userRepository.findById(userId)
                .orElseThrow(() -> new UsernameNotFoundException("User not found with id: " + userId));
        return new UserAuthenticated(user);
    }
}
```

Implementando a interface `Converter<Jwt, AbstractAuthenticationToken>`, a classe define o método `convert(Jwt jwt)`, que será chamado automaticamente toda vez que o Spring Security receber uma requisição autenticada com um JWT válido. O método inicia extraindo o `userId` da claim presente no token, utilizando a função `extractUser()`. Esse método lê o valor da claim `"userId"`, consulta o banco de dados por meio do `UserRepository` e busca a entidade `User` correspondente. Caso o usuário não seja encontrado, uma exceção do tipo `UsernameNotFoundException` é lançada, impedindo o acesso.

Uma vez recuperado o usuário, o método encapsula esse objeto em uma instância de `UserAuthenticated`, que implementa `UserDetails`. Em seguida, o método `convert()` copia as permissões (authorities) do usuário e retorna uma instância de `UsernamePasswordAuthenticationToken`. O segundo parâmetro dessa instância — que normalmente representa as credenciais (como a senha) — é definido como `null`, uma vez que as credenciais já foram verificadas no momento da autenticação e não precisam ser armazenadas ou reutilizadas durante o ciclo da requisição.

Essa abordagem promove a separação de responsabilidades entre autenticação e autorização. O token JWT carrega apenas o identificador do usuário, garantindo que os dados sensíveis permaneçam sob controle do servidor. A resolução do usuário no banco, a partir do ID fornecido, permite associar informações atualizadas e aplicar políticas de segurança mais precisas. Dessa forma, a classe `CustomJwtAuthenticationConverter` atua como um elo entre o token JWT e o contexto de segurança do Spring, permitindo que a autenticação seja concluída de forma segura e controlada.

Percebam que essa implementação **NÃO É** a mais performática possível, já que cada requisição gera uma consulta no banco. Estamos adotando-a apenas por fins de simplicidade e para mostrar que podemos configurar nosso processo de autenticação e autorização livremente. A forma mais correta seria fazer uso de cache (com uso de Redis, por exemplo) e/ou embeddar as claims completas no Token (e confiar nelas!).

### 3.3. Classe `UserAuthenticated.java`

A classe `UserAuthenticated` atua como um adaptador entre a entidade `User` da aplicação e a interface `UserDetails` do Spring Security, permitindo que o mecanismo de autenticação do Spring reconheça os dados do usuário. Ela encapsula um objeto do tipo `User`, armazenando as informações necessárias para autenticação e autorização.

```java
package br.ifsp.edu.todo.model;

import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.GrantedAuthority;
import java.util.Collection;
import java.util.List;

public class UserAuthenticated implements UserDetails {

    private final User user;

    public UserAuthenticated(User user) {
        this.user = user;
    }

    public User getUser() {
        return user;
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return user.getRoles().stream()
                .map(role -> (GrantedAuthority) () -> role.getRoleName().name())
                .toList();
    }

    @Override
    public String getPassword() {
        return user.getPassword();
    }

    @Override
    public String getUsername() {
        return user.getUsername();
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }

}
```

Ao implementar `UserDetails`, a classe fornece métodos obrigatórios como `getUsername()`, `getPassword()` e `getAuthorities()`. O método `getAuthorities()` converte os papéis (roles) associados ao usuário em objetos do tipo `GrantedAuthority`, os quais são utilizados pelo Spring Security para aplicar regras de autorização. Os métodos booleanos `isAccountNonExpired()`, `isAccountNonLocked()`, `isCredentialsNonExpired()` e `isEnabled()` estão todos configurados para retornar `true`, assumindo que todas as contas estão ativas, não expiradas e válidas — embora, em uma aplicação mais robusta, esses valores possam ser derivados de campos da própria entidade `User`.

Por enquanto estamos deixando tudo como `true`, mas evidentemente em uma aplicação real temos que configurar todas essas propriedades. 😊

Além dos métodos da interface `UserDetails`, a classe fornece um método adicional `getUser()`, que permite acessar diretamente o objeto encapsulado `User`. Isso é útil, por exemplo, em controladores que utilizam a anotação `@AuthenticationPrincipal`, permitindo que a aplicação acesse o usuário autenticado de forma prática e segura.

Essa abordagem é comum em projetos que utilizam Spring Security e desejam manter a separação entre as responsabilidades da camada de segurança e as entidades de domínio da aplicação. Ao transformar `User` em `UserAuthenticated`, o Spring Security consegue tratar os dados do usuário conforme suas necessidades internas, sem impor restrições diretas à modelagem da entidade principal.

### 3.4. Classe `JwtService.java`

```java
package br.ifsp.edu.todo.service;

import java.time.Instant;
import org.springframework.security.oauth2.jwt.JwtClaimsSet;
import org.springframework.security.oauth2.jwt.JwtEncoder;
import org.springframework.security.oauth2.jwt.JwtEncoderParameters;
import org.springframework.stereotype.Service;

import br.ifsp.edu.todo.model.User;

@Service
public class JwtService {
    private final JwtEncoder jwtEncoder;
    
    public JwtService(JwtEncoder encoder) {
        this.jwtEncoder = encoder;
    }
    
    public String generateToken(User user) {
        Instant now = Instant.now();
        long expire = 3600L;
    
        var claims = JwtClaimsSet.builder()
                .issuer("spring-security")
                .issuedAt(now)
                .expiresAt(now.plusSeconds(expire))
                .subject(user.getUsername())
                .claim("userId", user.getId())
                .build();
    
        return jwtEncoder.encode(JwtEncoderParameters.from(claims)).getTokenValue();
    }
    
}
```

A classe `JwtService` é responsável por gerar tokens JWT válidos a partir das informações de um usuário autenticado. Para isso, ela injeta um `JwtEncoder` (provavelmente o bean `NimbusJwtEncoder` definido na `SecurityConfig`) e define um método chamado `generateToken(User user)`. Dentro desse método, são criadas claims customizadas que representam o conteúdo do token, incluindo o emissor (`issuer`), o momento da emissão (`issuedAt`), a data de expiração (`expiresAt`, configurada para uma hora adiante), o sujeito (`subject`, representando o `username` do usuário) e o escopo (`scope`), que é extraído das roles do usuário. Essas claims são então agrupadas em um `JwtClaimsSet` e passadas ao encoder, que assina o token com a chave privada. O resultado é uma string JWT que poderá ser utilizada pelo cliente em requisições futuras.

### 3.5. Classe `UserDetailsService.java`

```java
package br.ifsp.edu.todo.service;

import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

import br.ifsp.edu.todo.model.User;
import br.ifsp.edu.todo.model.UserAuthenticated;
import br.ifsp.edu.todo.repository.UserRepository;

@Service
public class UserDetailService implements UserDetailsService {
    private final UserRepository userRepository;

    private UserDetailService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepository.findByUsername(username)
                .orElseThrow(() -> new UsernameNotFoundException("User not found with username: " + username));
        return new UserAuthenticated(user);
    }
}
```

A classe `UserDetailService` implementa a interface `UserDetailsService`, que é uma das exigências do Spring Security para carregar os dados de um usuário com base em seu nome de usuário (username). Quando o método `loadUserByUsername` é chamado (normalmente como parte do fluxo de autenticação), ele consulta o `UserRepository` para buscar o usuário no banco de dados. Se encontrado, esse usuário é encapsulado na classe `UserAuthenticated`, que implementa a interface `UserDetails` e expõe as informações necessárias para a autenticação e autorização (como username, senha e authorities). Caso contrário, uma exceção `UsernameNotFoundException` é lançada.

### 3.6. Classe `AuthenticationService.java`

```java
package br.ifsp.edu.todo.service;

import org.springframework.security.core.Authentication;
import org.springframework.stereotype.Service;

import br.ifsp.edu.todo.model.User;
import br.ifsp.edu.todo.repository.UserRepository;

@Service
public class AuthenticationService {
    private final JwtService jwtService;
    private final UserRepository userRepository;
    
    public AuthenticationService(JwtService jwtService, UserRepository userRepository) {
        this.jwtService = jwtService;
        this.userRepository = userRepository;
    }
    
    public String authenticate(Authentication authentication) {
        String username = authentication.getName();     
        User user = userRepository.findByUsername(username)
            .orElseThrow(() -> new RuntimeException("User not found"));
        return jwtService.generateToken(user);
    }
}
```

A classe `AuthenticationService` atua como intermediária entre a autenticação do Spring Security e a geração do JWT. Ela injeta o `JwtService` e o `UserRepository`. Quando o método `authenticate(Authentication authentication)` é chamado, a classe extrai o nome de usuário do objeto `Authentication`, recupera os dados completos do usuário no repositório, e então invoca o `JwtService` para gerar e retornar um token JWT. Essa classe, portanto, centraliza a lógica de emissão de tokens com base em um usuário autenticado previamente.

É importante notar que essas três classes de serviço se integram no fluxo de login: o `UserDetailService` fornece os detalhes do usuário para autenticação inicial, o `AuthenticationService` emite o JWT após a autenticação, e o `JwtService` realiza a geração do token propriamente dita. É um padrão comum e eficaz em aplicações Spring Boot que adotam autenticação stateless com JWT. 😊

### 3.7. Classe `AuthenticationController.java`

```java
package br.ifsp.edu.todo.controller;

import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import br.ifsp.edu.todo.dto.authentication.AuthenticationDTO;
import br.ifsp.edu.todo.service.AuthenticationService;

@RestController
@RequestMapping("/api/auth")
public class AuthenticationController {
    private final AuthenticationService authenticationService;

    public AuthenticationController(AuthenticationService authenticationService) {
        this.authenticationService = authenticationService;
    }

    @PostMapping("authenticate")
    public String authenticate(@RequestBody AuthenticationDTO request) {
        UsernamePasswordAuthenticationToken authentication = new UsernamePasswordAuthenticationToken(
                request.getUsername(), request.getPassword());
        return authenticationService.authenticate(authentication);
    }
}
```

A classe `AuthenticationController` define o endpoint responsável por processar as requisições de autenticação de usuários em uma aplicação protegida por JWT. Anotada com `@RestController` e mapeada com `@RequestMapping("/api/auth")`, ela expõe uma rota HTTP do tipo `POST` em `/api/auth/authenticate`, a qual espera receber no corpo da requisição um objeto do tipo `AuthenticationDTO`. Esse DTO contém os dados de autenticação fornecidos pelo cliente, como nome de usuário e senha.

Ao receber a requisição, o método `authenticate` instancia um objeto `UsernamePasswordAuthenticationToken`, utilizando os dados extraídos do DTO. Esse token representa uma tentativa de autenticação e é enviado ao serviço `AuthenticationService`, que cuida da validação do usuário e geração do token JWT. O retorno do serviço é uma `String` com o JWT, que é devolvida como resposta ao cliente.

Portanto, essa classe atua como o ponto de entrada para o processo de autenticação, intermediando a comunicação entre o cliente e a lógica de autenticação definida no serviço, e devolvendo o token JWT a ser usado nas requisições futuras protegidas.

### 3.8 Classe `AuthenticationDTO.java`

```java
package br.ifsp.edu.todo.dto.authentication;

import jakarta.validation.constraints.NotBlank;
import lombok.Data;

@Data
public class AuthenticationDTO {
    @NotBlank(message = "Please, enter your username.")
    private String username;
    @NotBlank(message = "Please, enter your password.")
    private String password;
}
```

Essa aqui é bem simples, vamos pular a explicação para evitar da explicação ficar gigante.
