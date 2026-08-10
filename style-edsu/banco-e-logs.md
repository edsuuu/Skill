# Validações, banco de dados e logs

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

## Rastreabilidade — ação relevante deixa rastro

Toda ação que importa precisa poder ser rastreada depois:

- **Log** no canal certo (regra acima).
- **Toast** pro usuário quando é ação de UI (sucesso/erro visível, não falha
  muda).
- **Avalie caso a caso** se faz sentido **persistir um registro em banco**
  (auditoria/ledger) — nem toda ação precisa, mas as de dinheiro, postagem,
  credencial e afins geralmente sim. Pense antes de decidir que não.
