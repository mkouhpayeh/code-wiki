# Loading Page

## Enable Overlay
``` js title="site-overlay.js"
$(function () {
    var $overlay = $("#loadingOverlay");

    function showOverlay() {
        $overlay.show();
    }

    function shouldIgnoreOverlay(element) {
        return $(element).closest("[data-no-overlay='true']").length > 0;
    }

    // Show loader on all form submits (e.g., Login, filters, etc.)
    $(document).on("submit", "form", function (e) {
        if (shouldIgnoreOverlay(this)) return;

        if ($.validator && $(this).data("unobtrusiveValidation")) {
            if (!$(this).valid()) return;
        }

        showOverlay();
    });

    // Show loader on anchor clicks (except JS, tabs, new tabs, etc.)
    $(document).on("click", "a", function (e) {
        var href = $(this).attr("href");

        if (shouldIgnoreOverlay(this)) return;

        if (
            !href || href === "#" || href.startsWith("javascript:") ||
            $(this).attr("target") === "_blank" ||
            $(this).attr("data-toggle") === "tab"
        ) return;

        showOverlay();
    });

    // Show loader on dropdown changes (e.g., Arbeitgeber-Auswahl)
    $(document).on("change", "select", function () {
        if (shouldIgnoreOverlay(this)) return;

        showOverlay();
    });

    // Show/hide loader during AJAX calls (optional)
    $(document).ajaxStart(function () {
        showOverlay();
    });

    $(document).ajaxStop(function () {
        $overlay.hide();
    });
});
```

``` html title="_LoadingOverlay.cshtml"
<div id="loadingOverlay" style="display:none; position: fixed; width: 100%; height: 100%; top:0; left:0; background-color:#666; z-index: 9999; opacity:.60; filter:alpha(opacity=50);">
    <div style="text-align: center; color:#fff; padding-top:200px;">
        <img src="~/images/Spinner-1s-200px.gif" alt="Loading..." /><br />
        <span style="font-weight: bold; font-size: 26px;">
            Loading ...<br />
        </span>
    </div>
</div>
```

``` html title="Layout.cshtml"
@Html.Partial("_LoadingOverlay")
```

## Disable Overlay
Use 'data-no-overlay="true"' on any parent element to prevent overlay on specific forms or buttons.

```
@using (Html.BeginForm("DownloadDokument", "Item", FormMethod.Post, new { @class = "d-inline", data_no_overlay = "true" }))
{
    @Html.AntiForgeryToken()
    @Html.Hidden("Id", Model.Id)
    <button type="submit" class="btn btn-info btn-sm">
        <span data-feather="download"></span> Download
    </button>
}
```

```
<a href="/Home/Index" data-no-overlay="true">Go Home</a>
```

```
<button type="submit" data-no-overlay="true">Submit without overlay</button>
```

```
<select name="ArbeitgeberId" data-no-overlay="true" onchange="this.form.submit();">
    <option value="1">Firma A</option>
</select>
```
