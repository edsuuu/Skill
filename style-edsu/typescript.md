# TypeScript

- **Sempre classes — uma classe por arquivo.** Nada de arquivo com funções
  soltas exportadas; a lógica vive em métodos da classe dona.

  ```ts
  // ✅ video-player.ts
  export class VideoPlayer {
      play(): void { /* ... */ }
      private buffer(): void { /* ... */ }
  }

  // ❌ funções soltas exportadas
  export function play(): void { /* ... */ }
  export function buffer(): void { /* ... */ }
  ```
- **Zero comentários no front TS.** Nenhum comentário em `resources/js` (ou
  equivalente) — nem prosa, nem em cima de variável. O nome explica; se não
  explica, renomeie.
- Import no topo, nada de `await import(...)` no meio de função (regra em
  [php.md](php.md), vale igual aqui).
- Corrente de `===` na mesma variável → `.includes()` (regra em
  [comparacoes.md](comparacoes.md)).
