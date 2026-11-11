# Astroneer VR Mod — AGENT.md

## Статус проекта

**Дата обновления**: 2026-11-12
**Аккаунт**: Vano3667
**Репозиторий**: github.com/Vano3667/Astroneer-mod
**Фаза**: 0.3 — Анализ ассетов

---

## Что сделано на 2026-11-12

- [x] Установлен .NET Desktop Runtime 10.0.15 (через winget)
- [x] FModel скопирован в `C:\ASTRONEER\tools\FModel` (без кириллицы)
- [x] UEVR скопирован в `C:\ASTRONEER\tools\UEVR` (без кириллицы)
- [x] Написан Python-скрипт для анализа .pak (`C:\ASTRONEER\tools\extract_pak.py`)
- [x] Проанализирован `Astro-WindowsNoEditor.pak`: **43 280 записей, версия 8, footer найден**
- [x] Обнаружена **обфускация имён файлов** в индексе .pak

---

## КЛЮЧЕВОЕ ОТКРЫТИЕ

**.pak файл Astroneer имеет обфусцированные/зашифрованные имена файлов.** Из 43 280 записей только ~560 имеют осмысленные имена. Остальные — мусор/бинарные данные вместо имён. Это типично для пиратских репаков.

**НО**: данные (сами файлы) читаются — мы можем извлечь их по смещениям в `C:\ASTRONEER\tools\extract_pak.py`.

---

## План действий для агентов (Фаза 0.3 → Фаза 1 → Фаза 2)

### Агент А: Извлечение ассетов
**Файл-основа**: `C:\ASTRONEER\tools\extract_pak.py`
**Задача**: Допилить скрипт, чтобы он извлекал файлы без имён (по смещениям с авто-именами типа file_0001.bin)

### Агент Б: UEVR-анализ (приоритет!)
**Инструмент**: `C:\ASTRONEER\tools\UEVR\UEVRInjector.exe`
**Задача**: Запустить Astroneer через UEVR, получить список ресурсов в рантайме, протестировать базовый VR

### Агент В: Изучение структуры .pak
**Задача**: Найти способ деобфускации имён. Возможные подходы:
- Проверить `Astrooptional-WindowsNoEditor.pak` (52 КБ) — он может содержать ключ
- Поискать AES ключ в бинарниках игры (dumpbin, strings)
- Проверить `Astro\Binaries\Win64\Astro-Win64-Shipping.exe` (основной бинарник)

---

## Установленные инструменты

| Инструмент | Версия | Путь | Статус |
|---|---|---|---|
| FModel | Nov 2026 | `C:\ASTRONEER\tools\FModel\FModel.exe` | ⚠️ Баг MainWindow (обход: .bat с DOTNET_BUNDLE_EXTRACT_BASE_DIR) |
| UEVR | 1.05 | `C:\ASTRONEER\tools\UEVR\UEVRInjector.exe` | ⚠️ Падает из-за кириллицы в TEMP |
| Android NDK | r21e | `C:\ASTRONEER\tools\android-ndk\android-ndk-r21e\` | ✅ Готов |
| GitHub CLI | 2.67.0 | `C:\Program Files\GitHub CLI\gh.exe` | ✅ Работает |
| .NET Desktop 10 | 10.0.15 | winget | ✅ Установлен |
| Python | 3.12.10 | `py -3.12` | ✅ Работает |

---

## Структура репозитория

```
Astroneer-mod/
├── AGENT.md                 ← этот файл
├── astroneer_vr_prompt.md   ← исходный промт
├── tools/
│   ├── extract_pak.py       ← Python-скрипт для .pak
│   ├── pak_filelist.txt     ← список категорий
│   ├── launch_fmodel.bat    ← запуск FModel
│   ├── FModel/              ← FModel (GUI)
│   ├── UEVR/                ← UEVR (инжектор)
│   ├── android-ndk/         ← Android NDK r21e
│   └── temp/                ← чистый TEMP (без кириллицы)
└── astroneer_assets/        ← будут извлечённые файлы
```

---

## Технический контекст

- **Версия UE**: 4.23.1
- **Версия игры**: 1.33.14.0 Shipping
- **Путь к игре**: `C:\ASTRONEER\`
- **Главный .pak**: `Astro\Content\Paks\Astro-WindowsNoEditor.pak` (2.8 ГБ, 43 280 файлов)
- **Проблема с путями**: КИРИЛЛИЦА в `C:\Users\Иван` ломает FModel и UEVR. ВСЕ инструменты запускать из `C:\ASTRONEER\tools\`
- **TEMP без кириллицы**: `$env:TEMP = "C:\ASTRONEER\tools\temp"` перед запуском GUI-инструментов
- **GitHub**: только через `gh api`, git напрямую заблокирован песочницей
- **GH_TOKEN**: установлен в переменной окружения. ВАЖНО: НИКОГДА не коммитить токен в репозиторий. Скомпрометированный токен отозван.

---

## История сессий

### Session 1 (2026-11-11): Настройка окружения
- Определена версия UE: 4.23.1
- Установлены GitHub CLI, FModel, UEVR, Android NDK
- Создан репозиторий, настроен workflow

### Session 2 (2026-11-12): Анализ .pak
- Установлен .NET Desktop Runtime 10.0.15
- Обнаружена проблема с кириллицей в путях
- Написан Python-скрипт для анализа .pak
- ОБНАРУЖЕНА обфускация имён в индексе .pak
- Требуется деобфускация или извлечение без имён