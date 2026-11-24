# Requisitos do SentinelTrade

Status: **Implementado** (código e testes), **Parcial** ou **Pendente**.

## Requisitos funcionais

| ID | Requisito | Origem | Status |
|----|-----------|--------|--------|
| RF01 | Autenticar o usuário com senha e segundo fator (MFA) | Caso de uso 2 | Implementado |
| RF02 | Bloquear a conta temporariamente após tentativas de login excessivas | Caso de uso 2 | Implementado |
| RF03 | Enviar ordem de compra ou venda | Caso de uso 1 | Implementado |
| RF04 | Validar saldo, posição em carteira, limite de risco e situação do mercado antes de enviar | Caso de uso 3 | Implementado |
| RF05 | Prevenir a duplicidade de ordens | Enunciado | Implementado |
| RF06 | Enviar a ordem à Bolsa simulada e acompanhar o ciclo de vida até execução, rejeição, cancelamento ou falha | Caso de uso 1 | Implementado |
| RF07 | Recuperar falhas da Bolsa por reenvio seguro | Enunciado | Implementado |
| RF08 | Registrar log de auditoria imutável dos eventos relevantes | Caso de uso 4 | Implementado |
| RF09 | Permitir ao administrador consultar os logs de auditoria, registrando a própria consulta | Caso de uso 5 | Implementado |
| RF10 | Consultar as próprias ordens e o histórico de operações | Descrição do projeto | Implementado |
| RF11 | Cancelar ordem ainda não finalizada | Enunciado | Implementado |
| RF12 | Autorizar cada ação pelo perfil (investidor ou administrador) no servidor | Enunciado | Implementado |
| RF13 | Notificar o investidor sobre execução, rejeição, cancelamento ou falha | Descrição do projeto | Pendente (o evento é gravado no log, mas não há envio de notificação) |
| RF14 | Gerir investidores, contas, carteiras, ativos e limites financeiros | Descrição do projeto | Parcial (cadastro por carga de dados `python main.py seed`; sem telas ou rotas de administração) |
| RF15 | Obter cotações em tempo quase real de um provedor externo | Descrição do projeto | Pendente (o preço do ativo é um valor fixo no banco) |

Os casos de uso 1 a 5 são os do diagrama entregue. RF10 e RF11 vêm do enunciado e da descrição do projeto e não aparecem como casos de uso no diagrama.

## Requisitos não funcionais

| ID | Requisito | Status |
|----|-----------|--------|
| RNF01 | Validar no servidor toda entrada por lista de valores permitidos (allowlist) | Implementado |
| RNF02 | Usar consultas parametrizadas; nunca concatenar entrada em SQL | Implementado |
| RNF03 | Guardar senhas apenas como hash seguro (Argon2id ou scrypt) | Implementado |
| RNF04 | Emitir tokens assinados de curta duração | Implementado |
| RNF05 | Manter segredos fora do código e do repositório (`.env`) | Implementado |
| RNF06 | Não revelar detalhes internos em mensagens de erro | Implementado (núcleo testado; camada de API ainda não executada) |
| RNF07 | Não gravar senhas, tokens ou chaves nos logs | Implementado |
| RNF08 | Limitar o tamanho das requisições e o número de tentativas | Parcial (tamanho de corpo e bloqueio de conta; sem limite por IP) |
| RNF09 | Orientação a objetos com encapsulamento, herança e interfaces, separada em camadas | Implementado |
| RNF10 | Persistência em banco relacional SQL | Implementado |
| RNF11 | Testes automatizados das regras críticas, em especial validação e transição de estados | Implementado |

## Segurança e DevSecOps: perguntas reflexivas respondidas para o SentinelTrade

| # | Pergunta | Resposta para o projeto |
|---|----------|-------------------------|
| 1 | Quais dados entram e qual regra valida cada um? | Ativo, quantidade e chave de idempotência entram em `OrdemRequest`; e-mail, senha e código MFA entram em `ServicoAutenticacao`; filtros de período, usuário e ordem entram em Consultar Logs. Cada campo tem formato, tamanho e faixa definidos. |
| 2 | Uma entrada manipulada pode causar injeção, execução indevida, vazamento ou indisponibilidade? | O risco é injeção de SQL e abuso por excesso de requisições. Mitigação prevista: consultas parametrizadas, allowlist e limite de requisições. |
| 3 | Quem pode executar a ação e como a autorização é validada no servidor? | Investidor envia ordens; administrador consulta logs. A verificação de perfil é feita em cada requisição, no servidor. |
| 4 | O usuário tem só as permissões necessárias? | Implementado: dois perfis (investidor e administrador), sem permissões compartilhadas. |
| 5 | Quais dados são sensíveis e onde ficam? | Senha (guardada só como hash), segredo do MFA, saldo e posições. Trafegam por TLS e não vão para os logs. |
| 6 | Há credenciais ou chaves expostas em código, repositório ou logs? | Segredos ficam em variáveis de ambiente (`.env`, listado no `.gitignore`). O repositório traz só o `.env.example`, sem valores. |
| 7 | O tratamento de erro revela informações a um atacante? | Não deve: mensagens genéricas para o usuário e detalhes só no log interno. |
| 8 | As dependências têm vulnerabilidades conhecidas? | Será verificado com `pip-audit` a cada alteração. |
| 9 | Há revisão e testes para cenários inválidos, abusivos e não autorizados? | Há testes de validação, de transição de estados, de autenticação, de auditoria e de acesso não autorizado (pasta `tests/`). |
| 10 | O pipeline bloqueia vulnerabilidades críticas, segredos e dependências inseguras? | `bandit`, `pip-audit` e `pytest` rodam localmente (veja "Comandos da esteira"). Ainda não há integração contínua automática. |
| 11 | Como o sistema reage a uma API externa indisponível ou malformada? | Timeout, reenvio com a mesma chave de idempotência, ordem em FALHA após o limite de tentativas e alerta ao administrador. |
| 12 | Há logs de auditoria, monitoramento, alerta, rollback e resposta a incidentes? | Log de auditoria imutável com hash encadeado e alerta ao administrador quando a gravação falha. Monitoramento e plano de rollback ainda não definidos. |
