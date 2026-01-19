---
date: 2026-01-18
categories:
  - Programação
  - C#
---

# C# 10: referência

Sintaxe e padrões modernos do C# 10 (.NET 6+).

<!-- more -->

## Tipos básicos

```csharp
// Inteiros
byte b = 255;              // 0 a 255
short s = 32767;           // -32k a 32k
int i = 2147483647;        // -2bi a 2bi
long l = 9223372036854775807L;

// Decimais
float f = 3.14f;           // 7 dígitos
double d = 3.14159265359;  // 15-16 dígitos
decimal m = 19.99m;        // 28-29 dígitos (dinheiro)

// Outros
bool flag = true;
char c = 'A';
string texto = "Hello";

// Nullable
int? nullable = null;
string? nullableStr = null;
```

## Strings

```csharp
// Interpolação
string nome = "João";
string msg = $"Olá, {nome}!";

// Verbatim (escapa \ automaticamente)
string path = @"C:\Users\João";

// Raw string literals (C# 11+)
string json = """
    {
        "nome": "João",
        "idade": 30
    }
    """;

// Métodos úteis
str.ToUpper();
str.ToLower();
str.Trim();
str.Split(',');
str.Contains("texto");
str.StartsWith("pre");
str.Replace("old", "new");
string.IsNullOrEmpty(str);
string.IsNullOrWhiteSpace(str);
string.Join(", ", lista);
```

## File-scoped namespace

```csharp
// Antes (C# 9)
namespace MeuProjeto.Services
{
    public class UserService
    {
        // ...
    }
}

// C# 10+
namespace MeuProjeto.Services;

public class UserService
{
    // ...
}
```

## Global usings

```csharp
// GlobalUsings.cs (ou qualquer arquivo)
global using System;
global using System.Collections.Generic;
global using System.Linq;
global using System.Threading.Tasks;
global using MeuProjeto.Models;

// Agora disponível em todos os arquivos do projeto
```

!!! tip "Implicit usings"
    No .NET 6+, habilite `<ImplicitUsings>enable</ImplicitUsings>` no `.csproj` para usings automáticos.

## Records

```csharp
// Record class (referência, imutável por padrão)
public record Person(string Name, int Age);

// Uso
var pessoa = new Person("João", 30);
var outra = pessoa with { Age = 31 };  // cria cópia modificada

// Igualdade por valor
var p1 = new Person("João", 30);
var p2 = new Person("João", 30);
Console.WriteLine(p1 == p2);  // true

// Record struct (C# 10)
public readonly record struct Point(int X, int Y);

// Record com corpo
public record Person(string Name, int Age)
{
    public string Greeting => $"Olá, {Name}!";
}
```

## Classes e structs

```csharp
public class User
{
    // Propriedades
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public string? Email { get; set; }

    // Init-only (imutável após construção)
    public DateTime CreatedAt { get; init; }

    // Computed property
    public bool HasEmail => !string.IsNullOrEmpty(Email);

    // Construtor
    public User(int id, string name)
    {
        Id = id;
        Name = name;
        CreatedAt = DateTime.UtcNow;
    }

    // Métodos
    public void UpdateEmail(string email) => Email = email;
}

// Struct
public struct Coordinate
{
    public double Lat { get; init; }
    public double Lon { get; init; }
}

// Readonly struct (totalmente imutável)
public readonly struct Money
{
    public decimal Amount { get; }
    public string Currency { get; }

    public Money(decimal amount, string currency)
    {
        Amount = amount;
        Currency = currency;
    }
}
```

## Pattern matching

```csharp
// is pattern
if (obj is string s)
{
    Console.WriteLine(s.ToUpper());
}

// switch expression
string GetStatus(int code) => code switch
{
    200 => "OK",
    201 => "Created",
    400 => "Bad Request",
    404 => "Not Found",
    >= 500 and < 600 => "Server Error",
    _ => "Unknown"
};

// Property pattern
string GetDiscount(User user) => user switch
{
    { Age: < 18 } => "10%",
    { Age: >= 65 } => "15%",
    { IsPremium: true } => "20%",
    _ => "0%"
};

// Tuple pattern
string GetQuadrant(int x, int y) => (x, y) switch
{
    ( > 0, > 0) => "Q1",
    ( < 0, > 0) => "Q2",
    ( < 0, < 0) => "Q3",
    ( > 0, < 0) => "Q4",
    _ => "Origin or Axis"
};

// List patterns (C# 11+)
int[] numbers = { 1, 2, 3 };
if (numbers is [1, 2, 3])
{
    Console.WriteLine("Match!");
}

if (numbers is [var first, .., var last])
{
    Console.WriteLine($"First: {first}, Last: {last}");
}
```

## Null handling

```csharp
// Null-conditional
string? nome = pessoa?.Nome;
int? tamanho = lista?.Count;

// Null-coalescing
string valor = texto ?? "default";
lista ??= new List<string>();  // atribui se null

// Null-forgiving (quando você sabe que não é null)
string nome = pessoa!.Nome;

// Required (C# 11+)
public class User
{
    public required string Name { get; set; }
}
```

## Collections

```csharp
// List
var lista = new List<int> { 1, 2, 3 };
lista.Add(4);
lista.Remove(2);
lista.Contains(3);

// Dictionary
var dict = new Dictionary<string, int>
{
    ["um"] = 1,
    ["dois"] = 2
};
dict.TryGetValue("um", out var valor);

// HashSet
var set = new HashSet<string> { "a", "b" };
set.Add("c");
set.Contains("a");

// Array
int[] arr = { 1, 2, 3 };
int[] arr2 = new int[10];
var span = arr.AsSpan();

// Range e Index
int[] nums = { 0, 1, 2, 3, 4, 5 };
var last = nums[^1];       // 5 (último)
var slice = nums[1..4];    // { 1, 2, 3 }
var fromEnd = nums[^3..];  // { 3, 4, 5 }
```

## LINQ

```csharp
var users = new List<User>();

// Query syntax
var adults = from u in users
             where u.Age >= 18
             orderby u.Name
             select u;

// Method syntax (preferido)
var adults = users
    .Where(u => u.Age >= 18)
    .OrderBy(u => u.Name)
    .ToList();

// Métodos comuns
users.Where(u => u.Active);
users.Select(u => u.Name);
users.OrderBy(u => u.Name);
users.OrderByDescending(u => u.Age);
users.First();
users.FirstOrDefault();
users.Single();
users.SingleOrDefault();
users.Any(u => u.Admin);
users.All(u => u.Active);
users.Count();
users.Sum(u => u.Score);
users.Average(u => u.Age);
users.Max(u => u.Age);
users.Min(u => u.Age);
users.GroupBy(u => u.Department);
users.Distinct();
users.Take(10);
users.Skip(5);
users.ToList();
users.ToArray();
users.ToDictionary(u => u.Id);
```

## Async/Await

```csharp
// Método async
public async Task<User> GetUserAsync(int id)
{
    var response = await httpClient.GetAsync($"/users/{id}");
    response.EnsureSuccessStatusCode();
    return await response.Content.ReadFromJsonAsync<User>();
}

// Async void (apenas para event handlers)
private async void Button_Click(object sender, EventArgs e)
{
    await DoSomethingAsync();
}

// Task.WhenAll (paralelo)
var tasks = ids.Select(id => GetUserAsync(id));
var users = await Task.WhenAll(tasks);

// Task.WhenAny (primeiro a completar)
var first = await Task.WhenAny(task1, task2);

// ConfigureAwait (libraries)
await SomeAsync().ConfigureAwait(false);

// Cancelamento
public async Task DoWork(CancellationToken ct)
{
    while (!ct.IsCancellationRequested)
    {
        await Task.Delay(1000, ct);
    }
}

// ValueTask (performance)
public ValueTask<int> GetCachedValueAsync()
{
    if (_cache.TryGetValue(key, out var value))
        return ValueTask.FromResult(value);

    return new ValueTask<int>(FetchValueAsync());
}
```

## Lambdas e delegates

```csharp
// Lambda
Func<int, int> dobro = x => x * 2;
Func<int, int, int> soma = (a, b) => a + b;
Action<string> print = msg => Console.WriteLine(msg);

// Lambda com tipo explícito (C# 10)
var parse = (string s) => int.Parse(s);

// Lambda com atributos (C# 10)
var validate = [Required] (string s) => s.Length > 0;

// Delegate
public delegate void Callback(string message);
Callback cb = msg => Console.WriteLine(msg);

// Events
public event EventHandler<UserEventArgs>? UserCreated;
UserCreated?.Invoke(this, new UserEventArgs(user));
```

## Generics

```csharp
// Classe genérica
public class Repository<T> where T : class
{
    private readonly List<T> _items = new();

    public void Add(T item) => _items.Add(item);
    public T? GetById(int id) => _items.FirstOrDefault();
}

// Método genérico
public T Parse<T>(string json) where T : new()
{
    return JsonSerializer.Deserialize<T>(json) ?? new T();
}

// Constraints
where T : class          // tipo referência
where T : struct         // tipo valor
where T : new()          // construtor sem parâmetros
where T : BaseClass      // herda de
where T : IInterface     // implementa
where T : notnull        // não-nulo
```

## Interfaces

```csharp
public interface IRepository<T>
{
    Task<T?> GetByIdAsync(int id);
    Task<IEnumerable<T>> GetAllAsync();
    Task AddAsync(T entity);
    Task UpdateAsync(T entity);
    Task DeleteAsync(int id);
}

// Default interface methods (C# 8+)
public interface ILogger
{
    void Log(string message);

    void LogError(string message) => Log($"ERROR: {message}");
    void LogWarning(string message) => Log($"WARN: {message}");
}

// Implementação
public class UserRepository : IRepository<User>
{
    public async Task<User?> GetByIdAsync(int id)
    {
        // implementação
    }
    // ...
}
```

## Exception handling

```csharp
try
{
    var result = await ProcessAsync();
}
catch (HttpRequestException ex) when (ex.StatusCode == HttpStatusCode.NotFound)
{
    // 404 específico
}
catch (HttpRequestException ex)
{
    logger.LogError(ex, "HTTP error");
    throw;  // re-throw mantendo stack trace
}
catch (Exception ex)
{
    logger.LogError(ex, "Unexpected error");
    throw new ApplicationException("Failed to process", ex);
}
finally
{
    // sempre executa
}

// Throw expression
var user = await GetUserAsync(id) ?? throw new NotFoundException();
```

## Dependency Injection

```csharp
// Program.cs (.NET 6+)
var builder = WebApplication.CreateBuilder(args);

// Registrar serviços
builder.Services.AddScoped<IUserService, UserService>();
builder.Services.AddSingleton<ICacheService, CacheService>();
builder.Services.AddTransient<IEmailService, EmailService>();

// Com factory
builder.Services.AddScoped<IDbConnection>(sp =>
    new SqlConnection(builder.Configuration.GetConnectionString("Default")));

var app = builder.Build();

// Injeção via construtor
public class UserController
{
    private readonly IUserService _userService;

    public UserController(IUserService userService)
    {
        _userService = userService;
    }
}

// Primary constructor (C# 12+)
public class UserController(IUserService userService)
{
    public async Task<User> Get(int id) => await userService.GetAsync(id);
}
```

## Minimal API (.NET 6+)

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/", () => "Hello World!");

app.MapGet("/users/{id}", async (int id, IUserService service) =>
{
    var user = await service.GetByIdAsync(id);
    return user is not null ? Results.Ok(user) : Results.NotFound();
});

app.MapPost("/users", async (User user, IUserService service) =>
{
    await service.CreateAsync(user);
    return Results.Created($"/users/{user.Id}", user);
});

app.MapPut("/users/{id}", async (int id, User user, IUserService service) =>
{
    await service.UpdateAsync(id, user);
    return Results.NoContent();
});

app.MapDelete("/users/{id}", async (int id, IUserService service) =>
{
    await service.DeleteAsync(id);
    return Results.NoContent();
});

app.Run();
```

## Entity Framework Core

```csharp
// DbContext
public class AppDbContext : DbContext
{
    public DbSet<User> Users => Set<User>();
    public DbSet<Order> Orders => Set<Order>();

    public AppDbContext(DbContextOptions<AppDbContext> options)
        : base(options) { }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<User>(e =>
        {
            e.HasKey(u => u.Id);
            e.Property(u => u.Name).IsRequired().HasMaxLength(100);
            e.HasMany(u => u.Orders).WithOne(o => o.User);
        });
    }
}

// Queries
var users = await context.Users
    .Where(u => u.Active)
    .Include(u => u.Orders)
    .OrderBy(u => u.Name)
    .ToListAsync();

var user = await context.Users.FindAsync(id);

// Insert
context.Users.Add(new User { Name = "João" });
await context.SaveChangesAsync();

// Update
user.Name = "João Silva";
await context.SaveChangesAsync();

// Delete
context.Users.Remove(user);
await context.SaveChangesAsync();

// Raw SQL
var users = await context.Users
    .FromSqlRaw("SELECT * FROM Users WHERE Active = 1")
    .ToListAsync();
```

## Configuração

```csharp
// appsettings.json
{
    "ConnectionStrings": {
        "Default": "Server=localhost;Database=App;..."
    },
    "AppSettings": {
        "ApiKey": "xxx",
        "MaxRetries": 3
    }
}

// Classe de configuração
public class AppSettings
{
    public string ApiKey { get; set; } = "";
    public int MaxRetries { get; set; }
}

// Program.cs
builder.Services.Configure<AppSettings>(
    builder.Configuration.GetSection("AppSettings"));

// Uso
public class MyService
{
    private readonly AppSettings _settings;

    public MyService(IOptions<AppSettings> options)
    {
        _settings = options.Value;
    }
}

// Direto
var connectionString = builder.Configuration.GetConnectionString("Default");
var apiKey = builder.Configuration["AppSettings:ApiKey"];
```

## Testes (xUnit)

```csharp
public class UserServiceTests
{
    private readonly Mock<IUserRepository> _repoMock;
    private readonly UserService _service;

    public UserServiceTests()
    {
        _repoMock = new Mock<IUserRepository>();
        _service = new UserService(_repoMock.Object);
    }

    [Fact]
    public async Task GetById_ReturnsUser_WhenExists()
    {
        // Arrange
        var user = new User { Id = 1, Name = "João" };
        _repoMock.Setup(r => r.GetByIdAsync(1))
            .ReturnsAsync(user);

        // Act
        var result = await _service.GetByIdAsync(1);

        // Assert
        Assert.NotNull(result);
        Assert.Equal("João", result.Name);
    }

    [Theory]
    [InlineData(1, true)]
    [InlineData(0, false)]
    [InlineData(-1, false)]
    public void IsValidId_ReturnsExpected(int id, bool expected)
    {
        var result = _service.IsValidId(id);
        Assert.Equal(expected, result);
    }
}
```

## Links

- [C# Docs](https://learn.microsoft.com/en-us/dotnet/csharp/){:target="_blank"}
- [.NET API Browser](https://learn.microsoft.com/en-us/dotnet/api/){:target="_blank"}
- [C# Cheatsheet](https://github.com/milanm/csharp-cheatsheet){:target="_blank"}
