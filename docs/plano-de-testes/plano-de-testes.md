# Plano de testes — ISO/IEC/IEEE 29119

> Entrega 2 · 09/11/2026 · Projeto A3 — Gestão da Qualidade de Software

O plano segue a estrutura de Plano de Teste da ISO/IEC/IEEE 29119-3, organiza o trabalho pelos processos da 29119-2 e escolhe as técnicas pela 29119-4; ele termina numa decisão Go / Conditional / No-Go baseada nos critérios de conclusão da seção 4.4.

| Parte da família | Tema | Uso neste projeto |
| --- | --- | --- |
| 29119-1 | Conceitos e vocabulário | Termos do plano: item de teste, base de teste, condição de teste, incidente, critério de conclusão |
| 29119-2 | Processos de teste | Gestão de testes (planejamento, monitoramento e controle, conclusão) e processo de teste dinâmico (projeto e implementação, ambiente, execução, relato de incidentes) |
| 29119-3 | Documentação de teste | Estrutura deste plano e dos documentos da Entrega 3 (seção 4.3) |
| 29119-4 | Técnicas de teste | Técnicas de caixa-preta e caixa-branca da seção 4.2 |
| 29119-5 | Teste orientado a palavras-chave | Não aplicada |

## 1. Contexto dos testes

### 1.1 Projeto e subprocessos de teste

Plano de teste de projeto, único, para a A3 de Gestão da Qualidade de Software. Ele cobre os subprocessos de teste unitário, de integração, de sistema, de aceitação, de regressão e o teste estático (inspeção), detalhados na seção 4.1.

### 1.2 Itens de teste

Versão `0.0.1-SNAPSHOT` do rest-spring-test: `PersonController`, `ControllerExceptionHandler`, `PersonService`, `PersonRepository`, `Person` (regras de validação) e os arquivos `application*.yml`. Base de teste: código-fonte, anotações OpenAPI, README do repositório e os requisitos derivados na [matriz de rastreabilidade](../rastreabilidade/matriz-de-rastreabilidade.md).

### 1.3 Escopo

| Dentro do escopo | Fora do escopo |
| --- | --- |
| Os 6 endpoints de `/person` e suas regras de negócio | Testes de carga e desempenho |
| Validação de entrada e códigos de status HTTP | Testes de interface gráfica (o sistema não tem front-end) |
| Tratamento de erros e formato das respostas | Segurança avançada (pentest, autenticação: o sistema não tem) |
| Persistência em H2 e MySQL | Compatibilidade com outros bancos |
| Documentação OpenAPI / Swagger | Pipeline de CI/CD (opcional na A3) |
| Qualidade interna do código (análise estática e cobertura) | |

### 1.4 Premissas e restrições

- Não existe documento de requisitos; a base de teste é o próprio código e sua documentação.
- Os defeitos encontrados são registrados e propostos como melhoria, sem corrigir o código original.
- Equipe de 3 pessoas e prazo até 16/11 para execução e relatórios.
- Os testes de integração com TestContainers exigem Docker em ao menos uma máquina.
- Pipeline de integração contínua fica fora do escopo obrigatório.

### 1.5 Partes interessadas

| Parte interessada | Interesse no teste |
| --- | --- |
| Prof. Emerson dos Santos Paduan | Avalia as entregas e a Quality Decision |
| Equipe de QA (Rafael, Gian Carlo, Theobaldo) | Planeja, executa e reporta os testes |
| Mantenedor do projeto (Douglas Fragoso) | Destinatário potencial dos incidentes e melhorias |
| Consumidores da API | Dependem do comportamento correto dos endpoints |

## 2. Comunicação dos testes

| Comunicação | Meio | Quando |
| --- | --- | --- |
| Relatório de incidente | GitHub Issues do repositório do grupo | A cada incidente encontrado |
| Relatório de status de teste | README / quadro do repositório | Ao fim de cada bloco de execução |
| Checkpoints com o professor | Apresentação em aula | 09/11 e 16/11 |
| Relatório de conclusão de teste | Relatório PDF e slides | 16/11 e apresentação (23/11 ou 30/11) |

## 3. Registro de riscos

Os riscos de produto (R01 a R06) e de projeto (R07 a R09) estão na [análise de riscos](../riscos/analise-de-riscos.md). Seguindo a abordagem baseada em risco da 29119, a prioridade de cada risco define a ordem de projeto e execução dos testes.

## 4. Estratégia de teste

### 4.1 Subprocessos de teste

| Subprocesso | O que verifica | Abordagem | Ferramentas |
| --- | --- | --- | --- |
| Teste unitário | `PersonService`, `ControllerExceptionHandler` e as regras de validação de `Person`, isolados | Caixa-branca: teste de comandos e de ramos | JUnit 5, Mockito, Jakarta Validator |
| Teste de integração | Service + Repository + banco real; consultas `findByEmail` e `updatePerson` | Caixa-branca e caixa-preta | `@DataJpaTest` (H2), TestContainers (MySQL) |
| Teste de sistema | Endpoints completos com a aplicação rodando | Caixa-preta: classes de equivalência, valor-limite, tabela de decisão, suposição de erro | Postman, Rest-Assured |
| Teste de aceitação | Requisitos atendidos do ponto de vista do consumidor da API | Execução dos casos mapeados na matriz, com evidência | Postman, Swagger UI |
| Teste de regressão | Os 40 testes originais e os novos continuam passando | Suíte completa a cada alteração | `mvnw clean test`, Surefire |
| Teste estático | Bugs, vulnerabilidades, code smells, duplicação; credenciais e tratamento de exceções | Análise estática + revisão técnica com checklist | SonarQube / SonarLint |

### 4.2 Técnicas de projeto de teste (29119-4)

| Técnica | Categoria na 29119-4 | Onde se aplica |
| --- | --- | --- |
| Particionamento em classes de equivalência | Baseada em especificação | Campos obrigatórios (válido, vazio, ausente, só espaços), formato de e-mail, id existente / inexistente / não numérico |
| Análise de valor-limite | Baseada em especificação | Tamanho das colunas no limite e no limite + 1 (20/21, 100/101, 10/11, 50/51); `page` e `size` zero e negativos |
| Teste de tabela de decisão | Baseada em especificação | Cadastro e atualização combinando "dados válidos?", "e-mail já existe?" e "id existe?" |
| Teste de comandos | Baseada em estrutura | Todas as linhas de `PersonService` e `ControllerExceptionHandler` |
| Teste de ramos / decisões | Baseada em estrutura | Os `if` de `save` e `update`, os `Optional` vazios e presentes, o `contains` do handler |
| Suposição de erro | Baseada em experiência | E-mail com `+` e ponto na URL, rota inexistente, método HTTP não suportado |

### 4.3 Entregáveis de teste (29119-3)

| Documento da 29119-3 | Conteúdo neste projeto | Entrega |
| --- | --- | --- |
| Plano de teste | Este documento | 09/11 |
| Especificação de projeto de teste | Condições de teste por requisito e matriz de rastreabilidade | 09/11 |
| Requisitos de dados e de ambiente de teste | Seções 4.5 e 4.6 | 09/11 |
| Especificação de casos de teste | CT01 a CT34 com entradas, pré-condições e resultados esperados | 16/11 |
| Especificação de procedimento de teste | Coleção Postman e classes de teste JUnit / Rest-Assured | 16/11 |
| Registro de execução de teste | Relatórios Surefire, prints e logs | 16/11 |
| Relatório de incidente | Issues no GitHub | A cada incidente |
| Relatório de status de teste | Andamento da execução e dos incidentes | 16/11 |
| Relatório de conclusão de teste | Métricas (JaCoCo, SonarQube), incidentes e Quality Decision | 16/11 |

### 4.4 Critérios de conclusão dos testes

Os critérios de conclusão também são os critérios de aceitação da release.

| Critério | Meta para Go |
| --- | --- |
| Casos de teste aprovados | ≥ 95% |
| Defeitos de severidade Crítica abertos | 0 |
| Defeitos de severidade Alta abertos | 0 (ou com contorno documentado → Conditional) |
| Cobertura de linhas (JaCoCo) | ≥ 80% |
| Cobertura de ramos (JaCoCo) | ≥ 70% |
| SonarQube: bugs e vulnerabilidades Blocker/Critical | 0 |
| SonarQube: Quality Gate | Aprovado |
| Testes de regressão (suíte original) | 100% passando |

Regra de decisão: **Go** se todos os critérios forem atendidos; **Conditional** se só houver defeitos Altos ou Médios com contorno e plano de correção; **No-Go** se houver defeito Crítico aberto ou critério de cobertura/Sonar muito abaixo da meta.

### 4.5 Requisitos de dados de teste

- Massa criada pelos próprios testes, só com nomes e e-mails fictícios; nenhum dado pessoal real.
- Conjuntos específicos: 12 pessoas para paginação; textos com 20/21, 100/101, 10/11 e 50/51 caracteres; e-mails inválidos; pares de pessoas com o mesmo e-mail.
- Ponto de partida: a coleção Postman do repositório.
- O H2 é apagado a cada reinício e o MySQL do TestContainers é descartado ao fim da execução, então cada execução começa do zero.

### 4.6 Requisitos de ambiente de teste

| Item | Configuração |
| --- | --- |
| Sistema operacional | Máquinas dos integrantes (Windows / Linux) |
| JDK / build | Java 21, Maven Wrapper |
| Banco | H2 (perfil `test`) para testes manuais; MySQL 8 via Docker para integração |
| Análise estática | SonarQube local (Docker) ou SonarLint na IDE |
| Versionamento | Repositório do grupo no GitHub |
| Gestão de incidentes | GitHub Issues do repositório do grupo |

### 4.7 Reteste e regressão

Reteste: todo caso que falhar é reexecutado depois de qualquer ajuste relacionado, para confirmar se o incidente é do sistema ou do teste. Regressão: a suíte completa (40 testes originais + novos) roda com `mvnw clean test` antes de cada medição de cobertura e antes do relatório de conclusão.

### 4.8 Critérios de início, suspensão e retomada

| Tipo | Critério |
| --- | --- |
| Início da execução | Aplicação compila e sobe localmente; os 40 testes originais executados e resultado registrado; casos de teste revisados pelo grupo |
| Suspensão | Aplicação não sobe, ou Docker indisponível para a integração (nesse caso, seguir só com H2 e registrar) |
| Retomada | Causa da suspensão corrigida e documentada no relatório de status |

### 4.9 Desvios da estratégia organizacional

Não há estratégia de teste organizacional, por ser um projeto acadêmico; o plano aplica diretamente os processos da 29119-2 e registra aqui qualquer desvio que surgir durante a execução.

### 4.10 Gestão de incidentes

Incidente é todo resultado diferente do esperado (29119-1). Cada um vira uma issue no GitHub com os campos do Relatório de Incidente da 29119-3: ID, título, item de teste, caso de teste de origem, passos para reproduzir, resultado esperado e obtido, evidência, severidade e prioridade. Fluxo: Aberto → Em análise → Confirmado como defeito (ou Rejeitado) → Fechado.

| Severidade | Definição |
| --- | --- |
| Crítica | Perda ou corrupção de dados, ou funcionalidade principal indisponível |
| Alta | Regra de negócio violada ou erro 500 em entrada comum, sem contorno simples |
| Média | Comportamento incorreto com contorno, ou status/mensagem HTTP errados |
| Baixa | Documentação, texto ou estilo de código |
