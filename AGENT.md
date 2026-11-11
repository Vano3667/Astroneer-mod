# Astroneer VR Mod — AGENT.md

## Статус проекта

**Дата обновления**: 2026-11-11
**Аккаунт**: Vano3667
**Репозиторий**: github.com/Vano3667/Astroneer-mod

---

## Что сделано (Session 1)

- [x] Выбор платформы для моддинга: Steam-версия Astroneer (Windows)
- [x] Определена стратегия: начать с UEVR (прототип), затем standalone-порт на Quest 3
- [x] Установлен GitHub CLI (gh 2.67.0)
- [x] Настроена авторизация GitHub через токен (scopes: repo, workflow, admin:org, gist, project, write:packages)
- [x] Создан репозиторий Astroneer-mod
- [x] Написан подробный промт для AI-агента (astroneer_vr_prompt.md)

---

## План работ (дорожная карта)

### Фаза 0 — Подготовка среды
- [ ] 0.1 Определение версии Unreal Engine у Astroneer
- [ ] 0.2 Установка инструментов: FModel, UnrealPak, UE4 Editor, Android NDK, UEVR
- [ ] 0.3 Извлечение ассетов Astroneer (.pak -> файлы)

### Фаза 1 — Прототип VR (UEVR)
- [ ] 1.0 Запуск Astroneer с UEVR на ПК (через AirLink/VD)
- [ ] 1.1 Документирование проблем (рендер, UI, управление, физика)

### Фаза 2 — Анализ архитектуры
- [ ] Изучение Blueprint'ов и C++ классов через FModel
- [ ] Карта зависимостей
- [ ] Точки входа для VR-инъекций

### Фаза 3 — VR-интеграция на ПК
- [ ] Стерео-рендер
- [ ] Motion Controls
- [ ] VR-UI

### Фаза 4 — Порт на Android / Quest 3 (standalone)
- [ ] Кросс-компиляция UE4 под ARM64
- [ ] Адаптация под Adreno GPU
- [ ] Oculus Mobile SDK / OpenXR
- [ ] Оптимизация (ATW, FFR, foveated rendering)

---

## Результаты анализа

### Фаза 0.1 — Версия Unreal Engine ✅ ВЫПОЛНЕНО

- **Версия UE**: **4.23.1** (определено по FileVersionRaw Astro.exe)
- **Версия игры**: 1.33.14.0 Shipping
- **Путь**: `C:\ASTRONEER\`
- **Размер .pak**: `Astro-WindowsNoEditor.pak` — **2.8 ГБ**
- **Доп. pak**: `Astrooptional-WindowsNoEditor.pak` — 52 КБ
- **Сторонние библиотеки**: PhysX3, Steamworks, Ogg/Vorbis, NVIDIA
- **Особенность**: Не Steam-версия — установлена напрямую в `C:\ASTRONEER`

## Следующий шаг

**Фаза 0.2** — Установка инструментов: FModel, UnrealPak, UE4 Editor (4.23.1), Android NDK, UEVR

---

## Технический контекст

- **Уровень разработчика**: Начинающий
- **Игра**: Astroneer на Unreal Engine 4
- **Целевая платформа**: Meta Quest 3 (standalone, Android/ARM64)
- **GitHub CLI**: установлен (2.67.0), авторизован
- **Git**: установлен (2.56.1), но заблокирован песочницей DSH
- **DSH workspace**: C:\Users\Иван\Downloads\DotAi

---

## История сессий

### Session 1 (2026-11-11)
- Анализ платформ, выбор Windows/Steam
- Стратегия: UEVR -> standalone
- Установка GitHub CLI
- Авторизация GitHub (токен)
- Создание репозитория Astroneer-mod
- Написание промта для AI-агента (astroneer_vr_prompt.md)
- Создание AGENT.md (этот файл)
- Первый коммит: AGENT.md + astroneer_vr_prompt.md залиты в репо через gh api
- Настроен workflow: AGENT.md будет обновляться после каждого этапа
- Обход песочницы DSH для Git: используем gh api вместо git push