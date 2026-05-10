# 📋 Requisitos — RF, RN e RNF

### Por que os requisitos importam?

Os diagramas e o banco de dados precisam ser **semanticamente fiéis ao domínio e às regras de negócio**. Isso significa que cada entidade, cada constraint SQL, cada relacionamento no diagrama precisa ter uma origem nos requisitos. Sem requisitos bem definidos, não tem como avaliar se os artefatos estão corretos.

Existem três tipos de requisitos:

---


## RF — Requisito Funcional

**O que é:** Descreve **o que o sistema deve fazer** — uma ação, comportamento ou funcionalidade que o usuário pode executar.

**Formato recomendado:**
```
RF[número]: O sistema deve permitir que [ator] [faça algo] [em qual contexto].
```

**Como reconhecer:** Se você consegue imaginar um botão, uma tela ou um endpoint para isso, provavelmente é um RF.

### Exemplo:

| ID | Requisito Funcional | Ator |
|---|---|---|
| RF01 | O sistema deve permitir que o usuário se cadastre com nome, e-mail e senha | Visitante |
| RF02 | O sistema deve permitir que o usuário faça login com e-mail e senha | Usuário |
| RF03 | O sistema deve exibir a listagem de produtos por categoria | Usuário |
| RF04 | O sistema deve permitir que o usuário adicione produtos ao carrinho | Usuário |
| RF05 | O sistema deve permitir que o usuário finalize um pedido | Usuário |
| RF06 | O sistema deve permitir que o usuário visualize seus pedidos anteriores | Usuário |
| RF07 | O sistema deve permitir que o administrador cadastre novos produtos | Administrador |
| RF08 | O sistema deve permitir que o administrador atualize o estoque de produtos | Administrador |
| RF09 | O sistema deve permitir que o usuário cadastre endereços de entrega | Usuário |
| RF10 | O sistema deve enviar confirmação por e-mail ao finalizar pedido | Sistema |

---

## RN — Regra de Negócio

**O que é:** Descreve **como o sistema deve se comportar** dentro de um contexto específico. São as restrições e políticas do domínio — o que é permitido, proibido ou obrigatório.

**Formato recomendado:**
```
RN[número]: Um(a) [entidade] deve/não pode [condição].
```

**Como reconhecer:** Se é uma restrição ou política específica do negócio (não uma funcionalidade visível ao usuário), é uma RN. RNs geralmente viram `CHECK`, `UNIQUE` ou validações no Service.

> **Diferença entre RF e RN:** O RF diz *"o usuário pode finalizar um pedido"*. A RN diz *"um pedido só pode ser finalizado se tiver ao menos 1 item e o estoque for suficiente"*. O RF é a ação; a RN é a restrição que governa essa ação.

### Exemplo:

| ID | Regra de Negócio | Entidade relacionada |
|---|---|---|
| RN01 | Um pedido deve conter ao menos 1 item | Pedido, ItemPedido |
| RN02 | O estoque de um produto não pode ficar negativo | Produto |
| RN03 | Um usuário deve ter ao menos 1 endereço cadastrado | Usuario, Endereco |
| RN04 | O preço de um produto deve ser maior que zero | Produto |
| RN05 | Um pedido com status CANCELADO não pode ser reaberto | Pedido |
| RN06 | O e-mail de cadastro deve ser único no sistema | Usuario |
| RN07 | O preço unitário em ItemPedido é fixado no momento da compra (não muda com o produto) | ItemPedido |
| RN08 | Um mesmo produto não pode aparecer duas vezes no mesmo pedido | ItemPedido |
| RN09 | Somente usuários com conta ativa podem fazer pedidos | Usuario |
| RN10 | Um produto inativo não pode ser adicionado a novos pedidos | Produto |

---

## RNF — Requisito Não Funcional

**O que é:** Descreve **qualidades do sistema** — desempenho, segurança, escalabilidade, manutenibilidade, usabilidade, etc. Não descreve o que o sistema *faz*, mas **como ele deve ser**.

**Formato recomendado:**
```
RNF[número]: O sistema deve [qualidade] em/com [condição mensurável].
```

**Como reconhecer:** Se não é uma funcionalidade nem uma restrição de negócio, mas sim uma característica de qualidade ou comportamento técnico do sistema, é um RNF.


### ⚠️ O problema dos RNFs vagos

Esse é o erro mais comum! RNFs vagos **não podem ser testados** — e um requisito que não pode ser verificado não serve para nada.

| ❌ RNF Vago — não verificável | ✅ RNF Técnico — verificável |
|---|---|
| "O sistema deve ser rápido." | "O endpoint `GET /produtos` deve retornar resposta em no máximo 500ms para até 50 requisições por segundo." |
| "O sistema deve ser seguro." | "Todos os inputs do usuário devem ser validados antes de atingir o banco. Payload inválido deve retornar HTTP 400 sem executar query SQL." |
| "O sistema deve ser fácil de usar." | "Mensagens de erro devem aparecer na mesma página do formulário, em português, com no máximo 80 caracteres." |
| "O sistema deve estar sempre disponível." | "O sistema deve ter disponibilidade de 99% medida em janelas de 30 dias." |

A diferença está em ter **um número, critério ou ferramenta** que permita verificar se o requisito foi cumprido.


### Critério SMART para RNFs

Um bom RNF deve ser:

| Letra | Critério | Pergunta a fazer |
|---|---|---|
| **S** | Específico | *"Para qual parte do sistema isso se aplica?"* |
| **M** | Mensurável | *"Como eu provo que está sendo cumprido? Qual o número?"* |
| **A** | Atingível | *"É tecnicamente possível com o stack do projeto?"* |
| **R** | Relevante | *"Impacta o usuário ou a manutenção do sistema?"* |
| **T** | Verificável | *"Existe um teste, métrica ou ferramenta que confirme?"* |


### Categorias de RNF 

#### 🚀 Desempenho (Performance)

Trata do tempo de resposta e da capacidade de processamento do sistema.

**RNF-PERF-01**
```
O endpoint GET /produtos deve retornar resposta HTTP 200 em no máximo
500ms, medido do recebimento da requisição até o envio da resposta,
em condições normais de uso (banco local, até 100 registros).

Verificação: teste com 10 usuários simultâneos durante 30 segundos.
```

**RNF-PERF-02**
```
O endpoint POST /pedidos deve processar a criação e redirecionar
(HTTP 201) em no máximo 400ms para payloads com até 10 itens.

Verificação: log de tempo no middleware do servidor.
```


#### 🔐 Segurança (Security)

Trata de proteção de dados e prevenção de ataques.

**RNF-SEC-01**
```
100% dos dados submetidos pelo usuário devem ser validados na camada
Service antes de qualquer interação com o banco de dados.

Verificação: revisão de código garantindo que nenhum Repository seja
chamado sem validação prévia. Teste com payload inválido deve retornar
HTTP 400 sem executar nenhuma query SQL.
```

**RNF-SEC-02**
```
Todas as queries SQL devem usar parâmetros posicionais ($1, $2, ...).
É proibido concatenar strings com dados do usuário para construir SQL.

Verificação: busca no código-fonte por concatenações de SQL com
variáveis de entrada. Qualquer ocorrência é uma violação.
```

**RNF-SEC-03**
```
Senhas não devem ser armazenadas em texto puro no banco de dados.
Deve ser aplicado hash com bcrypt (saltRounds mínimo de 10) antes do INSERT.

Verificação: inspecionar o valor da coluna senha_hash no banco após
cadastro — o valor deve começar com $2b$ (formato hash bcrypt).
```

#### 🏗️ Manutenibilidade (Maintainability)

Trata de como o código é organizado e o quão fácil é modificá-lo.

**RNF-MAINT-01**
```
A arquitetura deve seguir separação estrita de responsabilidades em camadas:
Controller, Service e Repository. Nenhum Controller deve executar queries
SQL diretamente.

Verificação: análise do código verificando que nenhum arquivo de Controller
contém chamadas diretas ao banco de dados.
```

**RNF-MAINT-02**
```
Cada entidade do domínio (Produto, Pedido, Usuario, ItemPedido) deve possuir
exatamente um arquivo em cada camada. Não deve haver lógica de negócio
duplicada entre Services diferentes.

Verificação: contagem de arquivos por pasta e revisão de métodos duplicados.
```


#### 📶 Disponibilidade (Availability)

Trata de quanto tempo o sistema está operacional e como ele lida com falhas.

**RNF-AVAIL-01**
```
Se a conexão com o banco de dados falhar na inicialização, o processo
deve encerrar com código de erro e logar a mensagem no terminal.

Verificação: derrubar o banco de dados e tentar iniciar o servidor —
o processo deve encerrar em no máximo 5 segundos com mensagem visível.
```

**RNF-AVAIL-02**
```
Rotas não encontradas devem retornar HTTP 404 com mensagem informativa.
Erros internos devem retornar HTTP 500 sem expor stack trace ao cliente
(apenas logar internamente).

Verificação: requisição para rota inválida deve retornar 404. Erro
forçado no Service deve retornar 500 sem mostrar o stack trace no body.
```


#### 🎨 Usabilidade (Usability)

Trata da experiência do usuário final com a interface.

**RNF-USA-01**
```
Mensagens de erro de validação exibidas na interface devem ser em
português e conter no máximo 80 caracteres. Devem aparecer na mesma
página do formulário, sem redirecionar o usuário, preservando os dados
já preenchidos.

Verificação: submeter formulário com dado inválido e verificar que
(a) a página não redireciona, (b) a mensagem aparece em destaque e
(c) os campos já preenchidos mantêm seus valores.
```


#### ⚙️ Portabilidade (Portability)

Trata de em quais ambientes o sistema funciona.

**RNF-PORT-01**
```
A aplicação deve funcionar com Node.js versão 18.x ou superior.
Todas as dependências devem estar listadas no package.json com versões
fixas, garantindo instalação reproduzível em qualquer máquina.

Verificação: clonar o repositório em máquina limpa e instalar as
dependências — a aplicação deve iniciar sem erros.
```


### Exemplo:

| ID | Categoria | Requisito (resumido) | Como Verificar | RFs cobertos |
|---|---|---|---|---|
| RNF-PERF-01 | Desempenho | GET /produtos responde em ≤ 500ms (100 registros) | Teste com usuários simultâneos | RF03, RF06 |
| RNF-PERF-02 | Desempenho | POST /pedidos responde em ≤ 400ms | Log de tempo no servidor | RF05 |
| RNF-SEC-01 | Segurança | 100% dos inputs validados antes do banco | Code review + teste com payload inválido | RF01, RF05, RF07 |
| RNF-SEC-02 | Segurança | SQL usa parâmetros posicionais ($1, $2...) | Busca no código por concatenações SQL | RF01–RF10 |
| RNF-SEC-03 | Segurança | Senhas com hash bcrypt (saltRounds ≥ 10) | Inspecionar coluna senha_hash no banco | RF01 |
| RNF-MAINT-01 | Manutenibilidade | Controllers não executam SQL direto | Análise de código por camada | RF03–RF08 |
| RNF-MAINT-02 | Manutenibilidade | Uma entidade = um arquivo por camada | Contagem de arquivos + revisão manual | RF05, RF07 |
| RNF-AVAIL-01 | Disponibilidade | Falha de conexão encerra processo com log | Derrubar banco e iniciar servidor | RF01–RF10 |
| RNF-AVAIL-02 | Disponibilidade | 404/500 sem expor stack trace | Rota inválida + erro forçado no Service | RF01, RF06 |
| RNF-USA-01 | Usabilidade | Erros em português, ≤ 80 chars, mesma página | Teste manual com dado inválido | RF01, RF05 |
| RNF-PORT-01 | Portabilidade | Funciona em Node.js ≥ 18.x, dependências fixas | Clone + install em máquina limpa | RF01–RF10 |

---

## Mapeamento RN → Constraint SQL

Os RNs devem aparecer como **constraints no SQL** ou **validações no Service**. Veja o mapeamento:

| RN | Onde é implementado |
|---|---|
| RN02 (estoque ≥ 0) | `CHECK (estoque >= 0)` em `produtos` |
| RN04 (preço > 0) | `CHECK (preco > 0)` em `produtos` |
| RN06 (e-mail único) | `UNIQUE` em `usuarios.email` |
| RN07 (preço fixado na compra) | Coluna `preco_unitario` própria em `itens_pedido` |
| RN08 (produto único por pedido) | `UNIQUE (pedido_id, produto_id)` em `itens_pedido` |
| RN01 (ao menos 1 item) | Validação no `PedidoService` |
| RN05 (cancelado não reabre) | Validação no `PedidoService` |

---

## Checklist antes de entregar

- [ ] RFs descrevem **ações** que o sistema realiza (têm ator e verbo)?
- [ ] RNs descrevem **restrições e regras** do negócio (não funcionalidades)?
- [ ] RNFs têm **critério mensurável** (número, porcentagem, ferramenta)?
- [ ] Nenhum RNF está vago (sem "o sistema deve ser rápido/seguro/fácil")?
- [ ] Cada RN tem uma entidade correspondente no Diagrama de Classes?
- [ ] Cada RN relevante está refletido em uma constraint SQL ou validação de código?
- [ ] Cada RNF tem um método de verificação definido?