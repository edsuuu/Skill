---
name: style-edsu
description: Convenções pessoais de código do Edsu para escrever ou modificar código — nomenclatura, estrutura de função, comentários, comparações, Livewire e simplicidade. Use ao criar/editar funções, classes ou arquivos, sobretudo em PHP/Laravel/Livewire. Complementa o ponytail (laziness); esta cuida do estilo concreto.
---

# Estilo de código do Edsu

Regra-mãe: **código simples, feijão com arroz**. `if/else`, `foreach`, o óbvio.
Nada de esperteza que alguém decifra às 3 da manhã. Se está ficando complexo,
quase sempre está errado — pare e simplifique antes de continuar.

Abaixo, o resumo das regras duras de cada bloco. **Antes de escrever código
que toca um bloco, leia o arquivo dele** — é onde estão os exemplos e os
detalhes.

## Blocos

### [planejamento.md](planejamento.md)
Todo plano tem fluxo explicativo (A → B → C → D pra chegar no E) + resultado
esperado; traz cenários que o usuário não previu; termina em checklist (do
que fazer ou testar); código porco se chama de porco, sem remorso; nunca
aceitar o plano de primeira — questionar e criticar antes de aplicar.

### [testes.md](testes.md)
Suíte de testes roda sempre com `.env.testing` (não existe? cria antes; o
`.env` é só da aplicação); teste unitário
só depois de perguntar ao usuário; entregar checklist de pontos de teste
(feliz/falha/borda) + HTML autocontido do checklist pra marcar o que foi
validado.

### [git-commits.md](git-commits.md)
Nunca commit/push/PR sem autorização explícita na hora; sempre em branch
(`feat/descricao-kebab`), nunca na `main`; Claude nunca como co-autor (sem
`Co-Authored-By`/"Generated with"); mensagem simples de uma linha
(`feat: criando um hover para coisa x`); nada de commit grande — feature
enorme vira desenvolvimento faseado por fluxo independente, e cada fase só
commita depois do usuário revisar o código.

### [nomenclatura.md](nomenclatura.md)
Código 100% em inglês (pt-BR só em rota, UI e comentário); sufixo no FINAL
(`SpotifyService`, nunca `ServiceSpotify`); nunca abreviar
variável (`$exception`, não `$e`); arquivo novo mora onde os iguais já moram
(o padrão do projeto vence o gosto pessoal).

### [funcoes-e-classes.md](funcoes-e-classes.md)
Early return sempre, nada de `if` aninhado; guards de mesmo desfecho num `if`
só com `||`; função nova só com reuso ou pra quebrar método grande; método
privado antes de classe nova; nunca classe Support/Helper pra cast — cast
nativo inline (`(string)`, `(int)`...); `filter`/`map` só quando ganham do
`foreach`.

### [comentarios.md](comentarios.md)
Prosa só em cima de função e só em caso extremo (o nome já explica); NUNCA em
cima de variável/propriedade; docblock de anotação (`@throws`, `@param`,
`@return`, `@var`) sempre que houver o que anotar.

### [php.md](php.md)
`declare(strict_types=1)` em todo arquivo; classes `final`; import no topo
(FQCN inline PROIBIDO, no TS idem); flags nativas sempre (`json_*` com
`JSON_THROW_ON_ERROR`); URL/token/scope de API nunca hardcoded — `.env` →
`config/` → `config(...)`.

### [typescript.md](typescript.md)
Sempre classes — uma classe por arquivo, sem funções soltas; zero comentários
no front TS; import no topo e `.includes()` valem igual aqui.

### [comparacoes.md](comparacoes.md)
Nulo → `is_null()`; array vazio → `empty()`/`count()`; corrente de `||` na
mesma variável → `in_array(..., true)` (TS: `.includes()`); string vazia →
normaliza com `mb_*` e compara `=== ''`.

### [banco-e-logs.md](banco-e-logs.md)
Escrita no banco em `try/catch`; todo `catch` loga (qualquer linguagem, nunca
vazio); `Log::channel(...)` sempre (facade default proibida), mensagem fixa
`[ERRO]`/`[INFO]`/`[WARN]` com exception no contexto; `DB::transaction()` pra
conjunto; validação antes do banco; `firstOrCreate` quando pode colidir;
ação relevante deixa rastro (log + toast + avaliar ledger).

### [livewire-blade.md](livewire-blade.md)
Rota de tela sempre `Route::view()` com blade intermediária — Livewire nunca
importado em routes, layout importado no wrapper (nada de `#[Layout]`),
parâmetro via `request()->route(...)`; back e front separados (`render()`
entrega dados prontos pra view `livewire.*`); sem `@php` no
blade (view-model pronto do `render()`); classe condicional via
`@class([...])`; cor/classe Tailwind nunca sai do back; propriedade pública é
fronteira de confiança; `mount()` sempre a primeira função e `render()`
sempre a última (meio: ações públicas → helpers privados);
`dispatch` sai de método, não solto na view.

## PERGUNTE antes — sempre ativo

- **Migration nova** (coluna, tabela, índice, alter): nunca crie por conta
  própria — pare e pergunte, o schema é decisão do dono do projeto.
- **Regra de negócio** (condição de status, cálculo, fluxo de estado,
  permissão, valor de corte): nunca mude por iniciativa própria, nem "de
  passagem" num refactor — pare e pergunte antes.
- **Já existe trecho que faz o mesmo?** Não decida sozinho entre centralizar
  (extrair e reusar — mexe em quem já usa) ou duplicar (copiar e seguir) —
  pergunte; a escolha é do dono do projeto.
