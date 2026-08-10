---
name: style-edsu
description: Convenções pessoais de código do Edsu para escrever ou modificar código — nomenclatura, estrutura de função, comentários, comparações, Livewire e simplicidade. Use ao criar/editar funções, classes ou arquivos, sobretudo em PHP/Laravel/Livewire. Complementa o ponytail (laziness); esta cuida do estilo concreto.
---

# Estilo de código do Edsu

Regra-mãe: **código simples, feijão com arroz**. `if/else`, `foreach`, o óbvio.
Nada de esperteza que alguém decifra às 3 da manhã. Se está ficando complexo,
quase sempre está errado — pare e simplifique antes de continuar.

---

## 1. Nomenclatura e organização

O que a classe/arquivo **é** vem como **sufixo, no FINAL** do nome.

```
✅ SpotifyService
✅ SpotifyDownloadController
✅ DownloadPlaylistJob
✅ VideoStatusEnum
✅ PostTaskData
❌ ServiceSpotify
```

Sufixos comuns: `*Service`, `*Controller`, `*Job`, `*Enum`, `*Data`,
`*Command`, `*Cast`, `*Exception`, `*Interface`.

**Nunca abrevie nome de variável.** Nome por extenso, sempre.

```php
✅ $exception   $repository   $userProfile   $index
❌ $e           $repo         $usrProf       $i
```

### Onde colocar o arquivo — siga o que o projeto já faz

Leia a estrutura existente antes de criar. O arquivo novo vai onde os
**iguais** já moram, não na raiz.

- Já existe **pasta que agrupa aquele tipo** (ex.: `Enums/` que acopla todos os
  enums)? Cria lá.
- O arquivo novo tem a **mesma natureza** de arquivos que já vivem numa pasta
  (ex.: 2 webhooks em `Webhook/` e você criou um terceiro na raiz)? **Move pra
  pasta deles.** Nunca deixa o irmão órfão na raiz.
- **Interface criada no service e usada na mesma camada** → fica **no próprio
  service**, não numa pasta de contracts global.
- Criou um service e já tem **2+ DTOs soltos na raiz** dele? Cria a pasta
  (`DTO/`, `Data/` — o nome que o projeto usar) e **move os DTOs pra lá**.

Regra-guia: o padrão do projeto vence o gosto pessoal. Se ele acopla por tipo,
acople por tipo; se agrupa por feature, agrupe por feature.

---

## 2. Estrutura de função

### Early return sempre — nada de `if` aninhado

Condições que levam ao **mesmo desfecho** vão num `if` só, com `||`. Não
empilhe dois `if` seguidos que fazem a mesma coisa.

```php
// ✅ assim — um if só, os dois guards retornam igual
public function handle(User $user): void
{
    if (is_null($user->email) || ! $user->isActive()) {
        return;
    }

    $this->notify($user);
}

// ❌ if aninhado
if (! is_null($user->email)) {
    if ($user->isActive()) {
        $this->notify($user);
    }
}
```

Guards com desfecho **diferente** continuam separados (cada um com seu
`return`/`throw`).

### Extraia função quando repetir OU quando ficar grande

Regra base: só vira função nova se for chamada em **mais de um lugar**. Usada
uma vez → inline; não crie função pra dar nome ao óbvio.

```php
// ❌ não crie isso — é chamado uma vez e é uma comparação boba
private function isMp4(string $video): bool
{
    return $video === 'mp4';
}

// ✅ deixa inline
if ($video === 'mp4') {
    // ...
}
```

**Exceção — tamanho:** se o método ficou grande demais pra ler de uma vez,
faz sentido quebrar em métodos privados menores, mesmo que só chamados uma
vez. O gatilho aqui é legibilidade, não reuso.

---

## 3. Classes

Evite classe que só tem 1–2 funções. Se a lógica é usada **num só lugar**, ela
é **método** da classe que consome — não uma classe própria.

```
✅ um método privado dentro do Service que já existe
❌ uma classe FooHelper nova só pra abrigar formatFoo()
```

Classe própria só quando: é reusada em vários lugares, tem estado próprio, ou
representa um conceito de domínio de verdade.

---

## 4. Comentários

- Comentário-prosa **só em cima da função**, e **apenas em casos extremos** —
  regra de negócio não óbvia, pegadinha de concorrência. Na dúvida, **não
  comente**: o nome já explica.
- **Nunca** comentário em cima de variável ou propriedade.
- **Docblock com anotação vai sempre em cima da função** — `@throws`, `@param`,
  `@return`, `@var` (tudo que a IDE/PHPStan usa). Isso não é "comentário
  extremo", é anotação: use sempre que houver o que anotar (exception lançada,
  tipo genérico de array, etc.). Docblock **só pra repetir o nome**, não.

```php
// ✅ docblock de anotação — método que lança
/**
 * @throws Throwable
 */
public function store(array $data): void { /* ... */ }

// ✅ anotação de tipo que o PHPStan precisa
/**
 * @param  array<int, VideoData>  $videos
 * @return Collection<int, Video>
 */
public function sync(array $videos): Collection { /* ... */ }
```

```php
// ✅ caso extremo: explica o PORQUÊ, não o QUÊ
// Claim atômico: o UPDATE precisa filtrar dispatched_at IS NULL senão dois
// ticks simultâneos postam o mesmo slot duas vezes.
public function claim(int $slotId): bool { /* ... */ }

// ❌ comentário que só repete o nome
// baixa o vídeo
public function downloadVideo(): void { /* ... */ }

// ❌ NUNCA em cima de variável
// nome do usuário
$name = $user->name;
```

---

## 5. PHP — cabeçalho obrigatório

Todo arquivo PHP gerado ou modificado **começa com**:

```php
<?php

declare(strict_types=1);
```

Classes sempre `final` salvo motivo real pra herança.

### Import inline → PROIBIDO

Todo import vai **no topo do arquivo**. Nunca referencie classe pelo FQCN no
meio do código — vai de `use` e nome curto. Mesma regra no TS: `import` no
topo, nada de `await import(...)`/`require(...)` no meio de função (exceto
quando o lazy-load é a razão de existir do código, e aí comenta o porquê).

```php
// ✅ use no topo
use App\Services\API\Discord\DiscordNotifierService;

$notifier = app(DiscordNotifierService::class);

// ❌ FQCN inline
$notifier = app(\App\Services\API\Discord\DiscordNotifierService::class);
```

**Sempre passe as flags** das funções nativas que as aceitam — sobretudo as
que trocam falha silenciosa por exceção. `json_decode`/`json_encode` **sempre**
com `JSON_THROW_ON_ERROR` (e o `@throws JsonException` em cima da função).

```php
// ✅
/**
 * @throws JsonException
 */
public function read(string $key): array
{
    return json_decode((string) $disk->get($key), true, 512, JSON_THROW_ON_ERROR) ?: [];
}

// ❌ sem flag — erro vira null silencioso
json_decode($disk->get($key), true);
```

---

## 6. Comparações

### Nulo → `is_null()`

Model/valor que pode retornar nulo (find, first, campo nullable): use
`is_null()`, não `=== null` nem `! $x`.

```php
// ✅
$user = User::find($id);
if (is_null($user)) {
    return;
}

// ❌
if ($user === null) { /* ... */ }
if (! $user)        { /* ... */ }
```

### Array vazio → `empty()` ou `count()`

```php
// ✅
if (empty($videos)) {
    return;
}
// ✅ quando você já precisa do número
if (count($videos) === 0) { /* ... */ }

// ❌
if ($videos === []) { /* ... */ }
if (! $videos)      { /* ... */ }
```

### Muitos `||` na mesma variável → `in_array`

Quando é a mesma variável comparada com vários valores, troca a corrente de
`||` por `in_array(..., true)` (sempre com o `strict` `true`).

```php
// ✅
if (in_array($video, ['mp4', 'mp3', 'mov'], true)) { /* ... */ }

// ❌
if ($video === 'mp4' || $video === 'mp3' || $video === 'mov') { /* ... */ }
```

Em TypeScript é o mesmo com `.includes()`:

```ts
// ✅
return ['entity.parse.failed', 'entity.too.large'].includes(type) || status === 400;

// ❌
return type === 'entity.parse.failed' || type === 'entity.too.large' || status === 400;
```

### String vazia → normaliza com `mb_*` e compara `=== ''`

Trate a string com as funções nativas `mb_*` (`mb_trim`, `mb_strtolower`)
antes de comparar. Comparação de vazio é `=== ''`, não `empty()` nem `! $str`.

```php
// ✅
$email = mb_strtolower(mb_trim((string) $socialUser->getEmail()));
if ($email === '') {
    return;
}

// ❌
if (empty($email))     { /* ... */ }
if (! trim($email))    { /* ... */ }  // trim sem mb_, e coerção frouxa
```

---

## 7. Validações e banco de dados

- Ação que escreve no banco vai **dentro de `try/catch`** — nunca deixe a
  falha vazar crua, e **nunca `catch` vazio**.
- **Todo `catch` loga — em QUALQUER linguagem.** Nunca `catch` vazio (nem só
  com comentário). Sem logger no contexto, pelo menos um `console.log`/print.
  Vale pra PHP, TS/Node, Python, tudo.

  ```ts
  // ✅ mesmo pra falha esperada, deixa rastro
  private async removeQuietly(path: string): Promise<void> {
      try {
          await unlink(path);
      } catch (error) {
          console.warn(`[WARN] falha ao remover ${path}`, error);
      }
  }
  ```
- **Nunca `Log::error()` / `Log::warning()` / `Log::info()` direto** (facade no
  canal default). **Sempre com channel**: o canal próprio do contexto se
  existir, senão `Log::channel('daily')`.
- **A mensagem NUNCA começa com `$exception->getMessage()`** — é horrível de
  rastrear. A mensagem é **descritiva e fixa**, prefixada com o **nível** entre
  colchetes: `[ERRO]` / `[INFO]` / `[WARN]`. O `getMessage()` e a exception vão
  no **contexto**, não no texto.

  ```php
  // ✅ mensagem fixa e descritiva, exception no contexto
  Log::channel('daily')->error('[ERRO] falha ao salvar o pedido', [
      'exception' => $exception,
      'message'   => $exception->getMessage(),
      'file'      => $exception->getFile(),
      'line'      => $exception->getLine(),
      'user_id'   => $userId,
  ]);

  Log::channel('youtube')->info('[INFO] upload concluído', [
      'video_id' => $videoId,
  ]);

  // ❌ getMessage como texto principal
  Log::channel('daily')->error($exception->getMessage(), ['exception' => $exception]);
  // ❌ facade no canal default
  Log::error('falha');
  ```
- Escrita que toca **mais de uma tabela/linha** (ou grava + dispara efeito)
  roda em **`DB::transaction()`**: ou tudo, ou nada. Sem estado meio-gravado.
- Valide **antes** de tocar o banco (guards no topo). O banco é o último passo,
  não o lugar de descobrir que o dado era inválido.
- Criação que pode colidir → **`firstOrCreate`** (ou `updateOrCreate`), nunca
  `create` cru que gera duplicata.

  ```php
  // ✅
  $tag = Tag::firstOrCreate(['slug' => $slug], ['name' => $name]);
  // ❌ cria duplicata se já existir
  $tag = Tag::create(['slug' => $slug, 'name' => $name]);
  ```

```php
// ✅ valida no topo, transaction pro conjunto, catch loga no canal certo
public function store(array $data): void
{
    if (is_null($data['user_id']) || empty($data['items'])) {
        return;
    }

    try {
        DB::transaction(function () use ($data): void {
            $order = Order::create(['user_id' => $data['user_id']]);
            $order->items()->createMany($data['items']);
        });
    } catch (Throwable $exception) {
        Log::channel('daily')->error('[ERRO] falha ao salvar o pedido', [
            'exception' => $exception,
            'message'   => $exception->getMessage(),
            'file'      => $exception->getFile(),
            'line'      => $exception->getLine(),
        ]);

        throw $exception;
    }
}
```

Escrita única e trivial (um `update` num campo) não precisa de transaction —
transaction é pra proteger **conjunto**.

### Rastreabilidade — ação relevante deixa rastro

Toda ação que importa precisa poder ser rastreada depois:

- **Log** no canal certo (regra acima).
- **Toast** pro usuário quando é ação de UI (sucesso/erro visível, não falha
  muda).
- **Avalie caso a caso** se faz sentido **persistir um registro em banco**
  (auditoria/ledger) — nem toda ação precisa, mas as de dinheiro, postagem,
  credencial e afins geralmente sim. Pense antes de decidir que não.

### Migration nova → PERGUNTE antes de criar

**Nunca saia criando migration.** Antes de gerar qualquer migration nova
(coluna, tabela, índice, alter), **pare e pergunte** — pode já existir outra
solução, ou o schema pode ser decisão do dono do projeto. Só cria depois do OK.

---

## 8. Integração de API

**URL/endpoint de serviço externo nunca fica hardcoded** em constante ou
variável de classe. Vai pro **`.env`** → exposto via **`config/`** → lido com
`config(...)`. Nada de `env()` espalhado pelo código.

Isso vale pra **qualquer URL**: base URL, endpoint, secret, token **e scopes
de OAuth** (que também são URL). Nada disso fica em `const` — tudo em
`config/`, lido com `config(...)`.

```php
// ❌ URL hardcoded na classe
private const string BASE_URL = 'https://www.googleapis.com/youtube/v3';
$response = Http::get('https://www.googleapis.com/youtube/v3/videos');

// ✅ .env → config → config()
// config/services.php
'youtube' => [
    'base_url' => env('YOUTUBE_API_URL'),
    'scopes'   => explode(',', (string) env('YOUTUBE_SCOPES')),
],
// no service
$response = Http::get(config('services.youtube.base_url').'/videos');
$scopes   = config('services.youtube.scopes');
```

---

## 9. Livewire / Blade

- **Nada de `@php` no blade.** Lógica, formatos, labels e datas vêm **prontos**
  do `render()` (view-models). O blade só exibe.

  ```blade
  {{-- ✅ classe condicional --}}
  <div @class(['badge', 'badge-active' => $isActive])>

  {{-- ❌ ternário dentro de class --}}
  <div class="badge {{ $isActive ? 'badge-active' : '' }}">

  {{-- ❌ @php no blade --}}
  @php $label = ucfirst($status); @endphp
  ```

- Classe condicional **sempre** via `@class([...])`, nunca ternário em `class=""`.
- **Cor/classe de Tailwind NUNCA sai do back.** Nada de `bg-white text-black`
  em string no componente. O back devolve só **estado/boolean**; o blade decide
  a classe no `@class([...])`.

  ```blade
  {{-- ✅ o blade mapeia estado → classe --}}
  <span @class([
      'bg-white text-black' => $isActive,
      'bg-slate-800 text-slate-400' => ! $isActive,
  ])>
  ```
  ```php
  // ❌ classe de Tailwind montada no back
  public string $badgeClass = 'bg-white text-black';
  ```
- Propriedade pública é **fronteira de confiança**: valide/saneie dentro das
  ações, nunca confie no valor cru.
- **Ordem dos métodos** no componente:
  `mount()` → ações públicas → helpers privados → `render()` por **último**.
- **`dispatch` sai de um método no back**, não solto na view. Crie a ação no
  componente e chame ela — nada de `wire:click="$dispatch(...)"` cru no blade.

  ```php
  // ✅ componente: a ação dispara o evento
  public function confirmDelete(int $id): void
  {
      $this->dispatch('item-deleted', id: $id);
  }
  ```
  ```blade
  {{-- ✅ --}}  <button wire:click="confirmDelete({{ $id }})">
  {{-- ❌ --}}  <button wire:click="$dispatch('item-deleted', { id: {{ $id }} })">
  ```

---

## 10. `filter`/`map`/reduce só quando ganham

Não corra pra corrente de `filter`/`map`/`reduce` por reflexo. Avalie a
abordagem mais simples primeiro — muitas vezes um `foreach` de uma passada é
mais claro. Use as funções de ordem superior **só quando** deixam o código
realmente mais legível (transformação direta 1:1, filtro óbvio), não pra
parecer esperto.

```php
// ✅ simples e direto
$titles = [];
foreach ($videos as $video) {
    if ($video->isReady()) {
        $titles[] = $video->title;
    }
}

// ✅ map/filter quando é transformação limpa e vale a pena
$titles = collect($videos)->filter->isReady()->pluck('title')->all();

// ❌ cadeia rebuscada onde um foreach resolvia melhor
$titles = array_values(array_map(
    fn ($video) => $video->title,
    array_filter($videos, fn ($video) => $video->isReady()),
));
```

---

## 11. PERGUNTE antes — regra de negócio e código duplicado

### Regra de negócio → NUNCA mude sem perguntar

Código que carrega regra de negócio (condição de status, cálculo, fluxo de
estado, permissão, valor de corte) **não se altera por iniciativa própria** —
nem "de passagem" durante um refactor ou bug fix. Percebeu que a tarefa exige
mudar uma regra de negócio? **Pare e pergunte** antes de mexer; só continua
depois do OK.

### Já existe trecho que faz o mesmo? → pergunte: centralizar ou duplicar

Antes de escrever, procure no sistema um trecho que já faça a mesma coisa.
Se existir, **não decida sozinho**: pergunte se é pra **centralizar** (extrair
e reusar o existente) ou **duplicar** (copiar e seguir) — centralizar mexe em
quem já usa e pode exigir retestar fluxos que estavam funcionando; essa
escolha é do dono do projeto.
