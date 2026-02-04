---
layout: aula
title: "6. DevOps e Operação"
parent: Aula 09 - Introdução aos Microsserviços
nav_order: 6
---

# **6. Empacotamento, Implantação e Operação (DevOps)**

A viabilidade dos microsserviços está diretamente ligada à automação e às práticas de DevOps.

## 6.1. Conteinerização

Cada microsserviço é empacotado com todas as suas dependências em uma unidade isolada e portátil chamada **contêiner**. Isso garante consistência entre os ambientes de desenvolvimento, teste e produção.

### Dockerfile

Um arquivo de texto que define passo a passo como construir a imagem de contêiner para uma aplicação.

**Exemplo prático:**
```dockerfile
FROM openjdk:17-slim
WORKDIR /app
COPY target/meu-servico-0.0.1.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**Tecnologia:** **Docker**

## 6.2. Orquestração Local

Para desenvolver e testar um sistema com múltiplos serviços localmente, é preciso uma forma de gerenciar todos os contêineres (serviços, bancos de dados, message broker).

### Docker Compose

Utiliza um arquivo YAML para definir e executar uma aplicação multi-contêiner.

**Exemplo:**
```yaml
version: '3.8'
services:
  servico-pedidos:
    build: ./pedidos
    ports:
      - "8080:8080"
    depends_on:
      - postgres
  
  servico-pagamentos:
    build: ./pagamentos
    ports:
      - "8081:8080"
  
  postgres:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: senha123
```

**Uso:**
```bash
docker-compose up -d  # Subir todos os serviços
docker-compose logs -f  # Ver logs
docker-compose down  # Parar todos
```

## 6.3. CI/CD (Continuous Integration / Continuous Deployment)

Cada microsserviço deve ter seu próprio **pipeline automatizado**. Esse pipeline é responsável por:

1. Compilar o código
2. Executar testes
3. Construir a imagem de contêiner
4. Implantá-la em um ambiente

**Tecnologias Comuns:**
- **GitHub Actions**
- **Jenkins**
- **GitLab CI**
- **CircleCI**

**Exemplo de Pipeline (GitHub Actions):**
```yaml
name: CI/CD
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Set up JDK
        uses: actions/setup-java@v2
      - name: Build
        run: mvn clean package
      - name: Build Docker image
        run: docker build -t meu-servico .
      - name: Push to registry
        run: docker push meu-servico
```

## 6.4. Orquestração em Produção

Gerenciar centenas de contêineres em produção exige uma plataforma de orquestração que lide com:

- Escalabilidade automática
- Recuperação de falhas
- Atualizações sem downtime
- Balanceamento de carga

**Tecnologia:** **Kubernetes** é o padrão de fato da indústria.

### Kubernetes: Conceitos Básicos

**Pods:** Menor unidade deployável (1+ contêineres)  
**Services:** Exposição de pods via rede  
**Deployments:** Gerenciamento de réplicas  
**ConfigMaps:** Configurações externas  
**Secrets:** Dados sensíveis

### 💻 Na Prática

**Startups:** Começam com Docker Compose  
**Scale-ups:** Migram para Kubernetes gerenciado (GKE, EKS)  
**Enterprises:** Kubernetes on-premise ou multi-cloud

