# Nomenclatura e organização

**Idioma: código 100% em inglês** — pastas, namespaces, classes, métodos,
propriedades, variáveis, migrations, colunas, env vars, config keys. pt-BR só
em: paths de rota (`/meus-videos`), strings de UI (labels, toasts) e
comentários.

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

## Onde colocar o arquivo — siga o que o projeto já faz

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
