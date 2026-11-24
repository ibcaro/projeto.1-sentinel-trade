# Evidências

## Execução dos testes automatizados

Comando executado na raiz do projeto:

```bash
python -m unittest discover -s tests -t . -v
```

Com `pytest` instalado, o mesmo resultado sai com o comando `pytest`.

Saída real da execução:

```text
test_ordem_com_dados_invalidos ... ok
test_ordem_valida ... ok
test_compra_aprovada ... ok
test_limite_de_risco_excedido ... ok
test_limite_exato_e_permitido ... ok
test_mercado_fechado ... ok
test_saldo_insuficiente ... ok
test_venda_aprovada_e_posicao_insuficiente ... ok
test_venda_de_ativo_que_nao_possui ... ok
test_hash_corrompido_nao_autentica ... ok
test_hash_e_verificacao ... ok
test_hashes_diferentes_para_mesma_senha ... ok
test_token_adulterado ... ok
test_token_de_outra_chave ... ok
test_token_expirado ... ok
test_token_valido ... ok
test_codigo_errado_ou_fora_da_janela ... ok
test_verificacao_devolve_passo ... ok
test_vetores_oficiais_rfc6238 ... ok
test_cancelamento_so_antes_do_desfecho ... ok
test_estados_finais ... ok
test_matriz_completa_de_transicoes ... ok
test_motivo_e_registrado ... ok
test_codigos_invalidos_e_ataques ... ok
test_codigos_validos ... ok
test_chave_idempotencia ... ok
test_codigo_mfa ... ok
test_email ... ok
test_preco_valido_com_duas_casas ... ok
test_precos_invalidos ... ok
test_quantidades_invalidas ... ok
test_quantidades_validas ... ok
test_acao_fora_da_lista_e_recusada ... ok
test_administrador_consulta_e_a_consulta_e_auditada ... ok
test_adulteracao_e_detectada_mesmo_sem_protecao_do_banco ... ok
test_banco_recusa_alterar_ou_apagar_registros ... ok
test_cadeia_de_hash ... ok
test_filtro_por_periodo ... ok
test_filtros ... ok
test_filtros_invalidos_sao_recusados ... ok
test_investidor_nao_consulta_logs ... ok
test_log_vazio_e_integro ... ok
test_remocao_de_registro_e_detectada ... ok
test_usuario_suspeito_e_neutralizado ... ok
test_bloqueio_apos_tentativas_excessivas ... ok
test_codigo_nao_pode_ser_reutilizado ... ok
test_log_nao_guarda_segredos ... ok
test_login_com_senha_e_segundo_fator ... ok
test_perfil_vem_do_banco ... ok
test_sucesso_zera_contador_de_falhas ... ok
test_todas_as_falhas_tem_a_mesma_mensagem ... ok
test_token_expira ... ok
test_bolsa_recusa_cancelamento_de_ordem_ja_processada ... ok
test_cancelar_ordem_enviada_libera_saldo ... ok
test_consultas_so_mostram_ordens_do_proprio_investidor ... ok
test_filtro_por_estado_e_paginacao_validados ... ok
test_nao_cancela_ordem_de_outro_investidor ... ok
test_nao_cancela_ordem_em_estado_final ... ok
test_ativo_inexistente ... ok
test_bolsa_fora_do_ar_deixa_ordem_em_falha_e_devolve_saldo ... ok
test_compra_executada_atualiza_saldo_e_carteira ... ok
test_entradas_invalidas_nao_geram_ordem ... ok
test_falha_transitoria_da_bolsa_e_recuperada_por_reenvio ... ok
test_mesma_chave_em_investidores_diferentes_e_permitida ... ok
test_ordem_duplicada_nao_executa_duas_vezes ... ok
test_rejeicao_pela_bolsa_libera_reserva ... ok
test_rejeicoes_de_negocio_ficam_registradas ... ok
test_resposta_invalida_da_bolsa_e_tratada_como_falha ... ok
test_saldo_insuficiente ... ok
test_toda_ordem_gera_trilha_de_auditoria ... ok
test_venda_executada_credita_saldo ... ok
test_banco_tem_restricoes_proprias ... ok
test_email_duplicado_e_recusado ... ok
test_estado_financeiro_e_persistido ... ok
test_sql_injection_em_busca_de_ativo ... ok
test_sql_injection_em_busca_de_usuario ... ok
test_transacao_desfaz_tudo_em_caso_de_erro ... ok

----------------------------------------------------------------------
Ran 77 tests in 2.564s

OK
```

## Imagem da execução

![Testes aprovados](testes-77-aprovados.png)

A imagem acima foi gerada a partir da saída real do comando. Para substituí-la por uma captura de tela do seu terminal, salve o print com o mesmo nome (`testes-77-aprovados.png`) nesta pasta.
