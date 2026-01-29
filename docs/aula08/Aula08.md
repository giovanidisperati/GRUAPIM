# Aula 08 – Segurança em APIs REST com JWT e Spring Security

Na Aula 08, construímos uma API REST de gerenciamento de tarefas (To-Do List) com uma arquitetura em camadas, separação de responsabilidades, validação, tratamento de exceções e testes automatizados. Agora, vamos dar mais um passo importante rumo a uma aplicação robusta e pronta para produção: **a implementação de segurança**!

Imagine agora as seguintes situações de uso da aplicação:

- Como usuário, eu quero me cadastrar e fazer login na aplicação.
- Como usuário autenticado, eu quero criar tarefas e garantir que **apenas eu** possa visualizá-las, editá-las ou excluí-las.
- Como administrador, eu preciso visualizar **todas as tarefas de todos os usuários**, a fim de gerar relatórios ou fazer auditorias.

Para atender esses requisitos, precisaremos evoluir nossa API incluindo:

- Cadastro de usuários com senha criptografada.
- Autenticação com **JWT (JSON Web Token)**.
- Proteção de endpoints com **Spring Security**, exigindo token válido para acesso.
- Associação de tarefas a usuários autenticados.
- Definição de **roles** (USER e ADMIN), permitindo controle de acesso baseado em permissões.

Com isso, nossa aplicação deixará de ser uma API pública e passará a oferecer funcionalidades **autenticadas** e **autorizadas**, respeitando o contexto de segurança de cada operação!

## 1. Introdução à Segurança em APIs REST

A segurança de APIs RESTful é um aspecto essencial no desenvolvimento de aplicações modernas, principalmente diante da crescente adoção de arquiteturas distribuídas. Uma abordagem bastante adotada para esse fim é o uso de JSON Web Tokens (JWT), padrão aberto (RFC 7519) que permite a representação segura de informações entre duas partes. O Spring Security oferece suporte nativo à autenticação baseada em JWT, dispensando a necessidade de bibliotecas externas e implementações manuais complexas. Esse suporte é viabilizado por meio do módulo OAuth2 Resource Server, que integra de forma transparente com o ecossistema Spring Boot. Antes de abordarmos a implementação do JWT com Spring Boot, entretanto, vamos entender um pouco dos fundamentos dessa tecnologia.

### 1.1. JSON Web Token (JWT): características e aplicações

O JSON Web Token (JWT) é um padrão aberto (RFC 7519) que define uma forma **compacta e auto-contida** de transmitir informações seguras entre partes como um objeto JSON ([JSON Web Token Introduction - jwt.io](https://jwt.io/introduction#:~:text=JSON%20Web%20Token%20,pair%20using%20RSA%20or%20ECDSA)). Internamente, um JWT (tipicamente no formato JWS – JSON Web Signature) é composto por três segmentos codificados em Base64URL, separados por pontos: um **cabeçalho** (header), um **payload** (carga útil) e uma **assinatura** ([Stateless Sessions for Stateful Minds: JWTs Explained and How You Can Make The Switch](https://auth0.com/blog/stateless-auth-for-stateful-minds/#:~:text=,validated%20against%20tampering%20using%20this)). O cabeçalho contém metadados sobre o token (por exemplo, `{ "alg":"HS256", "typ":"JWT"}`), enquanto o payload guarda declarações (“claims”) sobre a entidade (usuário) ou o token (como identificador de usuário, papéis, escopos, data de emissão/expiração, etc.) ([A Beginner's Guide to JWTs | Okta Developer](https://developer.okta.com/blog/2020/12/21/beginners-guide-to-jwt#:~:text=A%20JWT%20is%20a%20structured,are%20two%20types%20of%20JWTs)). A assinatura criptográfica, calculada sobre o cabeçalho e o payload, garante que o token não foi alterado e que foi emitido por uma fonte confiável ([JSON Web Token Introduction - jwt.io](https://jwt.io/introduction#:~:text=JSON%20Web%20Token%20,pair%20using%20RSA%20or%20ECDSA)). Opcionalmente, além de assinar (JWS), um JWT pode ser criptografado (JWE – JSON Web Encryption) para ocultar seu conteúdo de terceiros; porém, seu uso mais comum é na forma assinada, em que o payload permanece visível para quem possui o token. 

Para assinar e verificar JWT há dois modelos principais de algoritmos: simétricos e assimétricos.

Em **algoritmos simétricos** (ex.: HS256 – HMAC com SHA-256), há apenas uma chave secreta compartilhada entre as partes: o emissor do token usa esse segredo para assinar, e o destinatário usa o mesmo segredo para verificar a assinatura ([Signing Algorithms](https://auth0.com/docs/get-started/applications/signing-algorithms#:~:text=%2A%20HS256%20%28HMAC%20with%20SHA,This)).

Por outro lado, em **algoritmos assimétricos** (ex.: RS256 – assinatura RSA com SHA-256, ou algoritmos ECDSA), existe um par de chaves: o emissor mantém a chave *privada* para assinar o JWT, e os demais usam a chave *pública* correspondente para validar o token. A vantagem do RSA (ou ECDSA) é que apenas o detentor da chave privada pode emitir tokens válidos, enquanto qualquer serviço com a chave pública pode conferir a assinatura sem conhecer segredos compartilhados. Por essa razão, é geralmente recomendado usar RS256 em produção: se a chave privada for comprometida, ela pode ser rotacionada facilmente sem precisar redeploy dos serviços consumidores do token. Algoritmos simétricos (HMAC) são mais simples de implementar, mas exigem cuidado adicional para não vazar o segredo usado em múltiplas aplicações. Note-se também que JWTs suportam tanto apenas a assinatura (JWS) quanto também a criptografia de conteúdo (JWE) quando é necessário manter os dados ocultos, mas o mecanismo de assinatura é o que garante a **integridade** do token.

As principais motivações para adotar JWT decorrem de sua natureza **sem estado** (“stateless”) e padronização ampla. Como cada JWT é auto-suficiente, contendo todas as informações de autenticação/autorização necessárias, o servidor destinatário pode validar o token **localmente** (checando a assinatura) sem precisar consultar um banco de dados central a cada requisição. Isso facilita a escalabilidade horizontal de sistemas distribuídos: por não haver sessões armazenadas no servidor, vários serviços podem operar de forma independente e em paralelo, bastando compartilhar a chave de verificação. Em uma arquitetura de APIs REST, isso significa que a API receptor não precisa fazer round-trips ao provedor de identidade para checar o token, o que reduz latência e melhora desempenho. Por exemplo, provedores OAuth 2.0/OIDC costumam emitir *access tokens* no formato JWT exatamente para esse fim: o serviço de recursos (API) recebe o JWT e o valida localmente via assinatura, sem nova chamada à autoridade de autenticação. Além disso, o JWT pode carregar dados adicionais como o ID do usuário, papéis e escopos de acesso. Isso elimina várias consultas ao banco de dados na hora de autorizar ações, pois o serviço pode extrair essas informações diretamente do payload. Em outras palavras, ao embutir informações (como “claims” de autorização) no token, reduz-se a quantidade de “conversa” (chattiness) necessária no backend.

Outra motivação é a simplicidade para casos de *Single Sign-On* (SSO) e cenários *cross-domain*. O JWT é leve, baseado em JSON e independente de linguagem, de modo que pode ser transmitido entre domínios ou sistemas heterogêneos facilmente. Por exemplo, em fluxos SSO modernos um usuário pode autenticar em um domínio (ou aplicação) e então receber um JWT, que é enviado a outro domínio para comprovar a identidade sem nova entrada de credenciais. De fato, o padrão é usado amplamente em SSO devido ao seu pequeno overhead e capacidade de ser usado entre diferentes domínios. Dessa forma, JWT atende bem a casos onde múltiplos serviços ou aplicações em nuvem precisam compartilhar um mecanismo comum de autenticação sem depender de sessões centralizadas.

No dia a dia, o JWT aparece principalmente em autenticação de **APIs web** e **microsserviços**. É comum que APIs REST requeiram um token JWT no cabeçalho HTTP `Authorization: Bearer <token>` para liberar acesso a recursos protegidos. Nessas aplicações, o token indica quem é o usuário ou serviço requisitante e que permissões ele tem, e é validado em cada chamada sem criar estado no servidor. Em arquiteturas de microsserviços, o JWT facilita a **propagação de identidade** pelo sistema: um serviço que recebeu o token pode ler dele o ID do usuário ou outros dados de contexto e repassá-los a serviços downstream, sem precisar rediscutir autenticação a cada salto. Isto é útil tanto para autorizações internas entre microserviços quanto para cenários máquina-a-máquina (service-to-service), em que sistemas trocam JWTs para provar identidade e escopos de acesso. Igualmente, em fluxos OAuth/OIDC, o JWT pode aparecer tanto como access token quanto como ID token, dando suporte a logins federados em aplicações mobile, SPAs ou qualquer cliente web.

Fora do contexto de APIs, JWT também é usado em trocas de informações seguras entre servidores, aplicações móveis e bibliotecas de integração. Por exemplo, um cliente autenticado pode usar um JWT para fazer login em um sistema externo sem reenvio de senha (o token carrega a prova da autenticação prévia). Sua portabilidade e o fato de ser baseado em um padrão aberto o tornam também opção natural em soluções que exigem interoperabilidade entre diferentes empresas ou provedores de identidade.

### 1.2 Comparação com outras soluções de autenticação/autorização

Em aplicações web tradicionais, costuma-se usar **autenticação baseada em sessão** com cookies HTTP. Neste modelo stateful, o servidor gera um *session ID* (geralmente opaco) após o login, armazena os dados da sessão (ou um link a eles) internamente e envia o ID ao cliente via cookie. A cada requisição, o servidor consulta seu banco de sessões para carregar o contexto do usuário. Esse modelo permite revogar sessões instantaneamente (por exemplo, apagando a sessão no servidor) e utilizar proteções nativas de cookies (HttpOnly, SameSite, etc.), mas exige manter estado no servidor, o que pode dificultar o escalonamento horizontal. Em contraste, a autenticação baseada em tokens (como JWT) não requer armazenamento de sessão: cada requisição carrega o token autônomo, tornando o sistema **stateless**. Isso facilita a escalabilidade e a resiliência dos serviços, mas tem o custo de tornar a **revogação de sessão mais complexa**, pois o servidor não “lembra” quais tokens já emitiu – ele só valida a assinatura.

Outra comparação importante é entre **tokens JWT e tokens opacos** (como aqueles usados em algumas implementações OAuth2). Um token opaco é apenas um identificador aleatório vinculado a uma entrada no servidor de autorização; para verificar um token desse tipo é preciso fazer uma chamada de *introspecção* ao servidor que o emitiu. Em contrapartida, um JWT é *interpretável*: contém dados (claims) codificados em JSON e uma assinatura que qualquer parte confiável pode verificar localmente. Ou seja, o servidor de recursos não precisa chamar ninguém para validar um JWT – ele confia na assinatura. Como dito, isso torna a validação do JWT muito rápida (basta computar a assinatura localmente, sem acesso à rede), enquanto tokens opacos requerem um round-trip a um banco de dados ou endpoint de introspecção, o que adiciona latência. Em compensação, tokens opacos têm vantagem na revogação: basta removê-los do banco de dados e ficam inválidos imediatamente. Já para tokens JWT a revogação só ocorre quando o token expira (a menos que se crie uma lista de bloqueio), o que pode levar a atrasos indesejados . De forma resumida: JWT é **stateless e auto-contido**, facilitando autenticação distribuída sem estado compartilhado, enquanto soluções baseadas em sessão ou tokens opacos delegam a verificação a um servidor central e permitem revogação imediata em troca de maior acoplamento com esse servidor.

### 1.3 Limitações e desvantagens do JWT

Apesar das vantagens, o uso de JWT traz desvantagens importantes que devem ser consideradas. Em primeiro lugar, há a questão da **revogação e invalidação**. Como o JWT é validado apenas pela assinatura local, o servidor não sabe se o token foi tornado inválido antes de seu vencimento natural. Assim, se um token for vazado ou se o usuário tiver privilégios revogados, o JWT ainda poderá ser aceito até expirar, a não ser que se adote algum mecanismo extra (como listas negras de identificadores ou tempos de vida muito curtos). Isso torna a segurança mais frágil comparado a sessões que podem ser finalizadas pelo servidor a qualquer momento.

Além disso, os tokens JWT podem crescer de tamanho. Como carregam claims em JSON, cada requisição transporta esse peso extra. Se o payload incluir muitos dados (por exemplo, papéis extensos ou outros atributos), o tráfego de rede aumenta, podendo degradar o desempenho geral se usado de forma excessiva. Em arquiteturas de alto tráfego, isso deve ser balanceado: embora os tokens evitem ida constante ao banco de dados, tokens muito grandes podem virar gargalo de banda.

Do ponto de vista de segurança, a especificação JWT exige atenção especial. A validação de um token JWT corretamente é mais complexa do que parece – há muitas armadilhas e casos de borda. Por exemplo, já foram exploradas brechas relacionadas ao uso indevido do algoritmo `none`, que desabilita a verificação da assinatura. Portanto, recomenda-se usar bibliotecas maduras e configurá-las para aceitar apenas algoritmos esperados. Além disso, os dados no payload de um JWT assinado **não são criptografados** – qualquer servidor ou atacante que obtenha o token pode ler seu conteúdo. Assim, não se deve colocar informações sensíveis sem encriptá-las (caso se queira confidencialidade).

Também existem riscos no armazenamento dos tokens. Se um JWT for enviado ao cliente (como em uma aplicação web), armazená-lo em *localStorage* ou em JavaScript pode expor o token a ataques XSS. Por outro lado, armazenar o JWT em cookie HttpOnly evita acesso via script mas pode reintroduzir vulnerabilidade a CSRF caso não sejam adotadas contramedidas (como SameSite). Em resumo, o uso de JWT envolve um **trade-off entre segurança e desempenho**: ao facilitar operações sem estado e reduzir chamadas backend, abre espaço para novas formas de ataque se não for bem protegido. Boas práticas, como usar tokens de curta duração, algoritmos fortes, validação cuidadosa de claims (issuer, audience, expiração) e armazenamento seguro, são essenciais para mitigar esses riscos.  


### 1.4 JWT (JWS) vs JWE: assinatura e confidencialidade

Um JSON Web Token (JWT) padrão é apenas **assinado** (JWS) e codificado em Base64, mas **não criptografado**. Isso significa que qualquer um com o token pode ver seu conteúdo legível (claims) mas não pode alterá-lo sem invalidar a assinatura. Ou seja, um JWT pode ser assinado (JWS) ou encriptado (JWE). A assinatura (JWS) garante integridade, mas não oculta as informações. Para confidencialidade, existe o padrão JWE (JSON Web Encryption): cifrando o token inteiro, apenas o emissor e o destino podem ler as claims. Em outras palavras, “faz sentido criptografar um JWS se você quiser manter informações sensíveis ocultas do *bearer* (cliente) ou terceiros”. 

Contudo, o uso de JWE depende da necessidade de privacidade e do suporte dos componentes da aplicação (clientes e serviços consumidores) para decodificá-lo. Se o ambiente de consumo só suporta JWS, não há como usar JWE sem grande refatoração. Para ilustrar, o **RFC 7519** (especificação do JWT) recomenda que, caso o token contenha informações **privacidade-sensíveis**, deve-se tomar medidas para evitar vazamento: usar um JWT encriptado (JWE) *e* autenticar o destinatário, **ou** garantir que o JWT em texto claro só seja transmitido por canais seguros (HTTPS/TLS), **ou** simplesmente omitir esses dados sensíveis.

Nesse sentido, a criptografia de um claim de identificação (como, por exemplo, o ID do usuário) traria principalmente **confidencialidade**. Se a aplicação considera o ID do usuário como dado sensível ou pessoal (por exemplo, por regras de privacidade ou GDPR), o JWE impediria que qualquer interceptor lesse esse valor. Em um *ambiente de microserviços*, isso poderia evitar que serviços intermediários ou logs exponham o ID em texto claro. Além disso, JWE combinado com assinatura dupla (assinar-depois-encriptar) garante ao mesmo tempo integridade e confidencialidade dos claims. Ou seja, a vantagem é proteger **segredos ou PII** que se consideraria arriscado deixar no token, já que sem criptografia o payload é público. Como destacam fontes oficiais e especialistas, “se você não se sente confortável em expor até mesmo o ID/email do usuário (dados que podem ser considerados pessoais), alguns clientes podem optar por proteger até isso”. 

Por outro lado, a **criptografia de JWT acarreta custos significativos**. A operação de cifrar/descriptografar token consome mais CPU e é mais complexa de implementar corretamente. Estudos e guias de mercado apontam que manter um mecanismo de criptografia robusto é difícil: é preciso gerenciar chaves de forma segura, usar algoritmos modernos (e.g. AES-GCM) e evitar ataques (padding-oracle etc.) ([JSON Web Token for Java - OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html#:~:text=)). Em aplicações de alto tráfego, portanto, isso pode se tornar gargalo de desempenho. Além disso, todos os consumidores do token (serviços backend, APIs, etc.) precisam suportar JWE e ter as chaves para decifrar, o que complica a arquitetura de microsserviços.

Outro ponto: **criptografar não impede todos os ataques comuns**. Por exemplo, se um token for roubado via XSS ou através de um canal inseguro, o invasor ganha tanto o token cifrado quanto a capacidade de usá-lo como *bearer* – a criptografia só esconde o conteúdo, mas o token continua válido (a não ser que se implemente PoP ou lista de revogação). Em muitos casos, basta proteger o token em trânsito (TLS) e em repouso (cookie seguro ou armazenamento do dispositivo) para mitigar esses riscos. O próprio OWASP alerta que o payload de um JWT “não costuma ser criptografado, então deve-se revisar se há dados sensíveis incluídos” ([WSTG - Latest | OWASP Foundation](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/06-Session_Management_Testing/10-Testing_JSON_Web_Tokens#:~:text=The%20payload%20is%20it%20not,inappropriate%20data%20included%20within%20it)). Ou seja, a abordagem recomendada é **limitar o que vai no token**, evitando expor segredos, em vez de cifrá-lo simplesmente por precaução.

Em síntese, as **vantagens da criptografia** do claim de usuário são: 
- *Confidencialidade adicional*: somente partes autorizadas leem o ID (útil para dados pessoais sensíveis)
- *Poder incluir mais informação*: permite adicionar dados extras ao token sem medo de vazá-los (sob responsabilidade do JWE). 
- *Suporte oficial*: JWE é padrão oficial (RFC 7516) e algumas plataformas (Auth0, Okta, etc.) oferecem suporte a ele. 

Já as **desvantagens** são: 
- *Complexidade e desempenho*: criptografia torna o sistema mais complexo de configurar (chaves, algoritmos) e lento para criptografar/descriptografar a cada request.
- *Sobrecarga de infra*: todos os microsserviços/APIs devem gastar ciclos para decifrar, e precisam de um mecanismo seguro de distribuição de chaves. 
- *Limita uso no cliente*: em aplicações SPA ou mobile, o cliente raramente precisa ler o conteúdo do token (só envia no header). Mas se você criptografar, o cliente não consegue decodificar o token, o que talvez exija alterar fluxo (por exemplo, o token só é usado no backend). Na prática, um SPA não ganharia muito com isso, já que o próprio usuário sabe seu ID. 
- *Não resolve ameaças reais*: criptografia não impede token roubado de ser usado (bearer) nem evita XSS; e mesmo criptografado, o token deve seguir os cuidados padrões (TLS, expirations). 

Considerando isso, em **produção**, a prática comum é **não criptografar** o JWT apenas para proteger a informação de identificação do usuário. Em vez disso, recomenda-se seguir boas práticas de segurança: usar **assinatura forte** (p. ex. RSA ou ECDSA em vez de HS256 quando possível), transmitir sempre via HTTPS e usar short-lived tokens com `exp` curto. Em geral, coloque **somente o mínimo necessário** de dados no token – o ID do usuário no claim `sub` e talvez roles/permissões – e considere a comunicação com um backend seguro para obter qualquer outra informação.

Como ensina o guia da Curity, se houver necessidade de dados sensíveis, é mais seguro mantê-los fora do token (chamando, por exemplo, um endpoint de *userinfo*) do que cifrar o token inteiro ([JWT Security Best Practices | Curity](https://curity.io/resources/learn/jwt-best-practices/#:~:text=When%20it%20comes%20to%20access,directly%20in%20the%20ID%20token)).

Se ainda quiser confidencialidade maior sem criptografar o JWT, há **alternativas mais seguras ou eficientes**:

- **Tokens opacos + introspecção**: em vez de um JWT auto-contido, emita um token opaco (string aleatória) e armazene seus dados no servidor (ou use OAuth2 introspection). Assim, o token em si não revela nada ao cliente, e o servidor recupera o ID do usuário (e dados sensíveis) quando necessário. Essa abordagem requer manter estado no auth server, mas evita exposição de claims no cliente. 

- **BFF ou proxy de autenticação**: use um *backend for frontend* ou API gateway que gerencie o token. O cliente lida com um token leve, o gateway adiciona ou troca por tokens com mais privilégios para os microsserviços. O BFF também pode manter informações sensíveis no servidor, nunca expondo-as ao cliente.

- **Criptografia apenas de campos específicos**: em casos extremos, você poderia cifrar apenas certos valores dentro do JSON antes de colocar no JWT (não padronizado, arriscado) e deixar o JWT como JWS normal. Mas isso usualmente traz complexidade similar ao JWE sem todos os benefícios. 

- **Usuários autenticados**: lembre-se que, em um SPA ou app mobile, o próprio usuário já “conhece” seu ID (ele fez login) – logo, esconder o ID dele do próprio usuário geralmente não faz sentido. Se for preciso verificar algo no backend, o JWT assinado já garante que ele não foi adulterado.  

Em resumo, **criptografar o ID do usuário num JWT não é normalmente necessário**. A solução mais indicada é usar um JWT assinado (JWS) com claims mínimas, transportado via TLS, e tratar qualquer dado sensível fora do token. Se seu caso de uso realmente exige confidencialidade extra (por exemplo, jurisdições com regulações estritas de privacidade), considere JWE, mas avalie bem o custo. A maioria das fontes oficiais (JWT RFC, OWASP, Auth0) enfatiza: *não inclua dados sensíveis em tokens não criptografados*, e prefira soluções como canais seguros e design de token enxuto em vez de cifrar o JWT completo. 😊

### 1.5 JWT Com Spring Boot

A arquitetura de autenticação com JWT funciona da seguinte maneira: o cliente realiza uma requisição autenticada (geralmente via HTTP Basic) ao endpoint de emissão de tokens, como `/token`. Uma vez validada a autenticação, o servidor emite um JWT assinado digitalmente. Esse token é então enviado pelo cliente nas requisições subsequentes, por meio do cabeçalho `Authorization: Bearer <token>`. O servidor, por sua vez, valida o token recebido antes de permitir o acesso ao recurso protegido.

Como vimos, os tokens JWT são compostos por três partes codificadas em Base64 URL-safe e separadas por pontos: o header, que define o algoritmo de assinatura (como RS256); o payload, que contém os dados (claims) como nome de usuário e escopo de permissões; e a assinatura criptográfica, gerada com uma chave secreta ou privada. No contexto da nossa aplicação, optaremos por usar criptografia assimétrica, com chaves RSA, por ser considerada uma abordagem mais segura e escalável: a chave privada permanece protegida no servidor, enquanto a chave pública é usada para validar tokens.

No Spring Boot, configuramos essa abordagem por meio da anotação `@EnableWebSecurity` e da definição de um `SecurityFilterChain`, que configura a aplicação como um Resource Server. Dentro dessa cadeia, desativamos o CSRF (por se tratar de uma API stateless), exigimos autenticação para qualquer requisição e configuramos o sistema como sem estado (stateless), ou seja, sem uso de sessão HTTP. Para habilitar o suporte a JWTs, utilizamos o método `oauth2ResourceServer().jwt()`. Essa configuração requer um `JwtDecoder`, que pode ser definido como um bean utilizando a biblioteca `NimbusJwtDecoder`, passando a chave pública RSA obtida a partir de um arquivo `.pem` ou semelhante.

É importante ressaltar que o **SecurityFilterChain** do Spring Security é implementado via _Servlet Filters_, não via interceptores do Spring MVC. Internamente, o Spring registra um único `FilterChainProxy` (geralmente via `DelegatingFilterProxy`) no container web. O `FilterChainProxy` atua como um *middleware* que engloba várias cadeias de filtros de segurança configuradas na aplicação ([Architecture :: Spring Security](https://docs.spring.io/spring-security/reference/servlet/architecture.html#:~:text=Spring%20Security%E2%80%99s%20Servlet%20support%20is,typically%20wrapped%20in%20a%20DelegatingFilterProxy)). Cada *SecurityFilterChain* define uma lista ordenada de filtros de segurança (beans) a serem aplicados a certas requisições (por exemplo, filtrando por padrão de URL ou outro `RequestMatcher`). A cada requisição HTTP recebida, o `FilterChainProxy` escolhe a primeira *SecurityFilterChain* cujo critério de correspondência (`RequestMatcher`) bate com a requisição, e então invoca sequencialmente os filtros dessa cadeia. Em outras palavras, para cada requisição o Spring percorre suas cadeias de segurança configuradas e executa os filtros da cadeia correspondente.

Em termos de padrão de projeto, o **SecurityFilterChain** funciona como uma cadeia de responsabilidade (_Chain of Responsibility_ similar a “middleware”): é executado *antes* que o Spring MVC despache a requisição para os controladores, interceptando todas as requisições no nível do servlet. Assim, a cadeia de filtros de segurança do Spring Security age **globalmente e antes** dos interceptores de MVC, controlando autenticação, autorização e outras proteções. Essa abordagem baseado em filtros permite, por exemplo, aplicar regras de segurança dinâmicas por requisição (não apenas por URL estática), pois o `FilterChainProxy` pode usar qualquer detalhe da requisição (via `RequestMatcher`) para decidir quais filtros executar. Ou seja, temos muita flexibilidade ao configurar a segurança de nossa aplicação. 👨‍🏭

A geração dos tokens ocorre, por exemplo, em uma classe que implemente um `TokenService`, que utiliza o `JwtEncoder` (também baseado em RSA) para assinar os tokens. Os claims do token podem incluir informações como o emissor (`issuer`), momento da emissão (`issuedAt`), tempo de expiração (`expiresAt`) e o nome do usuário (`subject`), além de um escopo de permissões que pode ser derivado das roles do usuário autenticado. O token gerado é então retornado por um controlador, como um `AuthController`, que deve ser responsável por lidar com a autenticação e emissão do JWT.

Essa abordagem elimina a necessidade de um servidor de autorização dedicado, o que é suficiente para aplicações monolíticas ou sistemas pequenos. No entanto, conforme o sistema evolui e se torna distribuído, é recomendável adotar um Authorization Server (como o Spring Authorization Server ou serviços como Auth0, Keycloak e Okta), especialmente em cenários que exigem tokens de atualização (refresh tokens), isolamento de responsabilidades ou múltiplos serviços independentes. Veremos isso posteriormente ao lidarmos com Microsserviços. 🤠

Para realizar testes, pode-se utilizar ferramentas como o Postman. Basta realizar uma requisição POST ao endpoint que implementa a autenticação para obter o token JWT, e então usá-lo como Bearer Token nas requisições subsequentes. 

Quando expomos uma API REST para o mundo, precisamos garantir que **apenas usuários autorizados** possam acessá-la — especialmente se a API manipula dados sensíveis ou pessoais, como no nosso caso. Para isso, é comum aplicar camadas de segurança que validam quem está fazendo a requisição (autenticação) e se essa pessoa tem permissão para executá-la (autorização).

A imagem abaixo sintetiza o fluxo que acabamos de descrever ✍️

![Diagrama de Sequência - Fluxo de uma requisição em uma Aplicação Spring Boot com uso de JWT](diagrama_sequencia_jwt.png)

---

## 2. Estrutura do Projeto 

Vamos dar continuidade, como mencionamos inicialmente, à aplicação de To-do List. Nossa aplicação ganhará novas classes e pastas relacionadas à segurança, conforme o projeto evolui. A seguir, apresentaremos passo a passo os principais arquivos e explicações, mantendo a mesma organização e estilo da Aula 07.

Após a implementação, a estrutura do projeto ficará como mostrado a seguir:

```
.
├── src/
│   └── main/
│       └── java/
│           └── br/ifsp/edu/todo/
│               ├── config/
│               │   ├── ModelMapperConfig.java
│               │   └── SecurityConfig.java
│               ├── controller/
│               │   ├── AuthenticationController.java
│               │   ├── TaskController.java
│               │   └── UserController.java
│               ├── dto/
│               │   ├── authentication/
│               │   │   ├── AuthenticationDTO.java
│               │   │   └── UserRegistrationDTO.java
│               │   ├── page/
│               │   │   └── PagedResponse.java
│               │   └── task/
│               │       ├── TaskRequestDTO.java
│               │       └── TaskResponseDTO.java
│               ├── exception/
│               │   ├── ErrorResponse.java
│               │   ├── GlobalExceptionHandler.java
│               │   ├── InvalidTaskStateException.java
│               │   ├── ResourceNotFoundException.java
│               │   └── UserAlreadyExistsException.java
│               ├── mapper/
│               │   └── PagedResponseMapper.java
│               ├── model/
│               │   ├── enumerations/
│               │   │   ├── Category.java
│               │   │   ├── ERole.java
│               │   │   └── Priority.java
│               │   ├── Role.java
│               │   ├── Task.java
│               │   ├── User.java
│               │   └── UserAuthenticated.java
│               ├── repository/
│               ├── security/
│               │   └── CustomJwtAuthenticationConverter.java
│               ├── service/
│               │   ├── AuthenticationService.java
│               │   ├── JwtService.java
│               │   ├── TaskService.java
│               │   ├── UserDetailService.java
│               │   ├── UserService.java
│               │   └── TodoApplication.java
│   └── resources/
│       ├── db/migration/
│       │   ├── V1_CreateTables.sql
│       │   ├── V2_InsertDefaultRoles.sql
│       │   └── V3_InsertDefaultUsers.sql
│       ├── static/
│       ├── templates/
│       ├── app.key
│       ├── app.pub
│       └── application.properties
├── test/
│   └── java/
│       └── br/ifsp/edu/todo/
│           └── task/
│               ├── TaskControllerTest.java
│               ├── TaskServiceTest.java
│           └── TodoApplicationTests.java
```

Vamos começar explorando as configurações de segurança com JWT, a estrutura de autenticação e o novo fluxo de controle de acesso!

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

---

## 4. Entendendo como tudo isso funciona em conjunto 👆🤓

O fluxo de autenticação começa com a chamada ao endpoint `/api/auth/authenticate`, que recebe o nome de usuário e a senha por meio de um objeto `AuthenticationDTO`. O `AuthenticationController` cria uma instância de `UsernamePasswordAuthenticationToken` com esses dados e a envia ao `AuthenticationService`.

O `AuthenticationService`, por sua vez, utiliza o nome de usuário para buscar o usuário correspondente no banco de dados. Caso o usuário exista, ele repassa esse objeto ao `JwtService`, que gera um token JWT contendo informações como o nome do usuário, o identificador (userId), a data de emissão e a data de expiração. Esse token é assinado com a chave privada RSA e retornado ao cliente.

As requisições subsequentes da aplicação cliente devem incluir o JWT no cabeçalho `Authorization`, no formato `Bearer <token>`. Quando essas requisições chegam, o Spring Security, por meio da configuração definida na classe `SecurityConfig`, intercepta os pedidos usando a `SecurityFilterChain`. O token é então validado com a chave pública, utilizando o `JwtDecoder`.

Durante a validação do token, o Spring utiliza o `CustomJwtAuthenticationConverter` para extrair a claim `userId` do JWT. A partir desse valor, a aplicação consulta novamente o banco para obter o objeto `User` correspondente. Esse usuário é encapsulado em uma instância de `UserAuthenticated`, que implementa `UserDetails`, e fornece as authorities necessárias para aplicar as regras de autorização da aplicação.

Com base nas regras definidas em `SecurityConfig`, apenas as rotas públicas (como `/api/auth/**` e `/api/users/register`) estão liberadas. Todas as demais requerem um token válido para acesso.

Esse ciclo garante que apenas usuários autenticados possam acessar os recursos protegidos, e que a verificação do token seja feita de forma segura e independente de sessão, conforme o padrão stateless adotado na aplicação. O uso de chaves RSA garante a integridade do token e impede que ele seja falsificado sem acesso à chave privada.

---

## 5. É isso? 🙏

Sim, é isso! Chegamos ao final da Aula 08. Essa é, sem dúvidas, a implementação mais complicada que fizemos no Spring (até agora, pelo menos).

A partir daqui, nossa API passou a contar com autenticação e autorização baseadas em JWT, o que permite proteger os dados dos usuários e garantir que apenas operações autorizadas sejam permitidas. Implementamos geração e validação de tokens, criptografia com RSA, controle de acesso via roles e configuração completa do Spring Security para funcionar como um Resource Server stateless.

A abordagem adotada atende bem ao contexto de uma aplicação monolítica, com autenticação centralizada e controle direto sobre os usuários. Com isso, podemos garantir que cada usuário só possa acessar suas próprias tarefas e que administradores possam ter acesso ampliado. A separação entre os componentes (serviço, controlador, conversor e configuração de segurança) também facilita futuras manutenções ou evoluções.

Evidentemente há muito a melhorar se tratando de uma aplicação real, mas essa é uma introdução que balanceia a complexidade inerente dessa implementação com a apresentação dos conceitos envolvidos.

A partir daqui, podemos começar a pensar em desafios mais avançados, como:

- Controle de acesso baseado em roles por endpoint;
- Geração de refresh tokens;
- Integração com OAuth2 e OpenID Connect;
- Cache de dados do usuário para melhorar performance;
- E migração para uma arquitetura baseada em microsserviços.

Por enquanto, temos tudo que precisamos para garantir autenticação segura e acesso controlado na nossa API REST.

---

## 6. E agora, José? 🦜

Com a segurança implementada, nossa aplicação já é capaz de autenticar usuários e restringir o acesso a recursos protegidos. A partir daqui, podemos evoluir a API para lidar com novos cenários e boas práticas, como:

- Adicionar **refresh tokens**, permitindo sessões mais duradouras sem precisar de login constante.
- Integrar com **provedores de identidade externos**, como **Keycloak**, **Auth0** ou o **Spring Authorization Server**.
- Implementar **controle de acesso por recurso**, garantindo que um usuário só possa acessar ou modificar as próprias tarefas.
- Pensar em soluções de **cache**, como **Redis**, para evitar a consulta ao banco a cada requisição autenticada.
- Embutir mais dados diretamente no JWT, reduzindo chamadas desnecessárias ao banco.
- Preparar a aplicação para uso em **arquiteturas distribuídas**, como em ambientes de microsserviços.

Apesar dessas possibilidades, nosso foco agora deve ser consolidar a base construída. A autenticação com JWT já está ativa e funcional, e as próximas etapas envolvem aplicar essa segurança ao restante da aplicação.

Na próxima aula, vamos introduzir o controle de acesso por roles, diferenciando permissões de usuários comuns e administradores. Também vamos explorar o uso da anotação `@AuthenticationPrincipal` e aplicar as restrições diretamente nos controllers.

Vale destacar que ainda não refatoramos os controladores existentes para aproveitar essa estrutura, e que as `migrations` incluídas no projeto também não foram detalhadas até aqui. Sabem o que isso significa? Que teremos, como sempre...

---

## 7. Exercícios

Refatore os controladores da aplicação para utilizar a autenticação implementada com JWT. Aplique as regras de negócio descritas no início da aula:

- Usuários só podem acessar, editar ou excluir as **próprias tarefas**.
- Administradores podem visualizar **todas as tarefas**.

### Dicas práticas

- **Inclua o token nas requisições**: ao testar a API no Postman ou outro cliente HTTP, envie o JWT no cabeçalho:

  ```
  Authorization: Bearer <token>
  ```

- **Acesse o usuário logado com `@AuthenticationPrincipal`**:

  ```java
  @GetMapping("/me/tasks")
  public List<TaskResponseDTO> listUserTasks(@AuthenticationPrincipal UserAuthenticated authentication) {
      return taskService.findTasksByUser(authentication.getUser());
  }
  ```

- **Associe a tarefa ao usuário autenticado**:

  ```java
  @PostMapping("/tasks")
  public ResponseEntity<TaskResponseDTO> createTask(@RequestBody TaskRequestDTO dto,
                                                    @AuthenticationPrincipal UserAuthenticated authentication) {
      return ResponseEntity.ok(taskService.createTask(dto, authentication.getUser()));
  }
  ```

- **Proteja endpoints por role com `@PreAuthorize`**:

  ```java
  @PreAuthorize("hasRole('ADMIN')")
  @GetMapping("/admin/tasks")
  public List<TaskResponseDTO> listAllTasks() {
      return taskService.findAll();
  }
  ```