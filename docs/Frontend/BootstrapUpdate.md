## Update to Bootstrap 5

### Bundle changes
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
```
