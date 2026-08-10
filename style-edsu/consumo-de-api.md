# Consumo de API

Arquitetura em camadas, cada uma com um papel — e cada camada só existe
quando ganha o lugar dela:

```
UI (Livewire/Command/Job) → Service (regra de negócio) → Client (HTTP)
```

## Client — um por integração

A classe com as chamadas HTTP (Gateway). Esconde URL, auth, headers e formato
atrás de métodos com nome de negócio. URL/token/scope vêm de `config(...)`
(regra em [php.md](php.md)); retry/timeout se resolvem **decorando o
`Http::`** (`retry()`, `timeout()`, `throw()`) — nunca escrevendo classe de
retry/decorator na mão.

## Log obrigatório em TODA chamada

Toda chamada HTTP loga **headers enviados, body enviado e resposta** — é o
mapeamento pra rastrear qualquer conversa com a API depois. Segue as regras
de [banco-e-logs.md](banco-e-logs.md): canal próprio da integração, mensagem
fixa `[INFO]`/`[ERRO]`, dados no contexto. Secret/token no header vai
**mascarado** — log não é lugar de credencial.

```php
final class YoutubeVideosService
{
    public function find(string $videoId): array
    {
        $query = ['id' => $videoId];

        $response = Http::baseUrl(config('services.youtube.base_url'))
            ->retry(3, 200)
            ->withToken(config('services.youtube.token'))
            ->get('/videos', $query);

        Log::channel('youtube')->info('[INFO] chamada de busca de vídeo', [
            'headers'  => ['Authorization' => 'Bearer ***'],
            'body'     => $query,
            'status'   => $response->status(),
            'response' => $response->json(),
        ]);

        return $response->throw()->json();
    }
}
```

## Resposta: array direto — DTO só com objeto grande

O client devolve o `->json()` (array) e quem consome pega os campos com cast
explícito (`(string)`, `(int)` — regra em
[funcoes-e-classes.md](funcoes-e-classes.md)). **DTO (`*Data`) só quando o
objeto tem MUITOS dados** e espalhar array por vários pontos viraria bagunça
— aí o DTO centraliza a tradução num lugar. Pra resposta pequena (meia dúzia
de campos, 1-2 consumidores), DTO é cerimônia.

## Interface no client — só quando ganha o lugar

Interface com uma implementação eterna é cerimônia. Ela se justifica em dois
casos: existe (ou vai existir de verdade) um **segundo provedor
intercambiável** (aí é Strategy), ou o **teste precisa dublar a borda HTTP**.
Fora isso, o service depende da classe concreta e o container injeta igual.

Com 2+ provedores, a escolha da estratégia é um `match` — registry/factory é
upgrade pra quando a lista crescer:

```php
$poster = match ($platform) {
    'youtube' => app(YoutubePosterService::class),
    'tiktok'  => app(TiktokPosterService::class),
};
```

## Service intermediário — só quando existe regra

A diferença entre ter e não ter o service no meio é **onde mora a regra de
negócio**:

- **Chamada + regra** (persistir, validar duplicata, transaction, disparar
  job, reusar em 2+ entradas — Livewire hoje, Command/webhook amanhã) →
  service. A operação de negócio existe num lugar só.
- **Chamada pura e exibir** (buscar CEP e preencher form, autocomplete) →
  UI chama o client direto. Service que só faz `return $this->client->find()`
  é embrulho vazio — camada de passagem é proibida.

Upgrade barato: começou direto e apareceu a segunda regra ou o segundo
chamador → extrai o service **nesse momento**, não no dia 1.

## Injeção — nunca `new` na mão

Client e service entram por **construtor** e o container resolve. `new
HttpClient()` dentro do service mata a injeção: acopla na implementação e
impede mock/troca.

## Padrões que o Laravel JÁ é — não reimplemente

Singleton = container (`$this->app->singleton`); Facade = `Http::`/`Log::`;
Chain of Responsibility = middleware; Observer = events/listeners; Command =
jobs; Builder = query builder; Strategy = drivers por config; Decorator =
middleware do `Http::`. Escrever qualquer um desses na mão é duplicar o
framework com versão pior.
