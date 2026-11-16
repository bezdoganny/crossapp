# CrossApp
Наскрізний проєкт з крос-платформного програмування.

Предметна область: Склад. Сутності: Product, StockBatch, Warehouse, Movement.
Призначення: облік залишків товарів по партіях.

## Запуск
    dotnet build
    dotnet run --project src/Cli

## Середовище
.NET SDK 10.0, Fedora Workstation

## Публікація

| RID       | Режим                | Розмір | Потрібен runtime |
|-----------|----------------------|--------|------------------|
| linux-x64 | self-contained       | 80  МБ | ні               |
| linux-x64 | framework-dependent  | 128 KБ | так (.NET 10)    |

Self-contained містить .NET Runtime і запускається без встановленого .NET, але каталог значно більший. Framework-dependent містить лише код застосунку, тому потребує встановленого runtime відповідної версії.

jjj