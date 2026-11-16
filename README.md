# eww config

Конфигурация [Eww](https://elkowar.github.io/eww/) для статус-бара в [Hyprland](https://hyprland.org/) (два монитора).

## Окна

- `bar-left` — бар на мониторе `0` (`HDMI-A-1`)
- `bar-right` — бар на мониторе `1` (`DP-1`)

Каждое окно — док-панель сверху (`32px`, `exclusive`). В текущем состоянии в барах отображаются:

- виджет рабочих столов для соответствующего монитора;
- индикатор активного монитора (тонкая линия снизу бара, цвет зависит от переменной `active_monitor`);
- заглушки (`test` / `another`) в центральной и правой секциях.

## Структура

```text
.
├── eww.yuck                 # точки входа, окна и компоновка баров
├── eww.scss                 # подключение стилей
├── css/
│   ├── _colors.scss         # цветовая палитра
│   ├── _bar.scss            # стили бара
│   └── _workspaces.scss     # стили рабочих столов
├── modules/
│   ├── variables.yuck       # глобальные переменные
│   └── workspaces.yuck      # виджет рабочих столов
├── scripts/
│   └── workspaces-listen    # демон состояния рабочих столов Hyprland
└── _legacy/                 # старые/неактивные модули и окна
```

## Рабочие столы

`scripts/workspaces-listen` (Python 3) получает состояние из `hyprctl -j` и слушает события Hyprland через `.socket2.sock`, выводя JSON для `deflisten`. Для каждого монитора формируется 10 слотов с полями:

- `local` / `global` — локальный и глобальный номер рабочего стола;
- `active` — активный рабочий стол на мониторе;
- `occupied` — есть открытые окна;
- `persistent` — постоянный рабочий стол.

Клик по кнопке переключает рабочий стол через `hyprctl dispatch 'hl.dsp.focus({ workspace = <global> })'`.

## Зависимости

- `eww`
- `Hyprland` (`hyprctl`, переменные окружения `XDG_RUNTIME_DIR` и `HYPRLAND_INSTANCE_SIGNATURE`)
- `python3`
- шрифт `JetBrains Nerd Font` (используется в стилях)

## Запуск

```sh
eww --config ~/.config/eww open-many bar-left bar-right
```

Закрыть окна:

```sh
eww --config ~/.config/eww close bar-left bar-right
```

## Примечания

- Имена мониторов и их порядок заданы в `eww.yuck` (`HDMI-A-1`, `DP-1`). При другой конфигурации мониторов их нужно обновить.
- В `_legacy/` находятся закомментированные/старые модули (datetime, system, docker, jira, projects и т.д.), сейчас в основной конфигурации не подключены.
