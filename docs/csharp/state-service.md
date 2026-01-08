# Local State Service

- This code describes a simple local state management pattern using Browser Local Storage combined with a Blazor scoped service.
- The goal of this approach is to:
    - Preserve user state across page reloads
    - Avoid unnecessary API calls
    - Improve user experience during multi-step flows
    - Centralize temporary session-related data
    - **This pattern is especially useful for practice sessions, wizards, and step-based workflows.**
  
---

1. Add JavaScript Helper
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

2. Add State Service Model
``` cs title="PracticeStateService.cs"
public sealed class PracticeStateService
{
    private const string StorageKey = "PracticeSession";
    private readonly IJSRuntime _jsRuntime;

    public long UserId { get; set; }
    public long ScopeId { get; set; }
    public string ScopeName { get; set; }
    public long LevelId { get; set; }
    public int PracticeQuestionCount { get; set; }
    public int ReviewQuestionCount { get; set; }
    public bool IsNew { get; set; } = false;
    public UserReviewType UserReviewType { get; set; } = UserReviewType.Normal;

    public bool HasData => UserId > 0 && ScopeId > 0 && LevelId > 0 && PracticeQuestionCount > 0 && ReviewQuestionCount > 0;

    public PracticeStateService(IJSRuntime jsRuntime)
    {
        _jsRuntime = jsRuntime;
    }

    public async Task SaveStateAsync(long userId, long scopeId, string scopeName, long levelId, int practiceQuestionCount, int reviewQuestionCount, bool isNew, UserReviewType userReviewType = UserReviewType.Normal)
    {
        UserId = userId;
        ScopeId = scopeId;
        ScopeName = scopeName;
        LevelId = levelId;
        PracticeQuestionCount = practiceQuestionCount;
        ReviewQuestionCount = reviewQuestionCount;
        IsNew = isNew;
        UserReviewType = userReviewType;

        PracticeRequestModel sessionData = new PracticeRequestModel
        {
            UserId = userId,
            LevelId = levelId,
            PracticeQuestionCount = practiceQuestionCount,
            ReviewQuestionCount = reviewQuestionCount,
            IsNew = isNew,
            ScopeId = scopeId,
            ScopeName = scopeName,
            UserReviewType = userReviewType
        };

        await _jsRuntime.InvokeVoidAsync("localStorageHelper.setItem", StorageKey, sessionData);
    }

    public async Task LoadStateAsync()
    {
        var sessionData = await _jsRuntime.InvokeAsync<PracticeRequestModel>("localStorageHelper.getItem", StorageKey);
        if (sessionData != null)
        {
            UserId = sessionData.UserId;
            IsNew = sessionData.IsNew;
            LevelId = sessionData.LevelId;
            PracticeQuestionCount = sessionData.PracticeQuestionCount;
            ReviewQuestionCount = sessionData.ReviewQuestionCount;
            ScopeId = sessionData.ScopeId;
            ScopeName = sessionData.ScopeName;
            UserReviewType = sessionData.UserReviewType;
        }
    }

    public async Task ClearStateAsync()
    {
        UserId = 0;
        IsNew = false;
        LevelId = 0;
        PracticeQuestionCount = 0;
        ReviewQuestionCount = 0;
        ScopeId = 0;
        ScopeName = string.Empty;
        UserReviewType = UserReviewType.Normal;

        await _jsRuntime.InvokeVoidAsync("localStorageHelper.removeItem", StorageKey);
    }
}
```

---

3. Add DI
``` cs title="Program.cs"
builder.Services.AddScoped<PracticeStateService>();
```
