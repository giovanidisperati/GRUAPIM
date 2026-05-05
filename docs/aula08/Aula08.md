---
layout: aula
title: Aula 08 - Segurança de APIs com JWT!
nav_order: 10
has_children: true
permalink: /aula08
---

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