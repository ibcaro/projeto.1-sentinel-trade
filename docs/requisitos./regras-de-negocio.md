# Regras de Negócio

| ID | Regra | Onde é aplicada |
|----|-------|-----------------|
| RN01 | Uma compra só é aceita se o saldo disponível cobrir o valor da ordem (quantidade × preço) | `servicos/risco_saldo.py` |
| RN02 | Uma venda só é aceita se a posição disponível (total menos reservada) cobrir a quantidade | `servicos/risco_saldo.py` |
| RN03 | O valor da ordem não pode ultrapassar o limite de risco do investidor (valor igual ao limite é permitido) | `servicos/risco_saldo.py` |
| RN04 | Ordens só são aceitas com o mercado do ativo aberto | `servicos/risco_saldo.py` |
| RN05 | A chave de idempotência é única por investidor; repetir a chave não cria nova ordem | `database/schema.sql`, `servicos/ordem.py` |
| RN06 | O ciclo de vida da ordem é PENDENTE → ENVIADA → EXECUTADA, REJEITADA ou FALHA; CANCELADA a partir de PENDENTE ou ENVIADA | `dominio/estados.py` |
| RN07 | EXECUTADA, REJEITADA, FALHA e CANCELADA são estados finais: não mudam mais | `dominio/estados.py` |
| RN08 | Só o dono da ordem pode cancelá-la ou consultá-la; para os outros, a ordem "não existe" | `servicos/ordem.py` |
| RN09 | Ao enviar, o valor (compra) ou a quantidade (venda) fica reservado; é devolvido se a ordem for rejeitada, falhar ou for cancelada | `servicos/ordem.py`, `dominio/entidades.py` |
| RN10 | Se a Bolsa falhar, o sistema reenvia até 3 tentativas, com espera crescente (0,2 s, 0,4 s); esgotadas, a ordem fica em FALHA e um alerta é registrado | `servicos/ordem.py` |
| RN11 | O log de auditoria só aceita novos registros (append-only); cada um guarda o hash do anterior | `database/schema.sql`, `servicos/auditoria.py` |
| RN12 | Se o log não puder ser gravado, a operação inteira é desfeita | `servicos/ordem.py` (transação) |
| RN13 | Após 5 falhas de login seguidas, a conta fica bloqueada por 15 minutos | `dominio/entidades.py` |
| RN14 | O código do segundo fator (TOTP) vale 30 segundos (com tolerância de um passo) e só pode ser usado uma vez | `dominio/seguranca.py`, `dominio/entidades.py` |
| RN15 | Quantidade de 1 a 10.000; preço maior que zero, com no máximo duas casas decimais | `dominio/validacao.py` |
| RN16 | Código de ativo no formato de quatro letras e um ou dois dígitos (ex.: PETR4) | `dominio/validacao.py` |
| RN17 | O token de acesso vale 15 minutos e o perfil do usuário é lido do banco, nunca do token | `dominio/seguranca.py`, `servicos/autenticacao.py` |
| RN18 | Só o administrador consulta os logs, e cada consulta (ou tentativa negada) também é registrada | `servicos/auditoria.py` |
| RN19 | Toda falha de login devolve a mesma mensagem, sem indicar se o e-mail, a senha ou o código errou | `servicos/autenticacao.py` |
