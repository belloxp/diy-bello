# DIYBELLO — Diagramas de Engenharia

> Os diagramas abaixo estão escritos em **Mermaid**. Para exportar em `.png`,
> cole o bloco de código em [mermaid.live](https://mermaid.live) e use *Actions → PNG*,
> ou instale a extensão *Markdown Preview Mermaid Support* no VS Code e capture a tela.

---

## 1. Diagrama de Classes (UML)

```mermaid
classDiagram
    class Usuario {
        +int id
        +string nome
        +string email
        +string senha
        +string cpf
        +string telefone
        +string cep
        +string rua
        +string numero
        +string complemento
        +string bairro
        +string cidade
        +string uf
        +enum nivel_acesso
        +boolean status
        +cadastrar()
        +autenticar()
        +atualizarPerfil()
    }

    class Categoria {
        +int id
        +string nome
        +string descricao
        +string slug
        +string imagem_url
        +boolean status
        +listarProdutos()
    }

    class Produto {
        +int id
        +string nome
        +string descricao
        +float preco
        +string slug
        +enum tipo
        +enum nivel
        +int tempo_minutos
        +int estoque_atual
        +string imagem_url
        +boolean status
        +int categoria_id
        +temEstoque(qtd) boolean
        +deduzirEstoque(qtd)
        +devolverEstoque(qtd)
    }

    class Estoque {
        +int id
        +int produto_id
        +enum tipo
        +int quantidade
        +int saldo_anterior
        +int saldo_atual
        +string motivo
        +date data
        +registrarEntrada()
        +registrarSaida()
        +estornar()
    }

    class Pedido {
        +int id
        +int usuario_id
        +enum status
        +float subtotal
        +float desconto
        +float frete
        +float total
        +string cupom
        +enum forma_pagamento
        +string observacao
        +calcularDesconto()
        +calcularFrete()
        +finalizar()
        +cancelar()
    }

    class ItemPedido {
        +int id
        +int pedido_id
        +int produto_id
        +string nome_produto
        +int quantidade
        +float preco_unitario
        +float subtotal
        +calcularSubtotal()
    }

    class Entrega {
        +int id
        +int pedido_id
        +enum status
        +string codigo_rastreio
        +string cep
        +string rua
        +string numero
        +string bairro
        +string cidade
        +string uf
        +date previsao_entrega
        +date data_entrega
        +gerarRastreio()
        +atualizarStatus()
    }

    Usuario "1" --> "0..*" Pedido : realiza
    Pedido "1" --> "1..*" ItemPedido : contem
    ItemPedido "0..*" --> "1" Produto : referencia
    Pedido "1" --> "1" Entrega : gera
    Categoria "1" --> "0..*" Produto : classifica
    Produto "1" --> "0..*" Estoque : movimenta
```

### Cardinalidades

| Relacionamento | Cardinalidade | Chave estrangeira | Ao apagar o pai |
|---|---|---|---|
| Usuario → Pedido | 1 : N | `pedidos.usuario_id` | CASCADE |
| Pedido → ItemPedido | 1 : N | `itens_pedido.pedido_id` | CASCADE |
| ItemPedido → Produto | N : 1 | `itens_pedido.produto_id` | RESTRICT |
| Pedido → Entrega | 1 : 1 | `entregas.pedido_id` (UNIQUE) | CASCADE |
| Categoria → Produto | 1 : N | `produtos.categoria_id` | RESTRICT |
| Produto → Estoque | 1 : N | `estoques.produto_id` | CASCADE |

---

## 2. Diagrama de Casos de Uso

```mermaid
flowchart LR
    Visitante(["👤 Visitante"])
    Cliente(["👤 Cliente<br/>(USER)"])
    Admin(["👤 Administrador<br/>(ADMIN)"])
    ViaCEP(["🌐 ViaCEP"])

    subgraph SISTEMA["SISTEMA DIYBELLO"]
        direction TB

        subgraph PUB["Área Pública (sem login)"]
            UC06["UC06 Criar conta"]
            UC07["UC07 Validar CPF"]
            UC08["UC08 Autocompletar endereco"]
            UC09["UC09 Fazer login"]
        end

        subgraph CLI["Área do Cliente"]
            UC01["UC01 Navegar na loja"]
            UC02["UC02 Buscar e filtrar produtos"]
            UC03["UC03 Consultar detalhe do produto"]
            UC04["UC04 Gerenciar carrinho"]
            UC05["UC05 Montar caixa personalizada"]
            UC10["UC10 Finalizar pedido"]
            UC11["UC11 Consultar historico"]
            UC12["UC12 Rastrear entrega"]
            UC13["UC13 Atualizar perfil"]
        end

        subgraph ADM["Área Administrativa"]
            UC14["UC14 Gerenciar catalogo"]
            UC15["UC15 Gerenciar categorias"]
            UC16["UC16 Movimentar estoque"]
            UC17["UC17 Gerenciar pedidos e entregas"]
            UC18["UC18 Gerenciar clientes"]
            UC19["UC19 Ver relatorios gerenciais"]
        end
    end

    Visitante --- UC06
    Visitante --- UC09

    Cliente --- UC01
    Cliente --- UC02
    Cliente --- UC03
    Cliente --- UC04
    Cliente --- UC05

    UC06 -.->|include| UC07
    UC06 -.->|include| UC08
    UC08 --- ViaCEP
    UC10 -.->|extend| UC04

    Cliente --- UC10
    Cliente --- UC11
    Cliente --- UC12
    Cliente --- UC13

    Admin --- UC14
    Admin --- UC15
    Admin --- UC16
    Admin --- UC17
    Admin --- UC18
    Admin --- UC19

    Cliente -.->|herda de| Visitante
    Admin -.->|herda de| Cliente
```

---

## 3. Modelo Entidade-Relacionamento (banco)

```mermaid
erDiagram
    USUARIOS ||--o{ PEDIDOS : "realiza"
    PEDIDOS ||--|{ ITENS_PEDIDO : "contem"
    PEDIDOS ||--|| ENTREGAS : "gera"
    PRODUTOS ||--o{ ITENS_PEDIDO : "e vendido em"
    PRODUTOS ||--o{ ESTOQUES : "movimenta"
    CATEGORIAS ||--o{ PRODUTOS : "classifica"

    USUARIOS {
        int id PK
        varchar(100) nome
        varchar(100) email UK
        varchar senha "hash bcrypt"
        varchar(11) cpf UK
        varchar(11) telefone
        varchar(8) cep
        varchar(150) rua
        varchar(10) numero
        varchar(100) complemento
        varchar(100) bairro
        varchar(100) cidade
        varchar(2) uf
        enum nivel_acesso "USER, ADMIN"
        boolean status
    }

    CATEGORIAS {
        int id PK
        varchar(100) nome UK
        text descricao
        varchar(100) slug UK
        varchar imagem_url
        boolean status
    }

    PRODUTOS {
        int id PK
        varchar(120) nome
        text descricao
        float preco
        varchar(150) slug UK
        enum tipo "CAIXA, COMBO, AVULSO"
        enum nivel "INICIANTE, INTERMEDIARIO, AVANCADO"
        int tempo_minutos
        int estoque_atual
        varchar imagem_url
        boolean status
        int categoria_id FK
    }

    ESTOQUES {
        int id PK
        int produto_id FK
        enum tipo "ENTRADA, SAIDA"
        int quantidade
        int saldo_anterior
        int saldo_atual
        varchar(150) motivo
        datetime data
    }

    PEDIDOS {
        int id PK
        int usuario_id FK
        enum status "PENDENTE, PAGO, CANCELADO"
        float subtotal
        float desconto
        float frete
        float total
        varchar(30) cupom
        enum forma_pagamento "PIX, CARTAO, BOLETO"
        varchar(255) observacao
    }

    ITENS_PEDIDO {
        int id PK
        int pedido_id FK
        int produto_id FK
        varchar(120) nome_produto
        int quantidade
        float preco_unitario
        float subtotal
    }

    ENTREGAS {
        int id PK
        int pedido_id FK "UNIQUE"
        enum status "PREPARANDO, EM_TRANSITO, ENTREGUE, CANCELADA"
        varchar(30) codigo_rastreio UK
        varchar(8) cep
        varchar(150) rua
        varchar(10) numero
        varchar(100) bairro
        varchar(100) cidade
        varchar(2) uf
        datetime previsao_entrega
        datetime data_entrega
    }
```

---

