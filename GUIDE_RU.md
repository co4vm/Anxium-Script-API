создание Lua скрипта для AnxiumHub

## Как загрузить скрипт
1. Открой Anxium Hub
2. Перейди во вкладку Scripts
3. Вставь raw ссылку на .lua файл с GitHub (или gist)
4. Нажми Load from GitHub
5. В списке появится скрипт (дата загрузки и author, если указан)
6. Нажми Start - скрипт запустится, кнопка станет Stop
7. Нажми Stop - скрипт выгрузится (вкладка исчезнет, эффекты из onUnload очистятся), кнопка снова Start
8. Слева появится новая вкладка с именем скрипта и иконкой (после Start)
9. В Settings -> Binds можно назначить клавиши на функции скрипта
10. Config Save сохраняет исходники скриптов, бинды клавиш и состояния функций
11. Config Load восстанавливает скрипты, клавиши и включённые функции, затем auto-start

## Start / Stop
- Start - компилирует и запускает .lua, вызывает RegisterScript, создаёт вкладку
- Stop - вызывает onUnload, снимает бинды скрипта с HUD, удаляет вкладку, скрипт можно запустить снова
- В onUnload обязательно чисти connections, instances, ESP, скорость и всё, что скрипт создал

## Сохранение в конфиг
Сохраняется автоматически:
- исходник скрипта (PluginScripts)
- клавиши биндов (Keybinds)
- значения Toggle / Slider / Dropdown (Config.Plugins[id], ключи tog_ / sld_ / dd_)
- состояние бинд-функций (Config.Plugins[id], ключи bind_*)

При Config Load:
1. Подставляются Keybinds и Plugins
2. Скрипты из списка auto-start
3. UI читает tog_ / sld_ из конфига
4. После onLoad Anxium включает функции, у которых bind_* был true
5. Binds HUD обновляется

В onLoad обязательно применяй сохранённые tog_ / sld_ к своей логике (скорость, ESP и т.д.), иначе визуал Toggle будет ON, а эффект нет

## Минимальный шаблон

```lua
local Anxium = getgenv().Anxium or shared.Anxium
if not Anxium then
    return warn("Запусти скрипт из вкладки Scripts в Anxium")
end

local state = { enabled = false }

Anxium.RegisterScript({
    id = "my_script",
    name = "My Script",
    author = "YourName",
    icon = "rbxassetid://79541605299928",
    version = "1.0.0",

    buildUI = function(parent, UI, API)
        local card = UI.Section("Main")

        local _, get, set = UI.Toggle(card, "Enabled", false, function(on)
            state.enabled = on
            -- включи/выключи свою логику
        end)

        -- get/set: бинд и конфиг видят одно состояние
        API.RegisterBind("My Script Enabled", get, set)

        UI.Slider(card, "Speed", 1, 100, 50, function(v)
            -- v = число
        end)

        UI.Button(card, "Ping", function()
            API.Notify("My Script", "pong")
        end)
    end,

    onLoad = function(API)
        local cfg = API.GetConfig()
        -- восстановить логику из сохранённых toggle/slider
        state.enabled = cfg.tog_Enabled == true
        -- set из Toggle уже вызван Anxium при restore bind_*; дублируй логику здесь при необходимости
        API.Notify("My Script", "Loaded")
    end,

    onUnload = function()
        state.enabled = false
        -- очистка
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
- UI.Section(title) - секция (карточка) с заголовком.
- UI.Toggle(parent, name, default, callback) - возвращает row, get, set.
- UI.Slider(parent, name, min, max, default, callback)
- UI.Dropdown(parent, name, options, default, callback)
- UI.Button(parent, name, callback)
- UI.Label(parent, text)

Ключи в конфиге:
- Toggle -> tog_Имя_С_Подчёркиваниями
- Slider -> sld_...
- Dropdown -> dd_...
- RegisterBind -> bind_Имя_Бинда

## Бинды и Binds HUD
```lua
API.RegisterBind("Отображаемое имя", getState, setState)
```
- getState - возвращает true/false (для HUD и сохранения)
- setState - принимает true/false, меняет состояние + UI

Лучший вариант: передавать get/set от UI.Toggle, тогда Toggle, бинд, конфиг и HUD синхронны

После RegisterBind бинд появляется в Settings -> Binds
В Binds HUD цвет клавиши зависит от getState

## API внутри скрипта
- Anxium.RegisterScript(def) - зарегистрировать скрипт и создать вкладку
- Anxium.UnloadScript(id) - выгрузить скрипт
- Anxium.IsLoaded(id) - проверить, загружен ли скрипт
- API.Notify(title, message) - уведомление
- API.GetConfig() - таблица настроек скрипта (tog_ / sld_ / bind_ ...)
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
- В onLoad применяй cfg.tog_* / cfg.sld_* к своей логике
- id должен быть уникальным. Если скрипт с таким id уже загружен, регистрация не пройдёт