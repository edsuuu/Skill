# Git / commits

- **NUNCA commite, pushe ou abra PR sem autorização explícita do usuário, na
  hora.** Autorização de um commit não vale pro próximo. Atenção redobrada
  em repo com automerge: push ≈ merge.
- **Sempre em branch — nunca direto na `main`.** Padrão de nome: `feat/`,
  `fix/`, `refactor/`, `chore/`, `docs/` + descrição curta em kebab-case
  (ex.: `feat/upload-de-video`).
- **Claude NUNCA vai como co-autor.** Nada de `Co-Authored-By:` nem
  "Generated with" no corpo — a mensagem é só o texto da mensagem.
- **Mensagem simples, uma linha**: `tipo: descrição curta`.

  ```
  ✅ feat: criando um hover para coisa x
  ✅ fix: corrige claim duplicado no download
  ❌ feat: implementa o sistema completo de hover incluindo estados de
     focus, transições, tokens de cor e ajustes de acessibilidade (...)
  ```
- **Nada de commit grande.** Commit gigante é sinal de que faltou fasear.
- **Feature enorme → desenvolvimento faseado.** Se dá pra ver que vai ficar
  grande, quebre em partes por fluxo independente (mexeu no fluxo A e no
  fluxo B, e editar um não afeta o teste do outro → são fases separadas).
  A cada fase: **pare, peça a revisão do código ao usuário, commite só
  depois do OK** — e aí continua a próxima fase. Nunca acumular tudo pra um
  commit só no final.
