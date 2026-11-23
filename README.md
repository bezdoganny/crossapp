# CrossApp

Наскрізний проєкт з крос-платформного програмування (.NET 10, C#).
Предметна область: **Склад**. Сутності: `Product`, `StockBatch`, `Warehouse`, `Movement`.
Призначення: облік залишків товарів по партіях.

Проєкт розширюється протягом семестру: тижні 3–5 додають дані та сховища, 10 — проєкт `Api`, 12 — Blazor-клієнт. Усі клієнти використовують один `Core`.

## Структура

```
CrossApp/
├── CrossApp.sln
├── README.md
├── .gitignore
├── data/
│   └── sample.csv              # вхідні дані: 10 коректних і 3 пошкоджені рядки
└── src/
    ├── Core/                   # бібліотека: уся логіка без консолі та файлів
    │   ├── EnvironmentInfo.cs  # інформація про середовище виконання
    │   ├── Dto/                # формати даних (records)
    │   │   ├── ProductDto.cs
    │   │   └── ImportResult.cs
    │   ├── Import/
    │   │   └── ProductCsvImporter.cs  # розбір CSV з накопиченням помилок
    │   └── Domain/
    │       └── Product.cs      # сутність з інваріантами
    └── Cli/                    # консольний застосунок
        └── Program.cs
```

Залежності: `Cli → Core`. Зворотного посилання немає, інакше виникає циклічна залежність.

## Вимоги

- .NET SDK 10.0 (`dotnet --list-sdks` має показувати `10.x`)
- Git

## Запуск

```bash
dotnet build
dotnet run --project src/Cli                    # sample.csv за замовчуванням
dotnet run --project src/Cli -- data/інший.csv  # будь-який файл
```

Імпорт читає файл у кодуванні UTF-8, роздільник `;`, перший рядок може бути заголовком `id;sku;name;unit;quantity`.

## Публікація

```bash
# self-contained: містить .NET Runtime, запускається без встановленого .NET
dotnet publish src/Cli -c Release -r linux-x64 --self-contained true -o publish/self-contained
./publish/self-contained/Cli

# framework-dependent: лише код застосунку, потрібен встановлений runtime
dotnet publish src/Cli -c Release -r linux-x64 --self-contained false -o publish/framework-dependent
dotnet ./publish/framework-dependent/Cli.dll
```

RID визначається вашою ОС і архітектурою (`linux-x64`, `win-x64`, `osx-arm64`). Перевірити можна у виводі `dotnet --info`.

| RID       | Режим               | Розмір publish | Потрібен runtime |
|-----------|---------------------|----------------|------------------|
| linux-x64 | self-contained      | 80  МБ         | ні               |
| linux-x64 | framework-dependent | 128 KБ         | так (.NET 10)    |

Розміри виміряно командою `du -sh publish/*`.

## Модулі

### Core/EnvironmentInfo (лабораторна 2)
Метод `EnvironmentInfo.Collect()` повертає record `EnvironmentReport` з даними про ОС, runtime, архітектуру, RID і каталог застосунку. Нічого не друкує, друк робить Cli.

### Core/Dto (лабораторна 3)
Records без поведінки, лише формати даних:
- `ProductDto(Id, Sku, Name, Unit, Quantity)`
- `ImportResult<T>(Items, Errors)`: успішні записи разом з переліком помилок.

### Core/Import (лабораторна 3)
`ProductCsvImporter.Load(path)` читає файл і розбирає кожен рядок через switch expression з патернами. Пошкоджені рядки не зупиняють імпорт: вони потрапляють у `Errors` з номером рядка. Числа розбираються з `CultureInfo.InvariantCulture`.

### Core/Domain (лабораторна 4)
`Product` захищає свої правила:
- Створення лише через `Product.Create(...)`, конструктор приватний.
- Залишок змінюється тільки методами `RegisterArrival` і `Issue`.
- `ToDto()` і `FromDto(dto)` пов'язують сутність з `ProductDto`. `FromDto` викликає `Create`, тож дані з файлу проходять ті самі перевірки.

## Інваріанти Product

| Правило | Тип винятку | Метод |
|---|---|---|
| Id, SKU і назва не порожні | `ArgumentException` | `Create`, `FromDto` |
| Початковий залишок не від'ємний | `ArgumentOutOfRangeException` | `Create`, `FromDto` |
| Кількість приходу більша за нуль | `ArgumentOutOfRangeException` | `RegisterArrival` |
| Кількість видачі більша за нуль | `ArgumentOutOfRangeException` | `Issue` |
| Видача не перевищує залишок | `InvalidOperationException` | `Issue` |

Тип винятку відповідає причині: `Argument*` для некоректного вхідного аргументу, `InvalidOperationException` для операції, яку неможливо виконати в поточному стані.

## Приклад виводу

```
=== Сценарій 1: успіх ===
P-001 [SKU-001] Цемент М400 25кг — 100 шт
P-001 [SKU-001] Цемент М400 25кг — 120 шт
Відновлено з DTO: P-001 [SKU-001] Цемент М400 25кг — 120 шт

=== Сценарій 2: порушення інваріантів ===
  видача більша за залишок: InvalidOperationException — Не можна видати 1000: залишок SKU-001 = 120
  порожній SKU: ArgumentException — SKU не може бути порожнім (Parameter 'sku')
```

## Плани розвитку

| Тиждень | Що додасться |
|---|---|
| 5 | Файлове сховище `FileCatalogStore` на основі DTO |
| 6 | Доменні події про зміну залишку |
| 7 | LINQ-звіти |
| 8 | Асинхронне читання та unit-тести |
| 10 | Проєкт `Api` |
| 11 | Таблиці бази даних (EF Core) |
| 12 | Blazor-клієнт |

## Ліцензія та автор

Навчальний проєкт. Автор: Давид В.М., ФЕІ-37.
