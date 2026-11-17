# Casos de Uso — SentinelTrade

## Atores

| Ator | Tipo | Papel |
|------|------|-------|
| Investidor | Primário | Acompanha cotações, mantém carteira, envia, cancela e consulta ordens |
| Administrador | Primário | Gerencia investidores, contas, ativos e limites; consulta logs de auditoria |
| Bolsa/Corretora Simulada | Secundário | Recebe ordens e devolve execução, rejeição ou cancelamento |
| Provedor de Cotações | Secundário | Fornece cotações e situação do mercado (simulado) |
| Serviço de Notificação | Secundário | Entrega avisos ao investidor |
| Sistema | Primário (casos internos) | Dispara os casos incluídos (validação, auditoria) |

## Índice

| # | Caso de uso | Ator primário | Diagrama de atividade |
|---|-------------|---------------|-----------------------|
| 1 | Autenticar com MFA | Investidor / Administrador | `02_autenticar_mfa` |
| 2 | Enviar Ordem | Investidor | `01_executar_ordem` |
| 3 | Validar Saldo e Risco | Sistema (incluído) | `01_executar_ordem` |
| 4 | Registrar Log de Auditoria Imutável | Sistema (incluído) | `04_registrar_log_auditoria` |
| 5 | Consultar Logs de Auditoria | Administrador | `05_consultar_logs` |
| 6 | Cancelar Ordem | Investidor | `03_cancelar_ordem` |
| 7 | Consultar Ordens e Histórico | Investidor | — |
| 8 | Consultar Cotações | Investidor | — |
| 9 | Acompanhar Ciclo de Vida da Ordem | Sistema (incluído) | `01_executar_ordem` |
| 10 | Notificar Investidor | Sistema (incluído) | `01_executar_ordem` |
| 11 | Gerenciar Cadastros e Limites | Administrador | — |

---

## 1. Autenticar com MFA
1. Ator Primário: Investidor ou Administrador
2. Pré-condições: Usuário cadastrado e ativo, com segundo fator configurado.

### Fluxo Principal:
1. O usuário informa e-mail e senha.
2. O sistema valida as entradas e verifica se a conta está bloqueada.
3. O sistema confere a senha (hash Argon2id ou bcrypt).
4. O sistema solicita o segundo fator.
5. O usuário informa o código.
6. O sistema valida o código e emite um token assinado de curta duração.
7. O sistema registra o evento no log de auditoria (inclui "Registrar Log de Auditoria Imutável").

### Fluxos Alternativos:
1. Credenciais inválidas: o sistema responde com mensagem genérica, incrementa o contador de falhas e registra no log.
2. Conta bloqueada por tentativas excessivas: o acesso é negado e a tentativa é registrada.
3. Código do segundo fator inválido ou expirado: o acesso é negado e a falha é registrada.

---

## 2. Enviar Ordem
1. Ator Primário: Investidor
2. Pré-condições: Investidor autenticado com MFA; ordem informada com ativo, quantidade e preço.

### Fluxo Principal:
1. O investidor solicita a ordem de compra ou venda.
2. O sistema valida a sessão, autoriza a ação pelo perfil e valida as entradas.
3. O sistema valida saldo, posição e risco (inclui "Validar Saldo e Risco").
4. O sistema persiste a ordem como PENDENTE e a envia à Bolsa/Corretora Simulada (estado ENVIADA).
5. A Bolsa/Corretora devolve o desfecho: EXECUTADA, REJEITADA ou FALHA.
6. O sistema atualiza a carteira (se executada), registra o log (inclui "Registrar Log de Auditoria Imutável") e notifica o investidor (inclui "Notificar Investidor").

### Fluxos Alternativos:
1. Validação de saldo e risco falha: a ordem é marcada como REJEITADA e o investidor é notificado.
2. Ordem duplicada (mesma chave de idempotência): a segunda requisição é descartada e a ordem original é retornada.
3. Falha transitória na corretora: o sistema reenvia com a mesma chave de idempotência até o limite de tentativas; esgotado o limite, a ordem fica em FALHA e o administrador é alertado.
4. Entrada inválida ou usuário sem permissão: a ordem é recusada sem detalhes internos e a tentativa é registrada.

---

## 3. Validar Saldo e Risco
1. Ator Primário: Sistema (incluído por "Enviar Ordem")
2. Pré-condições: Ordem recebida com ativo, quantidade e preço.

### Fluxo Principal:
1. O sistema consulta o saldo e a posição em carteira do investidor.
2. O sistema verifica o limite de risco e a situação do mercado (via Provedor de Cotações).
3. O sistema confirma a validação e devolve o controle ao caso de uso que o chamou.

### Fluxos Alternativos:
1. Saldo ou posição insuficiente: a validação falha e a ordem é rejeitada.
2. Limite de risco ultrapassado: a validação falha e a ordem é rejeitada.
3. Mercado fechado: a ordem é rejeitada com mensagem informativa.
4. Provedor de cotações indisponível ou resposta inválida: o sistema informa indisponibilidade temporária e não envia a ordem.

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
3. Filtros inválidos: o sistema rejeita a consulta com mensagem de validação.

---

## 6. Cancelar Ordem
1. Ator Primário: Investidor
2. Pré-condições: Investidor autenticado; ordem existente, de sua propriedade, em estado PENDENTE ou ENVIADA.

### Fluxo Principal:
1. O investidor solicita o cancelamento informando o ID da ordem.
2. O sistema valida a sessão, a autorização e a propriedade da ordem.
3. Se a ordem já foi enviada, o sistema solicita o cancelamento à Bolsa/Corretora.
4. O sistema atualiza o estado para CANCELADA e libera o saldo ou a posição reservados.
5. O sistema registra o log (inclui "Registrar Log de Auditoria Imutável") e notifica o investidor.

### Fluxos Alternativos:
1. Ordem em estado final (EXECUTADA, REJEITADA, CANCELADA ou FALHA): o cancelamento é recusado e a transição inválida é registrada.
2. Corretora recusa por já ter executado: o investidor é avisado e o estado passa a EXECUTADA.
3. Ordem inexistente ou de outro investidor: o sistema responde sem revelar a existência da ordem.

---

## 7. Consultar Ordens e Histórico
1. Ator Primário: Investidor
2. Pré-condições: Investidor autenticado.

### Fluxo Principal:
1. O investidor solicita a lista de ordens ou o histórico de operações, com filtros opcionais.
2. O sistema valida os filtros e retorna apenas as ordens do próprio investidor.
3. O sistema exibe estado, valores e datas de cada ordem.

### Fluxos Alternativos:
1. Nenhuma ordem encontrada: o sistema informa que não há resultados.
2. Filtros inválidos: o sistema rejeita a consulta com mensagem de validação.

---

## 8. Consultar Cotações
1. Ator Primário: Investidor
2. Pré-condições: Investidor autenticado.

### Fluxo Principal:
1. O investidor informa o ativo desejado.
2. O sistema valida o código do ativo e consulta o Provedor de Cotações.
3. O sistema valida a resposta recebida e exibe a cotação em tempo quase real.

### Fluxos Alternativos:
1. Provedor indisponível: o sistema exibe a última cotação em cache, indicando a data/hora, ou informa indisponibilidade.
2. Resposta malformada: o sistema descarta o dado e informa falha temporária.
3. Ativo inexistente: o sistema informa que o ativo não foi encontrado.

---

## 9. Acompanhar Ciclo de Vida da Ordem
1. Ator Primário: Sistema (incluído por "Enviar Ordem" e "Cancelar Ordem")
2. Pré-condições: Ordem criada.

### Fluxo Principal:
1. O sistema controla as transições: PENDENTE → ENVIADA → EXECUTADA, REJEITADA ou FALHA; e CANCELADA a partir de PENDENTE ou ENVIADA.
2. A cada transição, o sistema persiste o novo estado e registra o log.

### Fluxos Alternativos:
1. Transição inválida (por exemplo, de estado final para outro): o sistema recusa a mudança e registra a tentativa.

---

## 10. Notificar Investidor
1. Ator Primário: Sistema (incluído por "Enviar Ordem" e "Cancelar Ordem")
2. Pré-condições: Desfecho de ordem (execução, rejeição, cancelamento ou falha).

### Fluxo Principal:
1. O sistema monta a mensagem sem dados sensíveis.
2. O sistema envia a mensagem ao Serviço de Notificação.
3. O serviço confirma a entrega.

### Fluxos Alternativos:
1. Serviço de notificação indisponível: o sistema registra a falha e tenta novamente depois, sem afetar o estado da ordem.

---

## 11. Gerenciar Cadastros e Limites
1. Ator Primário: Administrador
2. Pré-condições: Administrador autenticado via MFA.

### Fluxo Principal:
1. O administrador escolhe o cadastro (investidor, conta, carteira, ativo ou limite financeiro).
2. O administrador cria, altera ou desativa o registro.
3. O sistema valida as entradas e aplica a mudança.
4. O sistema registra a ação no log de auditoria.

### Fluxos Alternativos:
1. Dados inválidos: o sistema rejeita a operação com mensagem de validação.
2. Registro com ordens em andamento: o sistema impede a exclusão e sugere desativar.
3. Acesso não autorizado: o sistema nega e registra a tentativa.
