# Casos de Uso — SentinelTrade
![Uploading image.png…]()

## Atores

| Ator | Papel |
|------|-------|
| Investidor | Envia ordens de compra e venda |
| Administrador | Consulta os logs de auditoria |
| Bolsa | Bolsa/Corretora simulada: recebe as ordens e devolve o resultado |

## Relações entre os casos de uso

| Caso de uso | Inclui |
|-------------|--------|
| 1. Enviar Ordem de Compra/Venda | 2. Autenticar com MFA, 3. Validar Saldo e Risco, 4. Registrar Log de Auditoria Imutável |
| 5. Consultar Logs de Auditoria | 4. Registrar Log de Auditoria Imutável |

---

## 1. Enviar Ordem de Compra/Venda
1. Ator Primário: Investidor
2. Ator Secundário: Bolsa
3. Pré-condições: Investidor autenticado (inclui "Autenticar com MFA"); ordem informada com ativo, quantidade e preço.

### Fluxo Principal:
1. O investidor solicita a ordem de compra ou venda.
2. O sistema autentica o investidor (inclui "Autenticar com MFA").
3. O sistema valida saldo, posição e risco (inclui "Validar Saldo e Risco").
4. O sistema envia a ordem à Bolsa.
5. A Bolsa devolve o resultado: executada ou rejeitada.
6. O sistema registra o evento no log (inclui "Registrar Log de Auditoria Imutável") e notifica o investidor.

### Fluxos Alternativos:
1. Validação de saldo e risco falha: a ordem é rejeitada e o investidor é informado.
2. Ordem duplicada: a segunda requisição é descartada.
3. Falha ou indisponibilidade da Bolsa: o sistema tenta reenviar sem duplicar a ordem; esgotadas as tentativas, a ordem fica com falha e o investidor é notificado.
4. Falha ao registrar o log: a operação é abortada (ver caso 4).

---

## 2. Autenticar com MFA
1. Ator Primário: Sistema (incluído por "Enviar Ordem de Compra/Venda")
2. Pré-condições: Usuário cadastrado, com segundo fator configurado.

### Fluxo Principal:
1. O usuário informa e-mail e senha.
2. O sistema valida as credenciais.
3. O sistema solicita o segundo fator.
4. O usuário informa o código.
5. O sistema valida o código e libera o acesso.

### Fluxos Alternativos:
1. Credenciais inválidas: o acesso é negado com mensagem genérica.
2. Código do segundo fator inválido ou expirado: o acesso é negado.
3. Tentativas excessivas: a conta é bloqueada temporariamente.

---

## 3. Validar Saldo e Risco
1. Ator Primário: Sistema (incluído por "Enviar Ordem")
2. Pré-condições: Ordem recebida com ativo, quantidade e preço.

### Fluxo Principal:
1. O sistema consulta o saldo e a posição em carteira do investidor.
2. O sistema verifica o limite de risco e a situação do mercado.
3. O sistema confirma a validação e devolve o controle ao caso de uso que o chamou.

### Fluxos Alternativos:
1. Saldo ou posição insuficiente: a validação falha e a ordem é rejeitada.
2. Limite de risco ultrapassado: a validação falha e a ordem é rejeitada.
3. Mercado fechado: a ordem é rejeitada com mensagem informativa.

---

## 4. Registrar Log de Auditoria Imutável
1. Ator Primário: Sistema (incluído por "Enviar Ordem" e "Consultar Logs")
2. Pré-condições: Evento relevante ocorrido (ordem criada, enviada, executada, rejeitada ou cancelada).

### Fluxo Principal:
1. O sistema monta o registro (usuário, ação, data/hora, ID da ordem).
2. O sistema grava o registro de forma append-only, com hash do registro anterior.
3. O sistema confirma a gravação.

### Fluxos Alternativos:
1. Falha na gravação: a operação é abortada e o erro é alertado ao administrador.

---

## 5. Consultar Logs de Auditoria
1. Ator Primário: Administrador
2. Pré-condições: Administrador autenticado via MFA.

### Fluxo Principal:
1. O administrador acessa a tela de auditoria e informa os filtros (período, usuário, ordem).
2. O sistema busca os registros e os exibe, somente leitura.
3. O sistema registra a própria consulta no log de auditoria.

### Fluxos Alternativos:
1. Nenhum registro encontrado: o sistema informa que não há resultados.
2. Acesso não autorizado: o sistema nega o acesso e registra a tentativa.
