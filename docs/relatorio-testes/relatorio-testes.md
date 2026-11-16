# Relatório Prático de Testes

> Execução de testes, cobertura de código, inspeção estática e Quality Decision

| Item | Descrição |
| --- | --- |
| Unidade Curricular | Gestão da Qualidade de Software |
| Professor | Emerson dos Santos Paduan |
| Integrantes | Rafael Stender Rocha, Gian Carlo Tozzatti, Theobaldo Ocerio da Silva |
| Sistema avaliado | [rest-spring-test](https://github.com/douglasfragoso/rest-spring-test) |
| Entrega | Entrega 3 — Quality Assurance em Sistemas Open Source |
| Data | 16/11/2026 |

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
