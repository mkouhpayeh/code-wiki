# Error Handling

## ModelState
Use this when: 

- You're returning the same view after validation fails (e.g. form errors)
- You want to show inline validation errors next to form fields
- You want to use @Html.ValidationSummary() or @Html.ValidationMessageFor(...) in Razor
- Doesn't persist across redirects
- Works only for the current request

``` cs
if (!ModelState.IsValid)
    return View(model); // stays on same page

```

---

## TempData
Use this when:

- You need to pass error/success messages across redirects
- You're doing Post-Redirect-Get (PRG) pattern
- You want to show a status banner or alert box on the next view
- Persists for one request after redirect
- Uses cookies or session under the hood
- Works in .NET Core (via ITempDataDictionary)

``` cs
TempData["ErrorMessage"] = "Something went wrong.";
return RedirectToAction("List");
```

``` cs
@if (TempData["ErrorMessage"] != null)
{
    <div class="alert alert-danger">
        @TempData["ErrorMessage"]
    </div>
}

```
