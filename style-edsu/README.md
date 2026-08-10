# style-edsu

Skill do Claude Code com as convenções pessoais de código do Edsu —
nomenclatura, estrutura de função, comentários, comparações, Livewire e
simplicidade. O ponto de entrada é o [SKILL.md](SKILL.md); cada bloco de
regras tem seu próprio arquivo (`nomenclatura.md`, `php.md`, etc.).

## Instalação

Clone o repositório e crie um symlink na pasta de skills do usuário — assim
qualquer `git pull` já atualiza a skill em todos os projetos:

```bash
git clone git@github.com:edsuuu/Skill.git ~/projects/Skill
```

```bash
ln -s ~/projects/Skill/style-edsu ~/.claude/skills/style-edsu
```

Se preferir sem symlink, uma cópia direta também funciona:

```bash
cp -R ~/projects/Skill/style-edsu ~/.claude/skills/style-edsu
```

Para ativar só em um projeto específico (em vez de global), aponte o link
para o `.claude/skills/` do projeto:

```bash
ln -s ~/projects/Skill/style-edsu /caminho/do/projeto/.claude/skills/style-edsu
```

Depois de instalada, a skill aparece na lista de skills da sessão e pode ser
invocada com `/style-edsu` — mas o normal é o Claude ativá-la sozinho ao
criar ou editar código, conforme a `description` do SKILL.md.
