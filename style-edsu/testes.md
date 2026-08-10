# Testes

- **Suíte de testes (unitário/feature) usa sempre `.env.testing`.** Se o
  arquivo não existir no projeto, crie antes de rodar — nunca rode a suíte em
  cima do `.env` de dev (o `.env` é só da aplicação rodando).
- **Teste unitário só com autorização.** Antes de sair criando teste
  unitário, pergunte se o usuário quer — a decisão de cobrir com unitário é
  dele, não sua.
- **Todo trabalho de teste devolve um checklist de pontos de teste**: o que
  precisa ser verificado, caso a caso (feliz, falha, borda), marcável.
- **Gere um HTML com o checklist completo** — uma página simples e
  autocontida com todos os pontos de teste, pra acompanhar/marcar o que já
  foi validado.
