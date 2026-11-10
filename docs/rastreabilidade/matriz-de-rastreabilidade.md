# Requisitos e matriz de rastreabilidade

> Entrega 2 · 09/11/2026 · Projeto A3 — Gestão da Qualidade de Software

Os 18 requisitos abaixo foram extraídos do código, das anotações Swagger e do README, e cada um está ligado a pelo menos um dos 34 casos de teste planejados. O sistema não tem documento de requisitos, então RN05, RN03 no PUT e os RNF são expectativas derivadas pela equipe.

## 1. Requisitos

| ID | Tipo | Descrição | Fonte |
| --- | --- | --- | --- |
| RF01 | Funcional | Listar pessoas com paginação e ordenação (padrão 10 por página, por `id`) | `GET /person` |
| RF02 | Funcional | Buscar pessoa por id | `GET /person/id/{id}` |
| RF03 | Funcional | Buscar pessoa por e-mail | `GET /person/email/{email}` |
| RF04 | Funcional | Cadastrar pessoa e retornar 201 | `POST /person` |
| RF05 | Funcional | Atualizar todos os dados de uma pessoa existente | `PUT /person` |
| RF06 | Funcional | Excluir pessoa por id e retornar 204 | `DELETE /person/id/{id}` |
| RN01 | Regra de negócio | Nome, sobrenome, endereço, gênero e e-mail são obrigatórios no cadastro e na atualização | `@NotBlank` em `Person` |
| RN02 | Regra de negócio | O e-mail deve ter formato válido | `@Email` em `Person` |
| RN03 | Regra de negócio | Dois cadastros não podem ter o mesmo e-mail | `PersonService.save` |
| RN04 | Regra de negócio | O id é obrigatório na atualização | `@NotNull` grupo `Update` |
| RN05 | Regra de negócio | Campos acima do tamanho da coluna devem ser rejeitados com 400 | Tamanhos em `@Column` (derivado) |
| RN06 | Regra de negócio | Recurso inexistente retorna 404 com corpo padronizado | `ControllerExceptionHandler` |
| RN07 | Regra de negócio | Parâmetro de tipo inválido retorna 400 | `ControllerExceptionHandler` |
| RNF01 | Não funcional | Documentação OpenAPI acessível e coerente com o comportamento real | Swagger UI |
| RNF02 | Não funcional | Erros sempre em JSON padronizado (timestamp, status, error, message, path) | `StandartError`, `ValidationError` |
| RNF03 | Não funcional | Configuração sem credenciais fixas no código | Boa prática (derivado) |
| RNF04 | Não funcional | Cobertura de testes ≥ 80% de linhas e ≥ 70% de ramos | Critério do plano (4.4) |
| RNF05 | Não funcional | Código sem bugs e vulnerabilidades Blocker/Critical | Critério do plano (4.4) |

## 2. Casos de teste planejados

| ID | Caso de teste | Resultado esperado | Nível | Técnica |
| --- | --- | --- | --- | --- |
| CT01 | Listar sem parâmetros com 12 pessoas cadastradas | 200, 10 itens, ordem por id | Sistema | Partição de equivalência |
| CT02 | Listar com `page=1&size=5&sort=firstName,desc` | 200, página e ordem pedidas | Sistema | Partição de equivalência |
| CT03 | Listar com `size=0` e `page=-1` | 200 com valores ajustados, sem erro 500 | Sistema | Valor-limite |
| CT04 | Buscar id existente | 200 + pessoa | Sistema | Partição de equivalência |
| CT05 | Buscar id inexistente | 404 + `StandartError` | Sistema | Partição de equivalência |
| CT06 | Buscar e excluir id `abc` | 400 "Type mismatch" | Sistema | Partição de equivalência |
| CT07 | Buscar e-mail existente (com ponto e `+`) | 200 + pessoa | Sistema | Partição de equivalência |
| CT08 | Buscar e-mail inexistente | 404 + `StandartError` | Sistema | Partição de equivalência |
| CT09 | Cadastrar pessoa válida | 201 + pessoa com id; GET encontra | Sistema | Partição de equivalência |
| CT10 | Cadastrar com cada campo obrigatório vazio, ausente ou só espaços | 400 com a mensagem do campo | Sistema | Partição de equivalência |
| CT11 | Cadastrar com e-mail `joao.com`, `joao@`, `@x.com` | 400 "must be a valid email" | Sistema | Partição de equivalência |
| CT12 | Cadastrar com e-mail já existente | 400 "Email already registered" | Sistema | Tabela de decisão |
| CT13 | Cadastrar com nome e sobrenome de 20 e 21 caracteres | 201 e 400 | Sistema | Valor-limite |
| CT14 | Cadastrar com endereço 100/101, gênero 10/11, e-mail 50/51 caracteres | 201 no limite, 400 acima | Sistema | Valor-limite |
| CT15 | Atualizar pessoa existente com dados válidos | 200; GET mostra os dados novos | Sistema | Tabela de decisão |
| CT16 | Atualizar id inexistente | 404 | Sistema | Tabela de decisão |
| CT17 | Atualizar sem id | 400 "The field id is required" | Sistema | Partição de equivalência |
| CT18 | Atualizar trocando o e-mail para o de outra pessoa | 400; e-mail continua único | Sistema | Tabela de decisão |
| CT19 | Atualizar com campo obrigatório vazio, e-mail inválido ou campo acima do tamanho | 400; nada alterado | Sistema | Partição de equivalência |
| CT20 | Excluir id existente e buscar em seguida | 204, depois 404 | Sistema | Partição de equivalência |
| CT21 | Excluir id inexistente | 404 | Sistema | Partição de equivalência |
| CT22 | Chamar rota inexistente, método não suportado e enviar JSON malformado | 404 / 405 / 400 em JSON padronizado | Sistema | Partição de equivalência |
| CT23 | Executar os exemplos do Swagger UI para POST e PUT | Exemplos funcionam como documentado | Aceitação | Revisão de documentação |
| CT24 | `PersonService.save`: e-mail novo e e-mail existente | Salva / lança `DatabaseException` | Unitário | Cobertura de decisão |
| CT25 | `PersonService.update`: id existente e inexistente | Chama `updatePerson` / lança exceção | Unitário | Cobertura de decisão |
| CT26 | `findById`, `findByEmail`, `deleteById` com `Optional` vazio e presente | Retorna / lança exceção | Unitário | Cobertura de decisão |
| CT27 | `handleDatabaseExceptionId` com as duas mensagens | 400 e 404 | Unitário | Cobertura de decisão |
| CT28 | Demais handlers (type mismatch, no handler, validação) | 400 / 404 com `ValidationError` | Unitário | Cobertura de comandos |
| CT29 | Validar `Person` com Validator nos grupos `Create` e `Update` | Violações esperadas por campo, incluindo `@Email` | Unitário | Cobertura de comandos |
| CT30 | `findByEmail` e `updatePerson` contra H2 | Consulta e UPDATE corretos | Integração | Caixa-branca |
| CT31 | CRUD completo contra MySQL (TestContainers), suíte original incluída | Todos passam | Integração / Regressão | Caixa-preta |
| CT32 | Análise estática no SonarQube | 0 Blocker/Critical; Quality Gate aprovado | Inspeção | Análise estática |
| CT33 | Revisão de código com checklist (credenciais, exceções, validação) | Itens do checklist atendidos | Inspeção | Revisão técnica |
| CT34 | Relatório JaCoCo da suíte completa | ≥ 80% linhas, ≥ 70% ramos | Medição | Cobertura |

## 3. Matriz de rastreabilidade

| Requisito | Casos de teste | Testes já existentes no repositório | Risco |
| --- | --- | --- | --- |
| RF01 | CT01, CT02, CT03, CT31 | Sim (Service, Controller, Repository, Integração) | — |
| RF02 | CT04, CT05, CT26, CT31 | Sim | — |
| RF03 | CT07, CT08, CT26, CT30, CT31 | Sim | — |
| RF04 | CT09, CT24, CT31 | Sim | R01, R03 |
| RF05 | CT15, CT16, CT25, CT30, CT31 | Sim | R02 |
| RF06 | CT20, CT21, CT26, CT31 | Sim | — |
| RN01 | CT10, CT19, CT29 | Não | — |
| RN02 | CT11, CT19, CT29 | Não | R01 |
| RN03 | CT12, CT18, CT24, CT27 | Parcial (só no cadastro) | R02 |
| RN04 | CT17, CT29 | Não | — |
| RN05 | CT13, CT14, CT19 | Não | R03 |
| RN06 | CT05, CT08, CT16, CT21, CT27 | Sim | R04 |
| RN07 | CT06, CT28 | Não | — |
| RNF01 | CT23 | Parcial (só carrega a página) | R06 |
| RNF02 | CT03, CT22, CT27, CT28 | Não | R03, R04 |
| RNF03 | CT33 | Não | R05 |
| RNF04 | CT34 | Relatório configurado, meta não definida | R08 |
| RNF05 | CT32, CT33 | Não | R07, R09 |

Todos os 18 requisitos têm ao menos um caso de teste (detalhados na [especificação de casos de teste](../casos-de-teste/casos-de-teste.md)); 8 deles (RN01, RN02, RN04, RN05, RN07, RNF02, RNF03, RNF05) não tinham nenhum teste no repositório original e RN03 só era testado no cadastro, e é aí que a equipe concentra os testes novos.
