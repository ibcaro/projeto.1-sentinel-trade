# Diagrama de Classes de Projeto — SentinelTrade

Este diagrama mostra **como** o sistema será construído em Python: separação em camadas (API, serviços de negócio, persistência e integrações), encapsulamento, herança, interfaces (classes abstratas) e tratamento de exceções.

## Camadas

| Camada | Pasta em `src/` | Responsabilidade |
|--------|-----------------|------------------|
| Interface (API) | `api/` | Recebe requisições (FastAPI), valida entradas (Pydantic) e chama os serviços |
| Regras de negócio | `servicos/` | Autenticação com MFA, validação de saldo e risco, envio de ordem, auditoria |
| Domínio | `dominio/` | Entidades e regras de transição de estado |
| Persistência | `persistencia/` | Acesso ao banco SQL com consultas parametrizadas |
| Integrações | `integracoes/` | Bolsa simulada (externa) |

## 1. Domínio (herança, encapsulamento e estados)

```mermaid
classDiagram
    class Usuario {
        <<abstract>>
        -int id
        -str email
        -str senha_hash
        -str segredo_mfa
        +verificar_senha(senha) bool
        +verificar_codigo_mfa(codigo) bool
    }
    class Investidor {
        -Conta conta
        -Carteira carteira
        -LimiteRisco limite
    }
    class Administrador {
    }
    class Conta {
        -Decimal saldo
        +debitar(valor)
        +reservar(valor)
    }
    class LimiteRisco {
        -Decimal valor_maximo_por_ordem
        +permite(valor) bool
    }
    class Carteira {
        -List posicoes
        +quantidade_de(ativo) int
    }
    class Ativo {
        -str codigo
        -Decimal preco_atual
        -bool mercado_aberto
    }
    class EstadoOrdem {
        <<enumeration>>
        PENDENTE
        ENVIADA
        EXECUTADA
        REJEITADA
        FALHA
    }
    class Ordem {
        -int id
        -str chave_idempotencia
        -EstadoOrdem estado
        -int quantidade
        -Decimal preco
        +transicionar(novo_estado)
    }
    class LogAuditoria {
        -str usuario
        -str acao
        -datetime data_hora
        -int id_ordem
        -str hash
        -str hash_anterior
    }

    Usuario <|-- Investidor
    Usuario <|-- Administrador
    Investidor --> Conta
    Investidor --> Carteira
    Investidor --> LimiteRisco
    Ordem --> EstadoOrdem
    Ordem --> Ativo
    Investidor "1" --> "0..*" Ordem
```

## 2. Serviços, interfaces e persistência

```mermaid
classDiagram
    class ErroDominio {
        <<exception>>
    }
    class SaldoInsuficienteError
    class RiscoExcedidoError
    class MercadoFechadoError
    class OrdemDuplicadaError
    class TransicaoInvalidaError
    ErroDominio <|-- SaldoInsuficienteError
    ErroDominio <|-- RiscoExcedidoError
    ErroDominio <|-- MercadoFechadoError
    ErroDominio <|-- OrdemDuplicadaError
    ErroDominio <|-- TransicaoInvalidaError

    class OrdemRequest {
        <<Pydantic>>
        +str ativo
        +int quantidade
        +str chave_idempotencia
    }
    class OrdemRouter {
        +criar_ordem(OrdemRequest)
    }
    class LogRouter {
        +consultar_logs(filtros)
    }

    class ServicoAutenticacao {
        +autenticar(email, senha, codigo)
    }
    class ServicoOrdem {
        +enviar_ordem(investidor, OrdemRequest)
    }
    class ServicoRiscoSaldo {
        +validar(investidor, ordem)
    }
    class ServicoAuditoria {
        +registrar(evento)
        +consultar(filtros)
    }

    class RepositorioOrdem {
        <<interface>>
        +salvar(ordem)
        +buscar_por_chave(chave)
    }
    class RepositorioLog {
        <<interface>>
        +anexar(registro)
        +ultimo_hash() str
        +buscar(filtros)
    }
    class RepositorioOrdemSQL
    class RepositorioLogSQL

    class BolsaGateway {
        <<interface>>
        +enviar(ordem) EstadoOrdem
    }
    class BolsaSimulada

    OrdemRouter --> ServicoOrdem
    OrdemRouter --> ServicoAutenticacao
    LogRouter --> ServicoAuditoria
    LogRouter --> ServicoAutenticacao
    ServicoOrdem --> ServicoRiscoSaldo
    ServicoOrdem --> ServicoAuditoria
    ServicoOrdem --> RepositorioOrdem
    ServicoOrdem --> BolsaGateway
    ServicoAuditoria --> RepositorioLog
    RepositorioOrdem <|.. RepositorioOrdemSQL
    RepositorioLog <|.. RepositorioLogSQL
    BolsaGateway <|.. BolsaSimulada
    ServicoRiscoSaldo ..> SaldoInsuficienteError
    ServicoRiscoSaldo ..> RiscoExcedidoError
    ServicoRiscoSaldo ..> MercadoFechadoError
    ServicoOrdem ..> OrdemDuplicadaError
```

## Ligação com os casos de uso

| Caso de uso | Onde é implementado |
|-------------|---------------------|
| Enviar Ordem de Compra/Venda | `OrdemRouter` → `ServicoOrdem` |
| Autenticar com MFA | `ServicoAutenticacao` |
| Validar Saldo e Risco | `ServicoRiscoSaldo` |
| Registrar Log de Auditoria Imutável | `ServicoAuditoria` → `RepositorioLog` (append-only, com hash do registro anterior) |
| Consultar Logs de Auditoria | `LogRouter` → `ServicoAuditoria` |

## Requisitos do enunciado atendidos

| Requisito | Como aparece no projeto |
|-----------|-------------------------|
| Encapsulamento | Atributos privados (`-`) nas entidades |
| Herança | `Usuario` → `Investidor` e `Administrador`; `ErroDominio` → erros específicos |
| Interfaces ou classes abstratas | `RepositorioOrdem`, `RepositorioLog`, `BolsaGateway`, `Usuario` |
| Separação entre interface, regras, persistência e integrações | As camadas da tabela do início |
| Tratamento seguro de exceções | Hierarquia `ErroDominio` |
| Transição de estados de ordem | `Ordem.transicionar` com `EstadoOrdem` |
| Prevenção de duplicidade | `chave_idempotencia` e `buscar_por_chave` |
| Logs de auditoria imutáveis | `RepositorioLog` só com `anexar` (sem alterar nem apagar) |
