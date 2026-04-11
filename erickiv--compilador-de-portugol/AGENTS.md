# Compilador Portugol - Instruções para IA (Copilot)

## Visão Geral da Arquitetura

Este é um compilador educacional de Portugol para Python, agora com **6 fases clássicas**:

```
Fonte (.por) → Lexer (ER/AFD) → Parser → Analisador Semântico → Código Intermediário (3-endereços) → Otimizador → Gerador de Código Python
```

**Princípio:** Pipeline unidirecional, cada fase consome a anterior, sem modificar etapas anteriores.

## Responsabilidades dos Módulos

### 1. `lexer.py` - Análise Léxica
- Usa Expressões Regulares (ER) documentadas no início do arquivo.
- Implementa AFDs explícitos (ver `automaton.py`) para identificadores, inteiros e reais.
- Ignora comentários `//` e `/* ... */`.
- Gera tokens com rastreamento de linha/coluna.

### 2. `automaton.py` - Autômatos Finitos Determinísticos
- Implementa AFDs para tokens principais.
- Usado pelo lexer para validação e documentação.

### 3. `parser.py` - Análise Sintática
- Parser descendente recursivo.
- Gera AST (`Programa`, `Comando`, `Expressao` etc).
- Exige declaração de variáveis antes de `inicio`.

### 4. `semantic.py` - Análise Semântica
- Valida tipos, escopo e uso de variáveis.
- Permite conversões implícitas.
- Emite avisos para variáveis não inicializadas.

### 5. `intermediate.py` - Geração de Código Intermediário
- Gera código de 3 endereços (three-address code) a partir da AST.
- Serve de entrada para o otimizador.

### 6. `optimizer.py` - Otimizações Clássicas
- Implementa:
  - Propagação de constantes
  - Dobra de constantes
  - Simplificações algébricas
  - Propagação de cópias
  - Eliminação de código morto
- Opera sobre o código intermediário.

### 7. `codegen.py` - Geração de Python
- Converte AST (ou código intermediário otimizado) em Python executável.
- Mapeia tipos: `inteiro→int`, `real→float`, `caracter→str`, `logico→bool`.
- Garante indentação correta.

### 8. `main.py` - Orquestração
- Controla o pipeline e flags de CLI.
- Permite executar cada fase separadamente (`--debug`, `--save`, `--intermediate`, `--optimize`).

### 9. `compilar.py` - CLI
- Ponto de entrada. Não bypassar `CompiladorPortugol` em `main.py`.

## Fluxos de Trabalho

### Compilar e Executar
```bash
# Compilar e executar normalmente
python compilar.py programa.por

# Modo debug (mostra todas as fases)
python compilar.py programa.por --debug

# Gerar código intermediário
python compilar.py programa.por --intermediate

# Otimizar código intermediário
python compilar.py programa.por --optimize

# Salvar Python gerado (não executa)
python compilar.py programa.por --save
```

## Adicionando Novos Recursos à Linguagem
1. Adicione o token em `ast_nodes.py` (enum `TipoToken`).
2. Atualize `palavras_chave` em `lexer.py` (se for palavra reservada).
3. Crie a classe AST em `ast_nodes.py`.
4. Implemente parsing em `parser.py`.
5. Adicione validação semântica em `semantic.py`.
6. Implemente geração de código intermediário em `intermediate.py`.
7. Adicione otimização em `optimizer.py` (se aplicável).
8. Implemente geração de Python em `codegen.py`.

## Tratamento de Erros
Use exceções customizadas de `exceptions.py`:
- `ErroLexico` - Léxico
- `ErroSintatico` - Sintaxe
- `ErroSemantico` - Semântica
- `ErroGeracaoCodigo` - Geração de código
Todas incluem linha/coluna para localização precisa.

## Padrões da Linguagem Portugol
- Atribuição: `<-` (não `=`)
- Operadores lógicos: `e`, `ou`
- Booleanos: `verdadeiro`, `falso`
- Entrada: `leia(var)`
- Saída: `escreva(expr, ...)`
- Estruturas: `se`, `entao`, `senao`, `fimse`, `enquanto`, `faca`, `fimenquanto`

## Sistema de Tipos
Tipagem fraca: só validação semântica, sem checagem em tempo de execução.

## Testes e Exemplos
- `exemplos/demo_completa.por` - Teste não interativo (todas as features)
- `exemplos/calculadora_imc.por` - Exemplo interativo
- `exemplos/bubble_sort.por` - Algoritmo com laços
- `exemplos/teste_otimizacoes.por` - Exercita otimizador

Padrão: Sempre testar com `demo_completa.por` após mudanças.

## Armadilhas Comuns
1. Lookahead: use `self.token_atual` e `_avancar()` no parser.
2. Indentação: use `_adicionar_linha()` no codegen.
3. Escopo: só escopo global.
4. Comentários: podem aparecer em qualquer lugar, lexer deve consumir antes de tokenizar.
5. EOF: lexer deve emitir `TipoToken.EOF` explicitamente.

## Dependências e Ambiente
- Python puro (stdlib)
- Python 3.11+
- `.por` em UTF-8
- CLI para Windows (cmd/powershell)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ErickIV)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/ErickIV)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
