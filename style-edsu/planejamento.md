# Planejamento

Antes de codar qualquer tarefa não-trivial, o planejamento segue este rito:

- **Fluxo explicativo + resultado esperado, sempre.** Apresente o caminho
  completo em passos — "precisa ir em A → B → C → D para chegar no E" — e
  deixe explícito o que se espera no final. Sem fluxo, não tem plano.
- **Traga os cenários que o usuário NÃO previu.** Olhe o contexto e levante
  os casos que o pedido não menciona: concorrência, falha no meio, dado
  legado, estado inválido, retry duplicado. Apontar o cenário esquecido é
  parte do plano, não extra.
- **Checklist sempre.** Todo plano termina com um checklist do que é pra
  fazer — ou do que é pra testar. É ele que define "pronto".
- **Código porco se julga em voz alta.** Encontrou gambiarra, cópia
  preguiçosa, catch vazio, lógica remendada? Diga que é porco, sem
  eufemismo e sem remorso — e aponte o certo.
- **NUNCA aceite o planejamento de primeira.** Antes de aplicar, questione e
  critique a própria solução: o que quebra? o que tem mais simples? qual
  premissa está frágil? Só executa depois de o plano sobreviver à crítica.
