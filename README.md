# API de Clientes – Spring Boot + Design Patterns (`spring-design-patterns-api`)

API REST para gestão de clientes com enriquecimento automático de endereços por meio das APIs **ViaCEP** e **BrasilAPI**, demonstrando a aplicação prática de padrões de projeto no ecossistema Spring.

## ✨ Objetivos

Projeto de estudo desenvolvido durante o **DIO Itaú Bootcamp**, com os seguintes objetivos:

- Consolidar o uso de **Spring Boot 3 (Java 17)**, **Spring Data JPA** e **OpenFeign**.
- Demonstrar padrões de projeto: **Strategy, Singleton, Facade, Repository e Proxy**.
- Aplicar os conceitos de **Inversão de Controle (IoC)** e **Injeção de Dependências (DI)**.
- Integrar APIs externas (**ViaCEP e BrasilAPI**) para enriquecimento automático de endereços.
- Utilizar **OpenAPI e Swagger UI** para documentar e testar os endpoints da API.

## 🧱 Tecnologias e Dependências

| Camada | Tecnologia |
|---|---|
| Linguagem | Java 17 |
| Framework | Spring Boot 3.x |
| API REST | Spring Web |
| Persistência | Spring Data JPA |
| Banco de dados | H2 (em memória) |
| Cliente HTTP | Spring Cloud OpenFeign |
| APIs externas | ViaCEP e BrasilAPI |
| Documentação | Springdoc OpenAPI / Swagger UI |
| Testes | Spring Boot Starter Test |
| Build | Maven |

## 🧩 Padrões de Projeto Aplicados

| Padrão | Aplicação | Classes principais |
|---|---|---|
| **Strategy** | Define um contrato para as operações de gerenciamento de clientes. | `ClienteService`, `ClienteServiceImpl` |
| **Singleton** | Os beans Spring possuem escopo singleton por padrão. | `ClienteServiceImpl`, repositories e clientes Feign |
| **Facade** | Expõe endpoints que simplificam o acesso às funcionalidades da aplicação. | `ClienteRestController` |
| **Repository** | Abstrai o acesso e a persistência dos dados. | `ClienteRepository`, `EnderecoRepository` |
| **Proxy** | Os clientes Feign encapsulam as chamadas HTTP às APIs externas. | `ViaCepService`, `BrasilApiService` |
| **IoC / DI** | O Spring gerencia os componentes e suas dependências. | Serviços, controllers e repositories |

### Fluxo de criação de um cliente

Ao receber uma requisição para cadastrar um cliente, a aplicação verifica se o endereço correspondente ao CEP já existe no banco de dados. Caso não exista, consulta uma API externa para obter os dados do endereço.

```text
POST /clientes
   |
   v
ClienteRestController
   |
   v
ClienteService
   |
   v
ClienteServiceImpl
   |
   +--> EnderecoRepository.findById(cep)
   |         |
   |         +--> Endereço encontrado
   |         |         |
   |         |         +--> Reutiliza o endereço existente
   |         |
   |         +--> Endereço não encontrado
   |                   |
   |                   +--> ViaCepService / BrasilApiService
   |                   |         |
   |                   |         +--> Consulta API externa
   |                   |
   |                   +--> EnderecoRepository.save()
   |
   +--> ClienteRepository.save()
```

Esse fluxo evita consultas externas desnecessárias quando o endereço já está cadastrado.

## 🗂️ Estrutura do Projeto

O repositório reúne os projetos práticos desenvolvidos durante o bootcamp, incluindo o desafio de projeto e o laboratório de padrões de projeto com Spring.

```text
spring-design-patterns-api/
├── desafio-de-projeto/
└── lab-padroes-projeto-spring/
    └── src/
        └── main/
            └── java/
                └── ...
                    ├── controller/
                    │   └── ClienteRestController.java
                    ├── service/
                    │   ├── ClienteService.java
                    │   ├── ViaCepService.java
                    │   ├── BrasilApiService.java
                    │   └── impl/
                    │       └── ClienteServiceImpl.java
                    ├── repository/
                    │   ├── ClienteRepository.java
                    │   └── EnderecoRepository.java
                    ├── model/
                    │   ├── Cliente.java
                    │   └── Endereco.java
                    └── config/
                        └── OpenApiConfig.java
```

## 🔌 Endpoints da API

**URL base:** `http://localhost:8080`

| Método HTTP | Rota | Descrição |
|---|---|---|
| `GET` | `/clientes` | Lista todos os clientes cadastrados. |
| `GET` | `/clientes/{id}` | Busca um cliente pelo identificador. |
| `POST` | `/clientes` | Cadastra um cliente e consulta os dados do endereço pelo CEP. |
| `PUT` | `/clientes/{id}` | Atualiza os dados de um cliente, incluindo o processamento do CEP. |
| `DELETE` | `/clientes/{id}` | Remove um cliente pelo identificador. |

## 📄 Exemplo de Requisição e Resposta

No cadastro de um cliente, informe o nome e o CEP. Os demais dados do endereço são preenchidos automaticamente a partir da consulta à API externa, conforme a implementação do serviço.

### Requisição — POST `/clientes`

```json
{
  "nome": "Maria Silva",
  "endereco": {
    "cep": "01001000"
  }
}
```

### Resposta — Exemplo simplificado

```json
{
  "id": 1,
  "nome": "Maria Silva",
  "endereco": {
    "cep": "01001000",
    "logradouro": "Praça da Sé",
    "bairro": "Sé",
    "localidade": "São Paulo",
    "uf": "SP",
    "ddd": "11"
  }
}
```

Os dados apresentados são ilustrativos e podem variar conforme a resposta da API externa e o mapeamento realizado pela aplicação.

## 🧪 Testando a API

### Linux / Bash — cURL

**Cadastrar um cliente:**

```bash
curl -X POST http://localhost:8080/clientes \
  -H "Content-Type: application/json" \
  -d '{"nome":"Maria","endereco":{"cep":"01001000"}}'
```

**Listar os clientes:**

```bash
curl -X GET http://localhost:8080/clientes
```

**Buscar um cliente por ID:**

```bash
curl -X GET http://localhost:8080/clientes/1
```

**Atualizar um cliente:**

```bash
curl -X PUT http://localhost:8080/clientes/1 \
  -H "Content-Type: application/json" \
  -d '{"nome":"Maria Silva","endereco":{"cep":"01001000"}}'
```

**Excluir um cliente:**

```bash
curl -X DELETE http://localhost:8080/clientes/1
```

### Windows — PowerShell

**Cadastrar um cliente:**

```powershell
Invoke-RestMethod `
  -Uri "http://localhost:8080/clientes" `
  -Method Post `
  -Body '{"nome":"Maria","endereco":{"cep":"01001000"}}' `
  -ContentType "application/json"
```

**Listar os clientes:**

```powershell
Invoke-RestMethod `
  -Uri "http://localhost:8080/clientes" `
  -Method Get
```

## 📘 Documentação — Swagger / OpenAPI

A aplicação utiliza Springdoc OpenAPI para gerar a documentação dos endpoints.

Após iniciar a aplicação, acesse:

- **Swagger UI:** http://localhost:8080/swagger-ui/index.html
- **Especificação OpenAPI em JSON:** http://localhost:8080/v3/api-docs

Os metadados da documentação são configurados por meio de `@OpenAPIDefinition`.

## 🚀 Como Executar

### Pré-requisitos

- Java 17
- Maven 3.9+ ou Maven Wrapper
- Git

### 1. Clonar o repositório

```bash
git clone https://github.com/diego-dev-brazil/spring-design-patterns-api.git
```

### 2. Acessar o diretório da aplicação

```bash
cd spring-design-patterns-api/lab-padroes-projeto-spring
```

### 3. Limpar a compilação anterior (opcional)

```bash
./mvnw clean
```

### 4. Executar a aplicação

```bash
./mvnw spring-boot:run
```

No Windows, utilize:

```powershell
.\mvnw.cmd spring-boot:run
```

Após a inicialização, a API estará disponível em `http://localhost:8080`, desde que a aplicação esteja configurada para utilizar essa porta.

## 🔍 Observações Técnicas

- **Chave do endereço:** o CEP é utilizado como identificador do endereço, permitindo sua reutilização por diferentes clientes.
- **Reutilização de endereços:** `Optional.orElseGet()` permite consultar os serviços externos somente quando o endereço não está presente no banco de dados, conforme o fluxo implementado.
- **Integração com APIs externas:** OpenFeign permite declarar clientes HTTP para acessar os serviços ViaCEP e BrasilAPI.
- **Persistência:** o banco H2 é utilizado em memória. Os dados são perdidos quando a aplicação é reiniciada.
- **Modelagem:** as entidades `Cliente` e `Endereco` representam os dados persistidos pela aplicação.
- **DTOs:** o projeto não utiliza DTOs para manter o foco nos padrões de projeto estudados. Em aplicações de produção, separar os modelos de domínio dos objetos de transporte pode trazer benefícios de manutenção e segurança.
- **Documentação:** Springdoc OpenAPI gera a documentação da API e disponibiliza uma interface para testar os endpoints.

### Configuração opcional do console H2

Para habilitar o console web do H2 durante o desenvolvimento, adicione ao arquivo `application.properties`:

```properties
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console
```

O acesso ao console também depende da configuração do datasource e das regras de segurança da aplicação.

## 🧑‍💻 Autor

**Diego Augusto Oliveira**

- [GitHub](https://github.com/diego-dev-brazil)
- [Repositório do projeto](https://github.com/diego-dev-brazil/spring-design-patterns-api)

---

Projeto desenvolvido para fins educacionais durante o DIO Itaú Bootcamp, com foco em Java, Spring Boot, integração de APIs e padrões de projeto.
