# Dependency Injection

## Dependency Injection Lifetimes
ASP.NET Core provides three main lifetimes for services registered in the Dependency Injection (DI) container.  
Each lifetime defines **how long an instance of a service lives** and **how it is shared**.

1️⃣ Transient (New Instance Every Time) </br>
    - A **new instance is created every time** the service is requested
    - Suitable for:
        - Lightweight classes
        - Stateless services
    - ❌ Not suitable for database connections
    - Example registration:
    
        ``` cs
        services.AddTransient<IEmailService, EmailService>();
        ```

2️⃣ Scoped (One Instance Per Request) </br>
    - One instance is created **per HTTP request**    
    - The **same instance** is shared within that request
    - Suitable for:
        - Web requests
        - DbContext
        - Unit of Work pattern
    - ✅ **Recommended for business logic**
    - Example registration:
        ``` cs
        services.AddScoped<IOrderService, OrderService>();
        ```
        
3️⃣ Singleton (Single Instance for the Entire App) </br>
    - Only **one instance for the whole application lifetime**
    - Shared across all users and requests
    - Suitable for:
        - Caching
        - Configuration
        - Logging
    - ⚠️ Must be **thread-safe**
    - Example registration:
        ``` cs
        services.AddSingleton<ICacheService, CacheService>();
        ```

⚠️ **Golden Rule** </br>
A service with a longer lifetime must NOT depend on a service with a shorter lifetime. **(This can cause memory leaks or unpredictable behavior)** </br>
- ❌ Invalid dependencies:
    - Singleton → Scoped
    - Singleton → Transient

🧭 **Quick Guide** </br>
- Helper / Utility code → **Transient**
- Business Logic → **Scoped**
- Database Access (DbContext) → **Scoped**
- Cache / Configuration → **Singleton**
