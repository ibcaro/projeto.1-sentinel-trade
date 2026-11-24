# Matriz de Rastreabilidade

Liga cada requisito ao caso de uso, às regras de negócio, ao código e aos testes.
Testes: `tests/test_dominio.py` (D) e `tests/test_sistema.py` (S).

| Requisito | Caso de uso | Regras | Código | Testes |
|-----------|-------------|--------|--------|--------|
| RF01 Autenticar com MFA | 2 | RN14, RN17, RN19 | `servicos/autenticacao.py`, `dominio/seguranca.py` | S: `TesteAutenticacao` (login, mesma mensagem, código de uso único, token expira); D: `TesteTotp`, `TesteToken`, `TesteSenha` |
| RF02 Bloqueio por tentativas | 2 | RN13 | `dominio/entidades.py` (`Usuario`) | S: `test_bloqueio_apos_tentativas_excessivas`, `test_sucesso_zera_contador_de_falhas` |
| RF03 Enviar ordem | 1 | RN15, RN16 | `servicos/ordem.py`, `dominio/validacao.py`, `api/ordens.py` | D: `TesteValidacaoAtivo`, `TesteValidacaoQuantidadePreco`, `TesteOrdemEntidade`; S: `test_compra_executada_atualiza_saldo_e_carteira`, `test_venda_executada_credita_saldo`, `test_entradas_invalidas_nao_geram_ordem` |
| RF04 Validar saldo e risco | 3 | RN01 a RN04 | `servicos/risco_saldo.py` | D: `TesteRiscoSaldo`; S: `test_rejeicoes_de_negocio_ficam_registradas`, `test_saldo_insuficiente` |
| RF05 Prevenir duplicidade | 1 | RN05 | `database/schema.sql`, `servicos/ordem.py` | S: `test_ordem_duplicada_nao_executa_duas_vezes`, `test_mesma_chave_em_investidores_diferentes_e_permitida` |
| RF06 Ciclo de vida e Bolsa | 1 | RN06, RN07, RN09 | `dominio/estados.py`, `integracoes/bolsa.py` | D: `TesteTransicoes` (todas as combinações de estados); S: `test_rejeicao_pela_bolsa_libera_reserva` |
| RF07 Recuperar falhas da Bolsa | 1 | RN10 | `servicos/ordem.py` | S: `test_falha_transitoria_da_bolsa_e_recuperada_por_reenvio`, `test_bolsa_fora_do_ar_deixa_ordem_em_falha_e_devolve_saldo`, `test_resposta_invalida_da_bolsa_e_tratada_como_falha` |
| RF08 Log imutável | 4 | RN11, RN12 | `servicos/auditoria.py`, `database/schema.sql` | S: `TesteAuditoria` (cadeia de hash, banco recusa alterar e apagar, adulteração e remoção detectadas), `test_toda_ordem_gera_trilha_de_auditoria`, `test_transacao_desfaz_tudo_em_caso_de_erro` |
| RF09 Consultar logs | 5 | RN18 | `servicos/auditoria.py`, `api/auditoria.py` | S: `test_administrador_consulta_e_a_consulta_e_auditada`, `test_investidor_nao_consulta_logs`, `test_filtros`, `test_filtros_invalidos_sao_recusados`, `test_filtro_por_periodo` |
| RF10 Consultar ordens | (fora do diagrama) | RN08 | `servicos/ordem.py`, `api/ordens.py` | S: `test_consultas_so_mostram_ordens_do_proprio_investidor`, `test_filtro_por_estado_e_paginacao_validados` |
| RF11 Cancelar ordem | (fora do diagrama) | RN07, RN08, RN09 | `servicos/ordem.py`, `api/ordens.py` | S: `TesteCancelarEConsultar` (cancelar, estado final, ordem de outro, Bolsa recusa) |
| RF12 Autorização por perfil | 1, 5 | RN17, RN18 | `api/dependencias.py`, `servicos/autenticacao.py` | S: `test_perfil_vem_do_banco`, `test_investidor_nao_consulta_logs` (a camada HTTP ainda não foi executada) |
| RF13 Notificar investidor | 1 | — | — | Pendente |
| RF14 Gerir cadastros | — | — | `persistencia/seed.py` | Parcial: `test_estado_financeiro_e_persistido` |
| RF15 Cotações externas | — | — | — | Pendente |
| RNF01 Allowlist | 1, 2, 5 | RN15, RN16 | `dominio/validacao.py` | D: `TesteValidacao*` (inclui ataques de injeção) |
| RNF02 Consultas parametrizadas | todos | — | `persistencia/` | S: `test_sql_injection_em_busca_de_usuario`, `test_sql_injection_em_busca_de_ativo` |
| RNF03 Hash de senha | 2 | — | `dominio/seguranca.py` | D: `TesteSenha` |
| RNF04 Token assinado | 2 | RN17 | `dominio/seguranca.py` | D: `TesteToken` |
| RNF07 Logs sem segredos | 4 | — | `servicos/auditoria.py` | S: `test_log_nao_guarda_segredos`, `test_acao_fora_da_lista_e_recusada`, `test_usuario_suspeito_e_neutralizado` |
| RNF10 Banco SQL | — | — | `database/schema.sql` | S: `test_banco_tem_restricoes_proprias` |
