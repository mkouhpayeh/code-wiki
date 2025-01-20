## Validation Attributes

### RequiredIf
```csharp title="RequiredIf"
[RequiredIf("Art == 'Gutschrift'",
    ErrorMessageResourceName = "SelectChargeLabel",
    ErrorMessageResourceType = typeof(Resources.ObjectText))]  
public string GutschriftWann { get; set; } = string.Empty;
```

### AssertThat
```csharp title="AssertThat"
public class MyModel
{
    // This method must be accessible (public or internal) to be used in AssertThat
    public bool IsValidArt(string art)
    {
        // Custom validation logic
        return !string.IsNullOrWhiteSpace(art) && art.Length > 5;
    }

    [AssertThat("IsValidArt(Art)", 
        ErrorMessageResourceName = "InvalidArtMessage", 
        ErrorMessageResourceType = typeof(Resources.ObjectText))]
    public string Art { get; set; } = string.Empty;
}
```
