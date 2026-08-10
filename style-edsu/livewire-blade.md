# Livewire / Blade

- **Rota de tela é sempre `Route::view()` com blade intermediária** — classe
  Livewire NUNCA é importada em `routes/*.php`. O layout (`<x-app-layout>`)
  é importado **na blade intermediária**, nunca no componente (nada de
  `#[Layout]`). Parâmetro de rota entra via `request()->route(...)`.

  ```php
  // ✅ rota → blade wrapper
  Route::view('/editor-de-video/{cut}', 'video-editor.index')->name('video-editor.index');
  ```
  ```blade
  {{-- ✅ resources/views/video-editor/index.blade.php --}}
  <x-app-layout :title="__('Editor de vídeo')">
      <livewire:video-editor.index :uuid="request()->route('cut')" />
  </x-app-layout>
  ```
  ```php
  // ❌ componente Livewire direto na rota
  Route::get('/editor-de-video/{cut}', VideoEditorIndex::class);
  ```

- **Back e front sempre separados**: a classe só tem lógica e o
  `render()` entrega os dados prontos pra view do componente
  (`livewire.<area>.<nome>`) — nada de HTML/layout na classe.

  ```php
  // ✅ app/Livewire/Dashboard/Index.php
  final class Index extends Component
  {
      public function render(): View
      {
          return view('livewire.dashboard.index', [
              'videoCount' => Video::query()->where('user_id', Auth::id())->count(),
          ]);
      }
  }
  ```

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
- **Ordem dos métodos** no componente: `mount()` é **sempre a PRIMEIRA
  função** e `render()` **sempre a ÚLTIMA** — no meio, ações públicas e
  depois helpers privados. Sem exceção: função nova nunca entra antes do
  `mount()` nem depois do `render()`.
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
