[relatorio-qualidade-entrega3.md](https://github.com/user-attachments/files/34939538/relatorio-qualidade-entrega3.md)
# Relatório de Garantia da Qualidade

> Parte I — Inspeção de código  ·  Parte II — Relatório prático de testes

| Item | Descrição |
| --- | --- |
| Unidade Curricular | Gestão da Qualidade de Software |
| Professor | Emerson dos Santos Paduan |
| Integrantes | Rafael Stender Rocha, Gian Carlo Tozzatti, Theobaldo Ocerio da Silva |
| Sistema avaliado | [rest-spring-test](https://github.com/douglasfragoso/rest-spring-test) |
| Entrega | Entrega 3 — Quality Assurance em Sistemas Open Source (inspeção e testes) |
| Data | 16/11/2026 |

---

# Parte I — Relatório de Inspeção de Código

> Análise estática e revisão técnica do sistema rest-spring-test

## 1. Objetivo e escopo

Este relatório registra os resultados da inspeção de código do rest-spring-test, realizada como teste estático previsto no Plano de Testes da Entrega 2 (ISO/IEC/IEEE 29119). O objetivo é identificar defeitos, riscos e oportunidades de melhoria sem executar o sistema, e reunir evidências para a decisão de release do projeto.

Elementos inspecionados:

- Model e validações (`Person`)
- Controller e endpoints (`PersonController`)
- Service e regras de negócio (`PersonService`)
- Repository e consultas SQL (`PersonRepository`)
- Tratamento de exceções (`ControllerExceptionHandler`, `StandartError`)
- Testes unitários e de integração (`src/test/java`)
- Configurações Maven e dependências (`pom.xml`)
- Configurações dos bancos H2 e MySQL
- Perfis de execução (`application.yml`, `application-test.yml`, `application-prod.yml`)

## 2. Método

A inspeção combinou leitura do código-fonte e dos arquivos de configuração com revisão técnica orientada pelos riscos da análise de riscos. Cada achado recebeu um identificador (QA-nn) e foi registrado com evidência, identificação, análise técnica, impacto na qualidade, classificação e recomendação.

| Classificação | Significado |
| --- | --- |
| Defeito | Comportamento ou configuração incorreta que afeta o sistema ou a eficácia dos testes |
| Risco | Condição que pode causar falha ou instabilidade dependendo do ambiente ou do estado dos dados |
| Oportunidade de melhoria | Ponto que funciona, mas reduz manutenibilidade, cobertura ou clareza |
| Pendente de verificação | Hipótese levantada na leitura do código que precisa de execução para ser confirmada |
| Ponto positivo | Boa prática que contribui para a qualidade |

## 3. Resumo dos achados

Foram registrados 22 achados numerados, além das análises do `pom.xml` e dos arquivos de configuração (seção 5). Os defeitos se concentram nas regras de validação e de negócio e na precisão das asserções dos testes de integração; os pontos positivos estão na infraestrutura de testes.

| Classificação | Quantidade | Achados |
| --- | --- | --- |
| Defeito | 3 | QA-12, QA-22, QA-24 |
| Risco | 2 | QA-06, QA-15 |
| Oportunidade de melhoria | 10 | QA-13, QA-14, QA-16, QA-18, QA-19, QA-20, QA-21, QA-25, QA-26, QA-28 |
| Pendente de verificação | 2 | QA-23, QA-27 |
| Ponto positivo | 5 | QA-05, QA-07, QA-08, QA-17, QA-43 |

| ID | Achado | Área | Classificação |
| --- | --- | --- | --- |
| QA-05 | Organização dos testes de integração | Testes de integração | Ponto positivo |
| QA-06 | Dependência do Docker para execução dos testes de integração | Testes de integração | Risco |
| QA-07 | Utilização de banco de dados real nos testes | Testes de integração | Ponto positivo |
| QA-08 | Configuração dinâmica das propriedades de conexão | Testes de integração | Ponto positivo |
| QA-12 | Asserções do teste de cadastro sempre verdadeiras | Teste de integração (POST) | Defeito |
| QA-13 | Dependência entre os testes automatizados | Teste de integração | Oportunidade de melhoria |
| QA-14 | Ausência de verificação efetiva da persistência na atualização | Teste de integração (PUT) | Oportunidade de melhoria |
| QA-15 | Dependência de uma quantidade fixa de registros no teste de listagem | Teste de integração (GET) | Risco |
| QA-16 | Ausência de verificação do resultado da exclusão | Teste de integração (DELETE) | Oportunidade de melhoria |
| QA-17 | Verificação automatizada da disponibilidade da documentação Swagger | Teste de integração (Swagger) | Ponto positivo |
| QA-18 | Cobertura incompleta das verificações no teste de atualização | Teste de repositório | Oportunidade de melhoria |
| QA-19 | Verificação limitada da operação de salvamento | Teste de repositório | Oportunidade de melhoria |
| QA-20 | Ausência de retorno sobre o resultado da atualização | Repository | Oportunidade de melhoria |
| QA-21 | Dependência de consulta SQL nativa | Repository | Oportunidade de melhoria |
| QA-22 | Tipo de resposta documentado diverge do retornado no PUT | Controller / OpenAPI | Defeito |
| QA-23 | Ausência de verificação de e-mail duplicado durante a atualização | Service | Pendente de verificação |
| QA-24 | Validação de formato de e-mail não associada aos grupos de validação | Model (Person) | Defeito |
| QA-25 | Ausência de validação explícita dos limites de caracteres | Model (Person) | Oportunidade de melhoria |
| QA-26 | Identificação de exceções por comparação textual | Tratamento de exceções | Oportunidade de melhoria |
| QA-27 | Tratamento de rotas inexistentes dependente de configuração | Tratamento de exceções | Pendente de verificação |
| QA-28 | Inconsistência de nomenclatura na classe StandartError | Tratamento de exceções | Oportunidade de melhoria |
| QA-43 | Centralização de configurações dos testes | Testes | Ponto positivo |

## 4. Achados detalhados

### 4.1 Infraestrutura dos testes de integração

#### QA-05 — Organização dos testes de integração

**Evidência:** `src/test/java/com/restspringtest/Integration/`

**Identificação:** Os testes de integração têm uma classe-base (`AbstractIntegrationTest`) e classes específicas para o controller de pessoas e para a documentação Swagger.

**Impacto na qualidade:** Favorece o reaproveitamento de configurações e a separação de responsabilidades entre os testes.

**Classificação:** Ponto positivo — boa organização da suíte.

#### QA-06 — Dependência do Docker para execução dos testes de integração

**Evidência:** `AbstractIntegrationTest.java`, método `startContainers()`

**Identificação:** A classe usa TestContainers para iniciar um banco MySQL em contêiner Docker.

**Análise técnica:** `MySQLContainer` cria o banco a partir da imagem `mysql:8.0.28`, e `startContainers()` usa `Startables.deepStart()` para iniciar o contêiner e aguardar a inicialização. Os testes passam a usar um banco real, mas dependem de infraestrutura externa.

**Impacto na qualidade:** Sem Docker, ou com falha na inicialização do contêiner, os testes que dependem dessa configuração não executam.

**Classificação:** Risco de ambiente de testes.

**Recomendação:** Garantir Docker ativo no ambiente de execução e registrar a cobertura com e sem os testes de integração.

#### QA-07 — Utilização de banco de dados real nos testes

**Evidência:** `AbstractIntegrationTest.java`, declaração do objeto `MySQLContainer`

**Identificação:** Um contêiner MySQL fornece o banco de dados durante os testes de integração.

**Análise técnica:** A declaração `new MySQLContainer<>("mysql:8.0.28")` aproxima o ambiente de testes das condições reais. Diferente de testes só com mocks, permite verificar a integração entre a aplicação e o banco.

**Aspectos positivos:**

- Uso de um banco de dados real.
- Ambiente de testes separado da configuração de produção.
- Maior proximidade com as condições reais de integração.
- Possibilidade de identificar problemas de persistência.

**Impacto na qualidade:** Aumenta a confiabilidade dos testes de integração e a detecção de problemas na comunicação com o banco.

**Classificação:** Ponto positivo — boa prática de testes de integração.

#### QA-08 — Configuração dinâmica das propriedades de conexão

**Evidência:** `AbstractIntegrationTest.java`, métodos `createConnectionProperties()` e `initialize()`

**Identificação:** As propriedades de conexão com o banco de testes são obtidas dinamicamente do contêiner.

**Análise técnica:** `createConnectionProperties()` lê do contêiner `spring.datasource.url`, `spring.datasource.username` e `spring.datasource.password`; `initialize()` as injeta no ambiente do Spring com `MapPropertySource`.

**Aspectos positivos:**

- Evita fixar parâmetros de conexão no código dos testes.
- Facilita a inicialização do ambiente.
- Favorece o reaproveitamento da configuração.
- Reduz ajustes manuais entre ambientes de execução.

**Impacto na qualidade:** Contribui para a manutenibilidade e a confiabilidade da configuração de testes.

**Classificação:** Ponto positivo — boa prática de configuração.

#### QA-43 — Centralização de configurações dos testes

**Identificação:** As configurações dos testes ficam em um único local, em vez de repetidas em várias classes.

**Impacto na qualidade:** Facilita alterações futuras e melhora a organização do código. Nenhum defeito funcional foi identificado nessa classe.

**Classificação:** Ponto positivo — organização.

### 4.2 Qualidade dos testes automatizados

#### QA-12 — Asserções do teste de cadastro sempre verdadeiras

**Evidência:** Método `integrationTestGivenPerson_whenPostPerson_thenReturnPerson()`

**Identificação:** O teste atribui a resposta da API à variável de referência antes de comparar os campos.

```
Person createdPerson = objectMapper.readValue(content, Person.class);
person = createdPerson;
assertNotNull(createdPerson);
assertNotNull(createdPerson.getId());
assertEquals(person.getFirstName(), createdPerson.getFirstName());
assertEquals(person.getLastName(), createdPerson.getLastName());
```

**Análise técnica:** Após `person = createdPerson;`, as duas variáveis apontam para o mesmo objeto. As comparações de nome e sobrenome passam sempre que o objeto existir.

**Impacto na qualidade:** O teste não verifica se a API devolveu o nome e o sobrenome enviados na requisição, reduzindo sua capacidade de detectar erros no processamento ou no retorno dos dados.

**Classificação:** Defeito — deficiência na implementação de uma asserção.

**Recomendação:** Guardar os valores originais antes da requisição e compará-los com os valores retornados pela API.

#### QA-13 — Dependência entre os testes automatizados

**Evidência:** Atributo estático `person` e uso de `@Order`

**Identificação:** Os cenários compartilham a variável estática `private static Person person`: o cadastro a preenche, a atualização usa seu id e as consultas seguintes dependem dos dados nela.

**Análise técnica:** O resultado dos testes posteriores depende do sucesso dos anteriores. Se o cadastro falhar, os demais recebem um id nulo ou inexistente.

**Impacto na qualidade:** Dificulta a execução isolada dos testes, a manutenção da suíte e a identificação da origem das falhas.

**Classificação:** Oportunidade de melhoria na manutenibilidade e confiabilidade dos testes.

**Recomendação:** Criar dados independentes para cada cenário, com preparação e limpeza do ambiente de testes.

#### QA-14 — Ausência de verificação efetiva da persistência na atualização

**Evidência:** Método `integrationTestGivenPerson_whenPutPerson_thenReturnString()`

**Identificação:** O teste de atualização verifica a mensagem de retorno e compara o objeto original com o enviado.

```
assertEquals("Profile updated successfully", responseMessage);
assertNotEquals(person.getFirstName(), updatePerson.getFirstName());
assertNotEquals(person.getLastName(), updatePerson.getLastName());
```

**Análise técnica:** As duas últimas asserções só confirmam que os dados enviados são diferentes dos originais; não comprovam que o banco persistiu a alteração. Um teste posterior de consulta por id verifica os novos nomes, mas o cenário de atualização não se sustenta sozinho.

**Impacto na qualidade:** A verificação isolada do PUT é limitada e depende de outros cenários para confirmar a persistência.

**Classificação:** Oportunidade de melhoria na cobertura das verificações.

**Recomendação:** Após o PUT, consultar o registro e verificar se os dados foram de fato atualizados.

#### QA-15 — Dependência de uma quantidade fixa de registros no teste de listagem

**Evidência:** Método `integrationTest_whenGetAllPersons_thenReturnListOfPerson()`

**Identificação:** O teste espera exatamente dois registros: `assertEquals(2, page.getContent().size());`

**Análise técnica:** A expectativa depende do estado do banco durante a execução. Registros de outros testes ou de execuções anteriores alteram o resultado.

**Impacto na qualidade:** Compromete a estabilidade e a repetibilidade dos testes.

**Classificação:** Risco de dependência do estado do banco de dados.

**Recomendação:** Preparar um conjunto controlado de dados antes da execução e verificar a quantidade esperada para aquele cenário.

#### QA-16 — Ausência de verificação do resultado da exclusão

**Evidência:** Método `integrationTestGivenId_whenGetDeletePersonById_thenNoContent()`

**Identificação:** O teste de exclusão verifica apenas o código HTTP 204.

```
given().spec(specification)
    .pathParam("id", person.getId())
    .when().delete("/id/{id}")
    .then().statusCode(204);
```

**Análise técnica:** Não há consulta posterior para confirmar que o registro deixou de existir.

**Impacto na qualidade:** A validação confirma o código de resposta, mas não comprova o resultado da exclusão no sistema.

**Classificação:** Oportunidade de melhoria na validação funcional.

**Recomendação:** Após a exclusão, consultar o id removido e verificar o comportamento esperado para um registro inexistente (HTTP 404).

#### QA-17 — Verificação automatizada da disponibilidade da documentação Swagger

**Evidência:** `src/test/java/com/restspringtest/Integration/Tests/SwaggerIntegrationTest.java`, método `testShouldDisplaySwaggerUiPage()`

**Identificação:** Existe um teste de integração específico para a interface Swagger UI.

**Análise técnica:** Com JUnit 5, Spring Boot Test e Rest-Assured, o teste faz um GET em `/swagger-ui/index.html`, verifica o HTTP 200 e usa `assertTrue()` para confirmar a presença de "Swagger UI" no HTML retornado.

**Aspectos positivos:**

- Teste específico para a documentação da API.
- Verificação do código de resposta HTTP.
- Validação do conteúdo retornado pelo servidor.

**Limitação:** O teste não avalia se todos os endpoints estão documentados, se os esquemas de dados estão completos ou se os exemplos de requisição e resposta estão corretos (ver QA-22).

**Impacto na qualidade:** Contribui para a confiabilidade e a verificabilidade da documentação disponibilizada aos consumidores da API.

**Classificação:** Ponto positivo, com oportunidade de ampliação da cobertura funcional.

#### QA-18 — Cobertura incompleta das verificações no teste de atualização

**Evidência:** `src/test/java/com/restspringtest/Repository/PersonRepositoryTest.java`, método `testGivenPersonList_whenUpdate_ThenReturnPerson()`

**Identificação:** O teste atualiza cinco atributos de `Person`, mas verifica apenas quatro; `lastName` não tem asserção após a atualização.

**Análise técnica:** Uma falha na atualização do sobrenome não seria detectada por esse teste.

**Impacto na qualidade:** Reduz a cobertura das verificações funcionais da atualização.

**Classificação:** Oportunidade de melhoria na qualidade dos testes.

**Recomendação:** Acrescentar uma asserção confirmando que o sobrenome foi atualizado para "Doe2".

#### QA-19 — Verificação limitada da operação de salvamento

**Evidência:** `src/test/java/com/restspringtest/Repository/PersonRepositoryTest.java`, método `testGivenPersonObject_whenSave_ThenReturPerson()`

**Identificação:** O teste verifica apenas se o objeto retornado por `save()` não é nulo e se recebeu um id positivo.

**Análise técnica:** Não há verificação explícita dos demais atributos após o salvamento.

**Impacto na qualidade:** Uma inconsistência na persistência de algum atributo pode passar despercebida.

**Classificação:** Oportunidade de melhoria na cobertura dos testes de persistência.

**Recomendação:** Verificar os principais atributos após o salvamento, de preferência consultando novamente pelo id.

### 4.3 Model e validações

#### QA-24 — Validação de formato de e-mail não associada aos grupos de validação

**Evidência:** `src/main/java/com/restspringtest/Model/Person.java`, campo `email`

**Identificação:** A anotação `@Email` não declara os grupos `Create` e `Update`, ao contrário de `@NotBlank` no mesmo atributo.

**Análise técnica:** O controller valida explicitamente os grupos `Create.class` e `Update.class`. Como `@Email` pertence ao grupo `Default`, ela não é executada no cadastro nem na atualização.

**Impacto na qualidade:** A aplicação pode aceitar e-mails com formato inválido, comprometendo a integridade dos dados.

**Classificação:** Defeito — falha de configuração de validação identificada por análise estática.

**Recomendação:** Associar `@Email` aos grupos `Create` e `Update`, ou incluir o grupo `Default` na validação do controller.

#### QA-25 — Ausência de validação explícita dos limites de caracteres

**Evidência:** `src/main/java/com/restspringtest/Model/Person.java`, campos `firstName`, `lastName`, `address`, `gender` e `email`

**Identificação:** Os limites de tamanho estão definidos só em `@Column(length = ...)`, sem `@Size`.

**Análise técnica:** O tamanho da coluna no mapeamento JPA não valida o conteúdo recebido pela API; valores acima do limite chegam à camada de persistência.

**Impacto na qualidade:** Possibilidade de erros na persistência e ausência de mensagens de validação antecipadas e específicas para o usuário.

**Classificação:** Oportunidade de melhoria na validação de dados.

**Recomendação:** Adicionar `@Size(max = ...)` compatível com os limites das colunas.

### 4.4 Service e regras de negócio

#### QA-23 — Ausência de verificação de e-mail duplicado durante a atualização

**Evidência:** `src/main/java/com/restspringtest/Services/PersonService.java`, método `update()`

**Identificação:** O método verifica se o id existe, mas não se o novo e-mail já pertence a outro registro.

**Análise técnica:** O método `save()` verifica explicitamente e-mail duplicado antes do cadastro; a mesma regra de negócio não existe em `update()`.

**Impacto na qualidade:** Possível inconsistência nos dados cadastrais caso o banco não tenha restrição de unicidade no campo de e-mail.

**Classificação:** Possível falha de regra de negócio, pendente de verificação da restrição de unicidade no banco.

**Recomendação:** Verificar o e-mail na atualização, desconsiderando o próprio registro que está sendo atualizado.

### 4.5 Repository e consultas SQL

#### QA-20 — Ausência de retorno sobre o resultado da atualização

**Evidência:** `src/main/java/com/restspringtest/Repository/PersonRepository.java`, método `updatePerson()`

**Identificação:** O método de atualização retorna `void` e não informa quantos registros foram afetados.

**Análise técnica:** Com um id inexistente nenhuma linha é atualizada, mas o chamador não consegue distinguir esse caso de uma atualização bem-sucedida.

**Impacto na qualidade:** Limita o tratamento de atualizações sem correspondência no banco.

**Classificação:** Oportunidade de melhoria.

**Recomendação:** Retornar `int` com a quantidade de registros modificados e tratar o caso de nenhuma linha afetada.

#### QA-21 — Dependência de consulta SQL nativa

**Evidência:** `src/main/java/com/restspringtest/Repository/PersonRepository.java`, método `updatePerson()`

**Identificação:** A atualização usa SQL nativo via `@Query(nativeQuery = true)`.

**Análise técnica:** A consulta referencia diretamente a tabela `tb_person` e suas colunas físicas; mudanças no esquema exigem ajuste manual da instrução.

**Impacto na qualidade:** Maior acoplamento entre o repositório e a estrutura física do banco.

**Classificação:** Observação arquitetural.

**Recomendação:** Avaliar a necessidade da consulta nativa ou usar os recursos de abstração do JPA.

### 4.6 Controller e documentação da API

#### QA-22 — Tipo de resposta documentado diverge do retornado no PUT

**Evidência:** `src/main/java/com/restspringtest/Controller/PersonController.java`, método `update()`

**Identificação:** A documentação OpenAPI do `PUT /person` não corresponde à implementação.

**Análise técnica:** `@ApiResponse` declara `Person.class` como modelo da resposta HTTP 200, mas o método retorna `ResponseEntity<String>` com a mensagem "Profile updated successfully".

**Impacto na qualidade:** A documentação pode levar consumidores da API a interpretar incorretamente o formato da resposta, prejudicando integrações.

**Classificação:** Defeito — inconsistência documental.

**Recomendação:** Documentar a resposta como String ou alterar a implementação para retornar `Person`, conforme o comportamento desejado.

### 4.7 Tratamento de exceções

#### QA-26 — Identificação de exceções por comparação textual

**Evidência:** `src/main/java/com/restspringtest/Controller/Exception/ControllerExceptionHandler.java`, método `handleDatabaseExceptionId()`

**Identificação:** O tratamento de `DatabaseException` decide o tipo de erro comparando um trecho da mensagem.

**Análise técnica:** A classificação depende do texto "Email already registered". Uma mudança na mensagem altera o status HTTP devolvido (400 ou 404).

**Impacto na qualidade:** Acopla o tratamento de erros às mensagens das exceções, dificultando manutenção e evolução.

**Classificação:** Oportunidade de melhoria.

**Recomendação:** Usar exceções específicas ou identificadores próprios para cada categoria de erro.

#### QA-27 — Tratamento de rotas inexistentes dependente de configuração

**Evidência:** `src/main/java/com/restspringtest/Controller/Exception/ControllerExceptionHandler.java`, método `handleNotFoundException()`

**Identificação:** Existe tratamento específico para `NoHandlerFoundException`, retornando HTTP 404.

**Análise técnica:** O tratamento só é acionado se o Spring MVC estiver configurado para lançar `NoHandlerFoundException` em rotas sem correspondência. Sem executar uma requisição a uma rota inexistente não é possível confirmar o comportamento.

**Impacto na qualidade:** Possível inconsistência no formato das respostas de erro para rotas inexistentes.

**Classificação:** Ponto de investigação pendente de validação prática.

**Recomendação:** Conferir a configuração do Spring MVC e criar um teste de integração para uma rota inexistente (caso de teste CT22).

#### QA-28 — Inconsistência de nomenclatura na classe StandartError

**Evidência:** `src/main/java/com/restspringtest/Controller/Exception/StandartError.java`

**Identificação:** A classe usa a grafia "StandartError"; a grafia correta em inglês é "StandardError".

**Impacto na qualidade:** Pequena inconsistência que prejudica a padronização e a legibilidade do código.

**Classificação:** Melhoria de padronização e manutenção.

**Recomendação:** Renomear para `StandardError` e atualizar as referências.

## 5. Análise das configurações do projeto

### 5.1 Configuração Maven (pom.xml)

A análise do `pom.xml` identificou pontos positivos e oportunidades de melhoria nas dependências, nas ferramentas de teste e nos metadados do projeto.

**Pontos positivos:**

- Java 21 e Spring Boot 3.3.3.
- JUnit 5 e Mockito para testes automatizados.
- Rest-Assured para testes de integração da API.
- Testcontainers com MySQL, aproximando os testes do ambiente real de persistência.
- JaCoCo para análise de cobertura.
- Maven Surefire Report para geração de relatórios.

**Oportunidades de melhoria:**

- A seção de licenças do POM está vazia, sem identificar uma licença válida pelos metadados Maven.
- A configuração do JaCoCo diverge entre o identificador `report-aggregate` e o objetivo efetivamente executado (`report`).
- A geração de cobertura está vinculada à fase `post-integration-test`, que não é alcançada quando se executa só a fase `test`.
- A declaração simultânea de `springdoc-openapi-starter-webmvc-ui` e `springdoc-openapi-starter-webmvc-api` aparenta ser redundante.
- O Maven Surefire Report está na versão 2.22.0; recomenda-se revisar a atualização.

**Observação:** a falha de execução dos testes de integração observada pela equipe pode estar relacionada à indisponibilidade do Docker no ambiente da análise (ver QA-06). A causa deve ser confirmada nos relatórios de execução.

### 5.2 Configuração do banco MySQL (perfil de produção)

Na configuração do banco MySQL usada pelo perfil `prod`, foram identificados os pontos abaixo.

| Aspecto | Achado | Avaliação |
| --- | --- | --- |
| Segurança | Usuário `root` e senha `root` definidos diretamente no arquivo | Credenciais expostas no repositório |
| Persistência | `ddl-auto: update` permite que o Hibernate altere a estrutura do banco automaticamente | Arriscado em produção |
| Logs | `show-sql: true` exibe os comandos SQL no console | Excesso de informação em produção |
| Compatibilidade | Dialeto `MySQL8Dialect` declarado manualmente | Desnecessário em condições convencionais |

Recomenda-se separar as configurações de desenvolvimento, teste e produção, proteger as credenciais com variáveis de ambiente e gerenciar alterações do banco com ferramentas específicas de migração.

### 5.3 Perfis de execução (application.yml e application-prod.yml)

A aplicação usa perfis do Spring Boot para organizar os ambientes. O arquivo principal define o perfil `test` como padrão, configura o tratamento de rotas inexistentes e desativa os mapeamentos automáticos de recursos estáticos.

**Pontos positivos:**

- Uso de perfis para separar ambientes.
- Tratamento de rotas inexistentes integrado ao `ControllerExceptionHandler`.
- Desativação de recursos estáticos, coerente com uma API REST.

**Oportunidades de melhoria:**

- Revisar a definição do perfil `test` como padrão.
- Garantir a seleção explícita do perfil de produção na implantação.
- Avaliar o impacto da desativação de recursos estáticos sobre a interface Swagger.
- Revisar as credenciais e as configurações de persistência do perfil de produção.

### 5.4 Perfil de testes (application-test.yml)

O ambiente de testes usa H2 em memória, com a estrutura do banco criada e excluída automaticamente por `ddl-auto: create-drop`.

**Pontos positivos:**

- Banco em memória, facilitando a execução dos testes.
- Criação automática da estrutura do banco.
- Remoção da estrutura ao encerrar o contexto da aplicação.
- Menor dependência de um servidor de banco externo nos testes que usam H2.

**Observação técnica:** o projeto também usa Testcontainers com MySQL 8 nos testes de integração. Há, portanto, duas estratégias de persistência nos testes (H2 em memória e MySQL em contêiner), o que permite testar cenários diferentes, mas exige Docker disponível para a segunda.

## 6. Relação com a análise de riscos da Entrega 2

A inspeção confirmou na leitura do código os principais riscos de produto levantados na análise de riscos. A confirmação do efeito em execução fica com os casos de teste correspondentes da matriz de rastreabilidade.

| Risco (Entrega 2) | Achados que o confirmam | Situação após a inspeção |
| --- | --- | --- |
| R02 — E-mail duplicado via PUT | QA-23 | Regra ausente em `update()` confirmada no código; o efeito depende da execução do CT18 |
| R05 — Credenciais fixas | Seção 5.2 | Confirmado: usuário e senha `root/root` no perfil de produção |
| R01 — E-mail inválido aceito | QA-24 | Confirmado no código: `@Email` fora dos grupos validados; executar CT11 |
| R03 — Erro 500 por tamanho | QA-25 | Confirmado: campos sem `@Size`; executar CT13 e CT14 |
| R04 — Status pela mensagem | QA-26 | Confirmado: status decidido por comparação de texto |
| R06 — Swagger inconsistente | QA-22, QA-17 | Confirmado: PUT documentado como `Person` e retorna texto; teste do Swagger não cobre o contrato |
| R07 — Licença não identificável | Seção 5.1 | Reforçado: o POM também não declara licença |
| R08 — Dependência de Docker | QA-06 | Confirmado; falha observada na execução dos testes de integração possivelmente ligada ao Docker |

## 7. Conclusões e recomendações prioritárias

A estrutura do projeto é bem organizada: arquitetura em camadas, perfis de ambiente separados, testes de integração com banco real via TestContainers e ferramentas de cobertura e relatório já configuradas. Os problemas encontrados não estão na arquitetura, e sim em regras de validação e de negócio incompletas, em asserções de teste que verificam menos do que aparentam e em configurações sensíveis no perfil de produção.

Recomendações, em ordem de prioridade:

1. Aplicar a verificação de e-mail único também na atualização (QA-23) e criar restrição de unicidade na coluna `email`.
2. Remover as credenciais `root/root` do perfil de produção, usando variáveis de ambiente (seção 5.2).
3. Associar `@Email` aos grupos `Create` e `Update` (QA-24) e adicionar `@Size` aos campos com limite de coluna (QA-25).
4. Corrigir as asserções dos testes de integração (QA-12, QA-14, QA-16) e eliminar a dependência entre testes e do estado do banco (QA-13, QA-15).
5. Ajustar a documentação OpenAPI do PUT (QA-22) e substituir a comparação textual por exceções específicas (QA-26).
6. Corrigir a configuração do JaCoCo (fase e objetivo do relatório) para que a cobertura seja gerada de forma confiável (seção 5.1).
7. Tratar as melhorias menores: retorno de `updatePerson()` (QA-20), renomeação de `StandartError` (QA-28), revisão dos starters springdoc e da versão do Surefire.

---

# Parte II — Relatório Prático de Testes

> Execução de testes, cobertura de código, inspeção estática com SonarQube e Quality Decision

## 1. Introdução

Este relatório apresenta a avaliação da qualidade de software do projeto rest-spring-test, uma API REST desenvolvida em Java com Spring Boot, cujo objetivo é realizar operações de cadastro, consulta, atualização e exclusão de pessoas.

A avaliação foi realizada por meio de testes automatizados, análise de cobertura de código com JaCoCo e inspeção estática com SonarQube.

O objetivo é identificar evidências sobre a qualidade do sistema, verificar possíveis falhas, analisar a cobertura dos testes e propor melhorias, permitindo uma decisão fundamentada sobre sua aptidão para liberação.

## 2. Caracterização do sistema e ambiente de testes

O projeto analisado utiliza uma arquitetura monolítica organizada em três camadas principais:

- **Controller:** responsável por receber e responder às requisições HTTP.
- **Service:** responsável pelas regras de negócio.
- **Repository:** responsável pela comunicação com o banco de dados.

A entidade principal do sistema é `Person`, utilizada nas operações de gerenciamento de pessoas.

### 2.1 Tecnologias utilizadas

- Java 21;
- Spring Boot 3.3.3;
- Maven;
- MySQL 8;
- H2 Database;
- JUnit 5;
- Mockito;
- Rest-Assured;
- Testcontainers;
- JaCoCo;
- SonarQube.

Os testes foram executados em ambiente Windows, utilizando o Visual Studio Code, Maven e Docker Desktop.

## 3. Execução dos testes automatizados

A execução dos testes foi realizada por meio do comando:

```
.\mvnw.cmd clean test
```

Na primeira execução, foram identificados erros relacionados à incompatibilidade entre a versão da API Docker utilizada pelo Testcontainers e a versão exigida pelo Docker Desktop.

O cliente utilizado pelo Testcontainers estava tentando se comunicar utilizando a API 1.32, enquanto o ambiente Docker exigia, no mínimo, a versão 1.40.

Para solucionar o problema, foi criado o arquivo `docker-java.properties`, dentro de `src/test/resources`, contendo a configuração:

```
api.version=1.40
```

Após o ajuste, os testes foram executados novamente e concluídos com sucesso.

### 3.1 Resultados dos testes

Foram executados 40 testes automatizados, distribuídos da seguinte maneira:

| Classe de teste | Testes executados | Falhas | Erros |
| --- | --- | --- | --- |
| PersonControllerTest | 11 | 0 | 0 |
| PersonControllerIntegrationTest | 6 | 0 | 0 |
| SwaggerIntegrationTest | 1 | 0 | 0 |
| PersonRepositoryTest | 9 | 0 | 0 |
| PersonServiceTest | 12 | 0 | 0 |
| StartUpTests | 1 | 0 | 0 |
| **Total** | **40** | **0** | **0** |

**Resultado final:** 40 testes executados, nenhuma falha, nenhum erro e nenhum teste ignorado.

A compilação e a execução dos testes foram concluídas com BUILD SUCCESS.

![Figura 1 — Execução dos testes no Visual Studio Code: 40 testes executados, 0 falhas, 0 erros e BUILD SUCCESS, com geração do relatório JaCoCo.](img/01-execucao-testes-vscode.png)

*Figura 1 — Execução dos testes no Visual Studio Code: 40 testes executados, 0 falhas, 0 erros e BUILD SUCCESS, com geração do relatório JaCoCo.*

Esse resultado demonstra que os cenários automatizados existentes foram executados com sucesso no ambiente configurado. Entretanto, a aprovação dos testes não garante, isoladamente, a ausência de defeitos em todas as funcionalidades do sistema.

## 4. Análise de cobertura com JaCoCo

Foi utilizado o JaCoCo 0.8.12 para avaliar a cobertura do código-fonte pelos testes automatizados.

Os resultados gerais foram:

| Métrica | Resultado |
| --- | --- |
| Cobertura de instruções | 66% |
| Instruções não cobertas | 119 de 351 |
| Cobertura de branches | 100% |
| Branches não cobertos | 0 de 6 |

![Figura 2 — Relatório JaCoCo: visão geral da cobertura por pacote.](img/02-jacoco-visao-geral.png)

*Figura 2 — Relatório JaCoCo: visão geral da cobertura por pacote.*

### 4.1 Cobertura por pacote

A análise apresentou os seguintes resultados:

| Pacote | Cobertura |
| --- | --- |
| Controller.Exception | 23% |
| Pacote principal | 37% |
| Services | 100% |
| Controller | 100% |
| Model | 100% |
| Services.Exception | 100% |

Os resultados indicam que as camadas de Controller, Services e Model apresentaram cobertura de 100% nos indicadores exibidos pelo JaCoCo.

Por outro lado, o pacote Controller.Exception apresentou cobertura de apenas 23%, representando uma oportunidade significativa de ampliação dos testes.

Embora todos os 40 testes automatizados tenham sido aprovados, ainda existem partes do código que não foram exercitadas pelos testes. O tratamento de exceções representa o principal ponto de atenção identificado nesta análise.

### 4.2 Análise do ControllerExceptionHandler

A classe `ControllerExceptionHandler`, responsável pelo tratamento de exceções da aplicação, apresentou os seguintes resultados:

- Cobertura de instruções: 23%;
- 114 instruções não cobertas de um total de 149;
- 26 linhas não cobertas de um total de 35;
- 5 métodos não executados de um total de 7;
- Cobertura de branches: 100%.

![Figura 3 — JaCoCo: cobertura do pacote com.restspringtest.Controller.Exception.](img/03-jacoco-controller-exception.png)

*Figura 3 — JaCoCo: cobertura do pacote com.restspringtest.Controller.Exception.*

Foram identificados métodos sem cobertura relacionados ao tratamento de:

- `MethodArgumentNotValidException`;
- `ConstraintViolationException`;
- `handleTypeMismatchException`;
- `handleNotFoundException`.

Também foi identificado um método interno relacionado ao processamento de `ConstraintViolationException` sem cobertura.

![Figura 4 — JaCoCo: cobertura por método da classe ControllerExceptionHandler.](img/04-jacoco-controllerexceptionhandler.png)

*Figura 4 — JaCoCo: cobertura por método da classe ControllerExceptionHandler.*

> **Ponto de atenção — Cobertura do tratamento de exceções**
>
> Embora a cobertura de ramificações tenha atingido 100%, os resultados demonstram que grande parte do código responsável pelo tratamento de exceções não foi exercitada pela suíte automatizada.
>
> **Análise:** a baixa cobertura dessa classe representa uma lacuna nos testes automatizados, principalmente em cenários de tratamento de erros e validação de dados.
>
> **Recomendação:** criar testes específicos para os métodos de tratamento de exceções, verificando as respostas HTTP e as mensagens retornadas pela API em diferentes situações de erro.

É importante destacar que a ausência de cobertura não comprova, por si só, que esses tratamentos estejam funcionando incorretamente.

## 5. Inspeção estática com SonarQube

A inspeção estática foi realizada utilizando o SonarQube, com o objetivo de identificar problemas relacionados à segurança, confiabilidade e manutenibilidade do código.

A análise foi executada por meio do Maven e concluída com sucesso.

### 5.1 Resultados gerais

| Indicador | Resultado |
| --- | --- |
| Quality Gate | Passed |
| Segurança | D |
| Confiabilidade | C |
| Manutenibilidade | A |
| Cobertura | 67,8% |
| Duplicação de código | 0,0% |
| Problemas identificados | 5 |

![Figura 5 — SonarQube: visão geral do projeto rest-spring-test (Quality Gate Passed, 528 linhas de código).](img/05-sonarqube-visao-geral.png)

*Figura 5 — SonarQube: visão geral do projeto rest-spring-test (Quality Gate Passed, 528 linhas de código).*

Embora o Quality Gate tenha sido aprovado, foram identificados pontos que merecem atenção e correção.

### 5.2 Problemas identificados

#### 5.2.1 Utilização direta da entidade Person no método save()

| Item | Descrição |
| --- | --- |
| Arquivo | `PersonController.java` |
| Regra | `java:S4684` |
| Classificação | Problema relacionado à segurança |

O SonarQube identificou o uso da entidade persistente `Person` diretamente como corpo da requisição.

Essa abordagem pode aumentar o acoplamento entre a camada de entrada da API e a estrutura interna do banco de dados.

**Recomendação:** utilizar DTOs (Data Transfer Objects), permitindo controlar melhor os dados recebidos e evitar a exposição desnecessária de atributos internos.

![Figura 6 — SonarQube: entidade Person usada diretamente no método save() do PersonController.](img/06-sonar-s4684-save.png)

*Figura 6 — SonarQube: entidade Person usada diretamente no método save() do PersonController.*

#### 5.2.2 Utilização direta da entidade Person no método update()

| Item | Descrição |
| --- | --- |
| Arquivo | `PersonController.java` |
| Regra | `java:S4684` |
| Classificação | Problema relacionado à segurança |

Foi identificado o mesmo padrão no método responsável pela atualização de pessoas.

A utilização direta da entidade persistente pode permitir que a estrutura interna do modelo seja exposta à camada de entrada da aplicação.

**Recomendação:** implementar DTOs específicos para atualização, com controle dos campos que podem ser modificados.

![Figura 7 — SonarQube: entidade Person usada diretamente no método update() do PersonController.](img/07-sonar-s4684-update.png)

*Figura 7 — SonarQube: entidade Person usada diretamente no método update() do PersonController.*

#### 5.2.3 Injeção de dependência por atributo no Controller

| Item | Descrição |
| --- | --- |
| Arquivo | `PersonController.java` |
| Regra | `java:S6813` |
| Classificação | Melhoria de confiabilidade e manutenibilidade |

O SonarQube identificou a utilização de `@Autowired` diretamente no atributo `personService`.

**Recomendação:** substituir a injeção por atributo pela injeção via construtor, prática que favorece a organização do código, facilita os testes e melhora a manutenibilidade.

![Figura 8 — SonarQube: injeção por atributo (@Autowired) no PersonController.](img/08-sonar-s6813-controller.png)

*Figura 8 — SonarQube: injeção por atributo (@Autowired) no PersonController.*

#### 5.2.4 Injeção de dependência por atributo no Service

| Item | Descrição |
| --- | --- |
| Arquivo | `PersonService.java` |
| Regra | `java:S6813` |
| Classificação | Melhoria de manutenibilidade |

Foi identificado o mesmo padrão de injeção de dependência diretamente no atributo `personRepository`.

**Recomendação:** utilizar injeção via construtor, mantendo um padrão consistente na aplicação.

![Figura 9 — SonarQube: injeção por atributo (@Autowired) no PersonService.](img/09-sonar-s6813-service.png)

*Figura 9 — SonarQube: injeção por atributo (@Autowired) no PersonService.*

#### 5.2.5 Recomendação de testes adicionais na classe AbstractIntegrationTest

| Item | Descrição |
| --- | --- |
| Arquivo | `AbstractIntegrationTest.java` |
| Regra | `java:S2187` |
| Classificação | Oportunidade de melhoria na cobertura de testes |

O SonarQube identificou uma recomendação relacionada à ausência de testes para essa classe de suporte.

Entretanto, a aplicação possui testes de integração na classe `PersonControllerIntegrationTest`, com seis testes executados com sucesso.

Dessa forma, o apontamento deve ser analisado considerando a finalidade da classe `AbstractIntegrationTest`, que fornece configurações e recursos compartilhados para os testes de integração.

**Recomendação:** revisar a configuração e a estrutura dos testes para verificar se existe alguma oportunidade de melhoria na cobertura dessa classe.

![Figura 10 — SonarQube: apontamento de ausência de testes na classe AbstractIntegrationTest.](img/10-sonar-s2187-abstractintegrationtest.png)

*Figura 10 — SonarQube: apontamento de ausência de testes na classe AbstractIntegrationTest.*

### 5.3 Interpretação dos resultados

A análise do SonarQube demonstra que o projeto apresenta boa manutenibilidade geral, considerando a classificação A, e não apresenta duplicação de código na análise realizada.

Entretanto, as classificações de segurança D e confiabilidade C indicam a necessidade de atenção aos problemas identificados.

O Quality Gate aprovado representa o atendimento aos critérios configurados na ferramenta, mas não significa que todos os aspectos de qualidade estejam livres de riscos.

## 6. Consolidação das métricas de qualidade

Com base nos resultados obtidos nas ferramentas utilizadas, foi possível consolidar os seguintes indicadores:

| Indicador | Resultado |
| --- | --- |
| Testes automatizados executados | 40 |
| Testes aprovados | 40 |
| Testes com falha | 0 |
| Testes com erro | 0 |
| Cobertura JaCoCo | 66% |
| Cobertura SonarQube | 67,8% |
| Cobertura de branches JaCoCo | 100% |
| Duplicação de código | 0% |
| Problemas identificados pelo SonarQube | 5 |
| Quality Gate | Passed |

As métricas de cobertura do JaCoCo e do SonarQube foram mantidas separadas, pois são resultados apresentados por ferramentas distintas e podem considerar escopos ou critérios diferentes.

Os indicadores demonstram que a aplicação possui uma base de testes automatizados funcional, mas ainda apresenta oportunidades de melhoria na cobertura e na qualidade estrutural do código.

## 7. Avaliação de riscos e oportunidades de melhoria

A partir dos resultados obtidos, foram identificados os seguintes pontos de atenção:

| ID | Ponto observado | Classificação | Recomendação |
| --- | --- | --- | --- |
| 01 | Baixa cobertura do ControllerExceptionHandler | Lacuna de testes | Criar testes para os tratamentos de exceção |
| 02 | Utilização direta da entidade Person nas requisições | Risco de segurança e acoplamento | Implementar DTOs |
| 03 | Injeção de dependência por atributo | Oportunidade de melhoria | Utilizar injeção via construtor |
| 04 | Cobertura geral de instruções de 66% | Oportunidade de melhoria | Ampliar os cenários automatizados |
| 05 | Recomendação de testes para AbstractIntegrationTest | Oportunidade de melhoria | Revisar a estrutura dos testes de integração |

Além desses pontos, a análise do projeto também identificou aspectos que merecem atenção em uma avaliação mais ampla:

- Credenciais fixas `root/root` na configuração de produção;
- Ausência de um arquivo LICENSE na raiz do repositório, gerando incerteza sobre o licenciamento do projeto;
- Necessidade de verificar o tratamento de e-mails duplicados durante atualizações;
- Necessidade de avaliar o comportamento da API diante de dados inválidos e valores que ultrapassem os limites definidos para os campos.

Esses últimos pontos devem ser tratados como riscos ou itens de investigação, e não como defeitos confirmados, enquanto não houver evidências específicas obtidas por meio de testes ou inspeção direcionada.

## 8. Quality Decision — Decisão de qualidade

Com base nas evidências obtidas durante a execução dos testes automatizados, na análise de cobertura com JaCoCo e na inspeção estática com SonarQube, foi definida a decisão:

> ## CONDITIONAL GO

A decisão indica que o sistema apresenta condições técnicas favoráveis para continuar sua avaliação, mas ainda necessita de melhorias antes de uma recomendação irrestrita de liberação em ambiente de produção.

### 8.1 Justificativa

Os principais fatores positivos identificados foram:

- Execução bem-sucedida dos 40 testes automatizados;
- Ausência de falhas e erros na execução final;
- Cobertura de 100% dos branches analisados pelo JaCoCo;
- Ausência de duplicação de código identificada pelo SonarQube;
- Quality Gate aprovado;
- Cobertura de 100% nos pacotes Controller, Services e Model, conforme os indicadores do JaCoCo.

Por outro lado, foram identificados fatores que justificam a decisão condicional:

- Cobertura geral de instruções de 66%;
- Baixa cobertura da classe responsável pelo tratamento de exceções;
- Dois apontamentos relacionados ao uso direto da entidade persistente nas requisições;
- Necessidade de melhorias na injeção de dependências;
- Existência de riscos que ainda precisam de investigação, especialmente relacionados às configurações de produção e ao licenciamento.

### 8.2 Condições recomendadas antes da liberação

1. Ampliar os testes automatizados para os tratamentos de exceção.
2. Revisar a utilização direta da entidade Person nos métodos de criação e atualização.
3. Substituir a injeção de dependência por atributo pela injeção via construtor.
4. Revisar as credenciais utilizadas na configuração de produção.
5. Confirmar as condições de licenciamento do projeto.
6. Executar testes adicionais para os cenários de validação e integridade dos dados.

A decisão poderá ser reavaliada após a implementação das melhorias e a realização de novos testes.

## 9. Melhorias propostas

Com base nas evidências coletadas, são propostas as seguintes ações:

- Ampliar a cobertura dos testes, priorizando os métodos ainda não executados.
- Criar testes específicos para respostas de erro e exceções.
- Implementar DTOs para controlar os dados recebidos pela API.
- Padronizar a injeção de dependências por construtor.
- Revisar as configurações de segurança do ambiente de produção.
- Documentar corretamente o licenciamento do projeto.
- Manter a execução de testes automatizados e inspeções estáticas como parte do processo de desenvolvimento.

Essas ações podem contribuir para aumentar a confiabilidade, a segurança e a manutenibilidade da aplicação.

## 10. Lições aprendidas

Durante a avaliação, foi possível observar a importância de combinar diferentes técnicas de garantia da qualidade de software.

A execução dos testes automatizados permitiu verificar o comportamento dos principais componentes da aplicação, enquanto o JaCoCo demonstrou quais partes do código ainda não possuem cobertura adequada.

O SonarQube complementou a avaliação ao identificar problemas estruturais e oportunidades de melhoria que não necessariamente são detectados durante a execução dos testes.

Também foi possível identificar a importância da compatibilidade entre ferramentas e ambientes de execução, como ocorreu na configuração do Testcontainers com o Docker Desktop.

A atividade demonstrou que a aprovação dos testes não representa, isoladamente, uma garantia de qualidade. É necessário analisar os resultados em conjunto, considerar os riscos e estabelecer melhorias contínuas.

## 11. Conclusão

A avaliação do projeto rest-spring-test demonstrou que a aplicação possui uma estrutura funcional e uma suíte de testes automatizados capaz de validar os principais componentes analisados.

A execução final apresentou 40 testes aprovados, sem falhas ou erros. Entretanto, a análise de cobertura identificou uma lacuna significativa no tratamento de exceções, enquanto a inspeção estática apontou oportunidades de melhoria relacionadas à segurança, à confiabilidade e à organização do código.

Diante das evidências coletadas, a decisão de qualidade foi classificada como **Conditional Go**, indicando que o projeto apresenta resultados positivos, mas necessita de ajustes e verificações adicionais antes de uma liberação sem restrições.

Conclui-se que a qualidade de software deve ser avaliada continuamente, utilizando evidências técnicas, métricas, testes e inspeções para apoiar decisões mais seguras durante o desenvolvimento e a manutenção de sistemas.
