# Diagrama de Classes de Análise — SentinelTrade

Este diagrama mostra **o quê** o sistema precisa conhecer: os conceitos do domínio, seus atributos básicos e os relacionamentos de negócio. Ele não considera tecnologia, linguagem, banco de dados nem detalhes de implementação.

```mermaid
classDiagram
    class Usuario {
        nome
        email
        senha
        segundoFator
    }
    class Investidor {
        cpf
    }
    class Administrador {
    }
    class Conta {
        saldo
        situacao
    }
    class LimiteRisco {
        valorMaximoPorOrdem
        exposicaoMaxima
    }
    class Carteira {
    }
    class Posicao {
        quantidade
        precoMedio
    }
    class Ativo {
        codigo
        nome
        precoAtual
        mercadoAberto
    }
    class Ordem {
        tipo
        quantidade
        preco
        estado
        chaveIdempotencia
        dataHora
    }
    class LogAuditoria {
        usuario
        acao
        dataHora
        idOrdem
        hash
        hashAnterior
    }
    class Bolsa {
    }

    Usuario <|-- Investidor
    Usuario <|-- Administrador
    Investidor "1" --> "1..*" Conta : possui
    Investidor "1" --> "1" LimiteRisco : tem
    Investidor "1" --> "1" Carteira : mantém
    Carteira "1" --> "0..*" Posicao : contém
    Posicao "*" --> "1" Ativo : refere-se a
    Investidor "1" --> "0..*" Ordem : envia
    Ordem "*" --> "1" Ativo : negocia
    Ordem "*" --> "1" Bolsa : é enviada a
    Usuario "1" --> "0..*" LogAuditoria : gera
    Ordem "1" --> "0..*" LogAuditoria : é registrada em
    Administrador "1" --> "0..*" LogAuditoria : consulta
```

## De onde vem cada classe

| Classe | Origem no projeto |
|--------|-------------------|
| Usuario, Investidor, Administrador | Atores Investidor e Administrador; caso de uso Autenticar com MFA |
| Conta, LimiteRisco | Caso de uso Validar Saldo e Risco (saldo e limite de risco) |
| Carteira, Posicao, Ativo | Validar Saldo e Risco (posição em carteira); texto do projeto ("carteira de ativos") |
| Ordem | Caso de uso Enviar Ordem de Compra/Venda; estados PENDENTE, ENVIADA, EXECUTADA, REJEITADA, FALHA |
| LogAuditoria | Casos Registrar Log de Auditoria Imutável e Consultar Logs de Auditoria |
| Bolsa | Ator Bolsa (corretora simulada) |

## Estados da ordem

`PENDENTE → ENVIADA → EXECUTADA | REJEITADA | FALHA`. Uma ordem reprovada na validação de saldo e risco nasce REJEITADA.
