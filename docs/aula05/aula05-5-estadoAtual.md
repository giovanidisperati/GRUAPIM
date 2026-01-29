---
layout: aula
title: 5. Como está nossa aplicação?
parent: Aula 05 - Aprimorando nossas APIs Rest
nav_order: 5
---
## 5. Estado atual do sistema

Ao longo dessa aula estruturamos a aplicação seguindo boas práticas de organização de código e responsabilidades bem definidas entre os pacotes. Adotamos conceitos importantes como uso de DTOs para entrada e saída de dados, integração com o banco de dados relacional (MySQL) via Spring Data JPA, paginação e ordenação com `Pageable`, e mapeamento automático entre entidades e DTOs com ModelMapper. Também discutimos aspectos teóricos como a diferença entre entidades e DTOs, uso de Lombok para redução de boilerplate, princípios de Inversão de Controle e Injeção de Dependência, além da documentação interativa com Swagger/OpenAPI.

Nossa aplicação foi organizada em camadas: `config` para configurações globais, `controller` para os endpoints, `dto` para os objetos de transporte, `model` para as entidades JPA, `repository` para o acesso a dados, `exception` para tratamentos personalizados de erro, e `resources` para configurações de ambiente. Essa estrutura é nada mais do que uma sequência lógica do que já havíamos adotado nas aulas anteriores.

Nesse estágio atual, portanto, nosso sistema está com a seguinte estrutura de diretórios:

```
contacts-api/
├── pom.xml
├── src/
│   └── main/
│       ├── java/
│       │   └── br/
│       │       └── ifsp/
│       │           └── contacts/
│       │               ├── config/
│       │               │   └── MapperConfig.java
│       │               ├── controller/
│       │               │   ├── AddressController.java
│       │               │   └── ContactController.java
│       │               ├── dto/
│       │               │   ├── address/
│       │               │   │   ├── AddressRequestDTO.java
│       │               │   │   └── AddressResponseDTO.java
│       │               │   └── contact/
│       │               │       ├── ContactRequestDTO.java
│       │               │       ├── ContactResponseDTO.java
│       │               │       └── ContactPatchDTO.java
│       │               ├── exception/
│       │               │   └── ResourceNotFoundException.java
│       │               │   └── GlobalExceptionHandler.java
│       │               ├── model/
│       │               │   ├── Address.java
│       │               │   └── Contact.java
│       │               ├── repository/
│       │               │   ├── AddressRepository.java
│       │               │   └── ContactRepository.java
│       │               └── ContactsApiApplication.java
│       └── resources/
│           ├── application.properties
│           └── static/
└── target/
```

Os DTOs, Repositórios e Modelos já foram apresentados anteriormente, bem como o `MapperConfig` e o `ContactsApiApplication`. As exceções foram apresentadas na aula passada e não tiveram mudança. 

Vamos ver, portanto, como estão os nossos controllers.

**`ContactController.java`**

O código-fonte de nosso `ContactController` ficou da seguinte maneira:

```java
package br.ifsp.contacts.controller;

import org.modelmapper.ModelMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PatchMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseStatus;
import org.springframework.web.bind.annotation.RestController;

import br.ifsp.contacts.dto.contact.ContactPatchDTO;
import br.ifsp.contacts.dto.contact.ContactRequestDTO;
import br.ifsp.contacts.dto.contact.ContactResponseDTO;
import br.ifsp.contacts.exception.ResourceNotFoundException;
import br.ifsp.contacts.model.Address;
import br.ifsp.contacts.model.Contact;
import br.ifsp.contacts.repository.ContactRepository;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.tags.Tag;
import jakarta.validation.Valid;

@RestController
@RequestMapping("/api/contacts")
@Tag(name = "Contatos", description = "Operações relacionadas a contatos")
@Validated
public class ContactController {

    @Autowired
    private ContactRepository contactRepository;

    @Autowired
    private ModelMapper modelMapper;

    @Operation(summary = "Buscar todos os contatos paginados")
    @GetMapping
    public Page<ContactResponseDTO> getAllContacts(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size,
            @RequestParam(defaultValue = "nome") String sort) {

        Pageable pageable = PageRequest.of(page, size, Sort.by(sort));
        Page<Contact> contacts = contactRepository.findAll(pageable);
        return contacts.map(contact -> modelMapper.map(contact, ContactResponseDTO.class));
    }

    @Operation(summary = "Buscar contato por ID")
    @GetMapping("{id}")
    public ContactResponseDTO getContactById(@PathVariable Long id) {
        Contact contact = contactRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Contato não encontrado: " + id));
        return modelMapper.map(contact, ContactResponseDTO.class);
    }

    @Operation(summary = "Criar novo contato")
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public ContactResponseDTO createContact(@Valid @RequestBody ContactRequestDTO dto) {
        // Mapeia os campos simples
        Contact contact = new Contact(dto.getNome(), dto.getEmail(), dto.getTelefone());

        // Mapeia os endereços manualmente
        var addresses = dto.getAddresses().stream()
                .map(addrDto -> {
                    Address address = new Address();
                    address.setRua(addrDto.getRua());
                    address.setCidade(addrDto.getCidade());
                    address.setEstado(addrDto.getEstado());
                    address.setCep(addrDto.getCep());
                    address.setContact(contact); 
                    return address;
                }).toList();

        contact.setAddresses(addresses);

        Contact saved = contactRepository.save(contact);
        return modelMapper.map(saved, ContactResponseDTO.class);
    }

    @Operation(summary = "Atualizar contato por ID")
    @PutMapping("/{id}")
    public ContactResponseDTO updateContact(@PathVariable Long id, @Valid @RequestBody ContactRequestDTO dto) {
        Contact existingContact = contactRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Contato não encontrado: " + id));

        modelMapper.map(dto, existingContact);
        existingContact.getAddresses().forEach(addr -> addr.setContact(existingContact));
        Contact updated = contactRepository.save(existingContact);
        return modelMapper.map(updated, ContactResponseDTO.class);
    }

    @Operation(summary = "Atualização parcial de contato")
    @PatchMapping("/{id}")
    public ContactResponseDTO updateContactPartial(@PathVariable Long id, @RequestBody ContactPatchDTO dto) {
        Contact existingContact = contactRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Contato não encontrado: " + id));

        dto.getNome().ifPresent(existingContact::setNome);
        dto.getEmail().ifPresent(existingContact::setEmail);
        dto.getTelefone().ifPresent(existingContact::setTelefone);

        Contact saved = contactRepository.save(existingContact);
        return modelMapper.map(saved, ContactResponseDTO.class);
    }

    @Operation(summary = "Excluir contato")
    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void deleteContact(@PathVariable Long id) {
        contactRepository.deleteById(id);
    }

    @Operation(summary = "Buscar contatos pelo nome")
    @GetMapping("/search")
    public Page<ContactResponseDTO> searchContactsByName(@RequestParam String name, Pageable pageable) {
        return contactRepository.findByNomeContainingIgnoreCase(name, pageable)
                .map(contact -> modelMapper.map(contact, ContactResponseDTO.class));
    }
}
```

Optamos por manter os `import` para que vocês não se confundam, pois há annotations com os mesmos nomes em pacotes diferentes quando estamos falando de `Page` e `Pageable`.

Já o nosso `AddressController` ficou como mostrado a seguir.

**`AddressController.java`**

```java
package br.ifsp.contacts.controller;

import org.modelmapper.ModelMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseStatus;
import org.springframework.web.bind.annotation.RestController;

import br.ifsp.contacts.dto.address.AddressRequestDTO;
import br.ifsp.contacts.dto.address.AddressResponseDTO;
import br.ifsp.contacts.exception.ResourceNotFoundException;
import br.ifsp.contacts.model.Address;
import br.ifsp.contacts.model.Contact;
import br.ifsp.contacts.repository.AddressRepository;
import br.ifsp.contacts.repository.ContactRepository;
import io.swagger.v3.oas.annotations.Operation;
import jakarta.validation.Valid;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;

@RestController
@RequestMapping("/api/addresses")
public class AddressController {

    @Autowired
    private ContactRepository contactRepository;

    @Autowired
    private AddressRepository addressRepository;

    @Autowired
    private ModelMapper modelMapper;

    @Operation(summary = "Buscar todos os endereços de um contato")
    @GetMapping("/contacts/{contactId}")
    public Page<AddressResponseDTO> getAddressesByContact(@PathVariable Long contactId, Pageable pageable) {
        return addressRepository.findByContactId(contactId, pageable)
                .map(address -> modelMapper.map(address, AddressResponseDTO.class));
    }

    @Operation(summary = "Criar novo endereço para um contato")
    @PostMapping("/contacts/{contactId}")
    @ResponseStatus(HttpStatus.CREATED)
    public AddressResponseDTO createAddress(@PathVariable Long contactId, @RequestBody @Valid AddressRequestDTO dto) {
        Contact contact = contactRepository.findById(contactId)
                .orElseThrow(() -> new ResourceNotFoundException("Contato não encontrado: " + contactId));

        Address address = modelMapper.map(dto, Address.class);
        address.setContact(contact);
        Address saved = addressRepository.save(address);
        return modelMapper.map(saved, AddressResponseDTO.class);
    }
}
```

## 📚 Conclusões

Ao longo desta aula, demos mais um passo importante na consolidação das boas práticas no desenvolvimento de APIs RESTful com Spring Boot. 

Começamos discutindo a importância do uso de DTOs (Data Transfer Objects), separando os modelos internos da estrutura de dados trafegada pela API. Com isso, ganhamos maior controle sobre o que é exposto ao cliente, evitamos vazamentos de dados sensíveis, reduzimos o acoplamento entre as camadas e preparamos o terreno para uma evolução mais segura da aplicação. Vimos também as vantagens (e limitações) do uso de records e classes com Lombok, além das diferenças entre DTOs de request, response e atualizações parciais com PATCH.

Na sequência, implementamos a paginação e ordenação utilizando os recursos nativos do Spring Data JPA, o que nos permitiu trabalhar com grandes volumes de dados de forma mais performática e organizada. Discutimos a importância de oferecer ao cliente o controle sobre a quantidade e ordenação dos dados retornados, além dos impactos positivos na escalabilidade e usabilidade do sistema.

Depois, configuramos nossa aplicação para utilizar um banco de dados relacional real (MySQL/PostgreSQL) em vez do banco em memória (H2). Essa mudança nos aproximou de um ambiente mais próximo do mundo real, permitindo persistência entre execuções e maior controle sobre os dados. Também discutimos conceitos importantes como ORM, JPA e o papel das migrations no controle de versionamento do banco.

Por fim, integramos a ferramenta Swagger/OpenAPI para documentar nossa API de forma automática, interativa e acessível, promovendo uma comunicação mais clara entre o back-end e seus consumidores. Com apenas algumas anotações e configurações, conseguimos disponibilizar uma interface gráfica que facilita a exploração e o teste da nossa API — um recurso indispensável em qualquer aplicação moderna.

Além dos aspectos técnicos, a aula também reforçou princípios importantes da Engenharia de Software, como a separação de responsabilidades, a validação contextualizada, o desacoplamento entre camadas, o cuidado com overengineering e a importância de decisões técnicas conscientes, baseadas no contexto da aplicação e não apenas em modismos.

Se há uma mensagem principal que queremos reforçar aqui, é esta: não estamos apenas aprendendo frameworks ou ferramentas — estamos aprendendo a construir software com qualidade, clareza e propósito. E isso exige não só domínio técnico, mas também reflexão, criticidade e boas escolhas arquiteturais.

Continue praticando, testando e, acima de tudo, questionando o porquê de cada decisão técnica. Isso é o que transforma uma aplicação funcional em uma aplicação profissional — e um desenvolvedor iniciante em um desenvolvedor consciente.

É importante lembrar que ainda temos muito pela frente. Nas próximas aulas, abordaremos temas como segurança (autenticação e autorização com JWT), versionamento de APIs, e a construção de testes automatizados para garantir a qualidade e a confiabilidade do sistema. Esses tópicos aprofundarão ainda mais nossa aplicação e nosso conhecimento em desenvolvimento de sistemas, de forma geral.

Mas também é fundamental reconhecer o quanto já avançamos. Desde a nossa primeira aula, passamos por conceitos fundamentais de REST, criamos nossos primeiros endpoints, aprendemos a estruturar o projeto com camadas bem definidas, vimos o tratamento de exceções, adotamos boas práticas com DTOs e validações, configuramos a persistência com banco relacional, aplicamos paginação e ordenação, e finalizamos com a documentação interativa da API. Até que não estamos mal, né?! 🎉

É isso, pessoal! Nos vemos na próxima aula! Nessa semana não teremos exercícios para entrega. Apenas verifique ? Avalie os métodos e traga, na próxima aula, ao menos um aprimoramento nos controllers.
