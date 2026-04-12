
# CodeRabbit usage constraints

При использовании CodeRabbit CLI:

- Старайся запускать ревью на как можно меньшем, логически цельном наборе изменений.
- Не запускай CodeRabbit более 3 раз на один и тот же diff.
- Не пытайся игнорировать или “переписать” вывод CodeRabbit:
  - Если не согласен с рекомендацией, сначала объясни в чате, почему.
  - Только после этого можешь предложить альтернативное решение.

При имплементации рекомендаций CodeRabbit:

- В первую очередь исправляй:
  1) баги и логические ошибки,
  2) возможные проблемы с безопасностью,
  3) серьёзные перфоманс-проблемы.
- Стиль и мелкие “nit” правь только если это не ломает контекст и не раздувает diff.
- Не переписывай большие куски кода без необходимости только ради стиля.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DaneliyaPavel)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/DaneliyaPavel)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
