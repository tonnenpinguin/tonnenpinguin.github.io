---
date: 2025-09-25T14:43:19.348Z
title: "Solving Ecto Upserts with Partial Unique Indices"
---

Recently I encountered a PostgreSQL error while trying to upsert (insert or update) records with soft deletes. When using partial unique indices, Ecto's default upsert behavior doesn't work out of the box, resulting in errors like the following. Here's how I solved it.

```
** (Postgrex.Error) ERROR 42P10 (invalid_column_reference) there is no unique or
    exclusion constraint matching the ON CONFLICT specification
```

### Context

When implementing soft deletes (marking records as deleted instead of removing them), you often need unique constraints that only apply to active (non-deleted) records. This is where partial unique indices shine - they allow multiple "deleted" records with the same unique value while ensuring only one active record exists.

In this example, we want to ensure that each `tag` value exists only once among non-deleted records, but deleted records can have duplicate tag values.

Here's the Ecto schema with soft delete support:

```elixir
defmodule IdTag do
  use Ecto.Schema
  import Ecto.SoftDelete.Schema
  import Ecto.Changeset

  schema "idtags" do
    field :description, :string
    field :name, :string
    field :tag, :string

    soft_delete_schema()  # Adds deleted_at field
  end

  def changeset(id_tag, attrs) do
    id_tag
    |> cast(attrs, [:name, :tag, :description])
    |> validate_required([:name, :tag])
    |> unique_constraint(:tag)  # This won't work for partial indices
  end
end
```

The migration creates a partial unique index that only considers non-deleted records:

```elixir
defmodule Migrations.CreateIdtags do
  use Ecto.Migration
  import Ecto.SoftDelete.Migration

  def change do
    create table(:idtags) do
      add :name, :string
      add :tag, :string
      add :description, :string

      timestamps()
      soft_delete_columns()  # Adds deleted_at column
    end

    # Only enforce uniqueness for non-deleted records
    create unique_index(:idtags, [:tag], where: "deleted_at IS NULL")
  end
end
```

### The Upsert Challenge

The problem arises when trying to use Ecto's `on_conflict` with partial unique indices:

```elixir
# This fails because Ecto doesn't know about the partial index predicate
%IdTag{}
|> IdTag.changeset(attrs)
|> Repo.insert(
  on_conflict: [
    set: [
      name: attrs[:name],
      description: attrs[:description]
    ]
  ],
  conflict_target: [:tag]  # Missing the WHERE clause!
)
```

With a little hint from [StackOverflow](https://stackoverflow.com/a/54169587) the generated SQL shows the issue - no `WHERE deleted_at IS NULL` predicate:

```sql
INSERT INTO "idtags" AS i0 ("tag", ...) VALUES (...)
ON CONFLICT ("tag") DO UPDATE SET "name" = $8, "description" = $9 RETURNING "id" [...]
```

### The Solution: Using unsafe_fragment

The key insight from the [Elixir forum discussion](https://elixirforum.com/t/how-to-upsert-on-unique-partial-index-postgres/28977/2) is to use Ecto's `{:unsafe_fragment, ...}` to include the index predicate directly in the conflict target:

```elixir
%IdTag{}
|> IdTag.changeset(attrs)
|> Repo.insert(
  on_conflict: [
    set: [
      name: attrs[:name],
      description: attrs[:description]
    ]
  ],
  conflict_target: {:unsafe_fragment, ~s<("tag") WHERE deleted_at IS NULL>}
)
```

This generates the correct SQL that matches our partial index:

```sql
INSERT INTO "idtags" AS i0 ("tag", ...) VALUES (...)
ON CONFLICT ("tag") WHERE deleted_at IS NULL
DO UPDATE SET "name" = $8, "description" = $9
RETURNING "id" [...]
```

**Note**: While `unsafe_fragment` solves this specific case, use it with caution as it bypasses Ecto's SQL generation safety measures.

### When to Use This Pattern

This approach is particularly useful when you need to:

- Maintain unique constraints across soft-deleted records
- Allow duplicate values for deleted records
- Use Ecto's upsert functionality with complex index conditions

### Alternative Approaches

If you frequently work with partial indices, consider:

1. Creating a custom upsert function that handles your specific partial index predicates
2. Using raw SQL for complex conflict resolution scenarios
3. Implementing application-level conflict resolution logic

Hope you found this helpful!

If you need help with your Elixir/Phoenix project, feel free to [contact me](mailto:office@tonnenpinguin.solutions).
