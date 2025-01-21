## Update to Bootstrap 5

- Remove null reference error with adding this reference to each **Layout**
```csharp
@Scripts.Render("~/bundles/bootstrap")
```

### Bundle changes
- "~/Scripts/umd/popper.min.js" is removed and included in "bootstrap.bundle.min.js".
- "~/Scripts/bootstrap.min.js" is removed and as "bootstrap.bundle.min.js" already includes Bootstrap's JS.
    
```csharp title="BundleConfig.cs"
bundles.Add(new ScriptBundle("~/bundles/bootstrap").Include(
   `"~/Scripts/bootsrap.bundle.min.js",`
   `"~/Scripts/jquery.dataTables.min.js",`
    "~/Scripts/jquery.unobtrusive-ajax.min.js",
    "~/Scripts/moment.js",
    "~/Scripts/datetime.js",
    //"~/Scripts/dataTables.bootstrap4.min.js",
    "~/Scripts/jszip.min.js",
    "~/Scripts/dataTables.buttons.min.js",
    "~/Scripts/buttons.bootstrap4.min.js",
    "~/Scripts/buttons.html5.min.js",
    "~/Scripts/pdfmake.min.js",
    "~/Scripts/vfs_fonts.js"
    ));

// CSS bundle
bundles.Add(new StyleBundle("~/Content/css").Include(
   "~/Content/reset.css",
   "~/Content/bootstrap.min.css",
   "~/Content/jquery.dataTables.min.css",
   "~/Content/buttons.dataTables.min.css",
   "~/Content/font-awesome.min.css",
   "~/Content/Site.css"
));
```

### Tab view changes
- Replace data-toggle with data-bs-toggle: Bootstrap 5 uses "data-bs-toggle" instead of "data-toggle".
  
```html
<ul class="nav nav-tabs">
    <li class="nav-item"><a class="nav-link active" style="font-size:12px;" data-bs-toggle="tab" href="#persTab">Personaler</a></li>
    <li class="nav-item"><a class="nav-link" style="font-size:12px;" data-bs-toggle="tab" href="#fbTab">Firmenbetreuer</a></li>
    <li class="nav-item"><a class="nav-link" style="font-size:12px;" data-bs-toggle="tab" href="#adTab">Admins</a></li>
</ul>
```

```csharp
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
```
