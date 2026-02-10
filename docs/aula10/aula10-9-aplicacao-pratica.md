---
layout: aula
title: "9. Aplicação Prática"
parent: Aula 10 - Domain-Driven Design
nav_order: 9
---

# **9. Aplicação Prática - Do Conceito ao Código**

Vamos entender quando usar (e quando NÃO usar) DDD, com dicas práticas e armadilhas comuns.

## 9.1. Quando Usar DDD?

DDD **não é** para todos os projetos. Ele brilha em contextos específicos:

### ✅ Use DDD quando:

#### 1. **Domínio Complexo**
- Muitas regras de negócio inter-relacionadas
- Lógica que muda frequentemente
- Especialistas de domínio com conhecimento profundo
- **Exemplo**: Sistema financeiro, e-commerce com múltiplas integrações, plataforma de saúde

#### 2. **Sistema de Longa Vida**
- Projeto que evoluirá por anos
- Base de código que será mantida por times diferentes
- Investimento em qualidade compensa no longo prazo

#### 3. **Equipe Colaborativa**
- Desenvolvedores e especialistas podem trabalhar juntos
- Cultura de aprendizado mútuo
- Comunicação frequente é viável

#### 4. **Múltiplos Bounded Contexts**
- Sistema grande que precisa ser dividido
- Diferentes partes evoluem em ritmos diferentes
- Times distribuídos por áreas de negócio

#### 5. **Preparação para Microsserviços**
- Plano de migração para arquitetura distribuída
- Necessidade de definir fronteiras claras desde já

---

## 9.2. Quando NÃO Usar DDD?

### ❌ Evite DDD quando:

#### 1. **CRUD Simples**
- Sistema é basicamente cadastros e listagens
- Lógica de negócio é trivial ou inexistente
- **Alternativa**: Framework MVC tradicional, scaffolding

**Exemplo**: Sistema de cadastro de contatos, blog pessoal, intranet simples

#### 2. **Protótipos e MVPs**
- Incerteza sobre o modelo de negócio
- Tudo muda rapidamente
- Velocidade > estrutura
- **Alternativa**: Rails, Django, ferramentas RAD (Rapid Application Development)

#### 3. **Prazo Muito Curto**
- Deadline imediato (semanas)
- Time sem experiência em DDD
- Curva de aprendizado inviável

#### 4. **Equipe Pequena e Isolada**
- Desenvolvedores não têm acesso a especialistas
- Falta comunicação entre negócio e TI
- DDD precisa de colaboração para funcionar

#### 5. **Domínio Genérico**
- Problema já resolvido por bibliotecas prontas
- Não há diferencial competitivo
- **Exemplo**: Sistema de autenticação padrão, envio de e-mails

---

## 9.3. Estudos de Caso Reais

### 📊 Caso 1: E-commerce com DDD (Sucesso)

**Empresa**: Loja online de moda  
**Problema**: Sistema monolítico com lógica confusa de pedidos, estoque e pagamento  

**Solução com DDD**:
- Identificou 5 Bounded Contexts: Catalog, Cart, Order, Payment, Shipping
- Cada contexto com sua linguagem ubíqua
- Event Storming revelou processos ocultos

**Resultados**:
- ✅ Redução de 40% em bugs de negócio
- ✅ Times podem deployar independentemente
- ✅ Facilidade para adicionar novos canais (mobile, parceiros)

---

### 📊 Caso 2: Fintech com DDD (Sucesso Parcial)

**Empresa**: Startup de crédito  
**Problema**: Muitas regras de análise de crédito, compliance complexo  

**Solução com DDD**:
- Modelou agregado `CreditApplication` com invariantes rígidas
- Usou Domain Events para auditoria
- Separou Core Domain (análise de crédito) de Supporting (notificações)

**Resultados**:
- ✅ Modelo de domínio expressivo e testável
- ✅ Auditoria completa via eventos
- ⚠️ Overhead inicial alto (3 meses de modelagem)
- ⚠️ Curva de aprendizado da equipe

---

### 📊 Caso 3: Sistema Interno Simples (Overengineering)

**Empresa**: Empresa de logística  
**Problema**: Sistema de agendamento de reuniões interno  

**Erro**: Aplicaram DDD com Aggregates, VOs, ACL para CRUD simples

**Resultado**:
- ❌ Complexidade desnecessária
- ❌ Desenvolvimento 3x mais lento
- ❌ Time frustrado com "burocracia"

**Lição**: DDD-Lite para problema simples = desperdício

---

## 9.4. Armadilhas Comuns

### 🚨 Armadilha 1: DDD-Lite (Padrões Táticos Sem Estratégia)

**Sintoma**:
- Cria Entities, VOs, Aggregates sem conversar com especialistas
- Bounded Contexts definidos arbitrariamente
- Linguagem ubíqua inexistente

**Consequência**:
- Complexidade acidental sem ganho de modelagem
- "DDD não funciona para nós"

**Solução**:
- Sempre comece pela estratégia (Event Storming, Context Mapping)
- Tática sem estratégia = overengineering

---

### 🚨 Armadilha 2: Aggregates Muito Grandes

**Sintoma**:
- Agregado com 10+ entidades internas
- Transações lentas e bloqueadas
- Concorrência problemática

**Consequência**:
- Performance ruim
- Deadlocks
- Dificuldade de evolução

**Solução**:
- Regra: Aggregates pequenos (1-3 entidades)
- Use eventual consistency entre agregados
- Não tente garantir tudo atomicamente

---

### 🚨 Armadilha 3: Anemia Disfarçada

**Sintoma**:
```java
class Order {
    List<OrderLine> lines;
    
    // apenas getters e setters
}

class OrderService { // toda lógica aqui
    void addLine(Order order, ...) { ... }
    void calculateTotal(Order order) { ... }
}
```

**Consequência**:
- Model anêmico apesar da estrutura DDD
- Lógica espalhada em serviços

**Solução**:
- Comportamento vai **dentro** do agregado
- Services apenas orquestram, não implementam regras

---

### 🚨 Armadilha 4: Ignorar Eventual Consistency

**Sintoma**:
- Tentar transações distribuídas entre agregados
- Bloquear tudo para garantir consistência imediata

**Consequência**:
- Performance degradada
- Arquitetura não escala

**Solução**:
- Aceite eventual consistency
- Use Domain Events para sincronização
- Implemente Sagas quando necessário

---

## 9.5. Dicas de Ouro

### 💡 Dica 1: "Entenda Primeiro. Modele Depois."

A principal armadilha do DDD é **tratar como técnica o que é, na verdade, filosofia e estratégia**.

- DDD não é sobre criar `AggregateRoot.java`
- DDD não é sobre usar `@Entity` com `@Service`
- DDD não é sobre separar pastas com nomes bonitos

**DDD é sobre entender profundamente o domínio de negócio**, encontrar os limites naturais de cada parte, **explicitar os contratos entre elas** e garantir que **as decisões corretas sejam expressas no código** — com clareza, com coesão, e com autonomia.

---

### 💡 Dica 2: Comece Pequeno

Não tente aplicar DDD em tudo de uma vez:

1. **Escolha um subdomínio Core** para começar
2. **Faça Event Storming** com essa área específica
3. **Modele apenas esse contexto** com padrões táticos
4. **Aprenda com os erros**
5. **Expanda gradualmente** para outros contextos

---

### 💡 Dica 3: Invista em Conhecimento

DDD tem curva de aprendizado:

- **Leia os livros clássicos** (Evans, Vernon)
- **Participe de workshops** (Event Storming, Context Mapping)
- **Pratique modelagem** com casos reais
- **Não copie exemplos** sem entender o contexto

---

### 💡 Dica 4: Quando Parar de Modelar?

É comum ouvir: "mas quando o modelo está bom o suficiente?"

Uma boa regra:

> **Pare de modelar quando você consegue implementar as funcionalidades mais importantes do sistema de forma expressiva, compreensível e sem gambiarras.**

Se você ainda precisa "dar um jeitinho" para contornar inconsistências, criar validações espalhadas ou reimplementar lógicas em vários lugares... provavelmente ainda falta refinar o modelo ou dividir melhor os contextos.

---

### 💡 Dica 5: DDD é Evolutivo

O modelo de domínio **nunca está "pronto"**:

- Linguagem ubíqua evolui com o negócio
- Bounded Contexts podem ser reorganizados
- Agregados podem ser quebrados ou mesclados

**DDD não é waterfall**. É um processo contínuo de refinamento.

---

## 9.6. Checklist Prático

Antes de começar DDD no seu projeto, verifique:

### ✅ Estratégia
- [ ] Conversei com especialistas de domínio?
- [ ] Identifiquei subdomínios (Core, Supporting, Generic)?
- [ ] Defini Bounded Contexts com fronteiras claras?
- [ ] Criei um Context Map mostrando relacionamentos?
- [ ] Estabeleci linguagem ubíqua para cada contexto?

### ✅ Tática
- [ ] Modelei Aggregates com invariantes protegidas?
- [ ] Usei Value Objects para eliminar primitive obsession?
- [ ] Encapsulei lógica de negócio dentro das entidades?
- [ ] Defini Domain Events para comunicação?
- [ ] Criei Repositories para Aggregate Roots?

### ✅ Organização
- [ ] Estruturei pacotes por domínio (não por camada técnica)?
- [ ] Separei domain, application e infrastructure?
- [ ] Times estão alinhados com Bounded Contexts?

---

## 9.7. Conclusão

O Domain-Driven Design, quando bem aplicado, nos ajuda a **desenvolver sistemas mais compreensíveis, sustentáveis e alinhados com a realidade do negócio**. Ele **não é uma bala de prata**, mas é um conjunto de princípios que **evita que a complexidade do sistema cresça de forma descontrolada**.

Mais do que um padrão de arquitetura, o DDD nos convida a sermos **modeladores conscientes**, **desenvolvedores curiosos** e **colaboradores ativos do entendimento do problema**.

Se adotado com propósito, ele transforma não apenas o código — mas a forma como equipes inteiras pensam, falam e constroem software.

Na próxima e última seção, vamos ver o **Projeto Final** da disciplina, onde você aplicará tudo que aprendeu.

---

### ⏭️ Próxima Seção

Na próxima e última seção, vamos detalhar o **Projeto Final** da disciplina.

**[→ Ir para Projeto Final](aula10-10-projeto-final)**

---

## Referências Desta Seção

**EVANS, Eric.** *Domain-Driven Design: Tackling Complexity in the Heart of Software.* Boston: Addison-Wesley, 2003.

**VERNON, Vaughn.** *Implementing Domain-Driven Design.* Boston: Addison-Wesley, 2013.

**VERNON, Vaughn.** *Domain-Driven Design Distilled.* Boston: Addison-Wesley, 2016.

**KHONONOV, Vlad.** *Learning Domain-Driven Design: Aligning Software Architecture and Business Strategy.* Sebastopol: O'Reilly Media, 2021.
