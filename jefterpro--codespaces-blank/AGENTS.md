## Contrato tecnico do projeto

Estas instrucoes sao obrigatorias durante todo o desenvolvimento. Antes de
alterar o codigo, consultar este arquivo. Se uma solicitacao futura entrar em
conflito com ele, informar o conflito antes de modificar a arquitetura ou os
dados.

## Objetivo

Construir um aplicativo desktop local para um motorista de aplicativo
transformar metas financeiras em planejamento diario. O aplicativo deve
mostrar faturamento, despesas, lucro, meta do dia, valor realizado, valor
restante, necessidade dos proximos dias, folgas e ritmo em relacao aos
objetivos.

## Tecnologias e restricoes

- Usar Python, Tkinter/ttk e SQLite.
- O funcionamento deve ser offline, sem login, servidor ou dependencia de
	internet em tempo de execucao.
- A aplicacao deve ser preparada para empacotamento como `.exe` no Windows.
- Manter as dependencias de producao na biblioteca padrao sempre que possivel.
- Usar PyInstaller apenas no empacotamento; ferramentas como pytest e ruff sao
	dependencias de desenvolvimento.

## Arquitetura

- Preservar as camadas: presentation, application, domain e infrastructure.
- A interface nao deve conter regras financeiras nem acesso direto ao SQLite.
- Casos de uso devem coordenar a aplicacao; regras de negocio devem ficar no
	dominio; persistencia deve ficar em repositorios.
- Centralizar calculos em servicos de dominio reutilizaveis.
- Nao duplicar logica de calculo em telas, relatorios, consultas ou repositorios.
- Nao armazenar totais derivados como fonte principal da verdade; recalcular a
	partir dos lancamentos persistidos quando necessario.

## Navegacao e interface

- Usar uma unica janela principal.
- Exibir somente uma tela por vez dentro dessa janela.
- Telas secundarias devem ter botao `VOLTAR` e retornar ao contexto anterior.
- Nao criar janelas adicionais desnecessarias; dialogos curtos de confirmacao
	ou erro sao permitidos quando forem realmente necessarios.
- Manter a interface simples, legivel e adequada a uso frequente.
- Validar campos antes de persistir e exibir mensagens de erro acionaveis.
- Apos salvar, editar ou excluir um lancamento, atualizar os indicadores
	afetados.

## Regras financeiras

- Diferenciar claramente faturamento bruto, despesas, lucro operacional, lucro
	liquido, contribuicoes para metas e saldo disponivel.
- Faturamento e despesas devem ser registrados como lancamentos com data,
	valor, categoria ou origem e descricao opcional.
- Nao chamar um valor de lucro sem informar quais despesas entram no calculo.
- Despesas operacionais e despesas pessoais devem ser distinguiveis.
- Nao aplicar impostos, taxas, custos ou percentuais que nao tenham sido
	definidos pelo usuario ou documentados como configuracao explicita.
- Contribuicoes para metas devem ser registros explicitos. Nao transferir
	automaticamente faturamento ou lucro para uma meta sem regra confirmada.
- Valores monetarios invalidos, negativos quando nao permitidos ou sem valor
	devem ser rejeitados antes da gravacao.

## Regras de calculo

- Faturamento bruto e a soma dos lancamentos classificados como faturamento no
	periodo solicitado.
- Despesas totais sao a soma das despesas do periodo, respeitando os filtros de
	categoria definidos para o indicador.
- Lucro deve ser calculado como faturamento menos as despesas que a definicao
	do indicador incluir.
- Saldo da meta e valor alvo menos valor inicial menos contribuicoes acumuladas;
	quando a meta for atingida, exibir saldo zero e status concluido.
- Dias trabalhaveis devem excluir folgas e indisponibilidades conforme o
	planejamento do periodo.
- Necessidade diaria e saldo restante dividido pelos dias trabalhaveis
	restantes.
- Necessidade futura deve considerar somente dias trabalhaveis futuros dentro
	do periodo da meta.
- Diferenca de ritmo e valor realizado acumulado menos valor esperado
	acumulado.
- Quando nao houver dias trabalhaveis restantes ou o valor esperado for zero,
	tratar o caso explicitamente; nunca dividir por zero nem inventar resultado.
- Qualquer nova formula deve ter nome, definicao, casos-limite e testes.

## Arredondamento e valores monetarios

- Nunca usar `float` para representar dinheiro.
- Persistir dinheiro como inteiros em centavos.
- Usar conversao validada na entrada e formatacao somente na apresentacao.
- Arredondar apenas em pontos definidos da regra e sempre para centavos.
- Manter o mesmo criterio de arredondamento em telas, relatorios e testes.
- Nao ocultar diferencas de arredondamento nos totais.

## Regras de datas

- Usar datas locais explicitas no formato ISO `YYYY-MM-DD` no banco.
- Validar inicio, fim e pertencimento dos lancamentos aos periodos.
- Considerar a data do lancamento conforme a regra confirmada para o projeto;
	nao trocar silenciosamente data da corrida por data de recebimento.
- Tratar virada de mes, ano, periodos vazios e anos bissextos com testes.
- Nao depender implicitamente do fuso horario do computador para calcular dias.
- Alteracoes retroativas devem recalcular os indicadores afetados.

## Folgas e dias de trabalho

- Diferenciar dia trabalhado, folga planejada, indisponibilidade e dia sem
	informacao.
- Folga nao e automaticamente equivalente a dia sem faturamento.
- Contar folgas somente conforme o status registrado no planejamento.
- Nao compensar automaticamente uma folga ou alterar metas por causa dela sem
	uma regra confirmada.
- Alteracoes de folga devem atualizar a necessidade dos dias restantes e ser
	cobertas por testes.

## Multiplas metas

- Cada meta deve possuir identidade, periodo, valor alvo, status e contribuicoes
	proprias.
- Uma contribuicao deve estar vinculada a uma meta especifica.
- Nao atribuir o mesmo valor a duas metas sem uma distribuicao explicita.
- O dashboard deve distinguir totais gerais de valores destinados a cada meta.
- Conflitos entre metas concorrentes devem ser apresentados ao usuario; nao
	escolher prioridade silenciosamente.
- A regra de prioridade, rateio ou metas simultaneas deve ser configuravel ou
	confirmada antes de ser implementada.

## Historico e preservacao de dados

- Preservar dados historicos; nunca apagar ou sobrescrever lancamentos sem uma
	acao explicita do usuario.
- Confirmar exclusoes e registrar, quando houver suporte, o motivo da alteracao.
- Edicoes retroativas devem manter os dados validos e recalcular resultados.
- Nao fazer migracoes destrutivas nem alterar o significado de dados antigos
	sem migracao documentada.
- Funcionalidades novas devem continuar lendo dados criados por versoes
	anteriores.

## Banco de dados

- Usar SQLite local com chaves estrangeiras, restricoes e transacoes.
- Versionar o schema por migracoes; nao recriar o banco para aplicar mudancas.
- Usar repositorios para acesso aos dados e consultas parametrizadas.
- Ativar e testar integridade referencial.
- Armazenar o banco em uma pasta de dados apropriada do usuario no Windows,
	separada dos arquivos temporarios do `.exe`.
- Implementar backup e restauracao sem substituir dados sem confirmacao.
- Nao persistir segredos ou informacoes que nao sejam necessarios ao objetivo.

## Seguranca

- Nao incluir credenciais, tokens, senhas ou dados pessoais desnecessarios no
	codigo, banco ou logs.
- Usar consultas parametrizadas; nunca montar SQL com entrada do usuario.
- Validar e limitar entradas vindas da interface.
- Nao enviar dados para a internet.
- Evitar exibir valores financeiros completos em mensagens de erro ou logs.
- Tratar arquivos de backup e do banco com cuidado, sem excluir arquivos do
	usuario automaticamente.

## Testes e alteracoes

- Toda alteracao deve ser acompanhada de testes adequados ao risco.
- Testar primeiro regras de dominio e calculos, depois persistencia e
	interface quando aplicavel.
- Cobrir centavos, arredondamento, datas, periodos vazios, dias sem trabalho,
	folgas, metas atingidas, multiplas metas e alteracoes retroativas.
- Testar criacao e migracao do banco, transacoes, backup e restauracao.
- Executar a suite relevante antes de concluir qualquer etapa.
- Nao considerar uma alteracao concluida apenas porque a interface abriu.
- Nao corrigir testes enfraquecendo a regra; revisar a implementacao ou a
	especificacao e informar conflitos.

## Tratamento de erros

- Validar erros de entrada antes de acessar o banco.
- Usar excecoes de dominio e de infraestrutura distintas quando isso ajudar a
	apresentar mensagens corretas.
- Nao esconder excecoes silenciosamente.
- Nao mostrar rastreamentos tecnicos ao usuario final; registrar diagnostico
	seguro quando necessario.
- Garantir que falhas de gravacao nao deixem transacoes parcialmente aplicadas.
- Informar claramente quando um calculo nao puder ser determinado por falta de
	dados ou por ausencia de dias trabalhaveis.

## Decisoes que exigem confirmacao

Nao inventar regras para definir faturamento, lucro, impostos, taxas,
contribuicoes automaticas, prioridade entre metas, compensacao de folgas,
feriados, data de reconhecimento da receita ou tratamento de dias incompletos.
Quando uma dessas decisoes for necessaria, registrar a duvida e pedir
confirmacao antes de alterar o dominio ou o schema.

---
> Source: [JefterPro/codespaces-blank](https://github.com/JefterPro/codespaces-blank) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-17 -->
