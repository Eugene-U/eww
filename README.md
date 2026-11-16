# eww config

Конфигурация [Eww](https://elkowar.github.io/eww/) для статус-бара в [Hyprland](https://hyprland.org/) (два монитора).

## Окна

- `bar-left` — бар на мониторе `0` (`HDMI-A-1`)
- `bar-right` — бар на мониторе `1` (`DP-1`)

Каждое окно — док-панель сверху (`32px`, `exclusive`, `centerbox`-лейаут). В текущем состоянии в барах отображаются:

- **слева** — виджет рабочих столов для соответствующего монитора;
- **по центру** — заголовок активного окна;
- **справа** — заглушка (`test` / `another`);
- **снизу** — тонкая линия-индикатор активного монитора (цвет зависит от переменной `active_monitor`, обновляется из `hyprland.lua`).

## Структура

```text
.
├── eww.yuck                  # точки входа, окна и компоновка баров
├── eww.scss                  # подключение стилей
├── css/
│   ├── _colors.scss          # цветовая палитра
│   ├── _bar.scss             # стили бара
│   ├── _workspaces.scss      # стили рабочих столов
│   └── _active-window.scss   # стили виджета активного окна
├── modules/
│   ├── variables.yuck        # глобальные переменные
│   ├── workspaces.yuck       # виджет рабочих столов
│   └── active-window.yuck    # виджет активного окна
└── scripts/
    └── workspaces-listen     # демон состояния рабочих столов Hyprland
```

## Рабочие столы

`scripts/workspaces-listen` (Python 3) получает состояние из `hyprctl -j monitors/workspaces` и слушает события Hyprland через `.socket2.sock`, выводя JSON для `deflisten`. Интегрирован с плагином [`split-monitor-workspaces`](https://github.com/Duckonaut/split-monitor-workspaces) (`count = 10`, персистентные воркспейсы).

Для каждого монитора формируется 10 слотов с полями:

- `local` / `global` — локальный и глобальный номер рабочего стола (`local = ((global - 1) % 10) + 1`);
- `active` — активный рабочий стол на мониторе;
- `occupied` — есть открытые окна;
- `persistent` — постоянный рабочий стол.

Клик по кнопке переключает рабочий стол через `hyprctl dispatch 'hl.dsp.focus({ workspace = <global> })'`.

## Активное окно

`scripts/active-window-listen` (Python 3) получает данные из `hyprctl -j activewindow` и слушает события Hyprland (`activewindow`, `open/close/movewindow`, `focusedmon`, `workspace` и др.) через `.socket2.sock`, выводя JSON для `deflisten`:

- `title` — заголовок окна;
- `class` — класс приложения;
- `empty` — нет активного окна.

Виджет отображает заголовок по центру (с обрезкой длинных строк); при отсутствии активного окна текст пустой.

## Зависимости

- `eww`
- `Hyprland` (`hyprctl`, Lua config / `hl.dsp` dispatch), плагин `split-monitor-workspaces`
- `python3`
- переменные окружения Hyprland: `XDG_RUNTIME_DIR`, `HYPRLAND_INSTANCE_SIGNATURE` (для подписки на `.socket2.sock`)

## Запуск

```sh
# открыть оба бара
eww --config ~/.config/eww open-many bar-left bar-right

# перезагрузка конфига/стилей
eww --config ~/.config/eww reload

# закрыть окна
eww --config ~/.config/eww close bar-left bar-right
```

## Примечания

- Имена мониторов, их порядок и привязка окон заданы в `eww.yuck` (`HDMI-A-1`, `DP-1`). При другой конфигурации мониторов их нужно обновить.
- Количество локальных воркспейсов зафиксировано как `COUNT = 10` в `scripts/workspaces-listen` и должно соответствовать `workspace_count` в конфиге `split-monitor-workspaces`.
