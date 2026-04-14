# ◈ AI Coding Agent Instructions — P2P UDP Chat

## Visão Geral do Sistema

Sistema de chat P2P descentralizado em Python puro usando UDP para comunicação direta entre nós. **Nenhuma dependência externa** — apenas bibliotecas padrão (`socket`, `threading`, `tkinter`, `json`). Interface gráfica Tkinter com tema cyberpunk escuro.

**Arquitetura de camadas:**
- `Mensagem` (dataclass) → serialização/desserialização JSON
- `No` → socket UDP + thread de escuta + histórico thread-safe com `threading.Lock`
- `ChatApp` (Tkinter) → GUI principal com sidebar de vizinhos + canvas scrollável de mensagens

## Estrutura Crítica

### Sistema de Mensagens UDP

**IMPORTANTE:** UDP é stateless e fire-and-forget. Não há handshake ou confirmação de entrega.

```python
# Envio (método No.enviar)
sock.sendto(msg.serializar(), vizinho.endereco)  # Fire-and-forget

# Recepção (thread No._loop_escuta)
dados, _ = sock.recvfrom(65535)  # Bloqueia até receber datagrama
msg = Mensagem.desserializar(dados)
```

**Estrutura de mensagem** (todos os campos obrigatórios):
- `timestamp`, `remetente_nome/ip/porta`, `dest_nome/ip/porta`, `conteudo`
- `encaminhado` (bool) + `encaminhado_por` (str|None) — rastreamento de forwarding
- `tipo_pacote` → `"msg"` | `"ack_recebido"` | `"ack_lido"` para status de leitura
- `msg_id` (UUID) → rastreamento de confirmações de entrega

### Thread Safety

**SEMPRE use `self._lock` (threading.Lock) ao acessar:**
- `self._historico` — histórico de mensagens por vizinho
- `self._brutas` — mensagens raw para encaminhamento
- `self._status_msgs` — estados de confirmação por msg_id

```python
# Padrão correto
with self._lock:
    self._historico[vizinho.nome].append(...)
```

**Thread de escuta → GUI callback:**
```python
# No._processar chama:
if self._callback_nova_msg:
    self._callback_nova_msg(chave)

# ChatApp recebe via:
self.no._callback_nova_msg = self._on_nova_msg_thread_safe

# Enfileira na thread da GUI (NÃO toca widgets na thread de escuta!):
def _on_nova_msg_thread_safe(self, chave):
    self.after(0, lambda: self._processar_nova_msg(chave))
```

## Padrões de Codificação

### 1. Sistema de Cores (Paleta C)

Todas as cores vêm do dict `C` no topo de `chat_gui.py`. **Nunca use cores hardcoded**:

```python
# Correto
frame = tk.Frame(parent, bg=C["bg_panel"])
label.configure(fg=C["accent"], bg=C["bg_deep"])

# Errado
frame = tk.Frame(parent, bg="#0d1117")  # ❌ hardcoded
```

Cores por tipo de mensagem:
- `"eu"` → `C["bg_bubble_me"]` + `C["accent3"]` (verde)
- `"deles"` → `C["bg_bubble_other"]` + `C["accent"]` (ciano)
- `"fwd"` → `C["bg_bubble_fwd"]` + `C["accent4"]` (âmbar)
- `"fwd_sent"` → `C["bg_bubble_me"]` + `C["accent2"]` (violeta)

### 2. Widgets Tkinter

**Configuração padrão de Entry/Text:**
```python
entry = tk.Entry(
    parent,
    font=FONT_MONO,
    fg=C["text_pri"],
    bg=C["bg_deep"],
    insertbackground=C["accent"],  # Cursor colorido
    relief="flat",
    bd=6,  # Padding interno
)
```

**Botões estilizados com hover:**
```python
btn = tk.Button(
    parent, text="TEXTO",
    font=("Consolas", 9, "bold"),
    fg=fg_color, bg=bg_color,
    activebackground=fg_color,  # Inverte ao clicar
    activeforeground=bg_color,
    relief="flat", bd=0,
    cursor="hand2",
    command=callback
)
btn.bind("<Enter>", lambda e: btn.configure(bg=lighten(bg_color)))
btn.bind("<Leave>", lambda e: btn.configure(bg=bg_color))
```

### 3. Canvas Scrollável (Mensagens)

O `msg_frame` é embutido em um `Canvas` para scroll suave:

```python
canvas.create_window((0, 0), window=msg_frame, anchor="nw")
canvas.bind("<Configure>", self._on_canvas_resize)
msg_frame.bind("<Configure>", self._on_msgframe_resize)

# Scroll até o final após adicionar mensagens
self.after(60, self._scroll_bottom)
```

**SEMPRE use `after(delay, func)` para scroll automático** — permite widgets renderizarem antes de calcular posição.

### 4. Gerenciamento de Histórico

**Estrutura de histórico:**
```python
self._historico[vizinho_nome] = deque(maxlen=300)  # Fila circular
# Cada item: {"tipo": "eu"|"deles"|"fwd"|"fwd_sent", "msg": Mensagem}

self._brutas[vizinho_nome] = deque(maxlen=50)  # Para encaminhamento
# Armazena objetos Mensagem completos
```

**Roteamento de mensagens recebidas** (`No._processar`):
1. Se `msg.encaminhado` e `encaminhado_por` está em `_historico` → chave = `encaminhado_por`
2. Senão se `remetente_nome` está em `_historico` → chave = `remetente_nome`
3. Senão → cria nova entrada dinâmica para remetente desconhecido

### 5. Sistema de Encaminhamento

**Encaminhamento preserva o remetente original:**
```python
msg_fwd = Mensagem(
    remetente_nome=msg_original.remetente_nome,  # ← Autor original!
    remetente_ip=msg_original.remetente_ip,
    remetente_porta=msg_original.remetente_porta,
    dest_nome=destino.nome,
    dest_ip=destino.ip,
    dest_porta=destino.porta,
    conteudo=msg_original.conteudo,
    encaminhado=True,
    encaminhado_por=self.nome,  # ← Quem repassou
)
```

## Execução e Debugging

### Rodar o chat

**Com GUI de configuração (sem argumentos):**
```bash
python3 chat_gui.py
```

**Via linha de comando (teste local 3 nós):**
```bash
# Terminal 1
python3 chat_gui.py No_A 127.0.0.1 5001  No_B 127.0.0.1 5002

# Terminal 2
python3 chat_gui.py No_B 127.0.0.1 5002  No_A 127.0.0.1 5001  No_C 127.0.0.1 5003

# Terminal 3
python3 chat_gui.py No_C 127.0.0.1 5003  No_B 127.0.0.1 5002
```

**Formato dos argumentos:**
```bash
python3 chat_gui.py <nome> <ip> <porta> <viz1_nome> <viz1_ip> <viz1_porta> [<viz2_nome> <viz2_ip> <viz2_porta> ...]
```

### Debugging comum

**Mensagens não aparecem:**
- Verifique firewall (porta UDP bloqueada?)
- Confirme que o IP/porta do destinatário está correto
- Use `print()` em `No._processar()` para ver datagramas recebidos

**Race conditions:**
- SEMPRE use `with self._lock:` ao acessar `_historico` ou `_brutas`
- GUI freezando? Verifique se está rodando operações pesadas na thread principal

**Canvas não rola:**
- `_on_msgframe_resize()` deve chamar `canvas.configure(scrollregion=...)`
- Use `self.after(50, self._scroll_bottom)` após adicionar widgets

## Convenções de Nomenclatura

- **Variáveis privadas:** `_lock`, `_historico`, `_canvas` (prefixo `_`)
- **Métodos de callback:** `_ao_enviar`, `_ao_fechar`, `_on_nova_msg` (prefixo `_`)
- **Métodos públicos:** `enviar`, `encaminhar`, `get_historico` (sem prefixo)
- **Constantes:** `FONT_MONO`, `C` (maiúsculas)
- **Classes:** `PascalCase` (`ChatApp`, `TelaSetup`, `No`)

## Documentação de Código

**Docstrings obrigatórias para:**
- Classes (descrição do propósito + layout se for GUI)
- Métodos públicos complexos (ex: `_processar`, `serializar`)

**Comentários inline para:**
- Explicações de protocolos não óbvios (ex: por que UDP é fire-and-forget)
- Decisões de threading (ex: por que `after(0, ...)` em vez de chamar direto)

**Estilo:**
```python
def _processar(self, msg: Mensagem):
    """
    Armazena mensagem recebida no histórico correto.

    Roteamento:
      encaminhada por vizinho conhecido → conversa daquele vizinho
      remetente direto → conversa do remetente
      origem nova      → cria entrada dinâmica
    """
```

## Notas de Implementação

- **Sem servidor central:** cada nó roda independentemente
- **Porta UDP:** recomendado 5001-5009 (evite portas privilegiadas <1024)
- **Thread daemon:** `threading.Thread(..., daemon=True)` garante encerramento limpo
- **Encerramento:** `No.encerrar()` fecha socket → causa `OSError` em `recvfrom()` → thread de escuta termina
- **Polling GUI:** `self.after(200, self._tick)` verifica mudanças no histórico sem redesenho constante (evita flicker)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/giacon001) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
