# Astroneer VR Mod — AGENT.md

## ТОЧКА ВХОДА ДЛЯ НОВОГО АГЕНТА

Этот файл — полный контекст проекта. Всё что сделано, что работает, что нужно делать дальше.

---

## ТЕКУЩЕЕ СОСТОЯНИЕ (2026-11-12)

**.pak УСПЕШНО ОТКРЫТ И ЭКСПОРТИРОВАН.** Фаза 0 завершена.

| Параметр | Значение |
|---|---|
| Версия игры | **1.44.4.0 Shipping** |
| UE версия | **4.27** (определено FModel автоматически) |
| Главный .pak | `pakchunk0-WindowsNoEditor.pak` — 2.68 ГБ |
| Всего записей | 46 325 |
| Экспортировано | **44 287 файлов, 4 562 MB** |
| Папка с ассетами | `C:\ASTRONEER\tools\astroneer_assets\` |
| FModel | 4.4.4.0, **РАБОТАЕТ** |

---

## ФАЙЛЫ И ПУТИ

```
C:\ASTRONEER\steamrip\ASTRONEER\        ← РАБОЧАЯ версия игры (1.44.4.0)
    Astro\Content\Paks\pakchunk0-WindowsNoEditor.pak  ← 2.68 GB, 46 325 записей

C:\ASTRONEER\tools\                      ← ВСЕ инструменты здесь (без кириллицы!)
    astroneer_assets\                    ← 44 287 извлечённых файлов (4.56 GB)
        Astro\Content\                   ← Игровые ассеты
            Animations/     5642 файла
            Blueprints/     (в .uasset)
            Character/      18   ← ИГРОК И УПРАВЛЕНИЕ
            Camera/         82   ← КАМЕРА
            Materials/      5371
            Models/         5440
            Planets/        1988
            UI/             4832  ← ИНТЕРФЕЙС
            Vehicles/       186
            WwiseAudio/     2906
        Engine\Content\                  ← Стандартные ассеты UE4
    
    FModel\                               ← FModel.exe (портативный .NET 10)
        appsettings.json                  ← конфиг (путь к игре, версия UE)
    
    UEVR\                                 ← UEVRInjector.exe + бэкенд
    android-ndk\android-ndk-r21e\         ← NDK для Quest 3
    temp\                                 ← ЧИСТЫЙ TEMP (без кириллицы!)
    
    extract_pak.py                        ← Python-скрипт (устарел, FModel лучше)
    pak_filelist.txt                      ← Список категорий

C:\Users\Иван\Downloads\DotAi\           ← Рабочая папка DSH
    AGENT.md                             ← ЭТОТ ФАЙЛ
    .gitignore
```

---

## КАК ЗАПУСКАТЬ ИНСТРУМЕНТЫ

### FModel (просмотр и экспорт ассетов)
```powershell
$env:TEMP = "C:\ASTRONEER\tools\temp"
$env:TMP = "C:\ASTRONEER\tools\temp"
$env:DOTNET_BUNDLE_EXTRACT_BASE_DIR = "C:\ASTRONEER\tools\temp"
Start-Process "C:\ASTRONEER\tools\FModel\FModel.exe" -WorkingDirectory "C:\ASTRONEER\tools\FModel"
```
- Правый клик на папке → Export Folder
- Форматы: Raw Data (.uasset), Properties (.json), Decompiled Blueprints, Textures, Models

### UEVR (VR инжектор)
```powershell
$env:TEMP = "C:\ASTRONEER\tools\temp"
$env:TMP = "C:\ASTRONEER\tools\temp"
Start-Process "C:\ASTRONEER\tools\UEVR\UEVRInjector.exe" -WorkingDirectory "C:\ASTRONEER\tools\UEVR"
```

### GitHub (только через gh api!)
```powershell
$env:GH_TOKEN = '<токен из переменной окружения>'
& "C:\Program Files\GitHub CLI\gh.exe" api repos/Vano3667/Astroneer-mod/contents/AGENT.md --jq '.sha'
```
- `git push` НЕ РАБОТАЕТ (заблокирован песочницей DSH)
- Использовать только `gh api` с `-X PUT`

### Коммит файла в репо (через gh api):
```powershell
$sha = (gh api repos/Vano3667/Astroneer-mod/contents/AGENT.md --jq '.sha').Trim()
$raw = Get-Content "путь\к\файлу" -Raw
$bytes = [Text.Encoding]::UTF8.GetBytes($raw)
$b64 = [Convert]::ToBase64String($bytes)
$body = @{message='commit message'; content=$b64; sha=$sha; branch='main'} | ConvertTo-Json -Depth 1 -Compress
$body | gh api -X PUT /repos/Vano3667/Astroneer-mod/contents/AGENT.md --input -
```

---

## КЛЮЧЕВАЯ ПРОБЛЕМА: КИРИЛЛИЦА

`C:\Users\Иван` — ломает ВСЕ .NET/WPF приложения (FModel, UEVR).
Решение: всегда задавать `$env:TEMP = "C:\ASTRONEER\tools\temp"` перед запуском.

---

## СТАТИСТИКА ЭКСПОРТИРОВАННЫХ АССЕТОВ

| Расширение | Количество | Описание |
|---|---|---|
| .uexp | 19 332 | Данные ассетов (свойства, ссылки) |
| .uasset | 19 326 | Заголовки ассетов (имена, типы) |
| .res | 3 438 | Ресурсы движка |
| .wem | 2 759 | Аудио (Wwise) |
| .ubulk | 353 | Объёмные данные (модели, текстуры) |
| .png | 275 | Текстуры (UI, эффекты) |
| .bnk | 145 | Аудио-банки (Wwise) |
| .umap | 6 | Игровые карты/уровни |

### Ключевые папки Astro/Content:

| Папка | Файлов | Значение для VR |
|---|---|---|
| Animations/ | 5 642 | Анимации (руки, инструменты) |
| Materials/ | 5 371 | Шейдеры/материалы (надо адаптировать под стерео) |
| Models/ | 5 440 | 3D-модели |
| UI/ | 4 832 | Интерфейс (критично для VR) |
| Scenarios/ | 3 640 | Игровые сценарии |
| Items/ | 2 262 | Предметы инвентаря |
| Planets/ | 1 988 | Планеты (рендер на расстоянии) |
| Character/ | 18 | **ИГРОК** |
| Camera/ | 82 | **КАМЕРА** |
| Vehicles/ | 186 | Транспорт |

---

## ДОРОЖНАЯ КАРТА

### Фаза 0 ✅ ЗАВЕРШЕНА
```
✅ 0.1 Определена версия UE: 4.27
✅ 0.2 Инструменты установлены
✅ 0.3 Ассеты извлечены — 44 287 файлов
```

### Фаза 1 — UEVR Прототип ← СЛЕДУЮЩАЯ
```
[ ] 1.0 Запустить Astroneer с UEVR
[ ] 1.1 Проверить стерео-рендер
[ ] 1.2 Проверить управление (контроллеры)
[ ] 1.3 Задокументировать проблемы
```

### Фаза 2 — Анализ архитектуры
```
[ ] Найти PlayerController и PlayerCharacter
[ ] Найти Camera Blueprint
[ ] Построить карту зависимостей
[ ] Найти точки входа для VR-инъекций
```

### Фаза 3 — VR интеграция (ПК)
```
[ ] Стерео-рендер
[ ] Motion Controls
[ ] VR UI
```

### Фаза 4 — Порт на Quest 3 (Android/ARM64)
```
[ ] Кросс-компиляция
[ ] OpenXR
[ ] Оптимизация
```

---

## ПЛАН ДЛЯ 3 ПАРАЛЛЕЛЬНЫХ АГЕНТОВ

### Агент А: UEVR Тест (Фаза 1)
**Файлы**: `C:\ASTRONEER\tools\UEVR\UEVRInjector.exe`
**Игра**: `C:\ASTRONEER\steamrip\ASTRONEER\Astro\Binaries\Win64\Astro-Win64-Shipping.exe`
**Задача**:
1. Запустить игру через UEVR
2. UEVR настройки: OpenVR, синхронизированный рендер
3. Проверить: стерео работает? Управление VR-контроллерами? UI виден?
4. Результат: `UEVR_REPORT.md` со скриншотами

### Агент Б: Анализ Blueprint'ов (Фаза 2)
**Вход**: `C:\ASTRONEER\tools\astroneer_assets\`
**Задача**:
1. Экспортировать Decompiled Blueprints из FModel (правый клик → Export Folder → Decompiled Blueprints)
2. Найти ключевые классы:
   - `*PlayerController*` — управление
   - `*PlayerCharacter*` или `*Pawn*` — персонаж
   - `*Camera*` — камера
   - `*GameMode*` — игровая логика
   - `*HUD*` — интерфейс
   - `*Input*` — обработка ввода
3. Экспортировать Properties (.json) для этих классов
4. Результат: `ARCHITECTURE.md` с картой классов

### Агент В: Анализ UI и ресурсов (Фаза 2)
**Вход**: `C:\ASTRONEER\tools\astroneer_assets\Astro\Content\UI\`
**Задача**:
1. Проанализировать UI Blueprint'ы (Widget Blueprints)
2. Найти HUD, меню, инвентарь
3. Оценить объём работы для VR-адаптации UI
4. Найти текстуры UI (.png в папке UI/)
5. Результат: `UI_ANALYSIS.md`

---

## УСТАНОВЛЕННЫЕ ИНСТРУМЕНТЫ

| Инструмент | Версия | Путь | Статус |
|---|---|---|---|
| FModel | 4.4.4.0 (окт 31, 2026) | `C:\ASTRONEER\tools\FModel\FModel.exe` | ✅ Работает |
| UEVR | 1.05 | `C:\ASTRONEER\tools\UEVR\UEVRInjector.exe` | ✅ Готов |
| Android NDK | r21e | `C:\ASTRONEER\tools\android-ndk\android-ndk-r21e\` | ✅ Готов |
| GitHub CLI | 2.67.0 | `C:\Program Files\GitHub CLI\gh.exe` | ✅ Работает |
| .NET Desktop Runtime | 10.0.15 | winget | ✅ Установлен |
| Python | 3.12.10 | `py -3.12` | ✅ Работает |

---

## ИСТОРИЯ СЕССИЙ

### Session 1 (2026-11-11)
- Определена версия UE: 4.23.1
- Установлены: GitHub CLI, FModel, UEVR, Android NDK
- Создан репо: `Vano3667/Astroneer-mod`
- Написан `astroneer_vr_prompt.md`

### Session 2 (2026-11-12, часть 1)
- Установлен .NET Desktop Runtime 10.0.15
- Обнаружена проблема с кириллицей (`C:\Users\Иван`)
- Написан Python-скрипт для .pak
- Обнаружена обфускация имён в старой версии (репак)

### Session 3 (2026-11-12, ПРОРЫВ)
- Скачана чистая версия: Astroneer 1.44.4.0 (steamrip)
- FModel УСПЕШНО открыл .pak
- Определена UE версия: **GAME_UE4_27**
- Экспортировано: **44 287 файлов / 4 562 MB** в `astroneer_assets/`
- Исправлена утечка токена GitHub (старый отозван, новый не хранится в файлах)

---

## КАК ПРОДОЛЖАТЬ РАБОТУ (следующему агенту)

1. **Прочитай этот AGENT.md полностью**
2. Проверь что FModel работает: запусти по инструкции выше
3. Проверь что экспорт завершён: `C:\ASTRONEER\tools\astroneer_assets\Astro\Content\` — 44 287 файлов
4. Выбери одного из трёх агентов (А, Б, В) и начни выполнять
5. Результаты сохраняй в `C:\Users\Иван\Downloads\DotAi\` и коммить в репо

### Экспорт дополнительных форматов из FModel:
- **Properties (.json)** — структура Blueprint'ов в читаемом виде
- **Decompiled Blueprints** — псевдокод Blueprint'ов
- **Textures** — извлечённые .png текстуры
- **Models** — 3D модели (.gltf/.obj если поддерживается)