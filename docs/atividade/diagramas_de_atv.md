# Diagramas de Atividade — SentinelTrade

Os diagramas de atividade complementam os casos de uso e mostram a lógica interna de cada um: decisões, fluxos de exceção e passos em paralelo. Cada raia (Investidor, Sistema, Bolsa) representa um ator do diagrama de casos de uso.

## 1. Enviar Ordem de Compra/Venda

Caso de uso: **Enviar Ordem de Compra/Venda**, que inclui Autenticar com MFA, Validar Saldo e Risco e Registrar Log de Auditoria Imutável.

```mermaid
flowchart TD
    INI(("●")) --> A

    subgraph INV["Investidor"]
        A["Solicitar ordem de compra/venda"]
        Z["Receber resultado da ordem"]
    end

    subgraph SIS["Sistema"]
        B["Autenticar com MFA"]
        C{"Autenticado?"}
        R1["Negar acesso"]
        D["Validar e sanitizar entradas"]
        E{"Entrada válida?"}
        R2["Rejeitar entrada"]
        FK{{"Validar saldo e risco (em paralelo)"}}
        S1["Consultar saldo e posição"]
        S2["Verificar limite de risco e mercado"]
        JN{{"Juntar resultados"}}
        G{"Saldo, risco e mercado OK?"}
        R3["Marcar ordem como REJEITADA"]
        H["Persistir ordem como PENDENTE"]
        I["Enviar ordem à Bolsa"]
        M1["Atualizar para EXECUTADA e atualizar carteira"]
        M2["Atualizar para REJEITADA"]
        M3["Reenviar com a mesma chave; se esgotar, FALHA"]
        L["Registrar log de auditoria imutável"]
        N["Notificar investidor"]
    end

    subgraph BOL["Bolsa"]
        J["Processar ordem"]
        K{"Resultado"}
    end

    A --> B --> C
    C -- "não" --> R1 --> L
    C -- "sim" --> D --> E
    E -- "não" --> R2 --> L
    E -- "sim" --> FK
    FK --> S1 --> JN
    FK --> S2 --> JN
    JN --> G
    G -- "não" --> R3 --> L
    G -- "sim" --> H --> I --> J --> K
    K -- "executada" --> M1 --> L
    K -- "rejeitada" --> M2 --> L
    K -- "falha" --> M3 --> L
    L --> N --> Z --> FIM(("◉"))
```

**Pontos de segurança (UMLsec):** validação de entrada com allowlist, autenticação com segundo fator, ordem duplicada descartada por chave de idempotência e log append-only com hash do registro anterior.

## 2. Autenticar com MFA

Caso de uso: **Autenticar com MFA**, incluído por Enviar Ordem.

```mermaid
flowchart TD
    INI(("●")) --> A

    subgraph USR["Usuário"]
        A["Informar e-mail e senha"]
        C["Informar código do segundo fator"]
    end

    subgraph SIS["Sistema"]
        B{"Credenciais válidas?"}
        R1["Negar acesso com mensagem genérica"]
        BL{"Tentativas excessivas?"}
        R2["Bloquear conta temporariamente"]
        D{"Código válido?"}
        R3["Negar acesso"]
        OK["Liberar acesso com token de curta duração"]
        LG["Registrar no log de auditoria"]
    end

    A --> B
    B -- "não" --> BL
    BL -- "sim" --> R2 --> LG
    BL -- "não" --> R1 --> LG
    B -- "sim" --> C --> D
    D -- "não" --> R3 --> LG
    D -- "sim" --> OK --> LG
    LG --> FIM(("◉"))
```
