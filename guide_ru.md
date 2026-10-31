# создание Lua скрипта для AnxiumHub

## Как загрузить скрипт
1. Открой AnxiumHub
2. Перейди во вкладку Scripts
3. Вставь raw ссылку на .lua файл с GitHub (или gist)
4. Нажми Load from GitHub
5. В списке появится скрипт (дата загрузки и author, если указан)
6. Нажми Start
7. Слева появится новая вкладка с именем скрипта и иконкой
8. В Settings -> Binds можно назначить клавиши на функции скрипта
9. Config Save сохраняет исходники скриптов и их настройки
10. Config Load восстанавливает скрипты и запускает их автоматически

## Минимальный шаблон

```lua
local Anxium = getgenv().Anxium or shared.Anxium
if not Anxium then
    return warn("Запусти скрипт из вкладки Scripts в Anxium")
end

Anxium.RegisterScript({
    id = "my_script",
    name = "My Script",
    author = "YourName",
    icon = "rbxassetid://79541605299928",
    version = "1.0.0",

    buildUI = function(parent, UI, API)
        local card = UI.Section("Main")

        local _, get, set = UI.Toggle(card, "Enabled", false, function(on)
            -- on = true/false
        end)

        API.RegisterBind("My Script Enabled", get, set)

        UI.Slider(card, "Speed", 1, 100, 50, function(v)
            -- v = число
        end)

        UI.Dropdown(card, "Mode", {"A", "B"}, "A", function(v)
            -- v = выбранная строка
        end)

        UI.Button(card, "Ping", function()
            API.Notify("My Script", "pong")
        end)
    end,

    onLoad = function(API)
        -- вызывается после создания UI
    end,

    onUnload = function()
        -- вызывается при выгрузке скрипта
    end,
})
```

## Поля RegisterScript
- id (обязательно) - уникальный id скрипта, без пробелов
- name - имя вкладки слева
- author - показывается в списке Scripts рядом с датой: author: Name
- icon - rbxassetid иконки вкладки
- version - необязательная строка версии

## Иконки
Иконку для вкладки можно взять на сайте:
https://www.icons.rest/

Скопируй rbxassetid иконки и вставь в поле icon, например:
icon = "rbxassetid://113164012451432"

## UI элементы
- UI.Section(title) - секция (карточка) с заголовком
- UI.Toggle(parent, name, default, callback) - возвращает row, get, set.
- UI.Slider(parent, name, min, max, default, callback)
- UI.Dropdown(parent, name, options, default, callback)
- UI.Button(parent, name, callback)
- UI.Label(parent, text)

Значения Toggle / Slider / Dropdown автоматически сохраняются с конфигом

## Бинды
```lua
API.RegisterBind("Отображаемое имя", getState, setState)
```
- getState - функция, возвращает true/false.
- setState - функция, принимает true/false и меняет состояние + UI

После RegisterBind бинд появляется в Settings -> Binds

## API внутри скрипта
- Anxium.RegisterScript(def) - зарегистрировать скрипт и создать вкладку
- Anxium.UnloadScript(id) - выгрузить скрипт
- Anxium.IsLoaded(id) - проверить, загружен ли скрипт
- API.Notify(title, message) - уведомление
- API.GetConfig() - таблица настроек скрипта
- API.SetConfig(table) - заменить настройки скрипта
- API.Id - id скрипта
- API.Tab - Frame вкладки
- API.Theme - тема меню

## Как выложить на GitHub
1. Создай репозиторий (можно public)
2. Залей файл, например test.lua
3. Открой файл на GitHub и нажми Raw
4. Скопируй raw ссылку вида:
   https://raw.githubusercontent.com/USER/REPO/BRANCH/test.lua
5. Эту ссылку вставляй в Anxium -> Scripts -> Load from GitHub

Gist тоже работает: используй raw ссылку gist

## Важно
- Скрипт должен вызывать Anxium.RegisterScript. Без этого вкладка не создастся
- Запускай только через Scripts -> Start, а не через обычный executor loadstring (иначе API может быть недоступен)
- В onUnload обязательно чисти connections, instances, ESP и т.д
- id должен быть уникальным. Если скрипт с таким id уже загружен, регистрация не пройдёт