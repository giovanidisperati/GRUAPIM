---
layout: aula
title: Aula 06 - Service Layers e Testes Automatizados
nav_order: 8
has_children: true
permalink: /aula06
---

# Aula 06 – Refatoração dos Controllers, camada de serviço e Introdução aos Testes Automatizados

Na Aula 05, estruturamos nossa API com uso de DTOs, paginamos os endpoints, substituímos o banco em memória por um banco relacional real e documentamos todos os nossos recursos com Swagger. Agora, avançaremos um passo além na maturidade da nossa aplicação: vamos realizar uma **refatoração completa dos nossos controllers**, otimizar o uso do `ModelMapper` para reduzir mapeamentos manuais, separar a lógica da controller na camada de serviço e, ainda, iniciaremos a **introdução aos testes automatizados** da aplicação.

Como sempre, seguiremos um processo prático dando início de onde paramos na aula anterior. Começaremos com a **refatoração dos endpoints** por meio da adoção ao padrão `ResponseEntity`, um dos mais recomendados para retornos HTTP em APIs Spring Boot.