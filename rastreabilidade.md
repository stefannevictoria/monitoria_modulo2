# 🔗 Rastreabilidade — RN → Entidade → Tabela

## O que é rastreabilidade?

Rastreabilidade é a capacidade de **seguir o caminho de um requisito** desde sua definição até sua implementação. Ela responde à pergunta: *"Como eu sei que esta regra de negócio está realmente implementada no sistema?"*

Iremos abordar três matrizes de rastreabilidade:
1. **RN → Entidade → Tabela** 
2. **RF → RN → Endpoint**
3. **RNF evoluído para contexto de dados** 

---

## Matriz 1: RN → Entidade → Tabela

Esta é a principal. Mostra como cada Regra de Negócio aparece no Diagrama de Classes e onde é implementada no banco de dados.

| ID RN | Regra de Negócio | Entidade (Diagrama de Classes) | Tabela (DER/SQL) | Implementação |
|---|---|---|---|---|
| RN01 | Um pedido deve ter ao menos 1 item | `Pedido` (multiplicidade `1..*` para `ItemPedido`) | `ITENS_PEDIDO` | Validação no `PedidoService`; multiplicidade `||—\|{` no DER |
| RN02 | Estoque não pode ficar negativo | `Produto.estoque: Integer` | `produtos.estoque` | `CHECK (estoque >= 0)` em `0003_create_produtos.sql` |
| RN03 | Usuário deve ter ao menos 1 endereço | `Usuario` (composição `1..*` com `Endereco`) | `ENDERECOS` | Validação no `UsuarioService`; cardinalidade `\|{` no DER |
| RN04 | Preço de produto > 0 | `Produto.preco: Decimal` | `produtos.preco` | `CHECK (preco > 0)` em `0003_create_produtos.sql` |
| RN05 | Pedido cancelado não pode ser reaberto | `Pedido.cancelar(): void` + `StatusPedido` | `pedidos.status` | Validação no `PedidoService`; `CHECK (status IN (...))` |
| RN06 | E-mail deve ser único | `Usuario.email: String` | `usuarios.email` | `UNIQUE` em `0001_create_usuarios.sql` |
| RN07 | Preço fixado no momento da compra | `ItemPedido.precoUnitario: Decimal` | `itens_pedido.preco_unitario` | Campo próprio em `ITENS_PEDIDO`, não FK para preço do produto |
| RN08 | Produto não se repete no mesmo pedido | `ItemPedido` | `itens_pedido` | `UNIQUE (pedido_id, produto_id)` em `0006_create_itens_pedido.sql` |
| RN09 | Somente usuários ativos fazem pedidos | `Usuario.ativo: Boolean` | `usuarios.ativo` | Validação no `PedidoService`; campo `ativo BOOLEAN DEFAULT true` |
| RN10 | Produto inativo não entra em pedidos novos | `Produto.ativo: Boolean` | `produtos.ativo` | Validação no `PedidoService` |

---

## Matriz 2: RF → RN → Endpoint

Esta matriz liga os Requisitos Funcionais às Regras de Negócio que cada um deve respeitar, e ao endpoint que os implementa.

| ID RF | Requisito Funcional | RNs Associadas | Endpoint | Método HTTP |
|---|---|---|---|---|
| RF01 | Cadastro de usuário | RN06 (e-mail único) | `/usuarios` | POST |
| RF02 | Login de usuário | RN09 (usuário ativo) | `/auth/login` | POST |
| RF03 | Listagem de produtos por categoria | RN10 (produto ativo) | `/produtos?categoria={id}` | GET |
| RF04 | Adicionar produto ao carrinho | RN02, RN10 | `/carrinho/itens` | POST |
| RF05 | Finalizar pedido | RN01, RN02, RN07, RN09 | `/pedidos` | POST |
| RF06 | Visualizar pedidos do usuário | — | `/usuarios/{id}/pedidos` | GET |
| RF07 | Cadastrar produto (admin) | RN04 | `/admin/produtos` | POST |
| RF08 | Atualizar estoque (admin) | RN02 | `/admin/produtos/{id}/estoque` | PATCH |
| RF09 | Cadastrar endereço | RN03 | `/usuarios/{id}/enderecos` | POST |
| RF10 | Envio de e-mail de confirmação | — | (interno, disparado após RF05) | — |

---

## Matriz 3: RNFs no Contexto de Dados

Os RNFs precisam ser expandidos para cobrir aspectos específicos de **armazenamento, integridade, desempenho e segurança** dos dados.

| ID RNF | Requisito | Categoria | Artefato Relacionado | Como é garantido |
|---|---|---|---|---|
| RNF01 | Respostas em < 2s para 95% das requisições | Desempenho | `CREATE INDEX` nas migrations | Índices em `email`, `categoria_id`, `usuario_id` e `status` |
| RNF02 | Senhas com hash bcrypt | Segurança | `usuarios.senha_hash VARCHAR(255)` | Coluna não armazena senha em texto; hash feito no Service |
| RNF03 | Integridade referencial via FK | Integridade | Todas as migrations com FK | `FOREIGN KEY` + `ON DELETE RESTRICT/CASCADE` em todas as tabelas dependentes |
| RNF04 | Suportar 100 usuários simultâneos | Escalabilidade | Pool de conexões + índices | Índices nas colunas de filtro; pool configurado no ORM |
| RNF05 | Dados financeiros com precisão decimal | Precisão | `DECIMAL(10,2)` em `preco`, `preco_unitario`, `total` | Tipo `DECIMAL` garante 2 casas decimais sem erro de ponto flutuante |

---

## Como criar sua própria matriz

1. **Liste todas as suas RNs** 
2. **Para cada RN, identifique:**
   - Qual classe do Diagrama de Classes implementa essa regra?
   - Qual tabela/coluna no DER corresponde?
   - Como está implementado no SQL? (constraint? validação no código?)
3. **Monte a tabela**

> 💡 **Dica:** Se uma RN não aparece em nenhuma outra parte do sistema, é sinal de que ela foi esquecida na modelagem. A matriz serve justamente para detectar esses buracos!
