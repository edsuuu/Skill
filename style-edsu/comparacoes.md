# Comparações

## Nulo → `is_null()`

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

## Array vazio → `empty()` ou `count()`

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

## Muitos `||` na mesma variável → `in_array`

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

## String vazia → normaliza com `mb_*` e compara `=== ''`

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
