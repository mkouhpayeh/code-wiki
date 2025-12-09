# Patterns

## Declaration Pattern

### Pattern Matching
Allows to check both the type of ano object and also declare a vriable of that type in a single statement.

```
if (product != null && product is Bio) // ❌
{
    var p = (Bio)product;
    p.Price += 10;
}

if (product is Bio p) // ✔️
{
    p.Price += 10;
}
```
