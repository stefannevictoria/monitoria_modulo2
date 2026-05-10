# 🔄 Diagrama de Sequência UML

## O que é?

O **Diagrama de Sequência** mostra a **ordem das interações** entre os componentes do sistema ao longo do tempo. Diferente do Diagrama de Classes (que mostra estrutura), o de Sequência mostra **comportamento**: quem chama quem, em qual ordem, e o que retorna.

Na arquitetura em camadas que vocês usam, o fluxo padrão é:

```
Client → Controller → Service → Repository → Banco de Dados
                                             ↙
                       ← ← ← ← ← ← ← ← ← ←
```

---

## Elementos do Diagrama de Sequência

### Lifelines (linhas de vida)
Cada participante é representado por uma caixa no topo com uma linha vertical tracejada descendo. Representam objetos ou componentes que participam da interação.

### Mensagens
Setas horizontais entre as lifelines. Existem dois tipos principais:

| Tipo | Representação | Quando usar |
|---|---|---|
| **Síncrona** | `——>` seta sólida com ponta cheia | Chamada que espera resposta antes de continuar |
| **Assíncrona** | `——>` seta sólida com ponta aberta | Chamada que não bloqueia (ex: envio de e-mail, fila) |
| **Retorno** | `- - ->` seta tracejada | Resposta de volta para quem chamou |

### Activation Boxes (barras de ativação)
Retângulos finos sobre a lifeline que indicam quando aquele componente está "ativo" (processando).

---

## Arquitetura Controller → Service → Repository

Entenda o papel de cada camada:

| Camada | Responsabilidade |
|---|---|
| **Controller** | Recebe a requisição HTTP, valida entrada básica, chama o Service |
| **Service** | Contém a **lógica de negócio**. Orquestra operações, valida regras de negócio |
| **Repository** | Faz a comunicação com o banco de dados (queries SQL / ORM) |
| **Banco de Dados** | Armazena e retorna os dados |

> 💡 O Controller **não** deve ter lógica de negócio. O Repository **não** deve ter regras de negócio. Cada camada tem seu papel!

---

## Exemplo 1: Criar um Pedido (fluxo síncrono)

```mermaid
sequenceDiagram
    participant C as Client
    participant PC as PedidoController
    participant PS as PedidoService
    participant PR as PedidoRepository
    participant ProdR as ProdutoRepository
    participant DB as Banco de Dados

    C->>PC: POST /pedidos (usuarioId, itens[])
    activate PC

    PC->>PS: criarPedido(usuarioId, itens[])
    activate PS

    PS->>ProdR: verificarEstoque(produtoId, quantidade)
    activate ProdR
    ProdR->>DB: SELECT estoque FROM produtos WHERE id = ?
    activate DB
    DB-->>ProdR: estoque disponível
    deactivate DB
    ProdR-->>PS: true (estoque ok)
    deactivate ProdR

    PS->>PR: salvar(pedido)
    activate PR
    PR->>DB: INSERT INTO pedidos ...
    activate DB
    DB-->>PR: pedido criado (id: 42)
    deactivate DB
    PR-->>PS: Pedido{id: 42}
    deactivate PR

    PS->>ProdR: atualizarEstoque(produtoId, -quantidade)
    activate ProdR
    ProdR->>DB: UPDATE produtos SET estoque = estoque - ? ...
    activate DB
    DB-->>ProdR: updated
    deactivate DB
    ProdR-->>PS: void
    deactivate ProdR

    PS-->>PC: Pedido{id: 42, status: PENDENTE}
    deactivate PS

    PC-->>C: 201 Created { pedidoId: 42 }
    deactivate PC
```

---

## Exemplo 2: Login de Usuário com envio de e-mail assíncrono

```mermaid
sequenceDiagram
    participant C as Client
    participant UC as UsuarioController
    participant US as UsuarioService
    participant UR as UsuarioRepository
    participant ES as EmailService
    participant DB as Banco de Dados

    C->>UC: POST /login (email, senha)
    activate UC

    UC->>US: autenticar(email, senha)
    activate US

    US->>UR: buscarPorEmail(email)
    activate UR
    UR->>DB: SELECT * FROM usuarios WHERE email = ?
    activate DB
    DB-->>UR: Usuario{id: 1, senha_hash: ...}
    deactivate DB
    UR-->>US: Usuario
    deactivate UR

    US->>US: verificarSenha(senha, hash)
    Note right of US: Validação interna (bcrypt)

    %% Mensagem assíncrona - não bloqueia o fluxo
    US-)ES: notificarLogin(usuario)
    Note over ES: Assíncrono: o Service não<br/>espera essa resposta

    US-->>UC: token JWT
    deactivate US

    UC-->>C: 200 OK { token: "eyJ..." }
    deactivate UC
```

> 🔑 **Diferença visual:** A seta `-)` indica mensagem **assíncrona** (ponta aberta). Ela foi enviada, mas o fluxo continua sem esperar retorno.

---

## Exemplo 3: Buscar produto por ID (fluxo com erro)

```mermaid
sequenceDiagram
    participant C as Client
    participant PC as ProdutoController
    participant PS as ProdutoService
    participant PR as ProdutoRepository
    participant DB as Banco de Dados

    C->>PC: GET /produtos/999
    activate PC

    PC->>PS: buscarPorId(999)
    activate PS

    PS->>PR: findById(999)
    activate PR
    PR->>DB: SELECT * FROM produtos WHERE id = 999
    activate DB
    DB-->>PR: (vazio / null)
    deactivate DB
    PR-->>PS: null
    deactivate PR

    PS-->>PC: lança ProdutoNaoEncontradoException
    deactivate PS

    PC-->>C: 404 Not Found { erro: "Produto não encontrado" }
    deactivate PC
```

---

## Checklist antes de entregar

- [ ] Cada participante tem sua lifeline (linha de vida)?
- [ ] As mensagens síncronas usam seta sólida com ponta cheia?
- [ ] As mensagens assíncronas usam seta com ponta aberta?
- [ ] Os **retornos** usam linha tracejada?
- [ ] As barras de ativação indicam quando cada componente está ativo?
- [ ] O fluxo segue a ordem: Controller → Service → Repository → Banco?
