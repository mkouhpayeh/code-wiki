## Initialization

```
builder.Property(p=>p.Code)
    .IsRequired()
    .HasMaxLength(10);

nuilder.Property(p=>p.Code)
    .Metadata.SetAfterSaveBehavior(PropertySaveBehavior.Ignore);
```
