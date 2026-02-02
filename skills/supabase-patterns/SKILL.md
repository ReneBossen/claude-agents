# Supabase Patterns

## Authentication

- **Handled by Supabase Auth** - No custom JWT implementation
- Users authenticate via Supabase Auth (email/password, OAuth providers)
- Mobile/web apps use Supabase client libraries for authentication
- Backend validates Supabase JWT tokens via middleware
- Extract user ID from validated token claims using `auth.uid()`

## Row-Level Security (RLS)

### Always Enable RLS

```sql
ALTER TABLE {table_name} ENABLE ROW LEVEL SECURITY;
```

### Common Patterns

**User owns the row:**
```sql
CREATE POLICY "Users can manage own data"
    ON {table} FOR ALL
    USING (auth.uid() = user_id)
    WITH CHECK (auth.uid() = user_id);
```

**Read own, insert own:**
```sql
CREATE POLICY "Users can view own data"
    ON {table} FOR SELECT
    USING (auth.uid() = user_id);

CREATE POLICY "Users can insert own data"
    ON {table} FOR INSERT
    WITH CHECK (auth.uid() = user_id);
```

**Friend-based access:**
```sql
CREATE POLICY "Users can view friends data"
    ON {table} FOR SELECT
    USING (
        user_id = auth.uid()
        OR user_id IN (
            SELECT friend_id FROM friendships
            WHERE user_id = auth.uid() AND status = 'accepted'
        )
    );
```

## Schema Conventions

### Naming
| Element | Convention | Example |
|---------|------------|---------|
| Tables | snake_case, plural | `step_entries`, `group_memberships` |
| Columns | snake_case | `user_id`, `created_at` |
| Primary Keys | `id` (UUID) | `id UUID PRIMARY KEY` |
| Foreign Keys | `{referenced_table}_id` | `user_id`, `group_id` |
| Indexes | `idx_{table}_{column(s)}` | `idx_steps_user_id` |

### Standard Columns

Every table should include:
```sql
id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
created_at TIMESTAMPTZ DEFAULT NOW(),
updated_at TIMESTAMPTZ DEFAULT NOW()
```

### Data Types
| Use Case | Type |
|----------|------|
| Primary keys | `UUID DEFAULT gen_random_uuid()` |
| Foreign keys | `UUID REFERENCES {table}(id)` |
| Timestamps | `TIMESTAMPTZ DEFAULT NOW()` |
| Boolean flags | `BOOLEAN DEFAULT false` |
| JSON data | `JSONB DEFAULT '{}'` |

### Foreign Key Constraints

Always specify ON DELETE behavior:
```sql
-- Delete with parent
user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE

-- Nullify on delete
related_user_id UUID REFERENCES auth.users(id) ON DELETE SET NULL

-- Prevent delete
category_id UUID NOT NULL REFERENCES categories(id) ON DELETE RESTRICT
```

## Repository Pattern with Supabase

### Entity Separation

**Domain Model** (clean, no Supabase attributes):
```csharp
public class Group
{
    public Guid Id { get; set; }
    public string Name { get; set; }
    public int MemberCount { get; set; }  // Calculated
}
```

**Entity Class** (with Supabase mapping):
```csharp
[Table("groups")]
internal class GroupEntity : BaseModel
{
    [PrimaryKey("id")]
    public Guid Id { get; set; }

    [Column("name")]
    public string Name { get; set; }

    public Group ToGroup(int memberCount) => new()
    {
        Id = Id,
        Name = Name,
        MemberCount = memberCount
    };
}
```

## Security Rules

- Store credentials in environment variables, never hardcode
- Trust RLS policies for authorization
- Use `SECURITY DEFINER` sparingly with explicit `search_path`
- Never hardcode user IDs in policies (use `auth.uid()`)
