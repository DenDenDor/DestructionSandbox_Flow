# Unity Project Workflow

[Назад в README](../../README.md)

## Оглавление

- [Структура папок](#структура-папок)
    - [Addressables](#addressables)
    - [Configs (Resources)](#configs-resources)
    - [Scenes](#scenes)
    - [Scripts](#scripts)
    - [ThirdParty](#thirdparty)
- [Модули и asmdef](#модули-и-asmdef)
    - [Расположение модулей](#расположение-модулей)
    - [Структура модуля](#структура-модуля)
    - [Правила Runtime и Editor кода](#правила-runtime-и-editor-кода)
    - [Структура папок внутри модуля](#структура-папок-внутри-модуля)
        - [Facade](#facade)
        - [Controllers](#controllers)
        - [Helpers](#helpers)
        - [Enums](#enums)
        - [Installer](#installer)
        - [Factories](#factories)
        - [Providers](#providers)
        - [Models](#models)
    - [asmdef и зависимости](#asmdef-и-зависимости)
    - [Границы модуля](#границы-модуля)
- [Dependency Injection](#dependency-injection)
    - [GlobalLifetimeScope](#globallifetimescope)
    - [Installer модуля](#installer-модуля)
    - [Зависимости и границы модулей в DI](#зависимости-и-границы-модулей-в-di)

## Структура папок

### Addressables

В проекте используется Addressables, поэтому все ассеты, которые попадают в билд или должны подгружаться в рантайме, размещаются внутри папки: 
`Assets/Addressables/`

Правила:
- Ассеты должны быть логически сгруппированы по папкам
- Запрещено хранить ассеты “россыпью” в корне `Addressables`
- Структура папок отражает домен/систему/контекст использования

Примеры:
- `Assets/Addressables/UI/Windows/GameWindow/`
- `Assets/Addressables/UI/Windows/InventoryWindow/`
- `Assets/Addressables/Game/VFX/`

---

### Configs (Resources)

Все конфиги проекта хранятся в `Assets/Resources/Configs/`

Нейминг:
- строчный
- разделитель — нижнее подчёркивание (snake_case)

Пример:
- `wallet_config.asset`
- `enemy_balance_config.asset`

Создание конфигов:
- для удобства конфиги создаются через CreateAssetMenu по пути `Game Data/...`

Пример:
```csharp
[CreateAssetMenu(menuName = "Game Data/Game flow/Wallet config",fileName = "wallet_config")]
public class WalletConfig : ScriptableObject { }
```

Правила:
- Папки в menuName отражают систему/подсистему

---

### Scenes

Все сцены лежат в: `Assets/Scenes/`

Правила:
- Сцены группируются по назначению (`Gameplay` / `UI` / `Test` и т.п.)
- Тестовые/временные сцены не должны попадать в релизный список

Примеры:
- `Assets/Scenes/Boot/Boot.unity`
- `Assets/Scenes/Gameplay/Level_01.unity`
- `Assets/Scenes/Test/UI_Playground.unity`

---

### Scripts

Весь код организован модульно и лежит в: `Assets/Scripts/Modules/`

Правила:
- Каждый модуль — отдельная папка

Пример:
- `Assets/Scripts/Modules/UiModule/`
- `Assets/Scripts/Modules/SaveModule/`
- `Assets/Scripts/Modules/BootstrapperModule/`
- `Assets/Scripts/Modules/AddressablesManagementModule/`

---

### ThirdParty

Сторонние пакеты и библиотеки (если они не приходят через `Package Manager`) хранятся в: `Assets/ThirdParty/`

Правила:
- Не смешивать `ThirdParty` код с вашим кодом
- Не править код `ThirdParty` без явной причины (если правим — фиксируем это отдельной задачей и документируем)

Примеры:
- `Assets/ThirdParty/TextMesh Pro/`

<hr style="border: 0; height: 6px; background: #2f2f2f; margin: 40px 0;">

## Модули и asmdef

В проекте используется модульная архитектура с разделением кода на `Runtime` и `Editor` через `Assembly Definition` (`asmdef`).
Каждый модуль изолирован, имеет явные зависимости и собственные сборки.

### Расположение модулей

Все модули проекта располагаются в папке: `Assets/Scripts/Modules/`

Каждый модуль представлен отдельной папкой `Assets/Scripts/Modules/<ModuleName>`.

Пример:
- `Assets/Scripts/Modules/UiModule/`
- `Assets/Scripts/Modules/VContainerModule/`
- `Assets/Scripts/Modules/GameModules/`

---

### Структура модуля

Внутри каждого модуля используется строгое разделение на `Runtime` и `Editor`.

Структура модуля:
````
Scripts/Modules/<ModuleName>/
    Runtime/
        <FolderName>/
            ...
        Scripts.Modules.<ModuleName>.Runtime.asmdef

    Editor/
        <FolderName>/
            ...
        Scripts.Modules.<ModuleName>.Editor.asmdef
````

Пример для модуля UI:
````csharp
Scripts/Modules/SaveModule/
    Runtime/
        Facade/
            SaveModuleFacade.cs            
        Scripts.Modules.SaveModule.Runtime.asmdef
            
    Editor/
        SaveModuleEditorUtils.cs
        Scripts.Modules.SaveModule.Editor.asmdef
````

---

### Правила `Runtime` и `Editor` кода

#### Runtime

Правила:
- Используется в билде
- Не должен содержать ссылок на `UnityEditor`
- Собирается сборкой `Scripts.Modules.<ModuleName>.Runtime`
- Весь код, который используется в игре, обязан находиться в `Runtime`-папке модуля.

---

#### Editor

Правила:
- Используется только в редакторе Unity. `Editor`-код никогда не должен использоваться в рантайме.
- Может использовать `UnityEditor`
- Может ссылаться на `Runtime` сборки
- Собирается сборкой `Scripts.Modules.<ModuleName>.Editor`

В `Editor`-папке размещаются:
- кастомные инспекторы
- property drawers
- editor windows
- меню-команды (MenuItem)
- инструменты и утилиты для разработки

---

### Структура папок внутри модуля

Каждый модуль имеет стандартную внутреннюю структуру папок. Структура одинаковая для `Runtime` и (при необходимости) для `Editor`, но ниже описан именно `Runtime` как основа.

Рекомендуемая структура:

````
Scripts.Modules.<ModuleName>.Runtime/
    Facade/
    Controllers/
    Factories/
    Helpers/
    Enums/
    Installer/
    Providers/
    Models/
````

---

#### Facade

Папка `Facade` содержит публичную точку входа в модуль.

Содержимое:
- `<ModuleName>Facade`
- `I<ModuleName>Facade`

Правила:
- Внешний код взаимодействует с модулем через `I<ModuleName>Facade`. Исключения составляют сервисные модули, которые сами по себе не выполняют какой-то логики и не содержат фасада, но реализуются в других модулях.
- Доступ к внутренним контроллерам, провайдерам и фабрикам из других модулей запрещён.
- `Facade` не содержит “тяжёлой” логики — он оркестрирует и делегирует в `Controllers`

Пример:
- `ISaveModuleFacade`, `SaveModuleFacade`
- `IScenariosModuleFacade`, `ScenariosModuleFacade`

---

#### Controllers

Папка `Controllers` содержит классы, реализующие конкретную логику.

Нейминг:
- По умолчанию: `<Logic>Controller`
- Допускается более специфичный нейминг, если он лучше отражает смысл

Правила:
- Один контроллер — одна область ответственности
- Контроллеры не должны содержать `Editor`-зависимостей 
- Контроллеры могут зависеть от `Providers`, `Factories`, `Models`

Примеры:
- `DebugMenuController`
- `IWalletItemController`, `WalletItemController`

`Controller` не занимаются долгосрочным хранением данных:
- `Controller` берёт данные из `UserDataProvider`
- выполняет логику/расчёты
- может иметь оперативное (runtime) состояние, относящееся к текущей сессии/контексту
- сохраняет изменённые данные обратно через `UserDataProvider`

---

#### Helpers

Папка `Helpers` содержит:
- Утилиты
- Расширения (extensions)
- EventArgs
- Мелкие вспомогательные структуры

Правила:
- `Helper`-код не должен “разрастаться” в бизнес-логику
- Если `Helper`-код начинает содержать поведение домена — выносится в `Controllers` или отдельную сущность

---

#### Enums

Папка `Enums` содержит перечисления модуля.

---

#### Installer

Папка `Installer` содержит:
- `<ModuleName>Installer`

Назначение:
- Регистрация зависимостей модуля в `DI-контейнере`

Правила:
- Все регистрации `Runtime`-модуля должны быть сосредоточены в `Installer`
- Модуль не должен “сам себя собирать” в рандомных местах — только через `Installer`

Пример:
- `WalletModuleInstaller`
- `GameObjectPoolModuleInstaller`

---

#### Factories

Папка `Factories` содержит фабрики создания объектов.

Нейминг:
- `<SomeType>Factory`
- `I<SomeType>Factory`

Правила:
- Если фабрика используется за пределами модуля — наружу экспортируется интерфейс `I<SomeType>Factory`, либо конкретный метод фабрики через фасад
- Фабрики не должны хранить состояние домена (для этого `Providers`)

Примеры:
- `IWalletItemFactory`, `WalletItemFactory`

---

#### Providers

Папка `Providers` содержит провайдеры данных модуля. В модуле различаются два основных класса данных:
- Данные модуля (Module Data / Config) — статические настройки и баланс, которые задаются дизайном и не меняются игроком в рантайме (или меняются только разработчиком).
- Пользовательские данные (User Data) — прогресс игрока и любые значения, которые игрок может изменить и которые должны сохраняться между сессиями.

Типы провайдеров:
- `<ModuleName>DataProvider` - Провайдер данных модуля / конфигов (ScriptableObject, конфиги, таблицы и т.п.)
- `<ModuleName>UserDataProvider` - Провайдер пользовательских данных, прогресса

Примеры пользовательских данных:
- очки, опыт, деньги
- экипировка, инвентарь
- открытые улучшения/перки
- настройки игрока, если они должны сохраняться

Правила:
- `DataProvider` хранит и выдаёт конфигурацию модуля (статические данные).
- `UserDataProvider` хранит и выдаёт персистентные данные игрока, а также отвечает за их сериализацию

---

#### Models

Папка `Models` содержит модели данных модуля:
- Структуры/классы данных
- DTO (Data Transfer Object)
- State models
- Domain models (в рамках модуля)

Правила:
- модели не содержат поведения, кроме какого-то специфического внутреннего преобразования данных
- модели должны быть сериализуемыми при необходимости (если участвуют в сохранениях/конфигах)

---

### asmdef и зависимости

Каждый модуль имеет две сборки:
- `Scripts.Modules.<ModuleName>.Runtime`
- `Scripts.Modules.<ModuleName>.Editor`

Правила:
- `Scripts.Modules.<ModuleName>.Editor` может ссылаться на `Scripts.Modules.<ModuleName>.Runtime`
- `Scripts.Modules.<ModuleName>.Runtime` не может ссылаться на editor-сборки
- Циклические зависимости между asmdef запрещены
- Зависимости между модулями добавляются явно через Assembly Definition References.

---

### Границы модуля

Модуль рассматривается как изолированная единица.

Правила:
- Модуль предоставляет наружу только осознанный API
- Прямые зависимости от конкретных реализаций другого модуля запрещены
- взаимодействие между модулями должно происходить через фасады (там, где они есть) / интерфейсы, которые не помечены, как internal
- Внутренняя реализация модуля не должна протекать за его пределы.

<hr style="border: 0; height: 6px; background: #2f2f2f; margin: 40px 0;">

## Dependency Injection

В проекте используется внедрение зависимостей через `DI-контейнер` на базе `VContainer`:
https://vcontainer.hadashikick.jp/

DI применяется для:
- Изоляции модулей и контролируемых зависимостей
- Упрощения тестирования/замены реализаций
- Прозрачной инициализации систем через контейнер

---

### GlobalLifetimeScope

Все регистрации модулей выполняются в единой точке — классе `GlobalLifetimeScope`.
`GlobalLifetimeScope` является composition root приложения.

Пример:

````csharp
protected override void Configure(IContainerBuilder builder)
{
    UnityEventsModuleInstaller.Bind(builder);
    ScenariosModuleInstaller.Bind(builder);
    WindowsModuleInstaller.Bind(builder);
    GameInitializationModuleInstaller.Bind(builder);
    AddressablesManagementModuleInstaller.Bind(builder);
    GameObjectPoolModuleInstaller.Bind(builder);
    InputSystemModuleInstaller.Bind(builder);
    SteamworksModuleInstaller.Bind(builder);
    SaveModuleInstaller.Bind(builder);
    UserProfileModuleInstaller.Bind(builder);
    GameFlowModuleInstaller.Bind(builder);
    WalletModuleInstaller.Bind(builder);
}
````

Правила:
- Все инсталлеры регистрируются только через `GlobalLifetimeScope`
- Модуль не должен самостоятельно “где-то” создавать/строить контейнер
- `GlobalLifetimeScope` — единственное место, где определяется состав приложения

---

### Installer модуля

Каждый модуль, если он зависит от других модулей или сам является зависимостью, обязан иметь `<ModuleName>Installer` в папке `Installer/`.
`Installer/` содержит единственную публичную точку регистрации — метод `Bind`.

Пример:
`````csharp
public static class SaveModuleInstaller
{

    public static void Bind(IContainerBuilder containerBuilder)
    {
        containerBuilder.Register<IDataMapStorage, DataMapStorage>(Lifetime.Singleton);
        containerBuilder.Register<ISaveControllerFactory, SaveControllerFactory>(Lifetime.Singleton);
        containerBuilder.Register<ISaveModuleFacade, SaveModuleFacade>(Lifetime.Singleton);
    }

}
`````

Правила:
- Регистрация зависимостей модуля выполняется только внутри его `Installer`
- Наружу модуль экспортирует зависимости через `Facade`
- `Bind` не должен содержать бизнес-логику — только конфигурацию контейнера

---

### Зависимости и границы модулей в DI

Правила:
- модуль A может ссылаться на модуль B только через:
  - интерфейсы, предоставляемые B (если они не помечены, как `internal`)
  - публичный API B (`Facade`), если это предусмотрено
- прямые зависимости на конкретные реализации другого модуля запрещены
- зависимости между модулями должны быть видны в `asmdef` (`Assembly Definition References`)

<hr style="border: 0; height: 6px; background: #2f2f2f; margin: 40px 0;">