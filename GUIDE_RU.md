Гайд для создания lua скриптов под систему Scripts в Anxium

## Как это работает

1. Положи файл .lua в папку AnxiumHub/scripts (создаётся при первом запуске Anxium)
2. В меню Anxium открой вкладку Scripts
3. Нажми Refresh чтобы обновить список локальных файлов
4. Нажми Start у нужного скрипта
5. Если скрипт вызывает Anxium.RegisterScript, появится новая вкладка слева
6. Stop останавливает скрипт и вызывает onUnload

Настройки запущенных скриптов сохраняются в конфиг Anxium (.cfg)
Исходный код lua в конфиг не пишется. Файлы всегда лежат только в AnxiumHub/scripts

## Конфиг и скрипты

Если при Save ни один скрипт не запущен:
- в cfg нет данных о скриптах

Если один или несколько скриптов запущены, в cfg сохраняется:
- имя файла скрипта
- author (если указан)
- autoStart
- настройки UI (тумблеры, слайдеры, списки, бинды)

Source code в cfg не сохраняется
При Load конфига Anxium ищет файлы в AnxiumHub/scripts по имени
Если файла нет, скрипт не запустится (нужно положить lua вручную)

## Минимальный пример

```lua
local Anxium = (getgenv and getgenv().Anxium) or shared.Anxium
if not Anxium or type(Anxium.RegisterScript) ~= "function" then
    return warn("[script] Start only from Anxium -> Scripts")
end

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LP = Players.LocalPlayer

local conn

Anxium.RegisterScript({
    id = "example_speed",
    name = "Example Speed",
    author = "YourName",
    icon = "rbxassetid://113164012451432",

    buildUI = function(parent, UI, API)
        local card = UI.Section("Main")

        local _, getEnabled, setEnabled = UI.Toggle(card, "Enabled", false, function(on)
        end)

        UI.Slider(card, "Speed", 16, 100, 32, function(v)
        end)

        API.RegisterBind("Example Speed Enabled", getEnabled, setEnabled)

        UI.Button(card, "Ping", function()
            API.Notify("Example Speed", "pong")
        end)
    end,

    onLoad = function(API)
        local cfg = API.GetConfig()
        conn = RunService.Heartbeat:Connect(function()
            if cfg.tog_Enabled ~= true then return end
            local char = LP.Character
            local hum = char and char:FindFirstChildOfClass("Humanoid")
            if hum then
                hum.WalkSpeed = tonumber(cfg.sld_Speed) or 32
            end
        end)
    end,

    onUnload = function()
        if conn then
            conn:Disconnect()
            conn = nil
        end
        local char = LP.Character
        local hum = char and char:FindFirstChildOfClass("Humanoid")
        if hum then
            hum.WalkSpeed = 16
        end
    end,
})
```

## Поля RegisterScript

- id (обязательно) - уникальный идентификатор. Латиница, цифры, _
- name (обязательно) - имя вкладки в меню
- author (необязательно) - ник автора. Показывается в Scripts рядом с датой
- icon (необязательно) - rbxassetid://... иконка вкладки. Можно взять с https://www.icons.rest/
- buildUI (обязательно) - function(parent, UI, API) построение интерфейса
- onLoad (необязательно) - function(API) запуск логики после регистрации
- onUnload (необязательно) - function() вызов при Stop. Здесь нужно чистить connections и instances

## UI

UI.Section(title)
Карточка секции. Возвращает parent для Toggle, Slider и остальных элементов

UI.Toggle(parent, name, default, callback, colorKey?)
Возвращает row, get, set
Значение пишется в cfg как tog_Имя_с_подчёркиваниями
colorKey опционален: ключ цвета для палитры рядом с тумблером

UI.Slider(parent, name, min, max, default, callback)
Ключ в cfg: sld_Имя

UI.Dropdown(parent, name, options, default, callback)
Ключ в cfg: dd_Имя
options - таблица строк

UI.Button(parent, name, callback)
Кнопка без сохранения состояния

UI.Label(parent, text)
Текстовая подсказка.

Значения Toggle, Slider и Dropdown лежат в Config.Scripts[id] и сохраняются в .cfg только пока скрипт запущен

## API

- API.Id
- API.TabName
- API.Tab
- API.UI
- API.Theme
- API.Notify(title, text)
- API.GetConfig() - таблица настроек скрипта
- API.SetConfig(table)
- API.RegisterBind(displayName, getState, setState)

RegisterBind добавляет бинд в Settings Anxium. get и set обычно берутся из UI.Toggle

## Author

В RegisterScript укажи:

```lua
author = "co4vm",
```

В списке Scripts:

upload date: 01.11.2026 12:30:00  |  author: co4vm

Подпись author: серым, ник цветом темы. Автор читается из файла ещё до Start

## Иконки

Любой rbxassetid://...
Каталог: https://www.icons.rest/

```lua
icon = "rbxassetid://113164012451432",
```

## onLoad и onUnload

В onLoad:
- возьми cfg = API.GetConfig()
- применяй tog_ / sld_ / dd_
- создавай connections, ESP, hooks

В onUnload:
- Disconnect всех connections
- Destroy GUI и instances
- верни изменённые свойства персонажа или игры

Без onUnload после Stop логика может продолжать работать

## Файлы

AnxiumHub/scripts - твои .lua файлы
AnxiumHub/configs - конфиги .cfg

Положи скрипт в AnxiumHub/scripts, в меню Scripts нажми Refresh, затем Start

## Важно

- Скрипт должен вызывать Anxium.RegisterScript. Без этого вкладка не создастся
- Запускай только через Scripts -> Start, а не через обычный loadstring в executor
- id должен быть уникальным. Если такой уже запущен, сначала Stop
- В onUnload обязательно чисти connections и instances
- В onLoad применяй cfg.tog_* и cfg.sld_* к своей логике
- Код lua в конфиг не сохраняется. Храни файлы в AnxiumHub/scripts

## Чеклист

1. Проверка getgenv().Anxium и RegisterScript
2. Уникальный id
3. author (желательно)
4. icon (желательно)
5. buildUI с Section и Toggle/Slider
6. RegisterBind для важных тумблеров
7. onLoad читает cfg
8. onUnload чистит всё
9. Файл лежит в AnxiumHub/scripts
10. Scripts -> Refresh -> Start
