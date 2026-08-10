# Comentários

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
