# Estrutura de função e classes

## Early return sempre — nada de `if` aninhado

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

## Extraia função quando repetir OU quando ficar grande

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

## Classes

Evite classe que só tem 1–2 funções. Se a lógica é usada **num só lugar**, ela
é **método** da classe que consome — não uma classe própria.

```
✅ um método privado dentro do Service que já existe
❌ uma classe FooHelper nova só pra abrigar formatFoo()
```

Classe própria só quando: é reusada em vários lugares, tem estado próprio, ou
representa um conceito de domínio de verdade.

**NUNCA crie classe de Support/Helper pra fazer cast.** Conversão de tipo é
explícita e inline, com o cast nativo do que o valor é: `(string)`, `(int)`,
`(float)`, `(bool)`, `(array)`.

```php
// ✅ cast nativo, explícito, no lugar do uso
$videoId = (int) $request->input('video_id');
$email   = (string) $socialUser->getEmail();

// ❌ classe só pra abrigar um cast
final class CastSupport
{
    public static function toInt(mixed $value): int
    {
        return (int) $value;
    }
}
$videoId = CastSupport::toInt($request->input('video_id'));
```

## `filter`/`map`/reduce só quando ganham

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
