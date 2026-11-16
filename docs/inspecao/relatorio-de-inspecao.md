# Relatório de Inspeção de Código

> Entrega 3 · 16/11/2026 · Projeto A3 — Gestão da Qualidade de Software

Análise estática e revisão técnica do sistema [rest-spring-test](https://github.com/douglasfragoso/rest-spring-test) (Java 21, Spring Boot 3.3.3).

| Item | Descrição |
| --- | --- |
| Unidade Curricular | Gestão da Qualidade de Software |
| Professor | Emerson dos Santos Paduan |
| Integrantes | Rafael Stender Rocha, Gian Carlo Tozzatti, Theobaldo Ocerio da Silva |
| Entrega | Entrega 3 — Relatório de Inspeção |

## 1. Objetivo e escopo

Este relatório registra os resultados da inspeção de código do rest-spring-test, realizada como teste estático previsto no [Plano de Testes](../plano-de-testes/plano-de-testes.md) da Entrega 2 (ISO/IEC/IEEE 29119). O objetivo é identificar defeitos, riscos e oportunidades de melhoria sem executar o sistema, e reunir evidências para a decisão de release do projeto.

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

A inspeção combinou leitura do código-fonte e dos arquivos de configuração com revisão técnica orientada pelos riscos da [análise de riscos](../riscos/analise-de-riscos.md). Cada achado recebeu um identificador (QA-nn) e foi registrado com evidência, identificação, análise técnica, impacto na qualidade, classificação e recomendação.

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
| [QA-05](#qa-05) | Organização dos testes de integração | Testes de integração | Ponto positivo |
| [QA-06](#qa-06) | Dependência do Docker para execução dos testes de integração | Testes de integração | Risco |
| [QA-07](#qa-07) | Utilização de banco de dados real nos testes | Testes de integração | Ponto positivo |
| [QA-08](#qa-08) | Configuração dinâmica das propriedades de conexão | Testes de integração | Ponto positivo |
| [QA-12](#qa-12) | Asserções do teste de cadastro sempre verdadeiras | Teste de integração (POST) | Defeito |
| [QA-13](#qa-13) | Dependência entre os testes automatizados | Teste de integração | Oportunidade de melhoria |
| [QA-14](#qa-14) | Ausência de verificação efetiva da persistência na atualização | Teste de integração (PUT) | Oportunidade de melhoria |
| [QA-15](#qa-15) | Dependência de uma quantidade fixa de registros no teste de listagem | Teste de integração (GET) | Risco |
| [QA-16](#qa-16) | Ausência de verificação do resultado da exclusão | Teste de integração (DELETE) | Oportunidade de melhoria |
| [QA-17](#qa-17) | Verificação automatizada da disponibilidade da documentação Swagger | Teste de integração (Swagger) | Ponto positivo |
| [QA-18](#qa-18) | Cobertura incompleta das verificações no teste de atualização | Teste de repositório | Oportunidade de melhoria |
| [QA-19](#qa-19) | Verificação limitada da operação de salvamento | Teste de repositório | Oportunidade de melhoria |
| [QA-20](#qa-20) | Ausência de retorno sobre o resultado da atualização | Repository | Oportunidade de melhoria |
| [QA-21](#qa-21) | Dependência de consulta SQL nativa | Repository | Oportunidade de melhoria |
| [QA-22](#qa-22) | Tipo de resposta documentado diverge do retornado no PUT | Controller / OpenAPI | Defeito |
| [QA-23](#qa-23) | Ausência de verificação de e-mail duplicado durante a atualização | Service | Pendente de verificação |
| [QA-24](#qa-24) | Validação de formato de e-mail não associada aos grupos de validação | Model (Person) | Defeito |
| [QA-25](#qa-25) | Ausência de validação explícita dos limites de caracteres | Model (Person) | Oportunidade de melhoria |
| [QA-26](#qa-26) | Identificação de exceções por comparação textual | Tratamento de exceções | Oportunidade de melhoria |
| [QA-27](#qa-27) | Tratamento de rotas inexistentes dependente de configuração | Tratamento de exceções | Pendente de verificação |
| [QA-28](#qa-28) | Inconsistência de nomenclatura na classe StandartError | Tratamento de exceções | Oportunidade de melhoria |
| [QA-43](#qa-43) | Centralização de configurações dos testes | Testes | Ponto positivo |

## 4. Achados detalhados

### 4.1 Infraestrutura dos testes de integração

<a id="qa-05"></a>
#### QA-05 — Organização dos testes de integração

**Evidência:** `src/test/java/com/restspringtest/Integration/`

**Identificação:** Os testes de integração têm uma classe-base (`AbstractIntegrationTest`) e classes específicas para o controller de pessoas e para a documentação Swagger.

**Impacto na qualidade:** Favorece o reaproveitamento de configurações e a separação de responsabilidades entre os testes.

**Classificação:** Ponto positivo — boa organização da suíte.

<a id="qa-06"></a>
#### QA-06 — Dependência do Docker para execução dos testes de integração

**Evidência:** `AbstractIntegrationTest.java`, método `startContainers()`

**Identificação:** A classe usa TestContainers para iniciar um banco MySQL em contêiner Docker.

**Análise técnica:** `MySQLContainer` cria o banco a partir da imagem `mysql:8.0.28`, e `startContainers()` usa `Startables.deepStart()` para iniciar o contêiner e aguardar a inicialização. Os testes passam a usar um banco real, mas dependem de infraestrutura externa.

**Impacto na qualidade:** Sem Docker, ou com falha na inicialização do contêiner, os testes que dependem dessa configuração não executam.

**Classificação:** Risco de ambiente de testes.

**Recomendação:** Garantir Docker ativo no ambiente de execução e registrar a cobertura com e sem os testes de integração.

<a id="qa-07"></a>
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

<a id="qa-08"></a>
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

<a id="qa-43"></a>
#### QA-43 — Centralização de configurações dos testes

**Identificação:** As configurações dos testes ficam em um único local, em vez de repetidas em várias classes.

**Impacto na qualidade:** Facilita alterações futuras e melhora a organização do código. Nenhum defeito funcional foi identificado nessa classe.

**Classificação:** Ponto positivo — organização.

### 4.2 Qualidade dos testes automatizados

<a id="qa-12"></a>
#### QA-12 — Asserções do teste de cadastro sempre verdadeiras

**Evidência:** Método `integrationTestGivenPerson_whenPostPerson_thenReturnPerson()`

**Identificação:** O teste atribui a resposta da API à variável de referência antes de comparar os campos.

```java
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

<a id="qa-13"></a>
#### QA-13 — Dependência entre os testes automatizados

**Evidência:** Atributo estático `person` e uso de `@Order`

**Identificação:** Os cenários compartilham a variável estática `private static Person person`: o cadastro a preenche, a atualização usa seu id e as consultas seguintes dependem dos dados nela.

**Análise técnica:** O resultado dos testes posteriores depende do sucesso dos anteriores. Se o cadastro falhar, os demais recebem um id nulo ou inexistente.

**Impacto na qualidade:** Dificulta a execução isolada dos testes, a manutenção da suíte e a identificação da origem das falhas.

**Classificação:** Oportunidade de melhoria na manutenibilidade e confiabilidade dos testes.

**Recomendação:** Criar dados independentes para cada cenário, com preparação e limpeza do ambiente de testes.

<a id="qa-14"></a>
#### QA-14 — Ausência de verificação efetiva da persistência na atualização

**Evidência:** Método `integrationTestGivenPerson_whenPutPerson_thenReturnString()`

**Identificação:** O teste de atualização verifica a mensagem de retorno e compara o objeto original com o enviado.

```java
assertEquals("Profile updated successfully", responseMessage);
assertNotEquals(person.getFirstName(), updatePerson.getFirstName());
assertNotEquals(person.getLastName(), updatePerson.getLastName());
```

**Análise técnica:** As duas últimas asserções só confirmam que os dados enviados são diferentes dos originais; não comprovam que o banco persistiu a alteração. Um teste posterior de consulta por id verifica os novos nomes, mas o cenário de atualização não se sustenta sozinho.

**Impacto na qualidade:** A verificação isolada do PUT é limitada e depende de outros cenários para confirmar a persistência.

**Classificação:** Oportunidade de melhoria na cobertura das verificações.

**Recomendação:** Após o PUT, consultar o registro e verificar se os dados foram de fato atualizados.

<a id="qa-15"></a>
#### QA-15 — Dependência de uma quantidade fixa de registros no teste de listagem

**Evidência:** Método `integrationTest_whenGetAllPersons_thenReturnListOfPerson()`

**Identificação:** O teste espera exatamente dois registros: `assertEquals(2, page.getContent().size());`

**Análise técnica:** A expectativa depende do estado do banco durante a execução. Registros de outros testes ou de execuções anteriores alteram o resultado.

**Impacto na qualidade:** Compromete a estabilidade e a repetibilidade dos testes.

**Classificação:** Risco de dependência do estado do banco de dados.

**Recomendação:** Preparar um conjunto controlado de dados antes da execução e verificar a quantidade esperada para aquele cenário.

<a id="qa-16"></a>
#### QA-16 — Ausência de verificação do resultado da exclusão

**Evidência:** Método `integrationTestGivenId_whenGetDeletePersonById_thenNoContent()`

**Identificação:** O teste de exclusão verifica apenas o código HTTP 204.

```java
given().spec(specification)
    .pathParam("id", person.getId())
    .when().delete("/id/{id}")
    .then().statusCode(204);
```

**Análise técnica:** Não há consulta posterior para confirmar que o registro deixou de existir.

**Impacto na qualidade:** A validação confirma o código de resposta, mas não comprova o resultado da exclusão no sistema.

**Classificação:** Oportunidade de melhoria na validação funcional.

**Recomendação:** Após a exclusão, consultar o id removido e verificar o comportamento esperado para um registro inexistente (HTTP 404).

<a id="qa-17"></a>
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

<a id="qa-18"></a>
#### QA-18 — Cobertura incompleta das verificações no teste de atualização

**Evidência:** `src/test/java/com/restspringtest/Repository/PersonRepositoryTest.java`, método `testGivenPersonList_whenUpdate_ThenReturnPerson()`

**Identificação:** O teste atualiza cinco atributos de `Person`, mas verifica apenas quatro; `lastName` não tem asserção após a atualização.

**Análise técnica:** Uma falha na atualização do sobrenome não seria detectada por esse teste.

**Impacto na qualidade:** Reduz a cobertura das verificações funcionais da atualização.

**Classificação:** Oportunidade de melhoria na qualidade dos testes.

**Recomendação:** Acrescentar uma asserção confirmando que o sobrenome foi atualizado para "Doe2".

<a id="qa-19"></a>
#### QA-19 — Verificação limitada da operação de salvamento

**Evidência:** `src/test/java/com/restspringtest/Repository/PersonRepositoryTest.java`, método `testGivenPersonObject_whenSave_ThenReturPerson()`

**Identificação:** O teste verifica apenas se o objeto retornado por `save()` não é nulo e se recebeu um id positivo.

**Análise técnica:** Não há verificação explícita dos demais atributos após o salvamento.

**Impacto na qualidade:** Uma inconsistência na persistência de algum atributo pode passar despercebida.

**Classificação:** Oportunidade de melhoria na cobertura dos testes de persistência.

**Recomendação:** Verificar os principais atributos após o salvamento, de preferência consultando novamente pelo id.

### 4.3 Model e validações

<a id="qa-24"></a>
#### QA-24 — Validação de formato de e-mail não associada aos grupos de validação

**Evidência:** `src/main/java/com/restspringtest/Model/Person.java`, campo `email`

**Identificação:** A anotação `@Email` não declara os grupos `Create` e `Update`, ao contrário de `@NotBlank` no mesmo atributo.

**Análise técnica:** O controller valida explicitamente os grupos `Create.class` e `Update.class`. Como `@Email` pertence ao grupo `Default`, ela não é executada no cadastro nem na atualização.

**Impacto na qualidade:** A aplicação pode aceitar e-mails com formato inválido, comprometendo a integridade dos dados.

**Classificação:** Defeito — falha de configuração de validação identificada por análise estática.

**Recomendação:** Associar `@Email` aos grupos `Create` e `Update`, ou incluir o grupo `Default` na validação do controller.

<a id="qa-25"></a>
#### QA-25 — Ausência de validação explícita dos limites de caracteres

**Evidência:** `src/main/java/com/restspringtest/Model/Person.java`, campos `firstName`, `lastName`, `address`, `gender` e `email`

**Identificação:** Os limites de tamanho estão definidos só em `@Column(length = ...)`, sem `@Size`.

**Análise técnica:** O tamanho da coluna no mapeamento JPA não valida o conteúdo recebido pela API; valores acima do limite chegam à camada de persistência.

**Impacto na qualidade:** Possibilidade de erros na persistência e ausência de mensagens de validação antecipadas e específicas para o usuário.

**Classificação:** Oportunidade de melhoria na validação de dados.

**Recomendação:** Adicionar `@Size(max = ...)` compatível com os limites das colunas.

### 4.4 Service e regras de negócio

<a id="qa-23"></a>
#### QA-23 — Ausência de verificação de e-mail duplicado durante a atualização

**Evidência:** `src/main/java/com/restspringtest/Services/PersonService.java`, método `update()`

**Identificação:** O método verifica se o id existe, mas não se o novo e-mail já pertence a outro registro.

**Análise técnica:** O método `save()` verifica explicitamente e-mail duplicado antes do cadastro; a mesma regra de negócio não existe em `update()`.

**Impacto na qualidade:** Possível inconsistência nos dados cadastrais caso o banco não tenha restrição de unicidade no campo de e-mail.

**Classificação:** Possível falha de regra de negócio, pendente de verificação da restrição de unicidade no banco.

**Recomendação:** Verificar o e-mail na atualização, desconsiderando o próprio registro que está sendo atualizado.

### 4.5 Repository e consultas SQL

<a id="qa-20"></a>
#### QA-20 — Ausência de retorno sobre o resultado da atualização

**Evidência:** `src/main/java/com/restspringtest/Repository/PersonRepository.java`, método `updatePerson()`

**Identificação:** O método de atualização retorna `void` e não informa quantos registros foram afetados.

**Análise técnica:** Com um id inexistente nenhuma linha é atualizada, mas o chamador não consegue distinguir esse caso de uma atualização bem-sucedida.

**Impacto na qualidade:** Limita o tratamento de atualizações sem correspondência no banco.

**Classificação:** Oportunidade de melhoria.

**Recomendação:** Retornar `int` com a quantidade de registros modificados e tratar o caso de nenhuma linha afetada.

<a id="qa-21"></a>
#### QA-21 — Dependência de consulta SQL nativa

**Evidência:** `src/main/java/com/restspringtest/Repository/PersonRepository.java`, método `updatePerson()`

**Identificação:** A atualização usa SQL nativo via `@Query(nativeQuery = true)`.

**Análise técnica:** A consulta referencia diretamente a tabela `tb_person` e suas colunas físicas; mudanças no esquema exigem ajuste manual da instrução.

**Impacto na qualidade:** Maior acoplamento entre o repositório e a estrutura física do banco.

**Classificação:** Observação arquitetural.

**Recomendação:** Avaliar a necessidade da consulta nativa ou usar os recursos de abstração do JPA.

### 4.6 Controller e documentação da API

<a id="qa-22"></a>
#### QA-22 — Tipo de resposta documentado diverge do retornado no PUT

**Evidência:** `src/main/java/com/restspringtest/Controller/PersonController.java`, método `update()`

**Identificação:** A documentação OpenAPI do `PUT /person` não corresponde à implementação.

**Análise técnica:** `@ApiResponse` declara `Person.class` como modelo da resposta HTTP 200, mas o método retorna `ResponseEntity<String>` com a mensagem "Profile updated successfully".

**Impacto na qualidade:** A documentação pode levar consumidores da API a interpretar incorretamente o formato da resposta, prejudicando integrações.

**Classificação:** Defeito — inconsistência documental.

**Recomendação:** Documentar a resposta como String ou alterar a implementação para retornar `Person`, conforme o comportamento desejado.

### 4.7 Tratamento de exceções

<a id="qa-26"></a>
#### QA-26 — Identificação de exceções por comparação textual

**Evidência:** `src/main/java/com/restspringtest/Controller/Exception/ControllerExceptionHandler.java`, método `handleDatabaseExceptionId()`

**Identificação:** O tratamento de `DatabaseException` decide o tipo de erro comparando um trecho da mensagem.

**Análise técnica:** A classificação depende do texto "Email already registered". Uma mudança na mensagem altera o status HTTP devolvido (400 ou 404).

**Impacto na qualidade:** Acopla o tratamento de erros às mensagens das exceções, dificultando manutenção e evolução.

**Classificação:** Oportunidade de melhoria.

**Recomendação:** Usar exceções específicas ou identificadores próprios para cada categoria de erro.

<a id="qa-27"></a>
#### QA-27 — Tratamento de rotas inexistentes dependente de configuração

**Evidência:** `src/main/java/com/restspringtest/Controller/Exception/ControllerExceptionHandler.java`, método `handleNotFoundException()`

**Identificação:** Existe tratamento específico para `NoHandlerFoundException`, retornando HTTP 404.

**Análise técnica:** O tratamento só é acionado se o Spring MVC estiver configurado para lançar `NoHandlerFoundException` em rotas sem correspondência. Sem executar uma requisição a uma rota inexistente não é possível confirmar o comportamento.

**Impacto na qualidade:** Possível inconsistência no formato das respostas de erro para rotas inexistentes.

**Classificação:** Ponto de investigação pendente de validação prática.

**Recomendação:** Conferir a configuração do Spring MVC e criar um teste de integração para uma rota inexistente (caso de teste CT22).

<a id="qa-28"></a>
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

A inspeção confirmou na leitura do código os principais riscos de produto levantados na [análise de riscos](../riscos/analise-de-riscos.md). A confirmação do efeito em execução fica com os casos de teste correspondentes da [matriz de rastreabilidade](../rastreabilidade/matriz-de-rastreabilidade.md).

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
