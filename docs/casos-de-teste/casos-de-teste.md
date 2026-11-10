# Especificação de casos de teste — ISO/IEC/IEEE 29119-3

> Entrega 3 · versão de 10/11/2026 · Projeto A3 — Gestão da Qualidade de Software

Este documento detalha os 34 casos de teste planejados na [matriz de rastreabilidade](../rastreabilidade/matriz-de-rastreabilidade.md) (CT01 a CT34), com pré-condições, dados de entrada, passos e resultado esperado de cada um. O resultado esperado vem dos requisitos (oráculo), e não do comportamento atual do sistema. Quando a leitura do código indica que o sistema vai divergir do esperado, isso aparece numa nota **Leitura do código**: é uma hipótese a confirmar na execução, não um resultado.

## 1. Convenções

| Item | Convenção |
| --- | --- |
| URL base | `http://localhost:8080` (perfil `test`, H2). Nos testes de integração com TestContainers a porta é `8888` |
| Cabeçalhos | `Content-Type: application/json` em POST e PUT |
| Notação `X×n` | O caractere `X` repetido `n` vezes. Ex.: `A×20` = `AAAAAAAAAAAAAAAAAAAA` |
| `StandartError` | `{ timestamp, status, error, message: "texto", path }` |
| `ValidationError` | `{ timestamp, status, error: "Validation Error", message: ["texto", ...], path }` |
| Prioridade | Alta / Média / Base, conforme a [análise de riscos](../riscos/analise-de-riscos.md) |
| Status | Não executado → Aprovado / Reprovado / Bloqueado (preenchido no registro de execução) |

Um caso é **Aprovado** apenas quando todos os seus passos produzem o resultado esperado. Cada passo reprovado gera um relatório de incidente (issue no GitHub) que referencia o ID do caso e o número do passo (ex.: `CT14.5`).

## 2. Dados de teste

### 2.1 Pessoas de referência

Todos os dados são fictícios. Os testes de sistema criam a massa pelo próprio `POST /person`, já que o H2 começa vazio a cada inicialização.

| Ref. | firstName | lastName | address | gender | email |
| --- | --- | --- | --- | --- | --- |
| P1 | Ana | Souza | Rua das Flores, 100 | F | ana.souza@teste.com |
| P2 | Bruno | Lima | Av. Brasil, 200 | M | bruno.lima@teste.com |
| P3 | Carla | Dias | Rua XV, 15 | F | carla.dias+qa@teste.com |
| L01–L12 | Pessoa01 … Pessoa12 | Teste | Rua Teste, 1 … 12 | M | pessoa01@teste.com … pessoa12@teste.com |

Corpo JSON de P1, usado como base nos casos que alteram um único campo:

```json
{
  "firstName": "Ana",
  "lastName": "Souza",
  "address": "Rua das Flores, 100",
  "gender": "F",
  "email": "ana.souza@teste.com"
}
```

### 2.2 Valores-limite de tamanho

| Campo | Coluna | No limite (válido) | Limite + 1 (inválido) |
| --- | --- | --- | --- |
| firstName | VARCHAR(20) | `A×20` | `A×21` |
| lastName | VARCHAR(20) | `B×20` | `B×21` |
| address | VARCHAR(100) | `R×100` | `R×101` |
| gender | VARCHAR(10) | `G×10` | `G×11` |
| email | VARCHAR(50) | `a×40` + `@teste.com` (50) | `a×41` + `@teste.com` (51) |

### 2.3 Classes de equivalência

| Entrada | Classes válidas | Classes inválidas |
| --- | --- | --- |
| Campo texto obrigatório | Texto com 1 a N caracteres | Ausente (`null`), vazio (`""`), só espaços (`"   "`), acima de N caracteres |
| email | Formato `local@dominio.tld`, inclusive com `.` e `+` na parte local | Sem `@` (`joao.com`), sem domínio (`joao@`), sem parte local (`@teste.com`), com espaço (`joao silva@teste.com`) |
| id (path) | Inteiro de pessoa existente | Inteiro inexistente (`999`), não numérico (`abc`) |
| id (corpo do PUT) | Inteiro de pessoa existente | Ausente, inexistente |
| page / size | `page ≥ 0`, `1 ≤ size ≤ 2000` | `page < 0`, `size < 1`, `size > 2000` (o Spring ajusta, não rejeita) |

### 2.4 Tabelas de decisão

**Cadastro (`POST /person`)**

| Condição / ação | R1 | R2 | R3 |
| --- | --- | --- | --- |
| Dados válidos (obrigatórios, formato, tamanho)? | S | S | N |
| E-mail já cadastrado? | N | S | — |
| **Resultado esperado** | 201 + pessoa | 400 "Email already registered" | 400 "Validation Error" |
| **Casos** | CT09, CT13, CT14 | CT12 | CT10, CT11, CT13, CT14 |

**Atualização (`PUT /person`)**

| Condição / ação | R1 | R2 | R3 | R4 | R5 | R6 |
| --- | --- | --- | --- | --- | --- | --- |
| id informado? | N | S | S | S | S | S |
| Dados válidos? | — | N | S | S | S | S |
| id existe? | — | — | N | S | S | S |
| Novo e-mail pertence a outra pessoa? | — | — | — | N (e-mail novo) | N (mantém o próprio) | S |
| **Resultado esperado** | 400 "The field id is required" | 400 "Validation Error" | 404 | 200 | 200 | 400, nada alterado |
| **Casos** | CT17 | CT19 | CT16 | CT15 | CT15 | CT18 |

## 3. Índice dos casos

| ID | Caso | Requisitos | Riscos | Prioridade | Status |
| --- | --- | --- | --- | --- | --- |
| CT01 | Listagem padrão com 12 pessoas | RF01 | — | Base | Não executado |
| CT02 | Listagem com página, tamanho e ordenação | RF01 | — | Base | Não executado |
| CT03 | Listagem com parâmetros nos limites | RF01, RNF02 | — | Base | Não executado |
| CT04 | Buscar id existente | RF02 | — | Base | Não executado |
| CT05 | Buscar id inexistente | RF02, RN06 | R04 | Base | Não executado |
| CT06 | id não numérico em GET e DELETE | RN07 | — | Base | Não executado |
| CT07 | Buscar e-mail existente com `.` e `+` | RF03 | R02 | Base | Não executado |
| CT08 | Buscar e-mail inexistente | RF03, RN06 | — | Base | Não executado |
| CT09 | Cadastrar pessoa válida | RF04 | — | Base | Não executado |
| CT10 | Cadastrar com campo obrigatório inválido | RN01 | — | Base | Não executado |
| CT11 | Cadastrar com e-mail inválido | RN02 | R01 | Alta | Não executado |
| CT12 | Cadastrar com e-mail repetido | RN03 | R02 | Base | Não executado |
| CT13 | Limites de nome e sobrenome | RN05 | R03 | Alta | Não executado |
| CT14 | Limites de endereço, gênero e e-mail | RN05 | R03 | Alta | Não executado |
| CT15 | Atualizar pessoa existente | RF05 | — | Base | Não executado |
| CT16 | Atualizar id inexistente | RF05, RN06 | — | Base | Não executado |
| CT17 | Atualizar sem id | RN04 | — | Base | Não executado |
| CT18 | Atualizar para o e-mail de outra pessoa | RN03 | R02 | Alta | Não executado |
| CT19 | Atualizar com dados inválidos | RN01, RN02, RN05 | R01, R03 | Base | Não executado |
| CT20 | Excluir id existente | RF06 | — | Base | Não executado |
| CT21 | Excluir id inexistente | RF06, RN06 | — | Base | Não executado |
| CT22 | Rota inexistente, método não suportado e JSON malformado | RNF02 | — | Base | Não executado |
| CT23 | Exemplos e descrições do Swagger | RNF01 | R06 | Média | Não executado |
| CT24 | `PersonService.save` | RF04, RN03 | — | Base | Não executado |
| CT25 | `PersonService.update` | RF05 | — | Base | Não executado |
| CT26 | `findById`, `findByEmail`, `deleteById` | RF02, RF03, RF06 | — | Base | Não executado |
| CT27 | `handleDatabaseExceptionId` | RN03, RN06, RNF02 | R04 | Média | Não executado |
| CT28 | Demais handlers de exceção | RN07, RNF02 | — | Base | Não executado |
| CT29 | Regras de validação de `Person` | RN01, RN02, RN04 | R01 | Alta | Não executado |
| CT30 | Consultas do repositório contra H2 | RF03, RF05 | R02 | Base | Não executado |
| CT31 | Suíte de regressão com MySQL (TestContainers) | RF01–RF06 | R08 | Base | Não executado |
| CT32 | Análise estática no SonarQube | RNF05 | R07, R09 | Base | Não executado |
| CT33 | Revisão de código com checklist | RNF03, RNF05 | R05 | Alta | Não executado |
| CT34 | Cobertura JaCoCo | RNF04 | R08 | Média | Não executado |

## 4. Casos de teste de sistema (caixa-preta)

Executados com a aplicação rodando (`./mvnw spring-boot:run`) via Postman. Os casos de prioridade Alta também serão automatizados com Rest-Assured.

### CT01 — Listagem padrão com 12 pessoas

| Campo | Valor |
| --- | --- |
| Objetivo | Verificar a paginação padrão (10 itens, página 0, ordem crescente de `id`) |
| Rastreabilidade | RF01 |
| Nível / técnica | Sistema · Partição de equivalência |
| Pré-condições | Aplicação recém-iniciada; L01 a L12 cadastradas nessa ordem via `POST /person` |

| # | Ação | Resultado esperado |
| --- | --- | --- |
| 1 | `GET /person` | 200 |
| 2 | Conferir o corpo | `content` com 10 itens; `totalElements` = 12; `totalPages` = 2; `number` = 0; `size` = 10 |
| 3 | Conferir a ordem | `id` em ordem crescente; primeiro item = L01, último = L10 |
| 4 | `GET /person?page=1` | 200; `content` com 2 itens (L11 e L12); `last` = `true` |

### CT02 — Listagem com página, tamanho e ordenação

| Campo | Valor |
| --- | --- |
| Objetivo | Verificar que `page`, `size` e `sort` informados são respeitados |
| Rastreabilidade | RF01 |
| Nível / técnica | Sistema · Partição de equivalência |
| Pré-condições | L01 a L12 cadastradas |

| # | Ação | Resultado esperado |
| --- | --- | --- |
| 1 | `GET /person?page=1&size=5&sort=firstName,desc` | 200; `number` = 1; `size` = 5; `content` com 5 itens; `totalPages` = 3 |
| 2 | Conferir a ordem | `firstName` = Pessoa07, Pessoa06, Pessoa05, Pessoa04, Pessoa03 (segunda página da ordem decrescente) |
| 3 | `GET /person?size=5&sort=email,asc` | 200; Pessoa01 a Pessoa05 |

### CT03 — Listagem com parâmetros nos limites

| Campo | Valor |
| --- | --- |
| Objetivo | Verificar que valores-limite e inválidos de paginação não causam erro 500 |
| Rastreabilidade | RF01, RNF02 |
| Nível / técnica | Sistema · Valor-limite e suposição de erro |
| Pré-condições | L01 a L12 cadastradas |

| # | Ação | Resultado esperado |
| --- | --- | --- |
| 1 | `GET /person?size=1` | 200; 1 item; `totalPages` = 12 |
| 2 | `GET /person?size=0` | 200; tamanho ajustado para o padrão (10) |
| 3 | `GET /person?page=0` | 200; primeira página |
| 4 | `GET /person?page=-1` | 200; página ajustada para 0 |
| 5 | `GET /person?size=2000` | 200; 12 itens |
| 6 | `GET /person?size=2001` | 200; `size` limitado a 2000 |
| 7 | `GET /person?page=99` | 200; `content` vazio; `totalElements` = 12 |
| 8 | `GET /person?sort=campoInexistente` | 400 em `ValidationError` |

> **Leitura do código:** os passos 1 a 7 dependem do `PageableHandlerMethodArgumentResolver` do Spring, que corrige os valores sozinho, e devem passar. No passo 8, a `PropertyReferenceException` não tem handler e deve resultar em 500.

### CT04 — Buscar id existente

| Campo | Valor |
| --- | --- |
| Objetivo | Verificar a busca por id |
| Rastreabilidade | RF02 |
| Nível / técnica | Sistema · Partição de equivalência |
| Pré-condições | P1 cadastrada; `id` retornado anotado como `{idP1}` |

| # | Ação | Resultado esperado |
| --- | --- | --- |
| 1 | `GET /person/id/{idP1}` | 200 |
| 2 | Conferir o corpo | Os seis campos iguais aos de P1 |

### CT05 — Buscar id inexistente

| Campo | Valor |
| --- | --- |
| Objetivo | Verificar o 404 padronizado para id inexistente |
| Rastreabilidade | RF02, RN06 · R04 |
| Nível / técnica | Sistema · Partição de equivalência |
| Pré-condições | Nenhuma pessoa com id 999 |

| # | Ação | Resultado esperado |
| --- | --- | --- |
| 1 | `GET /person/id/999` | 404 |
| 2 | Conferir o corpo | `StandartError` com `status` = 404, `error` = "Resource not found", `message` = "Person not found, id: 999", `path` = "/person/id/999" e `timestamp` no formato `yyyy-MM-dd'T'HH:mm:ss'Z'` |

### CT06 — id não numérico em GET e DELETE

| Campo | Valor |
| --- | --- |
| Objetivo | Verificar o 400 para path variable de tipo errado |
| Rastreabilidade | RN07 |
| Nível / técnica | Sistema · Partição de equivalência |
| Pré-condições | Aplicação em execução |

| # | Ação | Resultado esperado |
| --- | --- | --- |
| 1 | `GET /person/id/abc` | 400; `ValidationError` com `message` = ["Type mismatch id"] |
| 2 | `DELETE /person/id/abc` | 400; mesmo corpo do passo 1 |
| 3 | `GET /person/id/1.5` | 400; mesmo corpo do passo 1 |

### CT07 — Buscar e-mail existente com `.` e `+`

| Campo | Valor |
| --- | --- |
| Objetivo | Verificar a busca por e-mail com caracteres especiais válidos na URL |
| Rastreabilidade | RF03 · R02 |
| Nível / técnica | Sistema · Partição de equivalência e suposição de erro |
| Pré-condições | P1 e P3 cadastradas |

| # | Ação | Resultado esperado |
| --- | --- | --- |
| 1 | `GET /person/email/ana.souza@teste.com` | 200; corpo = P1 (o `.com` não pode ser cortado) |
| 2 | `GET /person/email/carla.dias+qa@teste.com` | 200; corpo = P3 (o `+` não pode virar espaço) |
| 3 | `GET /person/email/carla.dias%2Bqa@teste.com` | 200; corpo = P3 |

Este caso é repetido depois do CT18 para medir o efeito de um e-mail duplicado (ver CT18, passo 4).

### CT08 — Buscar e-mail inexistente

| Campo | Valor |
| --- | --- |
| Objetivo | Verificar o 404 padronizado para e-mail inexistente |
| Rastreabilidade | RF03, RN06 |
| Nível / técnica | Sistema · Partição de equivalência |
| Pré-condições | Nenhuma pessoa com o e-mail `nao.existe@teste.com` |

| # | Ação | Resultado esperado |
| --- | --- | --- |
| 1 | `GET /person/email/nao.existe@teste.com` | 404 |
| 2 | Conferir o corpo | `StandartError` com `error` = "Resource not found" e `message` = "Person not found, email: nao.existe@teste.com" |

### CT09 — Cadastrar pessoa válida

| Campo | Valor |
| --- | --- |
| Objetivo | Verificar o cadastro com dados válidos (regra R1 da tabela de cadastro) |
| Rastreabilidade | RF04 |
| Nível / técnica | Sistema · Partição de equivalência |
| Pré-condições | Banco sem P1 |

| # | Ação | Resultado esperado |
| --- | --- | --- |
| 1 | `POST /person` com P1 | 201 |
| 2 | Conferir o corpo | Campos de P1 e `id` numérico gerado |
| 3 | `GET /person/id/{id}` com o id do passo 2 | 200; corpo igual ao do passo 2 |
| 4 | `POST /person` com P1 incluindo `"id": 500` | 201; o `id` é gerado pelo banco e ignora o 500, ou o pedido é rejeitado com 400. Em nenhum caso substitui um registro existente |

### CT10 — Cadastrar com campo obrigatório inválido

| Campo | Valor |
| --- | --- |
| Objetivo | Verificar a obrigatoriedade dos cinco campos no cadastro |
| Rastreabilidade | RN01 |
| Nível / técnica | Sistema · Partição de equivalência |
| Pré-condições | Aplicação em execução |
| Entradas | Corpo de P1 com um único campo alterado por execução |

Cada linha abaixo é executada três vezes: com o campo **ausente**, com valor **vazio** (`""`) e com valor **só de espaços** (`"   "`), num total de 15 execuções.

| # | Campo alterado | Resultado esperado |
| --- | --- | --- |
| 1 | firstName | 400; `ValidationError` com `message` contendo "The field firstname is required" |
| 2 | lastName | 400; "The field lastname is required" |
| 3 | address | 400; "The field address is required" |
| 4 | gender | 400; "The field gender is required" |
| 5 | email | 400; "The field email is required" |
| 6 | Corpo `{}` | 400; `message` com as cinco mensagens acima |
| 7 | Após os passos 1–6, `GET /person` | `totalElements` não mudou: nenhum cadastro inválido foi gravado |

### CT11 — Cadastrar com e-mail inválido

| Campo | Valor |
| --- | --- |
| Objetivo | Verificar a validação de formato do e-mail no cadastro |
| Rastreabilidade | RN02 · R01 |
| Nível / técnica | Sistema · Partição de equivalência |
| Prioridade | Alta |
| Pré-condições | Aplicação em execução; anotar `totalElements` de `GET /person` |
| Entradas | Corpo de P1 trocando apenas `email` |

| # | email | Resultado esperado |
| --- | --- | --- |
| 1 | `joao.com` | 400; `ValidationError` com "The field email must be a valid email" |
| 2 | `joao@` | 400; mesma mensagem |
| 3 | `@teste.com` | 400; mesma mensagem |
| 4 | `joao silva@teste.com` | 400; mesma mensagem |
| 5 | `GET /person` | `totalElements` igual ao anotado |

> **Leitura do código:** o controller valida só os grupos `Create`/`Update`, e o `@Email` não tem grupo, então a validação do controller **não** verifica o formato. Porém, o Hibernate valida o grupo `Default` antes do INSERT, então o POST provavelmente ainda devolve 400 com a mensagem certa, só que por uma camada diferente. Na execução, registrar no log se houve tentativa de INSERT. O mesmo erro no PUT não tem essa proteção (ver CT19, passo 6).

### CT12 — Cadastrar com e-mail repetido

| Campo | Valor |
| --- | --- |
| Objetivo | Verificar a unicidade do e-mail no cadastro (regra R2 da tabela de cadastro) |
| Rastreabilidade | RN03 · R02 |
| Nível / técnica | Sistema · Tabela de decisão |
| Pré-condições | P1 cadastrada |

| # | Ação | Resultado esperado |
| --- | --- | --- |
| 1 | `POST /person` com P2 trocando `email` para `ana.souza@teste.com` | 400 |
| 2 | Conferir o corpo | `StandartError` com `error` = "Email already registered" e `message` = "Email already registered: ana.souza@teste.com" |
| 3 | `POST /person` com P2 e `email` = `ANA.SOUZA@teste.com` | Registrar o resultado: a regra não define se a comparação ignora maiúsculas (observação para o relatório, não reprova o caso) |
| 4 | `GET /person/email/ana.souza@teste.com` | 200; continua sendo P1 |

### CT13 — Limites de nome e sobrenome

| Campo | Valor |
| --- | --- |
| Objetivo | Verificar o limite de 20 caracteres de `firstName` e `lastName` |
| Rastreabilidade | RN05 · R03 |
| Nível / técnica | Sistema · Valor-limite |
| Prioridade | Alta |
| Pré-condições | Aplicação em execução; cada passo usa um e-mail diferente para não cair no CT12 |

| # | Entrada (sobre P1) | Resultado esperado |
| --- | --- | --- |
| 1 | `firstName` = `A×20` | 201; valor gravado completo |
| 2 | `firstName` = `A×21` | 400 em `ValidationError` informando o tamanho máximo |
| 3 | `lastName` = `B×20` | 201 |
| 4 | `lastName` = `B×21` | 400 em `ValidationError` |
| 5 | `firstName` = `A×1` | 201 (menor valor válido) |

> **Leitura do código:** não há `@Size`; o limite existe só na coluna. Nos passos 2 e 4, o banco deve recusar o valor e a `DataIntegrityViolationException`, sem handler, deve virar 500 no formato padrão do Spring, e não em `ValidationError`.

### CT14 — Limites de endereço, gênero e e-mail

| Campo | Valor |
| --- | --- |
| Objetivo | Verificar os limites de 100 (`address`), 10 (`gender`) e 50 (`email`) caracteres |
| Rastreabilidade | RN05 · R03 |
| Nível / técnica | Sistema · Valor-limite |
| Prioridade | Alta |
| Pré-condições | Igual ao CT13 |

| # | Entrada (sobre P1) | Resultado esperado |
| --- | --- | --- |
| 1 | `address` = `R×100` | 201 |
| 2 | `address` = `R×101` | 400 em `ValidationError` |
| 3 | `gender` = `G×10` | 201 |
| 4 | `gender` = `G×11` | 400 em `ValidationError` |
| 5 | `email` com 50 caracteres (seção 2.2) | 201 |
| 6 | `email` com 51 caracteres (seção 2.2) | 400 em `ValidationError` |

> **Leitura do código:** mesmo comportamento esperado no CT13, ou seja, 500 nos passos 2, 4 e 6.

### CT15 — Atualizar pessoa existente

| Campo | Valor |
| --- | --- |
| Objetivo | Verificar a atualização de todos os campos (regras R4 e R5 da tabela de atualização) |
| Rastreabilidade | RF05 |
| Nível / técnica | Sistema · Tabela de decisão |
| Pré-condições | P1 cadastrada com id `{idP1}` |

| # | Ação | Resultado esperado |
| --- | --- | --- |
| 1 | `PUT /person` com `id` = `{idP1}`, `firstName` = "Ana Maria", `lastName` = "Souza Reis", `address` = "Rua Nova, 1", `gender` = "F", `email` = "ana.reis@teste.com" | 200; corpo "Profile updated successfully" |
| 2 | `GET /person/id/{idP1}` | 200; os cinco campos com os valores do passo 1 |
| 3 | `GET /person/email/ana.souza@teste.com` | 404 (e-mail antigo não existe mais) |
| 4 | `PUT /person` com `id` = `{idP1}`, mantendo `email` = "ana.reis@teste.com" e trocando só `address` = "Rua Velha, 2" | 200 (manter o próprio e-mail não pode ser tratado como duplicidade) |
| 5 | `GET /person/id/{idP1}` | `address` = "Rua Velha, 2" |

### CT16 — Atualizar id inexistente

| Campo | Valor |
| --- | --- |
| Objetivo | Verificar o 404 ao atualizar id inexistente (regra R3) |
| Rastreabilidade | RF05, RN06 |
| Nível / técnica | Sistema · Tabela de decisão |
| Pré-condições | Nenhuma pessoa com id 999 |

| # | Ação | Resultado esperado |
| --- | --- | --- |
| 1 | `PUT /person` com P1 e `id` = 999 | 404; `StandartError` com `message` = "Person not found, id: 999" |
| 2 | `GET /person` | Nenhum registro criado ou alterado |

### CT17 — Atualizar sem id

| Campo | Valor |
| --- | --- |
| Objetivo | Verificar a obrigatoriedade do `id` no PUT (regra R1) |
| Rastreabilidade | RN04 |
| Nível / técnica | Sistema · Partição de equivalência |
| Pré-condições | Aplicação em execução |

| # | Ação | Resultado esperado |
| --- | --- | --- |
| 1 | `PUT /person` com P1 sem o campo `id` | 400; `ValidationError` com "The field id is required" |
| 2 | `PUT /person` com P1 e `"id": null` | 400; mesma mensagem |
| 3 | `PUT /person` com P1 e `"id": "abc"` | 400 em `ValidationError` |

> **Leitura do código:** no passo 3 o erro ocorre na leitura do JSON (`HttpMessageNotReadableException`), que não tem handler. Deve voltar 400 no formato padrão do Spring, sem `ValidationError`.

### CT18 — Atualizar para o e-mail de outra pessoa

| Campo | Valor |
| --- | --- |
| Objetivo | Verificar que a atualização não permite duplicar e-mail (regra R6) e medir o efeito se permitir |
| Rastreabilidade | RN03 · R02 |
| Nível / técnica | Sistema · Tabela de decisão |
| Prioridade | Alta (exposição 9) |
| Pré-condições | P1 (`{idP1}`) e P2 (`{idP2}`) cadastradas |

| # | Ação | Resultado esperado |
| --- | --- | --- |
| 1 | `PUT /person` com os dados de P2, `id` = `{idP2}` e `email` = "ana.souza@teste.com" | 400; `StandartError` com `error` = "Email already registered" |
| 2 | `GET /person/id/{idP2}` | `email` continua "bruno.lima@teste.com" |
| 3 | `GET /person?size=50` | Nenhum e-mail aparece duas vezes |
| 4 | `GET /person/email/ana.souza@teste.com` | 200; corpo = P1 (repete o CT07) |
| 5 | `POST /person` com P3 e `email` = "ana.souza@teste.com" | 400 "Email already registered" (repete o CT12) |

> **Leitura do código:** o `update` não consulta o e-mail e a coluna não tem UNIQUE, então o passo 1 deve devolver 200 e gravar o duplicado. A partir daí, `findByEmail` encontra duas linhas para um retorno `Optional`, e os passos 4 e 5 devem falhar com 500 (`IncorrectResultSizeDataAccessException`). Se confirmado, o defeito é de severidade **Alta**: depois de um único PUT, o e-mail fica impossível de buscar e de cadastrar.

### CT19 — Atualizar com dados inválidos

| Campo | Valor |
| --- | --- |
| Objetivo | Verificar no PUT as mesmas regras de entrada do cadastro (regra R2) |
| Rastreabilidade | RN01, RN02, RN05 · R01, R03 |
| Nível / técnica | Sistema · Partição de equivalência e valor-limite |
| Pré-condições | P1 cadastrada com id `{idP1}` |
| Entradas | Corpo de P1 com `id` = `{idP1}` e um campo alterado por passo |

| # | Alteração | Resultado esperado |
| --- | --- | --- |
| 1 | `firstName` = `""` | 400; "The field firstname is required" |
| 2 | `lastName` ausente | 400; "The field lastname is required" |
| 3 | `address` = `"   "` | 400; "The field address is required" |
| 4 | `gender` = `""` | 400; "The field gender is required" |
| 5 | `email` ausente | 400; "The field email is required" |
| 6 | `email` = `joao.com` | 400; "The field email must be a valid email" |
| 7 | `firstName` = `A×21` | 400 em `ValidationError` |
| 8 | Após os passos 1–7, `GET /person/id/{idP1}` | Dados de P1 sem nenhuma alteração |

> **Leitura do código:** os passos 1 a 5 devem passar. No passo 6 o `@Email` não está no grupo `Update` e o UPDATE é SQL nativo, que não dispara a validação do Hibernate, então o e-mail inválido deve ser **gravado** com 200. Essa é a confirmação mais direta do R01. No passo 7 deve ocorrer 500, como no CT13.

### CT20 — Excluir id existente

| Campo | Valor |
| --- | --- |
| Objetivo | Verificar a exclusão e seu efeito |
| Rastreabilidade | RF06 |
| Nível / técnica | Sistema · Partição de equivalência |
| Pré-condições | P1 cadastrada com id `{idP1}` |

| # | Ação | Resultado esperado |
| --- | --- | --- |
| 1 | `DELETE /person/id/{idP1}` | 204; corpo vazio |
| 2 | `GET /person/id/{idP1}` | 404 |
| 3 | `DELETE /person/id/{idP1}` de novo | 404 |
| 4 | `POST /person` com P1 | 201 (o e-mail fica livre depois da exclusão) |

### CT21 — Excluir id inexistente

| Campo | Valor |
| --- | --- |
| Objetivo | Verificar o 404 ao excluir id inexistente |
| Rastreabilidade | RF06, RN06 |
| Nível / técnica | Sistema · Partição de equivalência |
| Pré-condições | Nenhuma pessoa com id 999; anotar `totalElements` |

| # | Ação | Resultado esperado |
| --- | --- | --- |
| 1 | `DELETE /person/id/999` | 404; `StandartError` com `message` = "Person not found, id: 999" |
| 2 | `GET /person` | `totalElements` igual ao anotado |

### CT22 — Rota inexistente, método não suportado e JSON malformado

| Campo | Valor |
| --- | --- |
| Objetivo | Verificar que todo erro sai em JSON padronizado |
| Rastreabilidade | RNF02 |
| Nível / técnica | Sistema · Partição de equivalência e suposição de erro |
| Pré-condições | Aplicação em execução |

| # | Ação | Resultado esperado |
| --- | --- | --- |
| 1 | `GET /pessoa` | 404; `ValidationError` com `message` = ["No handler found"] |
| 2 | `GET /person/nome/ana` | 404; mesmo corpo do passo 1 |
| 3 | `PATCH /person` | 405 em JSON com `timestamp`, `status`, `error`, `message` e `path` |
| 4 | `DELETE /person` (sem id) | 405 no mesmo formato |
| 5 | `POST /person` com corpo `{"firstName": "Ana",` (JSON incompleto) | 400 no mesmo formato |
| 6 | `POST /person` com P1 e `Content-Type: text/plain` | 415 no mesmo formato |

> **Leitura do código:** os passos 1 e 2 têm handler e devem passar. Os passos 3 a 6 não têm handler e devem cair no erro padrão do Spring Boot (`timestamp`, `status`, `error`, `path`, sem `message`). Divergência de severidade Baixa.

### CT23 — Exemplos e descrições do Swagger

| Campo | Valor |
| --- | --- |
| Objetivo | Verificar se a documentação OpenAPI corresponde ao comportamento real |
| Rastreabilidade | RNF01 · R06 |
| Nível / técnica | Aceitação · Revisão de documentação |
| Prioridade | Média |
| Pré-condições | Aplicação em execução; P1 cadastrada com id 1 |

| # | Ação | Resultado esperado |
| --- | --- | --- |
| 1 | Abrir `/swagger-ui/index.html` | Página carrega com os 6 endpoints |
| 2 | Abrir `/v3/api-docs` | JSON OpenAPI válido; licença MIT declarada em `info.license` |
| 3 | POST: "Try it out" com o exemplo sem alterações | 201 |
| 4 | PUT: "Try it out" com o exemplo sem alterações | 200 |
| 5 | Conferir a descrição do 200 de `GET /person/id/{id}` | Fala em busca por id |
| 6 | Conferir o schema do 200 de `PUT /person` | Igual ao corpo realmente devolvido |
| 7 | Conferir o PUT na lista de operações | Tem `summary` e `tags` como os outros endpoints |
| 8 | Conferir os 400 documentados de POST e PUT | Schema coerente com o corpo devolvido (`StandartError` para e-mail repetido, `ValidationError` para validação) |

> **Leitura do código:** os exemplos de POST e PUT não têm `email`, então os passos 3 e 4 devem devolver 400 "The field email is required". O 200 do GET por id diz "Successfully find by Email"; o PUT devolve texto mas declara `Person`, e não tem `@Operation`. Todos os passos de 3 a 8 tendem a reprovar, cada um com severidade Baixa.

## 5. Casos de teste unitários e de integração (caixa-branca)

Implementados em JUnit 5 dentro de `rest-spring-test/src/test/java`. Na coluna **Existente**, "sim" indica que a suíte original já tem um teste equivalente, que será reaproveitado.

### CT24 — `PersonService.save`

| Campo | Valor |
| --- | --- |
| Objetivo | Cobrir as duas saídas do `if (savedPerson.isPresent())` |
| Rastreabilidade | RF04, RN03 |
| Nível / técnica | Unitário · Cobertura de decisão |
| Ferramentas | JUnit 5 + Mockito (`@Mock PersonRepository`, `@InjectMocks PersonService`) |

| # | Cenário (mock) | Resultado esperado | Existente |
| --- | --- | --- | --- |
| 1 | `findByEmail` → `Optional.empty()` | Retorna a pessoa; `save` chamado 1 vez | Sim |
| 2 | `findByEmail` → `Optional.of(p)` | Lança `DatabaseException` com "Email already registered: {email}"; `save` nunca chamado | Sim (sem verificar `never()`) |

### CT25 — `PersonService.update`

| Campo | Valor |
| --- | --- |
| Objetivo | Cobrir as duas saídas do `if` de existência |
| Rastreabilidade | RF05 |
| Nível / técnica | Unitário · Cobertura de decisão |
| Ferramentas | JUnit 5 + Mockito |

| # | Cenário (mock) | Resultado esperado | Existente |
| --- | --- | --- | --- |
| 1 | `findById` → `Optional.of(p)` | `updatePerson` chamado 1 vez com os 6 argumentos de `p` | Sim |
| 2 | `findById` → `Optional.empty()` | Lança `DatabaseException` com "Person not found, id: {id}"; `updatePerson` nunca chamado | Sim |
| 3 | E-mail de `p` pertence a outra pessoa (`findByEmail` → pessoa com outro id) | Lança `DatabaseException` "Email already registered"; `updatePerson` nunca chamado | Não (deve falhar; documenta o R02) |

### CT26 — `findById`, `findByEmail`, `deleteById`

| Campo | Valor |
| --- | --- |
| Objetivo | Cobrir os `Optional` vazio e presente dos três métodos |
| Rastreabilidade | RF02, RF03, RF06 |
| Nível / técnica | Unitário · Cobertura de decisão |
| Ferramentas | JUnit 5 + Mockito |

| # | Método · cenário | Resultado esperado | Existente |
| --- | --- | --- | --- |
| 1 | `findById` · presente | Retorna a pessoa | Sim |
| 2 | `findById` · vazio | `DatabaseException` "Person not found, id: {id}" | Sim |
| 3 | `findByEmail` · presente | Retorna a pessoa | Não |
| 4 | `findByEmail` · vazio | `DatabaseException` "Person not found, email: {email}" | Sim |
| 5 | `deleteById` · presente | `repository.deleteById` chamado 1 vez | Sim |
| 6 | `deleteById` · vazio | `DatabaseException`; `repository.deleteById` nunca chamado | Sim |
| 7 | `findAll` · página com itens e vazia | Retorna a página do repositório | Sim |

### CT27 — `handleDatabaseExceptionId`

| Campo | Valor |
| --- | --- |
| Objetivo | Cobrir os dois ramos do `contains("Email already registered")` e a fragilidade da decisão por texto |
| Rastreabilidade | RN03, RN06, RNF02 · R04 |
| Nível / técnica | Unitário · Cobertura de decisão e suposição de erro |
| Prioridade | Média |
| Ferramentas | JUnit 5 + `MockHttpServletRequest` (handler instanciado diretamente) |

| # | Mensagem da `DatabaseException` | Resultado esperado |
| --- | --- | --- |
| 1 | "Email already registered: a@teste.com" | 400; `error` = "Email already registered"; `message` = a mensagem original; `path` = URI da requisição |
| 2 | "Person not found, id: 1" | 404; `error` = "Resource not found" |
| 3 | "email already registered: a@teste.com" (minúsculas) | 400 (é o mesmo erro de negócio) |
| 4 | `null` | Resposta de erro tratada, sem `NullPointerException` |

> **Leitura do código:** o passo 3 deve devolver 404, já que a comparação diferencia maiúsculas. O passo 4 deve lançar `NullPointerException` em `e.getMessage().contains(...)`. Os dois confirmam o R04.

### CT28 — Demais handlers de exceção

| Campo | Valor |
| --- | --- |
| Objetivo | Cobrir todos os comandos dos outros quatro handlers |
| Rastreabilidade | RN07, RNF02 |
| Nível / técnica | Unitário · Cobertura de comandos |
| Ferramentas | JUnit 5 + Mockito (exceções simuladas) ou MockMvc |

| # | Exceção | Resultado esperado |
| --- | --- | --- |
| 1 | `MethodArgumentTypeMismatchException` com `getName()` = "id" | 400; `message` = ["Type mismatch id"] |
| 2 | `NoHandlerFoundException` | 404; `message` = ["No handler found"] |
| 3 | `MethodArgumentNotValidException` com 2 erros de campo | 400; `message` com as 2 mensagens |
| 4 | `ConstraintViolationException` com 1 violação | 400; `message` com a mensagem da violação |
| 5 | Todos os anteriores | `error` = "Validation Error"; `path` = URI da requisição; `timestamp` preenchido |

### CT29 — Regras de validação de `Person`

| Campo | Valor |
| --- | --- |
| Objetivo | Verificar as anotações de validação por grupo, isoladas do Spring |
| Rastreabilidade | RN01, RN02, RN04 · R01 |
| Nível / técnica | Unitário · Cobertura de comandos |
| Prioridade | Alta |
| Ferramentas | JUnit 5 + `jakarta.validation.Validator` (`Validation.buildDefaultValidatorFactory()`) |

| # | Objeto · grupo | Resultado esperado |
| --- | --- | --- |
| 1 | P1 sem id · `Create` | 0 violações |
| 2 | P1 sem id · `Update` | 1 violação: "The field id is required" |
| 3 | P1 com id · `Update` | 0 violações |
| 4 | Cada campo texto `null`, `""` e `"   "` · `Create` e `Update` | 1 violação com a mensagem do campo (30 combinações; teste parametrizado) |
| 5 | P1 com `email` = `joao.com` · `Create` | 1 violação: "The field email must be a valid email" |
| 6 | P1 com `email` = `joao.com` · `Update` | 1 violação: mesma mensagem |
| 7 | P1 com `email` = `joao.com` · `Default` | 1 violação: mesma mensagem |
| 8 | P1 com `firstName` = `A×21` · `Create` | 1 violação de tamanho |

> **Leitura do código:** os passos 5 e 6 devem retornar 0 violações, e o passo 7 deve passar. Esse contraste prova que o `@Email` só vale no grupo `Default`. O passo 8 deve retornar 0 violações por falta de `@Size` (R03).

### CT30 — Consultas do repositório contra H2

| Campo | Valor |
| --- | --- |
| Objetivo | Verificar `findByEmail` e o UPDATE nativo `updatePerson` com banco real |
| Rastreabilidade | RF03, RF05 · R02 |
| Nível / técnica | Integração · Caixa-branca |
| Ferramentas | `@DataJpaTest` (H2) |

| # | Cenário | Resultado esperado | Existente |
| --- | --- | --- | --- |
| 1 | `findByEmail` de e-mail gravado | `Optional` com a pessoa | Sim |
| 2 | `findByEmail` de e-mail ausente | `Optional.empty()` | Sim |
| 3 | `updatePerson` em id existente | Os 5 campos alterados ao reler com `findById` após `flush`/`clear` | Sim |
| 4 | `updatePerson` em id inexistente | Nenhuma linha alterada | Sim |
| 5 | Gravar duas pessoas e usar `updatePerson` para dar à segunda o e-mail da primeira; chamar `findByEmail` | Documenta o efeito do R02: `IncorrectResultSizeDataAccessException` | Não |
| 6 | `updatePerson` com `firstName` = `A×21` | Exceção de integridade do banco (documenta o R03) | Não |

### CT31 — Suíte de regressão com MySQL (TestContainers)

| Campo | Valor |
| --- | --- |
| Objetivo | Garantir que a suíte original e a nova passam juntas contra MySQL real |
| Rastreabilidade | RF01 a RF06 · R08 |
| Nível / técnica | Integração / Regressão · Caixa-preta |
| Pré-condições | Docker ativo; imagem `mysql:8.0.28` disponível |

| # | Ação | Resultado esperado |
| --- | --- | --- |
| 1 | `docker info` | Docker responde |
| 2 | `./mvnw clean test` | `BUILD SUCCESS` |
| 3 | Ler o resumo do Surefire | Os 40 testes originais passam; nenhum `Skipped` por falta de Docker |
| 4 | `./mvnw site` e abrir `target/site/surefire-report.html` | Relatório gerado; anexar ao registro de execução |
| 5 | Repetir o passo 2 sem Docker | Registrar quantos testes falham ou são pulados (insumo do R08) |

Testes novos que documentam defeitos (CT25.3, CT27.3, CT27.4, CT29.5, CT29.6, CT29.8) ficam marcados com `@Tag("defeito")` e são excluídos da regressão com `-Dgroups='!defeito'` até que o defeito seja corrigido. Assim a suíte de regressão fica verde sem esconder os defeitos, que continuam executáveis com `-Dgroups=defeito`.

## 6. Casos de inspeção e medição

### CT32 — Análise estática no SonarQube

| Campo | Valor |
| --- | --- |
| Objetivo | Medir bugs, vulnerabilidades, code smells e duplicação |
| Rastreabilidade | RNF05 · R07, R09 |
| Nível / técnica | Inspeção · Análise estática |
| Pré-condições | Docker ativo |

| # | Ação | Resultado esperado |
| --- | --- | --- |
| 1 | `docker run -d --name sonarqube -p 9000:9000 sonarqube:lts-community` | SonarQube disponível em `http://localhost:9000` |
| 2 | Criar o projeto `rest-spring-test` e gerar um token | Token gerado |
| 3 | `./mvnw clean verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.host.url=http://localhost:9000 -Dsonar.token=<token> -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml` | Análise publicada |
| 4 | Conferir o painel | 0 bugs e 0 vulnerabilidades Blocker/Critical; Quality Gate "Passed" |
| 5 | Exportar as métricas (bugs, vulnerabilidades, security hotspots, code smells, dívida técnica, duplicação, cobertura) | Prints anexados ao relatório de inspeção |

### CT33 — Revisão de código com checklist

| Campo | Valor |
| --- | --- |
| Objetivo | Revisão técnica manual dos pontos que a análise estática não pega |
| Rastreabilidade | RNF03, RNF05 · R05 |
| Nível / técnica | Inspeção · Revisão técnica |
| Prioridade | Alta |
| Pré-condições | Código na versão `0.0.1-SNAPSHOT`; cada item revisado por dois integrantes |

Cada item é registrado como Atende / Não atende, com arquivo e linha.

| # | Item do checklist | Onde olhar | Atende se |
| --- | --- | --- | --- |
| 1 | Sem credenciais fixas | `application-prod.yml` | Usuário e senha vêm de variáveis de ambiente |
| 2 | Configuração segura em produção | `application-prod.yml` | `show-sql` desligado; `ddl-auto` diferente de `update` |
| 3 | Validação de formato aplicada nos grupos usados | `Person.email` | `@Email` com `groups = {Create.class, Update.class}` |
| 4 | Tamanho validado antes do banco | Campos de `Person` | `@Size` coerente com o `length` de cada `@Column` |
| 5 | Unicidade garantida no banco | `@Column(email)` | `unique = true` ou constraint equivalente |
| 6 | Unicidade garantida na atualização | `PersonService.update` | Verifica e-mail de outra pessoa |
| 7 | Decisão de erro por tipo, não por texto | `ControllerExceptionHandler` | Exceções específicas por situação |
| 8 | Handlers sem risco de NPE | `handleDatabaseExceptionId` | Trata mensagem nula |
| 9 | Erros de banco tratados | `ControllerExceptionHandler` | Handler para `DataIntegrityViolationException` |
| 10 | Injeção por construtor | Controller e Service | Sem `@Autowired` em atributo |
| 11 | Nomes corretos | `StandartError` | Nome de classe sem erro de grafia |
| 12 | Dependências atualizadas | `pom.xml`, dialetos | Sem `MySQL8Dialect` (descontinuado) e sem plugins antigos (`surefire-report` 2.22.0) |
| 13 | Licença identificável | Raiz do repositório | Arquivo `LICENSE` presente |

> **Leitura do código:** os itens 1 a 9 e 11 a 13 tendem a "Não atende". O caso só é aprovado se não houver item crítico (1, 5 e 6) sem tratamento.

### CT34 — Cobertura JaCoCo

| Campo | Valor |
| --- | --- |
| Objetivo | Medir cobertura de linhas e ramos da suíte completa |
| Rastreabilidade | RNF04 · R08 |
| Nível / técnica | Medição · Cobertura |
| Prioridade | Média |
| Pré-condições | CT31 executado |

| # | Ação | Resultado esperado |
| --- | --- | --- |
| 1 | `./mvnw clean test` com Docker | `target/site/jacoco/index.html` gerado |
| 2 | Ler o total do relatório | Cobertura de linhas ≥ 80% e de ramos ≥ 70% |
| 3 | Ler a cobertura por classe (`PersonService`, `ControllerExceptionHandler`, `PersonController`) | Registrar linhas e ramos de cada classe |
| 4 | Repetir sem Docker | Registrar a diferença (insumo do R08) |
| 5 | Comparar com a cobertura só da suíte original (antes dos testes novos) | Registrar o ganho obtido pela equipe |

## 7. Resumo

| Grupo | Casos | Passos especificados |
| --- | --- | --- |
| Sistema (caixa-preta) | CT01–CT22 | 93 (o CT10 se desdobra em 15 execuções) |
| Aceitação | CT23 | 8 |
| Unitário e integração (caixa-branca) | CT24–CT31 | 40 (o CT29.4 se desdobra em 30 combinações) |
| Inspeção e medição | CT32–CT34 | 23 |

A leitura do código indica reprovação provável em 12 casos: CT03, CT13, CT14, CT17, CT18, CT19, CT22, CT23, CT25, CT27, CT29 e CT33. Os mais graves são CT18 e CT19, que tocam R02 e R01. Esses casos devem ser executados primeiro, conforme a ordem de prioridade da análise de riscos.
