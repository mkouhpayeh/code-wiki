# Local State Service

- A lightweight local state management pattern for Blazor applications using Browser Local Storage and a scoped service. This approach is suitable for short-lived, client-side state that must survive page reloads without introducing server dependencies.
- The goal of this approach is to:
    - Preserve user state across page reloads
    - Avoid unnecessary API calls
    - Improve user experience during multi-step flows
    - Support wizards, practice sessions, or step-based workflows
    - **This pattern is especially useful for practice sessions, wizards, and step-based workflows.**
      
    > **This is not a replacement for server-side persistence or distributed caching.**
  
---

## Definition

Step 1: JavaScript Local Storage Helper
``` js title="site.js"
window.localStorageHelper = {
    setItem: function (key, value) {
        localStorage.setItem(key, JSON.stringify(value));
    },
    getItem: function (key) {
        let value = localStorage.getItem(key);
        return value ? JSON.parse(value) : null;
    },
    removeItem: function (key) {
        localStorage.removeItem(key);
    }
};
```

---

Step 2: State Service Implementation
``` cs title="PracticeStateService.cs"
public sealed class PracticeStateService
{
    private const string StorageKey = "PracticeSession";

    private readonly IJSRuntime _jsRuntime;
    private readonly ILogger<PracticeStateService> _logger;


    public long UserId { get; private set; }
    public long ScopeId { get; private set; }
    public string ScopeName { get; private set; } = string.Empty;
    public long LevelId { get; private set; }
    public int PracticeQuestionCount { get; private set; }
    public int ReviewQuestionCount { get; private set; }
    public bool IsNew { get; private set; }
    public UserReviewType UserReviewType { get; private set; } = UserReviewType.Normal;

    public bool HasData =>
        UserId > 0 &&
        ScopeId > 0 &&
        LevelId > 0 &&
        PracticeQuestionCount > 0 &&
        ReviewQuestionCount > 0;

     public PracticeStateService(
        IJSRuntime jsRuntime,
        ILogger<PracticeStateService> logger)
    {
        _jsRuntime = jsRuntime;
        _logger = logger;
    }

    public async Task SaveAsync(PracticeRequestModel model)
    {
        try
        {
            Apply(model);

            await _jsRuntime.InvokeVoidAsync(
                "localStorageHelper.setItem",
                StorageKey,
                model
            );
        }
        catch (Exception ex)
        {
            _logger.LogWarning(
                ex,
                "Failed to save practice state to local storage"
            );
        }
    }

    public async Task LoadAsync()
    {
        try
        {
            var model = await _jsRuntime.InvokeAsync<PracticeRequestModel>(
                "localStorageHelper.getItem",
                StorageKey
            );

            if (model is not null)
            {
                Apply(model);
            }
        }
        catch (Exception ex)
        {
            _logger.LogWarning(
                ex,
                "Failed to load practice state from local storage"
            );

            Reset();
        }
    }

    public async Task ClearAsync()
    {
        try
        {
            Reset();

            await _jsRuntime.InvokeVoidAsync(
                "localStorageHelper.removeItem",
                StorageKey
            );
        }
        catch (Exception ex)
        {
            _logger.LogWarning(
                ex,
                "Failed to clear practice state from local storage"
            );

            Reset();
        }
    }

    private void Apply(PracticeRequestModel model)
    {
        UserId = model.UserId;
        ScopeId = model.ScopeId;
        ScopeName = model.ScopeName;
        LevelId = model.LevelId;
        PracticeQuestionCount = model.PracticeQuestionCount;
        ReviewQuestionCount = model.ReviewQuestionCount;
        IsNew = model.IsNew;
        UserReviewType = model.UserReviewType;
    }

    private void Reset()
    {
        UserId = 0;
        ScopeId = 0;
        ScopeName = string.Empty;
        LevelId = 0;
        PracticeQuestionCount = 0;
        ReviewQuestionCount = 0;
        IsNew = false;
        UserReviewType = UserReviewType.Normal;
    }
}
```

---

Step 3: Dependency Injection
``` cs title="Program.cs"
builder.Services.AddScoped<PracticeStateService>();
```

---

## Usage

``` cs
@inject PracticeStateService State

protected override async Task OnInitializedAsync()
{
    await State.LoadAsync();

    if (!State.HasData)
    {
        await State.SaveAsync(new PracticeRequestModel
        {
            UserId = userId,
            ScopeId = scopeId,
            ScopeName = scopeName,
            LevelId = levelId,
            PracticeQuestionCount = 10,
            ReviewQuestionCount = 5,
            IsNew = true,
            UserReviewType = UserReviewType.Normal
        });
    }
}
```
