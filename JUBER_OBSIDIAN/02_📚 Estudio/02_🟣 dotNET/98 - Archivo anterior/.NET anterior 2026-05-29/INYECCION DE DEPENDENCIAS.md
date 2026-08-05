# Inyección de Dependencias en C# .NET

![[1762178838750.jpg|400]]
## 🧩 ¿Qué es?

Técnica que permite que una clase reciba sus dependencias desde afuera, en lugar de crearlas internamente. Generalmente se hace a través del constructor.

**Principios que cumple:**
- **D de SOLID** → Dependency Inversion (depender de abstracciones, no implementaciones)
- **Desacoplamiento** → Las clases no conocen implementaciones concretas
- **Testabilidad** → Fácil inyectar mocks en tests

---
## 🧠 Ejemplo Práctico

### Sin inyección de dependencias (acoplado)
```csharp
public class UserRegistrationService
{
    private readonly EmailService _emailService = new EmailService();
    private readonly UserRepository _userRepository = new UserRepository();
    
    public void Register(User user)
    {
        _userRepository.Save(user);
        _emailService.SendWelcome(user.Email);
    }
}
```

> [!warning] Problemas
> - Crea sus propias dependencias (acoplamiento fuerte)
> - Imposible testear sin enviar emails reales
> - Cambiar implementación requiere modificar la clase

### Con inyección de dependencias (desacoplado)
```csharp
public class UserRegistrationService
{
    private readonly IEmailService _emailService;
    private readonly IUserRepository _userRepository;
    private readonly ILogger<UserRegistrationService> _logger;

    public UserRegistrationService(
        IEmailService emailService,
        IUserRepository userRepository,
        ILogger<UserRegistrationService> logger)
    {
        _emailService = emailService;
        _userRepository = userRepository;
        _logger = logger;
    }

    public void Register(User user)
    {
        _userRepository.Save(user);
        _emailService.SendWelcome(user.Email);
        _logger.LogInformation("Usuario {Email} registrado", user.Email);
    }
}
```

> [!success] Beneficios
> - Depende de interfaces, no implementaciones
> - Testeable con mocks
> - Cambiar implementación solo requiere cambiar el registro

---
## 🔧 Registro en Program.cs
```csharp
builder.Services.AddScoped<IUserRepository, UserRepository>();
builder.Services.AddScoped<IEmailService, EmailService>();
builder.Services.AddScoped<UserRegistrationService>();
```

---
## 🔄 Ciclos de Vida (Lifetimes)

| Tipo | Creación | Duración | Destrucción |
|------|----------|----------|-------------|
| **Singleton** | Al iniciar la app | Toda la ejecución | Al cerrar la app |
| **Scoped** | Al iniciar un request HTTP | Durante el request | Al finalizar el request |
| **Transient** | Cada vez que se inyecta | Uso inmediato | Justo después de usarse |
### 🧬 Singleton
```csharp
builder.Services.AddSingleton<ICache, MemoryCache>();
```
- Una sola instancia para toda la aplicación
- **Usar para:** Configuraciones, cachés, servicios stateless compartidos
- **Cuidado:** No guardar estado específico de request
### 🧬 Scoped
```csharp
builder.Services.AddScoped<IUserRepository, UserRepository>();
```
- Una instancia por request HTTP
- **Usar para:** DbContext, repositorios, Unit of Work
- **Más común** en aplicaciones web
### 🧬 Transient
```csharp
builder.Services.AddTransient<IEmailService, EmailService>();
```
- Nueva instancia cada vez que se solicita
- **Usar para:** Servicios livianos y stateless
- **Cuidado:** Puede afectar performance si es costoso crear la instancia

---
## ⚠️ Errores Comunes

> [!danger] Captive Dependency
> Un Singleton que depende de un Scoped o Transient captura una instancia que debería ser efímera.
```csharp
// ❌ MAL: Singleton depende de Scoped
public class MySingleton
{
    private readonly IScopedService _scoped; // Problema!
}
```

**Regla:** Un servicio solo puede depender de servicios con igual o mayor tiempo de vida.
```
Transient → puede depender de → Transient, Scoped, Singleton
Scoped    → puede depender de → Scoped, Singleton
Singleton → puede depender de → Singleton (únicamente)
```

---
## 🎯 Resumen

> [!tip] Cuándo usar cada lifetime
> - **Singleton** → Servicios compartidos, configuración, caché
> - **Scoped** → Repositorios, DbContext, servicios por request
> - **Transient** → Servicios livianos sin estado

> [!note] Recuerda
> La inyección de dependencias no es solo una técnica, es la base para escribir código testeable, mantenible y desacoplado en .NET.