# Руководство по EUI

## Как устроен EUI

### Основные виды интерфейсов SS14

В SS14 существует три подхода к созданию UI:  
- **Client-Side UI** — локальные элементы, существующие только на клиенте (например, админская панель)  
- **BoundUserInterface** — классические интерфейсы, привязанные к конкретной сущности (например, UI консоли связи)  
- **EUI (Extensible User Interface)** — гибкая система, не привязанная к сущностям, с двусторонней коммуникацией между клиентом и сервером  

!!! Примечание
    Пока `BoundUserInterface` реализован в самом RobustToolbox, `EUI` реализован исключительно в контенте и может быть изменён при необходимости.

### Жизненный цикл EUI

1. **Инициация на сервере**  
   Всё начинается, когда сервер решает открыть интерфейс. Он создаёт экземпляр серверного EUI-класса.

2. **Открытие на клиенте**  
   Сервер отправляет приглашение клиенту. Клиент создаёт свой экземпляр EUI и открывает окно интерфейса.

3. **Обмен данными**
    - Сервер отправляет **полное состояние** (весь набор данных) при каждом обновлении  
    - Клиент может отправлять **сообщения** (например, о нажатии кнопок)  
    - Сервер помечает состояние как изменённое вызовом `StateDirty()`, чтобы уведомить клиент  

4. **Закрытие**  
   Только сервер может завершить EUI, отправив команду закрыть EUI. Клиент уничтожает класс EUI и закрывает окно.

### Важные особенности работы с EUI

1. **Сервер — владелец процесса**  
   Только сервер может инициировать открытие и закрытие интерфейса. Клиент может лишь попросить о закрытии.

2. **Всегда полное состояние**  
   Даже если изменилось одно поле, сервер отправляет весь объект состояния. Это упрощает синхронизацию.

3. **Жизненный цикл — ваша ответственность**  
   Всегда закрывайте EUI при завершении работы и освобождайте ресурсы в методе `Closed()`.

## Практика

Создадим обычный счётчик. На клиенте есть окно отображающее текст и цифру, а также под ними располагает кнопку "Клик". При каждом нажатии на кнопку цифра увеличивается на 1.

### Шаг 1: Состояние EUI

В `Content.Shared` создаём класс хранящий в себе текущее состояние EUI:

```csharp
namespace Content.Shared.Example;

[Serializable, NetSerializable]
public sealed class CounterEuiState : EuiStateBase
{
    public string DisplayText = "Начальное значение";
    public int CurrentCount = 0;
}
```

Этот объект будет передаваться целиком при каждом обновлении. Атрибуты `[Serializable, NetSerializable]` дают возможность отправлять класс по сети.

### Шаг 2: Реализуем серверную логику

В `Content.Server` создаём обработчик EUI. Здесь живёт бизнес-логика:

```csharp
namespace Content.Server.Example;

public sealed class CounterEui : BaseEui
{
    private string _text = "Счётчик: ";
    private int _count = 0;

    // Возвращаем полное состояние при запросе
    public override EuiStateBase GetNewState()
    {
        return new CounterEuiState {
            DisplayText = _text,
            CurrentCount = _count
        };
    }

    // Обрабатываем сообщения от клиента (например, нажатия кнопок)
    public override void HandleMessage(EuiMessageBase msg)
    {
        if (msg is IncrementCounterMessage incrementMsg)
        {
            _count += incrementMsg.Value;
            // Важно! Указываем, что состояние изменилось, иначе клиент посчитает, что изменений нет и обновления UI не произойдёт, счётчик не изменится
            StateDirty();
        }
    }

    // Всегда освобождаем ресурсы при закрытии
    public override void Closed()
    {
        Logger.Debug("EUI закрыт, ресурсы освобождены");
    }
}
```

**Ключевые методы сервера:**  
- `GetNewState()` — возвращает текущее состояние для синхронизации  
- `HandleMessage()` — обработчик действий от клиента  
- `StateDirty()` — волшебный метод, указывающий на необходимость обновления  
- `Opened()` — вызывается при открытии EUI  
- `Closed()` — вызывается при закрытии EUI - ваш шанс убрать за собой

### Шаг 3: Создаём клиентскую часть

В `Content.Client` реализуем отображение, лицо нашего EUI:

```csharp
namespace Content.Client.Example;

public sealed class CounterEui : BaseEui
{
    private CounterWindow? _window;

    // При открытии создаём окно
    public override void Opened()
    {
        _window = new CounterWindow();

        // Настройка обратных вызовов:
        _window.OnClose += () => SendMessage(new CloseEuiMessage());
        _window.OnIncrement += (value) => SendMessage(
            new IncrementCounterMessage { Value = value });

        _window.OpenCentered();
    }

    // Обновляем интерфейс при получении нового состояния
    public override void HandleState(EuiStateBase state)
    {
        if (state is not CounterEuiState casted) return;
        _window?.UpdateDisplay(casted.DisplayText, casted.CurrentCount);
    }

    // Закрываем окно при завершении
    public override void Closed()
    {
        _window?.Close();
    }
}
```

**Ключевые методы клиента:**  
- `Opened()` — инициализация интерфейса  
- `HandleState()` — реакция на обновления с сервера  
- `SendMessage()` — отправка действий на сервер  
- `Closed()` — финальная очистка

### Шаг 4: Запускаем EUI

На сервере открываем интерфейс для игрока:

```csharp
// Где-то в вашей системе
var eui = new CounterEui();
_euiManager.OpenEui(eui, playerSession);
```

# Конец
