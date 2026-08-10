# PHP — cabeçalho, imports e flags

Todo arquivo PHP gerado ou modificado **começa com**:

```php
<?php

declare(strict_types=1);
```

Classes sempre `final` salvo motivo real pra herança.

## Import inline → PROIBIDO

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

## Flags de funções nativas

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

## Integração de API — nada de URL hardcoded

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
