# Deploy to IIS

## .Net Core - Virtual App

``` csharp title="program.cs"
 var app = builder.Build();

app.UsePathBase("/Vitual Path");
```

``` csharp title="_Host.cshtml"
<base href="~/" />

<link rel="stylesheet" href="_content/Radzen.Blazor/css/standard-base.css">
<link rel="stylesheet" href="~/css/bootstrap.min.css" />
<link rel="stylesheet" href="~/css/bootstrap-icons.min.css" />
<link rel="stylesheet" href="~/css/site.css" />
<link rel="icon" type="image/png" href="favicon.png" />

<script src="~/js/site.js"></script>
```


``` csharp title="css"
@font-face {
    font-family: 'Poppins';
    src: url('./fonts/Poppins-Regular.ttf') format('truetype');
    font-weight: normal;
    font-style: normal;
}
```
