# 🧪 Projeto A3 — Gestão da Qualidade de Software

**Quality Assurance em Sistemas Open Source**

Repositório da equipe de QA responsável pela avaliação da qualidade do sistema open source **[rest-spring-test](https://github.com/douglasfragoso/rest-spring-test)**, desenvolvido por Douglas Fragoso.

> **Questão central:** *Com base em evidências técnicas, este software apresenta qualidade suficiente para ser considerado apto para uma release?*

---

## 👥 Integrantes

| Nome | RA | 
|------|----|--------|
| Rafael Stender Rocha | `12524117225` |
| Gian Carlo Tozzatti | `12524124240` |
| Theobaldo Ocerio da Silva | `1252421088` |

**Unidade Curricular:** Gestão da Qualidade de Software
**Professor:** Emerson dos Santos Paduan

---

## 📌 Sistema avaliado

| Item | Descrição |
|------|-----------|
| **Nome** | rest-spring-test |
| **Repositório original** | https://github.com/douglasfragoso/rest-spring-test |
| **Descrição** | API RESTful para cadastro e gerenciamento de pessoas (entidade `Person`) |
| **Linguagem** | Java 21 |
| **Framework** | Spring Boot 3.3.3 |
| **Build** | Maven |
| **Banco de dados** | H2 (perfil `test`) e MySQL (perfil `prod`) |
| **Persistência** | Spring Data JPA |
| **Documentação da API** | Swagger / OpenAPI |
| **Testes existentes** | JUnit 5, Mockito, Rest-Assured, TestContainers |
| **Cobertura** | JaCoCo |

### Funcionalidades (endpoints)

| Método | Rota | Descrição |
|--------|------|-----------|
| `GET` | `/person` | Lista pessoas (paginado) |
| `GET` | `/person/id/{id}` | Busca pessoa por ID |
| `GET` | `/person/email/{email}` | Busca pessoa por e-mail |
| `POST` | `/person` | Cadastra uma nova pessoa |
| `PUT` | `/person` | Atualiza uma pessoa existente |
| `DELETE` | `/person/id/{id}` | Remove pessoa por ID |

**Campos da entidade `Person`:** `id`, `firstName`, `lastName`, `address`, `gender`, `email`, todos com validação de obrigatoriedade via Bean Validation.

### Arquitetura em camadas

```
Controller  →  Service  →  Repository  →  Banco de Dados (H2 / MySQL)
    │
    └── ControllerExceptionHandler (tratamento centralizado de erros)
```

---

## ⚙️ Como executar localmente

### Pré-requisitos

- [Java 21 (JDK)](https://adoptium.net/)
- [Maven 3.9+](https://maven.apache.org/) (ou usar o wrapper `mvnw` incluso)
- [Docker](https://www.docker.com/) — **necessário para os testes de integração com TestContainers**
- [MySQL 8](https://dev.mysql.com/downloads/) — apenas para o perfil `prod`
- [Postman](https://www.postman.com/) ou Insomnia (opcional)

### 1. Clonar o repositório

```bash
git clone https://github.com/douglasfragoso/rest-spring-test.git
cd rest-spring-test/rest-spring-test
```

### 2. Executar com H2 (perfil `test`, padrão)

O perfil `test` já vem ativo no `application.yml`, não precisa de banco instalado:

```bash
./mvnw spring-boot:run
```

> No Windows, use `mvnw.cmd spring-boot:run`.

### 3. Executar com MySQL (perfil `prod`)

1. Crie o banco `db_test` no MySQL.
2. Ajuste usuário e senha em `src/main/resources/application-prod.yml`, se necessário.
3. Altere o perfil ativo no `application.yml` para `prod`, ou rode:

```bash
./mvnw spring-boot:run -Dspring-boot.run.profiles=prod
```

### 4. Acessar a aplicação

- **API:** http://localhost:8080/person
- **Swagger UI:** http://localhost:8080/swagger-ui/index.html
- **Postman:** importar o arquivo `Spring Test.postman_collection.json` da raiz do repositório original

---

## 🧪 Execução dos testes e cobertura

```bash
# Executa todos os testes e gera o relatório do JaCoCo
./mvnw clean test

# Gera o relatório HTML do Surefire
./mvnw site
```

| Relatório | Caminho |
|-----------|---------|
| Cobertura (JaCoCo) | `target/site/jacoco/index.html` |
| Execução dos testes (Surefire) | `target/site/surefire-report.html` |

> ⚠️ Os testes de integração usam **TestContainers** e só rodam com o Docker ativo.

---

## 🗺️ Ciclo de QA do projeto

**Seleção → Clonagem → Configuração → Execução → Compreensão → Planejamento → Teste → Inspeção → Automação → Medição → Avaliação**

| Etapa | Atividades | Ferramentas |
|-------|------------|-------------|
| Planejamento | Plano de testes, critérios de aceitação, análise de riscos, matriz de rastreabilidade | — |
| Testes caixa-preta | Partição de equivalência, valor-limite, tabela de decisão | Postman / Insomnia |
| Testes caixa-branca | Cobertura de instruções, decisões e caminhos | JUnit 5, JaCoCo |
| Automação | Testes unitários, de integração e de API | JUnit 5, Mockito, Rest-Assured, TestContainers |
| Inspeção | Análise estática e revisão de código | SonarQube / SonarLint |
| Gestão de defeitos | Registro e acompanhamento de defeitos | GitHub Issues |
| Integração contínua | Pipeline de build e testes | GitHub Actions |
| Avaliação | Métricas de qualidade e Quality Decision (Go / No-Go / Conditional) | — |

---

## 📅 Cronograma de entregas

| Checkpoint | Data | Conteúdo | Status |
|------------|------|----------|--------|
| Entrega 1 | 26/10 | Seleção do sistema, criação do GitHub, cadastro dos integrantes, README | ✅ |
| Entrega 2 | 09/11 | Documentação da arquitetura, Plano de Testes, Matriz de Rastreabilidade, Análise de Riscos | ⏳ |
| Entrega 3 | 16/11 | Casos de Teste, Testes Automatizados, Relatório de Inspeção (SonarQube), Relatório de Métricas, Quality Decision | ⏳ |
| Apresentação | 23/11 ou 30/11 | Apresentação oral (até 15 min) com demonstração do MVP | ⏳ |

---

## 📁 Estrutura do repositório

```
📦 projeto-a3-qa
 ┣ 📂 docs
 ┃ ┣ 📂 arquitetura          # Diagramas de arquitetura
 ┃ ┣ 📂 plano-de-testes      # Plano de testes e critérios de aceitação
 ┃ ┣ 📂 riscos               # Matriz Probabilidade × Impacto
 ┃ ┣ 📂 rastreabilidade      # Matriz de rastreabilidade
 ┃ ┣ 📂 casos-de-teste       # Casos de teste caixa-preta e caixa-branca
 ┃ ┣ 📂 inspecao             # Relatórios do SonarQube
 ┃ ┗ 📂 metricas             # Relatórios de cobertura e métricas
 ┣ 📂 rest-spring-test       # Código do sistema avaliado + testes criados pela equipe
 ┣ 📂 .github/workflows      # Pipeline de integração contínua
 ┗ 📜 README.md
```

---

## ⚖️ Licença e créditos

O sistema avaliado é de autoria de **[Douglas Fragoso](https://github.com/douglasfragoso)**. Este repositório tem finalidade exclusivamente acadêmica.

