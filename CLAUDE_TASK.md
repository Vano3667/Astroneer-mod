# Claude Opus 5.5 — Задача: Анализ архитектуры Astroneer

> Ты — Агент Б. Твоя задача — проанализировать архитектуру Astroneer и найти точки входа для VR-мода.
> Ты работаешь в облаке. У тебя нет доступа к ПК. Все данные уже извлечены для тебя.

---

## ТВОЯ ЗАДАЧА

Проанализировать Blueprint'ы Astroneer и составить карту ключевых классов для VR-модификации.

**Результат**: файл `ARCHITECTURE.md` в этом репозитории.

---

## ДАННЫЕ ДЛЯ АНАЛИЗА

Все данные в файле `claude_architecture_data.json` в этом репозитории.

Структура JSON:
```json
{
  "game": "Astroneer 1.44.4.0",
  "ue_version": "4.27",
  "folders": {
    "Camera": {
      "count": 41,
      "files": [
        {
          "path": "Camera/BackpackCameraRig.uasset",
          "own_path": "/Game/Camera/BackpackCameraRig",
          "asset_deps": ["/Game/Camera/CameraSpaceWalkingRig"],
          "script_deps": ["/Script/Astro", "/Script/CoreUObject", "/Script/Engine"],
          "size": 7012
        }
      ]
    }
  }
}
```

---

## КЛЮЧЕВЫЕ НАХОДКИ (уже известны)

### Контроллер игрока
- `/Game/Globals/PlayControllerInstance` — **главный PlayerController**
- Зависит от: FreeCamRig, DesignAstro (Character), Dropship, MultiTool, Compass, Beacon

### GameMode
- `/Game/Globals/AstroGameModeInstance` — основной GameMode
- `/Game/Globals/Astro_Gates_GameMode_Instance` — расширенный GameMode
- Зависит от: PlayControllerInstance, HUDInstance, DesignAstro (Character), карты Stages

### GameState
- `/Game/Globals/Astro_Gates_GameState_Instance` — состояние игры
- Управляет персистентностью акторов, ресурсами

### Камера
- `/Game/Camera/BackpackCameraRig` — камера от 3-го лица (за спиной)
- `/Game/Camera/CameraSpaceDrivingRig` — камера транспорта
- `/Game/Camera/VehicleGamepadFreeCursorCam` — свободная камера в транспорте
- Все камеры наследуются от `/Game/Camera/CameraSpaceWalkingRig`

### HUD / UI
- `/Game/UI/HUDInstance` — ГЛАВНЫЙ HUD
- Зависит от: ActionWheel, EmoteWheel, PlayWidgetInstance, PlanetSelectionHUD, Store
- Использует `/Script/UMG` (Unreal Motion Graphics)

### Персонаж
- `/Game/Character/DesignAstro` (есть в зависимостях GameMode и PlayController)
- 185 файлов скинов/палитр в Globals/CharacterCustomization

### Ввод
- `/Game/Globals/DiscreteInputDefinitionDatabase` — карта действий
- `/Game/Globals/InputKeyToIconDatabase` — иконки клавиш/кнопок

---

## ЧТО НУЖНО СДЕЛАТЬ

### Шаг 1: Построить карту зависимостей
Прочитай `claude_architecture_data.json`. Для каждого ключевого класса:
- Найди все его зависимости (`asset_deps`)
- Определи цепочку наследования
- Найди что ещё ссылается на этот класс

### Шаг 2: Найти точки VR-инъекций
Для VR-мода нужно переопределить:
1. **PlayerController** — добавить VR-трекинг головы и рук
2. **Camera** — заменить на стерео-камеру (Left/Right Eye)
3. **Pawn/Character** — добавить Motion Controller компоненты
4. **HUD** — адаптировать под VR (World Space Widget)
5. **Input** — маппинг VR-контроллеров

### Шаг 3: Оценить сложность
Для каждого компонента:
- Можно ли модифицировать Blueprint'ом или нужен C++?
- Сколько зависимостей нужно будет обновить?
- Какие ассеты затронуты?

### Шаг 4: Написать ARCHITECTURE.md
Формат:
```markdown
# Astroneer VR Architecture

## Ключевые классы
### PlayerController
- Путь: /Game/Globals/PlayControllerInstance
- Зависимости: [...]
- Стратегия VR-интеграции: [...]

## Карта зависимостей
(диаграмма в ASCII или mermaid)

## Точки инъекций
(приоритизированный список)

## Оценка сложности
(таблица)
```

---

## ВАЖНЫЙ КОНТЕКСТ

- **Игра**: Astroneer — песочница-выживание на procedurally generated планетах
- **Движок**: Unreal Engine 4.27
- **Цель**: Портировать на Meta Quest 3 (standalone Android/ARM64)
- **Стратегия**: Сначала прототип через UEVR на ПК, затем standalone-порт
- **Фаза 1** (параллельно): UEVR тест (Агент А — DeepSeek V4 Pro)
- **Фаза 2** (ты): Анализ архитектуры
- **Фаза 2** (параллельно): Анализ UI (Агент В — DeepSeek V4 Pro)

---

## КООРДИНАЦИЯ

Обновляй свой статус в `COORDINATION.md`. Когда закончишь — запиши результат в `ARCHITECTURE.md` и обнови статус на ✅.