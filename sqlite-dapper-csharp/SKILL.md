---
name: sqlite-dapper-csharp
description: >-
  Определяет стандарты работы с локальной базой данных SQLite через микро-ORM Dapper в C# / .NET:
  настройку пула соединений, прагмы производительности (WAL), параметризованные запросы,
  транзакции и тестирование в памяти. Активируется при проектировании репозиториев данных
  и локальных баз данных на C#.
---

# Регламент работы с SQLite и Dapper (sqlite-dapper-csharp)

Данный навык устанавливает архитектурные шаблоны и практики взаимодействия с локальной базой данных SQLite с использованием микро-ORM Dapper в проектах на .NET.

## Ключевые требования

1. Все SQL-запросы обязаны быть параметризованными для исключения SQL-инъекций.
2. Для повышения скорости записи включать режим упреждающей записи WAL (`PRAGMA journal_mode = WAL`).
3. Пакетные операции вставки и обновления выполнять исключительно внутри транзакции `IDbTransaction`.
4. Соединения оборачивать в блок `await using` или `using` для своевременного возврата ресурсов.

---

## Зависимости проекта

В `.csproj` файле:

```xml
<ItemGroup>
  <PackageReference Include="Microsoft.Data.Sqlite" Version="9.0.0" />
  <PackageReference Include="Dapper" Version="2.1.35" />
</ItemGroup>
```

---

## Фабрика соединений и инициализация базы данных

```csharp
using System.Data;
using Microsoft.Data.Sqlite;
using Dapper;

public interface IDbConnectionFactory
{
    Task<IDbConnection> CreateConnectionAsync(CancellationToken cancellationToken = default);
}

public class SqliteConnectionFactory : IDbConnectionFactory
{
    private readonly string _connectionString;

    public SqliteConnectionFactory(string connectionString)
    {
        _connectionString = connectionString;
    }

    public async Task<IDbConnection> CreateConnectionAsync(CancellationToken cancellationToken = default)
    {
        var connection = new SqliteConnection(_connectionString);
        await connection.OpenAsync(cancellationToken);

        await connection.ExecuteAsync("PRAGMA journal_mode = WAL; PRAGMA synchronous = NORMAL;");
        return connection;
    }
}
```

---

## Шаблон репозитория сущностей

Пример реализации слоя доступа к данным для складской номенклатуры:

```csharp
using System.Data;
using Dapper;

public record ItemRecord(long Id, string Sku, string Name, int Quantity, decimal Price);

public class ItemRepository
{
    private readonly IDbConnectionFactory _factory;

    public ItemRepository(IDbConnectionFactory factory)
    {
        _factory = factory;
    }

    public async Task InitializeSchemaAsync()
    {
        using var connection = await _factory.CreateConnectionAsync();
        const string sql = """
            CREATE TABLE IF NOT EXISTS Items (
                Id INTEGER PRIMARY KEY AUTOINCREMENT,
                Sku TEXT NOT NULL UNIQUE,
                Name TEXT NOT NULL,
                Quantity INTEGER NOT NULL,
                Price DECIMAL NOT NULL
            );
            CREATE INDEX IF NOT EXISTS IX_Items_Sku ON Items (Sku);
            """;
        await connection.ExecuteAsync(sql);
    }

    public async Task<ItemRecord?> GetBySkuAsync(string sku)
    {
        using var connection = await _factory.CreateConnectionAsync();
        const string sql = "SELECT Id, Sku, Name, Quantity, Price FROM Items WHERE Sku = @Sku LIMIT 1;";
        return await connection.QuerySingleOrDefaultAsync<ItemRecord>(sql, new { Sku = sku });
    }

    public async Task InsertBatchAsync(IEnumerable<ItemRecord> items)
    {
        using var connection = await _factory.CreateConnectionAsync();
        using var transaction = connection.BeginTransaction();

        const string sql = """
            INSERT INTO Items (Sku, Name, Quantity, Price)
            VALUES (@Sku, @Name, @Quantity, @Price);
            """;

        await connection.ExecuteAsync(sql, items, transaction);
        transaction.Commit();
    }
}
```

---

## Тестирование с базой данных в памяти

Для модульных тестов используется подключение `Data Source=:memory:`, соединение удерживается открытым на протяжении выполнения теста:

```csharp
using Microsoft.Data.Sqlite;
using Xunit;

public class ItemRepositoryTests
{
    [Fact]
    public async Task Should_Insert_And_Retrieve_Item()
    {
        using var memoryConnection = new SqliteConnection("Data Source=:memory:");
        await memoryConnection.OpenAsync();

        var factory = new TestConnectionFactory(memoryConnection);
        var repo = new ItemRepository(factory);

        await repo.InitializeSchemaAsync();
        await repo.InsertBatchAsync([new ItemRecord(0, "SKU-001", "Sensor", 10, 25.50m)]);

        var result = await repo.GetBySkuAsync("SKU-001");

        Assert.NotNull(result);
        Assert.Equal("Sensor", result.Name);
    }
}

public class TestConnectionFactory : IDbConnectionFactory
{
    private readonly SqliteConnection _connection;

    public TestConnectionFactory(SqliteConnection connection)
    {
        _connection = connection;
    }

    public Task<IDbConnection> CreateConnectionAsync(CancellationToken cancellationToken = default)
    {
        return Task.FromResult<IDbConnection>(_connection);
    }
}
```

---

## Чек-лист проверки

- Все запросы используют строго параметризованный синтаксис через анонимные типы Dapper.
- Нет конкатенации строк при формировании SQL.
- В коде отсутствуют комментарии.
- Пакетные вставки упакованы в транзакции.
- Приложение протестировано на сценарии параллельного чтения и записи.
