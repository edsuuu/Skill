# Criação de API

O request atravessa a cadeia que o Laravel já montou — nenhuma camada extra
inventada:

```
Rota (routes/api.php) → Middleware (auth/token, fail-closed)
  → FormRequest (validação na fronteira)
  → Controller magro → Service (regra de negócio)
  → Resource (formata a saída) → Exception com render() (erro vira JSON)
```

## FormRequest — validação na fronteira, nunca no controller

Todo endpoint que recebe dado tem seu `*Request` com `rules()`. O controller
já recebe validado (`$request->validated()`); `$request->input()` cru dentro
de controller é proibido.

## Controller magro, e UM por recurso

Recebe o validado, chama o service, devolve o Resource. Sem regra de negócio,
sem query, sem `response()->json([...], 4xx)` espalhado.

**Um controller por recurso**, com os métodos dele — `index`, `show`, `store`,
`update`, `destroy` e o que mais aquele recurso tiver. Um arquivo por ação
espalha sessenta arquivos de uma função só, e aí a maior parte de cada arquivo
é `<?php`, namespace e import: cabeçalho demais para código de menos.

`__invoke` fica para o recurso que tem **uma ação só de verdade** (um webhook,
um `/config`, um `/me`) — não para fatiar um CRUD.

```php
final class VideoController
{
    public function index(VideoService $service): AnonymousResourceCollection
    {
        return VideoResource::collection($service->recent());
    }

    public function store(StoreVideoRequest $request, VideoImportService $service): VideoResource
    {
        return new VideoResource($service->import($request->validated()));
    }

    public function destroy(Video $video, VideoService $service): Response
    {
        $service->remove($video);

        return response()->noContent();
    }
}
```

## Rota agrupada por prefixo

Caminho repetido vira `Route::prefix(...)`, e nome repetido vira
`Route::name(...)`. O arquivo de rotas passa a mostrar a forma da API em vez de
repetir `/servers/{server}/` vinte vezes.

```php
Route::name('api.')->middleware('auth:sanctum')->group(function (): void {
    Route::prefix('videos')->name('videos.')->group(function (): void {
        Route::get('/', [VideoController::class, 'index'])->name('index');
        Route::post('/', [VideoController::class, 'store'])->name('store');
        Route::delete('/{video}', [VideoController::class, 'destroy'])->name('destroy');
    });
});
```

Sub-recurso que pende de outro no caminho mas é recurso próprio (`/videos/{video}/comments`)
usa o prefixo de caminho do pai e o **nome dele mesmo** (`comments.index`), não o do pai.

## Resource — o retorno da controller é SEMPRE um Resource

A controller nunca devolve model cru nem array montado na mão: a saída passa
por um `*Resource` (`JsonResource`), que define o contrato público num lugar
só — refatora tabela/model sem quebrar consumidor. Coleção usa
`Resource::collection(...)`.

```php
final class VideoResource extends JsonResource
{
    /**
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'id'         => $this->uuid,
            'title'      => $this->title,
            'status'     => $this->status->value,
            'created_at' => $this->created_at->toIso8601String(),
        ];
    }
}
```

## Service — regra de negócio

Mesmas regras de sempre ([banco-e-logs.md](banco-e-logs.md)): validação de
negócio no topo, `DB::transaction()` pra conjunto, log no canal certo, e
falha vira exception própria — não `response()` dentro do service.

## Exception com render() próprio

Exception que fecha request HTTP implementa o próprio
`render(): JsonResponse` — o status code mora NELA, num lugar só, não
espalhado pelos controllers.

```php
final class VideoNotFoundException extends Exception
{
    public function render(): JsonResponse
    {
        return response()->json(['message' => 'Vídeo não encontrado.'], 404);
    }
}
```

## Middleware de auth — fail-closed

Endpoint protegido por token/auth valida no middleware, antes de tudo, e
**fail-closed**: env/config ausente = nega, nunca libera. Autenticação
específica (Sanctum, token próprio) e versionamento (`/api/v1`) são decisão
por projeto, não regra geral.
