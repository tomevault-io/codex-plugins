# KZ Serviços - App Cliente

## Projeto
App Flutter para clientes da KZ Serviços. Foco em solicitação de serviços de transporte/viagens e "outros serviços".

## Stack
- **Framework:** Flutter (Dart)
- **State Management:** A definir (Bloc/Cubit ou Riverpod)
- **Navegação:** GoRouter
- **Mapas:** Google Maps Flutter
- **Notificações:** Firebase Cloud Messaging (FCM)
- **Chat:** Mensagens em tempo real (sem banco de dados externo inicialmente)
- **Arquitetura:** Clean Architecture com separação de camadas

## Idioma
- Código em inglês (variáveis, funções, classes, comentários)
- UI/UX em português brasileiro (textos, labels, mensagens)
- Comunicação com o desenvolvedor em português brasileiro

## Convenções de Código
- Seguir o [Effective Dart](https://dart.dev/effective-dart)
- Nomes de arquivos em snake_case
- Nomes de classes em PascalCase
- Nomes de variáveis e funções em camelCase
- Widgets em arquivos separados, um widget por arquivo
- Máximo 300 linhas por arquivo — dividir se necessário
- Usar `const` sempre que possível
- Preferir composição a herança

## Estrutura do Projeto
```
lib/
  core/              # Constantes, temas, utils, widgets compartilhados
    constants/
    theme/
    utils/
    widgets/
  features/          # Cada feature é independente
    auth/
    home/
    trip/            # Serviço de viagem/corrida
      data/
        models/
        repositories/
      domain/
        entities/
        usecases/
      presentation/
        pages/
        widgets/
        bloc/
    other_services/  # Outros serviços
    chat/            # Chat com motorista
    notifications/
    payment/
    profile/
  l10n/              # Internacionalização
  routes/            # Configuração de rotas
```

## Ciclo de Vida da Viagem
```
criação → análise → aguardando_motoristas → aguardando_cliente_aceitar →
confirmação_central → pagamento → agendado → iniciada → finalizada
```

Cada estado deve ter sua própria representação visual e lógica de transição.

## Métodos de Pagamento
- À vista (PIX ou transferência)
- Parcelado (6% de taxa)
- Faturado (PJ - requer dados da empresa)

## Regras Importantes
- **NÃO conectar com banco de dados inicialmente** — usar dados mockados/locais
- Validação de endereços obrigatória
- Notificações push para mudanças de status
- Mapa em tempo real mostrando posição do motorista
- Preset de mensagens no chat ("Estou chegando", etc.)
- Botão de atendimento WhatsApp (número a definir)

## Formulário de Corrida
Campos obrigatórios:
1. Saudação
2. Data/Hora
3. Endereço de partida (rua, bairro, número, cidade, CEP)
4. Endereço de chegada (rua, bairro, número, cidade, CEP)
5. Número de passageiros
6. Tem criança? Quantas?
7. Mala? Quantidade e tamanho
8. Ida e volta?
9. Observações

## Workflow de Desenvolvimento
Usar as skills do Superpowers para todo desenvolvimento:
1. **brainstorming** — antes de qualquer feature nova
2. **writing-plans** — criar plano detalhado de implementação
3. **test-driven-development** — TDD obrigatório
4. **executing-plans** ou **subagent-driven-development** — execução do plano
5. **systematic-debugging** — para qualquer bug
6. **verification-before-completion** — antes de marcar como pronto

## Testes
- Testes unitários para toda lógica de negócio
- Testes de widget para componentes visuais
- Usar `mocktail` para mocks
- Mínimo 80% de cobertura nas features críticas (trip, payment, chat)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BlackElephant-from-Brazil)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/BlackElephant-from-Brazil)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
