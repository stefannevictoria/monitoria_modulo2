# 🏗️ Modelo Físico e Migrations SQL 

## O que é o Modelo Físico?

O **Modelo Físico** é a implementação real do banco de dados em SQL. É onde você escreve os `CREATE TABLE`, define os tipos exatos de cada coluna, e aplica as **constraints** (restrições) que garantem a integridade dos dados.

---

## O que são Migrations?

**Migrations** são arquivos SQL versionados que descrevem as mudanças no banco de dados de forma **reproduzível e ordenada**. Cada migration é um passo na "história" do seu banco.

### Por que são numeradas?

Porque o banco precisa ser criado **na ordem certa**. Se você tenta criar a tabela `PEDIDOS` antes de criar `USUARIOS`, vai dar erro — pois `PEDIDOS` tem uma FK que referencia `USUARIOS`.

```
0001_create_usuarios.sql      ← sem dependências externas
0002_create_categorias.sql    ← sem dependências externas
0003_create_produtos.sql      ← depende de CATEGORIAS
0004_create_enderecos.sql     ← depende de USUARIOS
0005_create_pedidos.sql       ← depende de USUARIOS e ENDERECOS
0006_create_itens_pedido.sql  ← depende de PEDIDOS e PRODUTOS
```


---

## Constraints importantes

| Constraint | O que faz | Exemplo |
|---|---|---|
| `NOT NULL` | Impede valor nulo (campo obrigatório) | `nome VARCHAR(100) NOT NULL` |
| `UNIQUE` | Garante que o valor não se repita | `email VARCHAR(255) UNIQUE` |
| `PRIMARY KEY` | Define a chave primária | `id SERIAL PRIMARY KEY` |
| `FOREIGN KEY` | Define chave estrangeira com referência | `FOREIGN KEY (usuario_id) REFERENCES usuarios(id)` |
| `CHECK` | Valida uma condição no valor | `CHECK (estoque >= 0)` |
| `DEFAULT` | Define valor padrão | `ativo BOOLEAN DEFAULT true` |
| `ON DELETE` | O que fazer quando o pai é deletado | `ON DELETE CASCADE` ou `ON DELETE RESTRICT` |

---

## As Migrations do ShopEasy

### 0001_create_usuarios.sql

```sql
-- Migration: 0001_create_usuarios
-- Descrição: Cria a tabela de usuários do sistema
-- Dependências: nenhuma

CREATE TABLE usuarios (
    id          SERIAL PRIMARY KEY,
    nome        VARCHAR(100)        NOT NULL,
    email       VARCHAR(255)        NOT NULL UNIQUE,
    senha_hash  VARCHAR(255)        NOT NULL,
    ativo       BOOLEAN             NOT NULL DEFAULT true,
    criado_em   TIMESTAMP           NOT NULL DEFAULT NOW(),

    CONSTRAINT chk_email_formato CHECK (email LIKE '%@%')
);

CREATE INDEX idx_usuarios_email ON usuarios(email);
```

---

### 0002_create_categorias.sql

```sql
-- Migration: 0002_create_categorias
-- Descrição: Cria a tabela de categorias de produtos
-- Dependências: nenhuma

CREATE TABLE categorias (
    id        SERIAL PRIMARY KEY,
    nome      VARCHAR(100) NOT NULL UNIQUE,
    descricao TEXT
);
```

---

### 0003_create_produtos.sql

```sql
-- Migration: 0003_create_produtos
-- Descrição: Cria a tabela de produtos
-- Dependências: categorias (0002)

CREATE TABLE produtos (
    id           SERIAL PRIMARY KEY,
    categoria_id INTEGER       NOT NULL,
    nome         VARCHAR(200)  NOT NULL,
    preco        DECIMAL(10,2) NOT NULL,
    estoque      INTEGER       NOT NULL DEFAULT 0,
    descricao    TEXT,
    ativo        BOOLEAN       NOT NULL DEFAULT true,

    CONSTRAINT fk_produto_categoria
        FOREIGN KEY (categoria_id)
        REFERENCES categorias(id)
        ON DELETE RESTRICT,

    CONSTRAINT chk_preco_positivo  CHECK (preco > 0),
    CONSTRAINT chk_estoque_positivo CHECK (estoque >= 0)
);

CREATE INDEX idx_produtos_categoria ON produtos(categoria_id);
CREATE INDEX idx_produtos_nome ON produtos(nome);
```

---

### 0004_create_enderecos.sql

```sql
-- Migration: 0004_create_enderecos
-- Descrição: Cria a tabela de endereços dos usuários
-- Dependências: usuarios (0001)

CREATE TABLE enderecos (
    id         SERIAL PRIMARY KEY,
    usuario_id INTEGER      NOT NULL,
    rua        VARCHAR(200) NOT NULL,
    cidade     VARCHAR(100) NOT NULL,
    estado     CHAR(2)      NOT NULL,
    cep        VARCHAR(9)   NOT NULL,
    padrao     BOOLEAN      NOT NULL DEFAULT false,

    CONSTRAINT fk_endereco_usuario
        FOREIGN KEY (usuario_id)
        REFERENCES usuarios(id)
        ON DELETE CASCADE,

    CONSTRAINT chk_cep_formato CHECK (cep ~ '^\d{5}-?\d{3}$')
);

CREATE INDEX idx_enderecos_usuario ON enderecos(usuario_id);
```

---

### 0005_create_pedidos.sql

```sql
-- Migration: 0005_create_pedidos
-- Descrição: Cria a tabela de pedidos
-- Dependências: usuarios (0001), enderecos (0004)

CREATE TABLE pedidos (
    id           SERIAL PRIMARY KEY,
    usuario_id   INTEGER       NOT NULL,
    endereco_id  INTEGER       NOT NULL,
    data_criacao TIMESTAMP     NOT NULL DEFAULT NOW(),
    status       VARCHAR(20)   NOT NULL DEFAULT 'PENDENTE',
    total        DECIMAL(10,2) NOT NULL DEFAULT 0.00,

    CONSTRAINT fk_pedido_usuario
        FOREIGN KEY (usuario_id)
        REFERENCES usuarios(id)
        ON DELETE RESTRICT,

    CONSTRAINT fk_pedido_endereco
        FOREIGN KEY (endereco_id)
        REFERENCES enderecos(id)
        ON DELETE RESTRICT,

    CONSTRAINT chk_status_valido
        CHECK (status IN ('PENDENTE', 'PAGO', 'ENVIADO', 'ENTREGUE', 'CANCELADO')),

    CONSTRAINT chk_total_positivo CHECK (total >= 0)
);

CREATE INDEX idx_pedidos_usuario ON pedidos(usuario_id);
CREATE INDEX idx_pedidos_status  ON pedidos(status);
```

---

### 0006_create_itens_pedido.sql

```sql
-- Migration: 0006_create_itens_pedido
-- Descrição: Cria a tabela de itens de cada pedido
-- Dependências: pedidos (0005), produtos (0003)

CREATE TABLE itens_pedido (
    id             SERIAL PRIMARY KEY,
    pedido_id      INTEGER       NOT NULL,
    produto_id     INTEGER       NOT NULL,
    quantidade     INTEGER       NOT NULL,
    preco_unitario DECIMAL(10,2) NOT NULL,

    CONSTRAINT fk_item_pedido
        FOREIGN KEY (pedido_id)
        REFERENCES pedidos(id)
        ON DELETE CASCADE,

    CONSTRAINT fk_item_produto
        FOREIGN KEY (produto_id)
        REFERENCES produtos(id)
        ON DELETE RESTRICT,

    CONSTRAINT chk_quantidade_positiva  CHECK (quantidade > 0),
    CONSTRAINT chk_preco_unitario_pos   CHECK (preco_unitario > 0),

    -- Garante que não haja duplicata do mesmo produto no mesmo pedido
    CONSTRAINT uq_item_pedido_produto UNIQUE (pedido_id, produto_id)
);

CREATE INDEX idx_itens_pedido     ON itens_pedido(pedido_id);
CREATE INDEX idx_itens_produto    ON itens_pedido(produto_id);
```

---

## Modelo Relacional (resumo textual)

O modelo relacional descreve os esquemas em formato textual. Convenção: **PK em negrito**, FK em *itálico*.

```
usuarios(**id**, nome, email, senha_hash, ativo, criado_em)

categorias(**id**, nome, descricao)

produtos(**id**, *categoria_id*, nome, preco, estoque, descricao, ativo)

enderecos(**id**, *usuario_id*, rua, cidade, estado, cep, padrao)

pedidos(**id**, *usuario_id*, *endereco_id*, data_criacao, status, total)

itens_pedido(**id**, *pedido_id*, *produto_id*, quantidade, preco_unitario)
```

---

## Checklist antes de entregar

- [ ] As migrations estão numeradas com prefixo (`0001_`, `0002_`, etc.)?
- [ ] A ordem respeita as dependências (tabelas referenciadas são criadas antes)?
- [ ] Cada `FOREIGN KEY` referencia uma tabela que já existe?
- [ ] Campos obrigatórios têm `NOT NULL`?
- [ ] Campos únicos têm `UNIQUE`?
- [ ] Regras de negócio estão em constraints `CHECK`?
- [ ] Há `CREATE INDEX` para colunas usadas em filtros e JOINs?
- [ ] O SQL roda sem erros do início ao fim, em ordem?
