# UI_ANALYSIS.md — Анализ UI-слоя Astroneer для VR-мода

**Агент:** В (DeepSeek V4 Pro)
**Фаза:** 2 — Анализ UI/ресурсов (Widget Blueprints, HUD, текстуры)
**Версия игры:** Astroneer 1.44.4.0 Shipping
**Движок:** Unreal Engine 4.27 (`GAME_UE4_27`)
**Целевая платформа:** Meta Quest 3, standalone ARM64/Android (прототип — UEVR на ПК)
**Дата:** 2026-11-12

---

## 0. Краткое резюме (TL;DR)

1. UI-слой Astroneer — **2407 ассетов**, из них **1587 (66%) — текстуры/иконки**, а не виджеты. Реальных Widget Blueprint'ов существенно меньше, чем кажется по общему числу.
2. **UMG используется 529 раз** — UI построен на стандартном Unreal Motion Graphics. Это хорошая новость: UMG в UEVR поддаётся адаптации, не требуется переписывать UI-фреймворк.
3. **Критическая проблема для VR: в игре нет ни одного VR-специфичного ассета.** Нет иконок VR-контроллеров (Oculus/Quest/OpenXR), нет world-space UI, нет лазерных указателей. Всё заточено под геймпад (Xbox/PlayStation/Switch) и клавиатуру/мышь.
4. **Вторая критическая проблема: не найден отдельный виджет инвентаря/рюкзака.** Судя по именам, инвентарь встроен внутрь `HUDInstance` — это монолит, который в VR придётся вскрывать и разделять. Требуется подтверждение через экспорт Properties.
5. **Данные о внутренней структуре виджетов отсутствуют.** Properties (.json) не экспортированы, `.uasset` — бинарники, локальный shell не работает. Все выводы о иерархии ниже сделаны по **именам файлов, путям и счётчикам зависимостей** и явно помечены как предварительные.

---

## 1. Методика и достоверность данных

### 1.1. Источники

| Источник | Объём | Что даёт |
|---|---|---|
| `pak_list_1444.txt` | 46 325 строк (23 162 ассета ×2: `.uasset` + `.uexp`) | Полный манифест пака 1.44.4.0 |
| Экспорт FModel `C:\ASTRONEER\tools\astroneer_assets\` | 44 287 файлов / 4 562 МБ | Дерево ассетов на диске |
| `Astro\Content\UI\` | **2407 `.uasset`** | Предмет анализа |
| `script_dependencies.json` | 23 C++ модуля, 17 090 ассетов | Зависимости от `/Script/` (замена SDK dump) |
| `claude_architecture_data.json` | 16 202 файла | Метаданные путей/размеров |
| `astroneer_configs.txt` | 108 799 Б | DefaultEngine/Game/Input.ini |
| `astroneer_vr_prompt.md` | 8 641 Б | ТЗ на VR-порт (Фазы 0–1) |

### 1.2. Что **не** удалось получить (важно для честности выводов)

- **Нет Properties (.json) экспорта.** В дереве экспорта всего два `.json` — `WwiseAudio\Windows\PluginInfo.json` и `SoundbanksInfo.json`. То есть проход «Properties» в FModel **не запускался**. Внутренняя структура виджетов (WidgetTree, привязки, layout) неизвестна.
- **Нет декомпилированных Blueprint'ов** (`.copy`/`.txt` от FModel).
- **Нет `.png`-текстур.** Текстуры существуют как `.uasset`-объекты (`T_UI_*`, `ui_icon_*`), но графические данные наружу не выгружены.
- **SDK dump недоступен** — требует запуска игры (нужен Steam) и нажатия INSERT → Dump SDK. Никогда не выполнялся.
- **Локальный shell (pwsh) не работает**: `SetNamedSecurityInfoW failed (Win32 5): grantWrite(C:\AI\Astroneer mod)`. Поэтому UAssetGUI/FModel CLI не запускались.

### 1.3. Как читать этот документ

Утверждения делятся на три класса:
- **[ФАКТ]** — подтверждено путём/счётчиком/цифрой из данных.
- **[ВЫВОД]** — логически следует из имён файлов и структуры папок, но не подтверждено содержимым.
- **[ГИПОТЕЗА]** — предположение, требующее проверки (указан способ проверки).

---

## 2. Числовая карта UI-слоя

**[ФАКТ]** Разбивка `Astro\Content\UI\` по папкам (измерено через glob/grep, сумма сходится к 2407):

| Папка | Файлов | Доля | Назначение |
|---|---:|---:|---|
| Корень `UI\` | 41 | 1.7% | HUD, скрины загрузки, курсоры, сплэши |
| `content\` | 292 | 12.1% | Иконки ресурсов, курсоры, кнопки-меши, компасы |
| `PauseMenu\` | 288 | 12.0% | Всё меню паузы, настройки, Custom Game |
| `Textures\` | **1587** | **65.9%** | Иконки/атласы/панели — **основная масса** |
| `Tutorial\` | 96 | 4.0% | Обучающие карточки, QuickHelp, Astropedia |
| `MicroTxnStore\` | 18 | 0.7% | Магазин микротранзакций |
| `fonts\` | 16 | 0.7% | Шрифты (NDAstroneer, Cica, Arial, DroidSans) |
| `Interactions\` | 12 | 0.5% | Универсальный тултип + компоненты |
| `Photos\` | 12 | 0.5% | Виджеты фоторежима и фильтры |
| `Notifications\` | 11 | 0.5% | Уведомления/тосты + кривые анимации |
| `CreativeMode\` | 7 | 0.3% | Креативные чекбоксы/слайдеры/меню |
| `CreativeDrone\` | 5 | 0.2% | Оверлей дрона + подсказки управления |
| `ComponentWidgets\` | 5 | 0.2% | Переиспользуемые WBP-компоненты |
| `ActionWheel\` | 4 | 0.2% | Колесо действий (Q-wheel) |
| `EmoteWheel\` | 3 | 0.1% | Колесо эмоций |
| `UIMeshes\` | 6 | 0.2% | 3D-меши UI (DataBar, item_selector) |
| `FunctionLibraries\` | 1 | 0.04% | `MTXStoreFunctionLibrary` |
| `Intro\` | 1 | 0.04% | `IntroCinematicUI` |
| `ModuleIcons\` | 1 | 0.04% | Тестовая иконка компаса |
| `Video\` | 1 | 0.04% | `LoadingSpinner` |
| **Итого** | **2407** | 100% | |

**[ВЫВОД]** Ключевой инсайт для планирования: **две трети UI-ассетов — это статика (иконки/текстуры), а не логика**. Для VR-порта это означает, что основной объём работ — не «переделать 2400 виджетов», а:
- переработать **~40–60 экранных виджетов** (HUD, меню, колёса),
- перегенерировать/дорисовать **VR-иконки управления** (сейчас их 0),
- адаптировать **529 UMG-зависимостей** к world-space рендерингу.

---

## 3. Widget Blueprints: что реально найдено

**[ФАКТ]** По префиксу `WBP_` найдено **43 файла**:

| Папка | Кол-во | Файлы |
|---|---:|---|
| `MicroTxnStore\` | 18 | `StoreMenu`, `StoreTile`, `StoreTilePanel`, `StoreBuyPanel`, `StoreSubTabBar`, `StoreSubTabCategory`, `BundleContent`, `BundleContentPanel`, `BundleSubItem`, `BundleTile`, `CostPanel`, `DescriptionBanner`, `FillerTile`, `LoadingSpinner`, `PurchaseCurrencyDialog`, `TileRibbon`, `VirtualCurrencyTile`, `VirtualCurrencyFeaturedTile` |
| `PauseMenu\SubMenus\CustomizationMenu\` | 14 | `CustomizationMenuPane`, `CustomizationActionBar`, `CustomizationDescriptionBanner`, `CustomizationSubTabBar`, `CustomizationSubTabCategory`, `CustomizationTile`, `CustomizationTileEmoteFlag`, `CustomizationTileGrid`, `EmoteSlot`, `EmoteSlotPanel`, `NewOutfitTile`, `OutfitButton`, `OutfitSubItem`, `OutfitTile` |
| `ComponentWidgets\` | 5 | `WBP_HorizontalDragDistance`, `WBP_KeybindingImage`, `WBP_MonospacedTimeDisplay`, `WBP_MonospacedQBitDisplay`, `WBP_PlatformAwareKeyImage` |
| `CreativeDrone\` | 5 | `ControlHint`, `ControlHint_Toggle`, `ControlHint_Vertical`, `ControlHintPanel`, `CreativeDroneOverlay` |
| `Tutorial\` | 2 | `Tutorial\Expansion\WBP_ExpansionPage`, `Tutorial\U36_Expansion\WBP_MegaTech_ExpansionPage` |
| Корень `UI\` | 1 | `WBP_PlanetSelectionHUD` |
| `Notifications\` | 1 | `WBP_Toast` |

**[ВЫВОД]** Префикс `WBP_` — **не** универсальный маркер в этом проекте. Основная масса виджетов названа без него: `HUDInstance`, `GameMenuRoot`, `ActionWheel`, `EmoteWheel`, `CompassWidget`, `UniversalTooltipWidget`, `NotificationBannerWidgetBP` и т.д. Если искать «виджеты» только по `WBP_`, потеряется ~90% UI. Это существенный вывод для оценки объёма: **фактическое число виджетов существенно больше 43**.

**[ГИПОТЕЗА]** Реальное число Widget Blueprint'ов — порядка **300–500** из 2407 (остальное: текстуры 1587, шрифты 16, меши/анимации/скелеты в `content\buttons`, кривые анимации, material-инстансы).
*Как проверить:* FModel → правый клик по `Astro\Content\UI` → **Properties (.json)**; в JSON поле `$type` = `WidgetBlueprintGeneratedClass` / `WidgetBlueprint` однозначно отделит виджеты. Ожидаемый объём экспорта — десятки МБ.

---

## 4. HUD и игровой слой (корень `UI\`)

**[ФАКТ]** 41 файл в корне `Astro\Content\UI\`. Разбивка по функциональным группам **[ВЫВОД по именам]**:

### 4.1. Главный HUD
| Ассет | Предполагаемая роль | VR-статус |
|---|---|---|
| `HUDInstance` | **Корневой виджет HUD** в игре | 🔴 Требует вскрытия и разделения |
| `PlayWidgetInstance` | Корневой виджет игрового сеанса | 🔴 То же |
| `WBP_PlanetSelectionHUD` | HUD выбора планеты | 🟡 Средне |
| `CompassWidget` | Компас | 🟢 Хорошо переносится |
| `ResourceWidget` | Отображение ресурсов | 🟡 Нужна привязка к руке |
| `SymbolWidget`, `SymbolPingWidget` | Символы/пинги на HUD | 🟢 Ок |
| `SessionOperationInProgressWidget` | Индикатор операции сессии | 🟢 Ок |
| `ErrorWidget` | Экран ошибки | 🟢 Ок |
| `lowoxygen`, `unpowered` | Индикаторы кислорода/энергии | 🟡 Дублируются на запястье |
| `PowerPingActorChargingMesh`, `PowerPingActorDischargingMesh` | 3D-меши пингов энергии | 🟢 3D — уже хорошо |

**[ВЫВОД]** `HUDInstance` — вероятный **монолит**: в Astroneer HUD содержит и полоски ресурсов, и инвентарь рюкзака, и подсказки взаимодействия. Для VR его придётся декомпозировать на world-space панели (запястье / перед игроком). Это **самая дорогая единичная задача** UI-порта.

### 4.2. Экраны загрузки и переходов
`LoadScreen`, `PAXLoadScreen`, `InterMapLoadScreen`, `FadeToBlackScreen`, `BlackBackground`, `LoadingSpinner`, `LoadingSpinner2`, `InterMapLoadingSpinner`, `PAXMapLoadingSpinner`, `LaunchSequenceLoadingSpinner`, `LogoSplash`, `LogoSplashActor`, `PressAToStart`, `IntroCinematicUI`.

**[ВЫВОД]** В VR **обязательны** к переработке: плоский 2D-лоадскрин в шлеме вызывает дискомфорт. Рекомендация — заменить на стерео-панораму/cubemap + пространственный спиннер. Объём: **низкий**, эффект: **высокий** (дешёвая победа).

### 4.3. Курсоры
`Cursor_Main_01`, `Cursor_Main_Small`, `GamepadCursor`; в `content\cursors\` — ~30 файлов (кисти терраформера, руки, захват).

**[ВЫВОД]** В VR 2D-курсор **не нужен** — заменяется лучом из контроллера. Однако `content\cursors\` содержит деки/кисти терраформера (`cursor_brush_ring_01a…05ccc`, `icon_glove_open`, `icon_hand_close/open`) — это **готовый визуал для VR-рук и превью кисти**. Переиспользуется почти без затрат.

### 4.4. Диалоги и предупреждения
`ConfirmLogout`, `ReconnectController`, `UnsupportedSaveWarningDialog`, `CustomGameNoRespawnsDeathScreen`, `AvatarButton`, `LaunchButton`, `PlayTogether`, `ExperimentalWatermark(+Actor)`, `EarlyAccessWatermark`.

**[ВЫВОД]** Плоские модальные окна → в VR нужен world-space диалог с лазерным указателем. `ExperimentalWatermark`/`EarlyAccessWatermark` в VR лучше **отключить** (лишний оверлей в глазах).

### 4.5. Шрифты (корень + `fonts\`)
`ASTROFONT`, `AstroFont_Bold` (корень) + 16 в `fonts\`: `NDAstroneer-Bold_V2`, `NDAstroneer-Regular_V2`, `NDAstroneer-Light_V2`, `NDAstroneer-Regular_Font`, `NDAstroneer-Bold_0_Font`, `AstroneerFont`, `Cica-Regular`, `Cica-Regular_reduced`, `Cica-Bold`, `arial`, `arialbd`, `arialbi`, `ariali`, `ariblk`, `bullet`, `DroidSansFallback`.

**[ФАКТ]** `DroidSansFallback` — признак локализации под CJK.
**[ВЫВОД]** Для VR критично: **мелкий текст в шлеме нечитаем**. Потребуется либо увеличить кегль/перекомпоновать layout, либо перейти на контролируемый угловой размер (≈0.5–0.8° на строку). `DroidSansFallback` — тяжёлый шрифт (полный CJK), на Quest может стоить памяти; проверить в профилировщике.

---

## 5. Меню паузы (`PauseMenu\`, 288 файлов)

**[ФАКТ]** Структура:
- корень `PauseMenu\` — 8 файлов (`GameMenuRoot`, `TitleBanner`, `FullScreenMOTD`, `ControllingUserDisplay`, `FriendsInGameListEntry`, `ExperimentalSettingMenuWarning`, `CreativeModeProgressionWarning`, `InsufficientSpaceForSaveFileWarning`);
- `PauseMenu\Components\` — ~44 переиспользуемых компонента (`AstroSlider`, `AstroMenuScrollBar(+Container)`, `GameMenuButtonWidget`, `GameMenuEntry*`, `GameMenuSection*`, `GameMenuTabBar(+Button_Base)`, `GameMenuColorPicker`, `GameMenuStylableTextBox`, `GameMenuDashedDivider`, `GameMenuEntry_DiscreteInputBinding` …);
- `PauseMenu\CustomGameMode\` + `Submenus\astropedia\` — крупнейшее поддерево (настройки Custom Game, Astropedia: планеты, сила тяжести, кислород, энергия, мегаструктуры);
- `PauseMenu\Popouts\PopoutContents\` — попапы (`ServerAdminManageListedPlayer`, `ServerAdminManageSavedGame`, `ServerNewGame`, `ServerSaveGame`, `StartNewGame`, `SwitchReconnect`);
- `PauseMenu\SubMenus\CustomizationMenu\` — 14 `WBP_*` кастомизации (скины, эмоции, аутфиты);
- `PauseMenu\UpsellDisplay\` — апселл (`T_REMOVE_ME_ExpansionTestImage` — тестовая заглушка, **REMOVE ME** прямо в имени).

**[ВЫВОД]** `GameMenuRoot` + `Components\` — **самая VR-дружелюбная часть UI**: это классический список с табами и скроллом. Перенос в world-space панель перед игроком реалистичен за **1–2 недели** работы без переписывания логики, если сохранить UMG-дерево и поменять только «хост» (WidgetComponent на motion controller / плавающая панель).

**[ВЫВОД]** `GameMenuEntry_DiscreteInputBinding` — **точка вмешательства для VR-биндингов**: здесь логично добавить маппинг «кнопка Touch → игровое действие».

⚠️ **Custom Game / Astropedia (крупнейшее поддерево меню)** — в VR низкоприоритетно. Можно **отложить на пост-релиз**: это контент для создателей миров, не для типичного VR-игрока. Экономия значительная.

---

## 6. Инвентарь и рюкзак — **пробел в данных**

**[ФАКТ]** В `Astro\Content\UI\` **нет** отдельного виджета с именем `Inventory*`, `Backpack*`, `ItemSlot*`, `Storage*`.

**[ГИПОТЕЗА]** Инвентарь/рюкзак реализован **внутри `HUDInstance`** (и/или `PlayWidgetInstance`), т.е. монолитно. Слоты предметов, скорее всего, построены динамически из C++-классов `/Script/Astro` (4119 обращений) без отдельных Blueprint-ассетов.

**Почему это критично для VR:** инвентарь — **центральный элемент** VR-взаимодействия (в Astroneer это «рюкзак на спине» → в VR идеально ложится на **запястье/тыльную сторону руки** или **физический рюкзак за спиной**). Если он зашит в `HUDInstance` без выделенных виджетов, потребуется:
1. вытащить логику слотов из монолита (риск: правка Blueprint'а размером в весь HUD);
2. либо **не трогать** монолит и рендерить `HUDInstance` целиком в world-space (компромисс: HUD «висит» перед лицом, но работает быстро).

**Способ проверки (обязательный):** экспорт Properties для `HUDInstance.uasset` → посмотреть `WidgetTree` на наличие дочерних `Inventory*`/`Backpack*`/`Slot*`.

---

## 7. Колёса действий и эмоций — лучший кандидат для VR

**[ФАКТ]**
- `ActionWheel\` (4): `ActionWheel`, `ActionWheelOption`, `ActionWheelOptionPlaceholder`, `ActionWheelRing`.
- `EmoteWheel\` (3): `EmoteWheel`, `EmoteWheelOption`, `EmoteWheelRing`.
- Текстуры: `Textures\Elements\ui_element_radial_ring_arc`, `ui_element_radial_ring_tab`.
- Туториалы: `Tutorial_ActionWheel_Gamepad/Keyboard`.
- В бинарнике `ActionWheel.uasset` удалось прочитать ASCII-строку **`GetAllModesOptionCount`** (функция-геттер количества опций) — подтверждает, что это **динамически заполняемое радиальное меню**, а не статичный экран.

**[ВЫВОД]** Радиальные колёса — **идеальный паттерн для VR**. Их не надо переделывать: достаточно отрендерить `ActionWheel` как world-space WidgetComponent, привязанный к контроллеру, и заменить ввод (мышь/стик → луч контроллера). Это **самый дешёвый и самый заметный** выигрыш в UX.

**Рекомендация:** начать VR-UI именно с `ActionWheel` — низкий риск, быстрый результат, проверка гипотезы «UMG в world-space работает».

---

## 8. Тултипы и взаимодействия (`Interactions\`, 12)

**[ФАКТ]**
- `UniversalTooltipWidget`
- `TooltipComponents\`: `TooltipBadgeWidget`, `TooltipInteractionPromptWidget`, `TooltipTitleWidget`, `TooltipDescriptionWidget`, `TooltipContentAreaWidget`, `TooltipContentInteractionPromptSpacer`, `TooltipContentSubsectionSpacer`
- `TooltipAnimationCurves\`: `BadgelessTooltipContentAreaPresentationAlphaCurve`, `StandardTooltipBadgePresentationScaleCurve`, `StandardTooltipConnectingLinePresentationScaleCurve`, `StandardTooltipContentAreaPresentationAlphaCurve`

**[ВЫВОД]** Архитектура тултипов **хорошо разложена на компоненты** — это плюс. Обратите внимание на `StandardTooltipConnectingLinePresentationScaleCurve`: тултип уже умеет рисовать **соединительную линию** между якорем и панелью. В VR это почти готовая механика «панель у объекта + линия к нему» — переиспользуется напрямую.

`/Script/AkAudio` встречается у `UniversalTooltipWidget` — значит тултип проигрывает звук; в VR **пространственный звук** должен быть привязан к позиции объекта.

---

## 9. Уведомления (`Notifications\`, 11)

**[ФАКТ]** `AstroPopupBadgeWidgetBP`, `NotificationBannerWidgetBP`, `NotificationTagWidgetBP`, `NotificationUnlockWidgetBP`, `NotificationToastWidgetBP`, `WBP_Toast` + `AnimationCurves\FullTag\*` (alpha/translation/lift) + `AnimationCurves\MiniBadge\*` (drift/alpha).

**[ВЫВОД]** В VR системы уведомлений **плоские тосты в углу экрана не работают** (пользователь не смотрит в угол). Нужен перенос в world-space: бейдж у объекта-источника либо на запястье. Кривые анимации (`TranslationCurve`, `LiftCurve`) придётся пересчитать под новую систему координат — но сами кривые переиспользуемы как шаблон.

Зависимость от `/Script/AkAudio` (через `AstroNotificationDatabase`) — звуковые уведомления, привязать к 3D-позиции.

---

## 10. Магазин микротранзакций (`MicroTxnStore\`, 18) — вне VR-скоупа

**[ФАКТ]** 18 `WBP_*` (см. §3) + `FunctionLibraries\MTXStoreFunctionLibrary`.

**[ВЫВОД]** Микротранзакции в standalone VR-моде **не нужны** (другая платёжная модель, требование Meta). Рекомендация: **исключить из VR-сборки целиком**. Это ~18 виджетов + фанк-либа, которые можно просто не подключать. Экономия: малая по объёму, но снимает вопрос платёжных SDK.

⚠️ Требует аккуратности: если `MTXStoreFunctionLibrary` вызывается из общего кода (не только магазина), «вырезать» не получится — только скрыть UI. **Проверить:** экспорт Properties/Blueprint-ссылок на `MTXStoreFunctionLibrary`.

---

## 11. Обучающая система (`Tutorial\`, 96) — крупный скрытый блок

**[ФАКТ]** Структура:
- Корень `Tutorial\` — 54 файла: `TutorialScreenBase`, `TutorialCardWrapper`, `AstropediaTutorialCardCarousel`, `CarouselPip`, `CarouselPipBar`, `CarouselPipBar_Vertical`, `PersistentObjective{Goal,Container,Header}` + ~22 `QuickHelp_*` (Controls/Terrain/Inventory/Interact/Resources/Flora/Planet) + ~18 `Tutorial_*` (Crafting, Printers, BaseBuilding, Power, Missions, Rails, LogicItems, AutoItems, SoilExcavation, ResearchPlatform, ResearchCatalog, DamageAndRespawn, ActionWheel).
- `Components\` — 13: `TutorialCaptionBox(+Creative)`, `TutorialTextBox(+_WithImage)`, `TutorialInputBox(+Double)`, `TutorialKeyImage`, `TutorialKeybindingImage`, `TutorialKeybindingDisplayData`, `TutorialPlusLabel`, `TutorialSectionHeaderBoxAutosized`, `TutorialCreativeControlBinding`, `TutorialCreativeGameplayOption`.
- `Creative\` — 7: `CreativeCatalog`, `CreativeControls`, `CreativeNewInteractions`, `CreativeNewItems`, `CreativePauseMenu`, `CreativeTerrainTool(+Window)`.
- `Expansion\` — 3 (`QuickHelp_GW_Tutorial_Intro`, `WBP_ExpansionPage`, `Glitchwalkers_Final_Art_16x9_`) + `U36_Expansion\` 18 (Astropedia Megatech: Biodome, DLS, Museum, IMT, OP + `Tutorial_FreeVsPaid`).
- `Tutorial\Expansion\T_keyart_glitchwalker_aug_21__2_` — текстура.

**[ВЫВОД]** Почти **каждый** туториал существует в двух вариантах: `_Gamepad` и `_Keyboard`. Это **готовый шаблон для третьего варианта — `_VR`**: структура уже параметризована по типу ввода (`TutorialKeybindingDisplayData`, `WBP_PlatformAwareKeyImage`, `TutorialKeybindingImage`).

**Рекомендация:** не переписывать туториалы, а **добавить ветку `VR`** в существующую систему вариаций. Это резко снижает объём работ (96 файлов → правка ~10 компонентов + тексты).

---

## 12. Креативный режим и дрон (`CreativeMode\` 7, `CreativeDrone\` 5)

**[ФАКТ]** `CreativeCheckboxWidget`, `CreativeSliderWidget`, `CreativeColorPickerButton`, `CreativeTerrainColorPicker`, `TerrainToolCreativeMenu`, `TerrainToolCreativeQuitButton`, `TerrainToolNullColorPickerButton`; `WBP_CreativeDroneOverlay`, `WBP_ControlHint(+_Toggle/_Vertical)`, `WBP_ControlHintPanel`.

**[ВЫВОД]** `CreativeDrone` — это **свободный полёт камеры**, что в VR почти бесплатно (наклон головы = полёт). Низкий приоритет, но технически просто. `TerrainToolCreativeMenu` — сложный (кисти/цвета), отложить.

---

## 13. Фоторежим (`Photos\`, 12)

**[ФАКТ]** `SelfiePhotoWidget`, `CameraPhotoWidget_Default`, `LEMPhotoWidget`, `LEM50PhotoWidget`, `SwitchCloseMessageWidget` + фильтры `Filter_Default/Greyscale/BlueSepia/PostCard/Holiday2019/Wanderer/LEM50`.

**[ВЫВОД]** В VR фоторежим — **сильный аргумент продаж** (стерео-скриншоты, 180°/360° панорамы). Но `Selfie` и `SwitchCloseMessageWidget` (привязка к железу Switch) — переделывать. Средний приоритет, **высокая ценность**.

---

## 14. Текстуры и иконки (`Textures\`, 1587 — 66% всего UI)

**[ФАКТ]** Крупные поддеревья:
- `Textures\Icons\` — **845** файлов, в т.ч. `Actions` (~50), `Atlas` (4 атласа: `icon_atlas_buttons_1/2/megatech/mission_log`), `Canisters` (7), `Controller\{NX, Playstation, Xbox}`, `Emotes` (7), `Gate` (4), `Generic` (~17), `Glyphs` (9), `Items` (~80), `Megatech` (~90), `Menu\Tab`, `Mouse-Kb`, `Sprites` (2 MI: `ui_flipbook_dna_icon`, `ui_flipbook_dualmonitor`);
- `Textures\ButtonsKeys\` — 13 (`ui_button_xbox_*`, `ui_key_*`, `ui_mouse_*`);
- `Textures\ControlPanels\` — ~21 (`UI_Panel_Background_Radius*`, `UI_Panel_Background_Square/Triangle/Grid_Tile`, `UI_Panel_Bracket_Radius`, `UI_Panel_Arrow_Curved`) + `RotatorMaterial`;
- `Textures\Elements\` — 5 (`ui_element_radial_ring_arc/tab`, `ui_element_use_circle_inner/outer`, `ui_element_use_square_inner/outer`);
- `Textures\GameMenu\` — 2 (`menu_checkbox_empty`, `terrain_color_select_outline`);
- корень `Textures\` — 3 (`ui_drill_arrow_01a`, `ui_drill_line_01a`, `ui_drill_tilt_indicator`);
- `content\Compass\` — `T_UI_Compass_Rings`, `M_UI_Compass_*`, `MI_UI_Compass_*`, `UI_Compass_Navpoint_{Square,Circle,Teardrop,Triangle,Hexagon}`, `UI_Compass_Elevation_Line`.

### 14.1. 🔴 Ключевая находка для VR: иконки ввода только под 2D-платформы

**[ФАКТ]** Полный список семейств иконок управления:
- **Xbox**: `ui_icon_xb_pad_a/b/x/y` (+цветные варианты), `dpad_*`, `bumper_*`, `trigger_*`, `thumbstick_*`, `view_*`
- **PlayStation**: `ui_icon_ps_pad_{cross,circle,square,triangle}`, `ps5_{create,options,touch_pad}`, `ps_dpad_*`, `ps_pad_l1/l2/r1/r2`, `ps_thumbstick_*`, `ps_share`
- **Switch (NX)**: `ui_icon_switch_button_{b,l,r,zl,zr}`, `bumper_*`, `dpad_*`, `dir_*`, `stick_*`, `trigger_*`, `plus/minus`
- **Клавиатура/мышь**: `content\pc\` (`icon_pc_key{A,D,E,Q,R,S,T,W,SHIFT,SPACE,TAB}`, `icon_pc_mouse_{lc,rc,mw}`, `pc_lmb/rmb/mmb`, `Mouse_Keyboard`), `Textures\ButtonsKeys\ui_key_*`, `ui_mouse_*`
- **VR**: **отсутствует полностью.** Поиск по `oculus|quest|vive|steamvr|openxr|vr_|_vr|motion` дал 0 совпадений (2 ложных — `ui_icon_tab_question`, `ui_icon_kb_question`).

**[ВЫВОД]** Для VR-порта **обязательно создать с нуля** семейство иконок VR-контроллеров: как минимум `ui_icon_vr_trigger`, `_grip`, `_a`, `_b`, `_x`, `_y`, `_thumbstick`, `_menu`, `_system`, `_stick_press`. Без них все подсказки управления (`TutorialInputBox`, `WBP_PlatformAwareKeyImage`, `WBP_KeybindingImage`, `GameMenuEntry_DiscreteInputBinding`) будут показывать VR-игроку иконки геймпада — **это ломает UX сильнее, чем отсутствие 6DOF**.

### 14.2. Почему текстур «нет» в отчёте Фазы 0

**[ФАКТ]** `.png` в экспорте — 0. Текстуры есть как `.uasset`-объекты, но пиксельные данные не выгружены.
**[ВЫВОД]** Для проверки размеров/атласов/сжатия нужен **отдельный проход FModel → Textures**. Без него нельзя оценить:
- разрешение иконок (хватит ли для VR-читаемости);
- наличие мип-уровней;
- вес VRAM (критично для Quest 3: ~4–5 ГБ доступной памяти).

---

## 15. Зависимости от C++ (`/Script/`) — что это значит для UI

**[ФАКТ]** Из `script_dependencies.json` (17 090 ассетов):

| Модуль | Использований | Значение для UI |
|---|---:|---|
| `/Script/CoreUObject` | 16 123 | Базовые типы |
| `/Script/Engine` | 15 261 | Акторы, WidgetComponent, компоненты |
| `/Script/Astro` | 4 119 | **Игровая логика: инвентарь, слоты, предметы** |
| `/Script/NavigationSystem` | 1 954 | (уникально: UI-кнопки-меши используют Nav) |
| `/Script/Terrain2` | 966 | Воксели/терраформинг (в UI — `PauseMenu\CustomGameMode\*Astropedia*`) |
| **`/Script/UMG`** | **529** | **UI-фреймворк** |
| `/Script/GameplayTags` | 424 | Тэги (Custom Game, модификаторы) |
| `/Script/AkAudio` | 263 | Wwise-звук (лоадскрины, тултипы, уведомления) |
| `/Script/SlateCore` | 140 | Стили/шрифты/курсоры (`ConfirmLogout`, `GamepadCursor`, `InterMapLoadingSpinner`, `CustomGameNoRespawnsDeathScreen`) |
| `/Script/InputCore` | 127 | Ввод (`PlayControllerInstance`, `CustomGameNoRespawnsDeathScreen`) |

**[ВЫВОД]** Важные следствия:
1. **UMG 529** — UI на стандартном стеке. UEVR умеет рендерить UMG в world-space, значит базовый путь есть.
2. **`/Script/Astro` 4119** — логика инвентаря/предметов в C++. Значит UI **нельзя переписать на Blueprint'ах целиком** — он дёргает нативные классы. В standalone-порте эти классы придётся либо линковать (невозможно без исходников), либо реверс-инжинирить. **Это главный архитектурный риск проекта, и он затрагивает именно UI.**
3. **`/Script/Terrain2` внутри `PauseMenu\CustomGameMode\*`** — меню настроек Custom Game напрямую зависит от воксельного модуля. В Android-порте Terrain2 — самая тяжёлая часть. **Практический вывод: Custom Game меню в VR-сборку не тащить.**
4. **`/Script/AkAudio` 263** — Wwise. Для Quest 3 нужен Wwise Android-рантайм; UI-звуки (тултипы, уведомления, лоадскрины) без него просто замолчат. Учесть в бюджете.

---

## 16. Оценка объёма VR-адаптации

### 16.1. Сводная таблица по категориям

| # | Категория | Ассетов | Сложность VR | Приоритет | Оценка* |
|---|---|---:|---|---|---|
| 1 | Колёса `ActionWheel`/`EmoteWheel` | 7 | **Низкая** | 🔴 Высокий | 3–5 дней |
| 2 | Меню паузы `GameMenuRoot` + Components | ~52 | **Низкая** | 🔴 Высокий | 1–2 недели |
| 3 | Загрузочные экраны / fade | ~14 | **Низкая** | 🔴 Высокий | 2–4 дня |
| 4 | **HUD `HUDInstance`/`PlayWidgetInstance`** | 2 (монолиты) | **Очень высокая** | 🔴 Высокий | 3–6 недель |
| 5 | **Инвентарь/рюкзак** (внутри HUD?) | ? | **Очень высокая** | 🔴 Критический | 3–6 недель |
| 6 | VR-иконки управления (создать с нуля) | 0 → ~10 | **Средняя** (арт) | 🔴 Критический | 1–2 недели |
| 7 | Тултипы `Interactions\` | 12 | **Низкая** | 🟡 Средний | 4–7 дней |
| 8 | Уведомления `Notifications\` | 11 | **Средняя** | 🟡 Средний | 1 неделя |
| 9 | Туториалы `Tutorial\` (ветка `_VR`) | 96 | **Средняя** | 🟡 Средний | 2–3 недели |
| 10 | Курсоры → VR-руки/луч | ~35 | **Средняя** | 🟡 Средний | 1–2 недели |
| 11 | Фоторежим `Photos\` | 12 | **Средняя** | 🟢 Низкий (но ценность высокая) | 1–2 недели |
| 12 | Креатив `CreativeMode\`/`CreativeDrone\` | 12 | **Средняя** | 🟢 Низкий | 1 неделя |
| 13 | Интро-синематик `IntroCinematicUI` | 1 | **Высокая** (стерео 360) | 🟢 Низкий | 1–2 недели |
| 14 | Custom Game / Astropedia | ~150 | **Очень высокая** (Terrain2) | ⚪ Отложить | — |
| 15 | Магазин `MicroTxnStore\` | 19 | — (**исключить**) | ⚪ Отложить | 0 |
| 16 | Текстуры/атласы (ре-экспорт + VR-читаемость) | 1587 | **Средняя** | 🟡 Средний | 1–2 недели |

\* Оценка для одного разработчика-новичка с ИИ-ассистом; **не включает** работу над рендером/движком.

### 16.2. Реалистичный итог

**[ВЫВОД]** Минимально жизнеспособный VR-UI (пункты 1, 2, 3, 6, 7 + частично 4):
**≈ 6–10 недель**. Полный перенос всего UI (без п.14–15): **≈ 4–6 месяцев**.

Это **согласуется** с предупреждением из `astroneer_vr_prompt.md` («6–12 месяцев для команды профессионалов») и подтверждает рекомендацию ТЗ: **первые 3 месяца — только прототип на UEVR**, без попыток standalone-порта.

---

## 17. Риски (отсортированы по опасности)

| Риск | Влияние | Вероятность | Митигация |
|---|---|---|---|
| **Инвентарь зашит в монолит `HUDInstance`** | Нет нормального VR-инвентаря → игра неиграбельна | Высокая | Экспорт Properties; при подтверждении — ранняя декомпозиция, до остальных работ |
| **Нет VR-иконок ввода** | VR-игрок видит иконки геймпада; обучение ломается | 100% | Создать арт-пакет VR-иконок в первую очередь |
| **Текст в VR нечитаем** (мелкие кегли, 1080p-layout) | Дискомфорт, невозможность пользоваться меню | Высокая | Аудит размеров после экспорта Properties; переход на угловой размер |
| **Flat-панели в world-space выглядят «наклейками»** | Ощущение дешёвого порта | Средняя | Дизайн-пасс: панели с глубиной, полупрозрачность, привязка к руке |
| **Зависимость UI от `/Script/Astro` (4119)** | Standalone-порт UI невозможен без реверса нативных классов | Высокая (для standalone) | Для прототипа — не проблема (UEVR работает в PC-процессе); для standalone — план реверса |
| **Wwise (`AkAudio`) на Android** | UI-звуки молчат | Средняя | Проверить наличие Wwise Android-рантайма до старта |
| **VRAM-бюджет Quest 3 при 1587 текстурах** | Не хватит памяти | Средняя | Проход Textures → аудит разрешений/мипов; атласирование; стриминг |
| **Custom Game зависит от Terrain2** | Тянет воксельный модуль в VR-сборку | Высокая | Исключить из VR-скоупа (уже рекомендовано) |
| **Данные о структуре виджетов отсутствуют** | Оценки могут вырасти в 2–3× | Высокая | **Немедленно** выполнить экспорт Properties (см. §18) |

---

## 18. Что нужно сделать, чтобы снять неопределённость

Приоритезированный список (все пункты требуют **работающего shell** — сейчас блокер):

1. **Экспорт Properties (.json) для `Astro\Content\UI`** — FModel → правый клик → Properties. Даёт: тип каждого ассета (Widget Blueprint vs текстура vs меш), `WidgetTree`, привязки, размеры шрифтов. **Снимает ~70% неопределённости этого документа.**
2. **Отдельный экспорт Textures** — получить `.png`/`.tga` + метаданные (разрешение, мипы, сжатие, вес VRAM).
3. **Декомпиляция ключевых Blueprint'ов**: `HUDInstance`, `PlayWidgetInstance`, `GameMenuRoot`, `ActionWheel`, `UniversalTooltipWidget` — понять реальную иерархию и найти инвентарь.
4. **Проверка `HUDInstance` на наличие дочерних `Inventory*`/`Backpack*`/`Slot*`** — закрывает критический пробел §6.
5. **Аудит `MTXStoreFunctionLibrary`** — можно ли вырезать магазин целиком.
6. **Замер размера шрифтов в `GameMenu*`** — оценка читаемости в VR.

---

## 19. Рекомендации по порядку работ (VR-UI)

**[ВЫВОД]** Оптимальная последовательность для прототипа на UEVR:

**Шаг 1 (проверка гипотезы, 3–5 дней).** `ActionWheel` в world-space на контроллере. Если работает — весь остальной UMG-план валиден. Если нет — проблема в UEVR/UMG, и её надо решать до всего остального.

**Шаг 2 (дешёвые победы, 1–2 недели).** Лоадскрины (стерео), `GameMenuRoot` в world-space панели, VR-иконки управления.

**Шаг 3 (ядро, 3–6 недель).** `HUDInstance`: вскрытие, декомпозиция, инвентарь на запястье/руке.

**Шаг 4 (полировка, 2–4 недели).** Тултипы, уведомления, туториалы (`_VR`-ветка), VR-руки.

**Вне скоупа:** `MicroTxnStore`, `CustomGameMode`/`Astropedia`, `TerrainToolCreativeMenu`.

---

## 20. Итог для `COORDINATION.md`

- **Статус:** Фаза 2 (UI) — первичный анализ **выполнен** на уровне инвентаризации и классификации.
- **Готово:** полная карта 2407 ассетов, разбивка по 19 папкам, 43 `WBP_*`, HUD-группа, меню, колёса, тултипы, уведомления, шрифты, текстуры, зависимости `/Script/`, оценка объёма и рисков.
- **Блокер:** отсутствие Properties (.json) и текстурного экспорта + нерабочий локальный shell → структурный анализ виджетов **невозможен**.
- **Следующий шаг агента В:** после починки shell — прогнать FModel Properties/Textures и обновить §3–§6 фактами вместо гипотез.

---

## Приложение A. Все `WBP_*` (43) — полный список

```
MicroTxnStore/          WBP_BundleContent, WBP_BundleContentPanel, WBP_BundleSubItem,
                        WBP_BundleTile, WBP_CostPanel, WBP_DescriptionBanner,
                        WBP_FillerTile, WBP_LoadingSpinner, WBP_PurchaseCurrencyDialog,
                        WBP_StoreBuyPanel, WBP_StoreMenu, WBP_StoreSubTabBar,
                        WBP_StoreSubTabCategory, WBP_StoreTile, WBP_StoreTilePanel,
                        WBP_TileRibbon, WBP_VirtualCurrencyFeaturedTile,
                        WBP_VirtualCurrencyTile
PauseMenu/SubMenus/CustomizationMenu/
                        WBP_CustomizationActionBar, WBP_CustomizationDescriptionBanner,
                        WBP_CustomizationMenuPane, WBP_CustomizationSubTabBar,
                        WBP_CustomizationSubTabCategory, WBP_CustomizationTile,
                        WBP_CustomizationTileEmoteFlag, WBP_CustomizationTileGrid,
                        WBP_EmoteSlot, WBP_EmoteSlotPanel, WBP_NewOutfitTile,
                        WBP_OutfitButton, WBP_OutfitSubItem, WBP_OutfitTile
ComponentWidgets/       WBP_HorizontalDragDistance, WBP_KeybindingImage,
                        WBP_MonospacedTimeDisplay, WBP_MonospacedQBitDisplay,
                        WBP_PlatformAwareKeyImage
CreativeDrone/          WBP_ControlHint, WBP_ControlHint_Toggle,
                        WBP_ControlHint_Vertical, WBP_ControlHintPanel,
                        WBP_CreativeDroneOverlay
Tutorial/               Tutorial/Expansion/WBP_ExpansionPage,
                        Tutorial/U36_Expansion/WBP_MegaTech_ExpansionPage
Notifications/          WBP_Toast
UI/                     WBP_PlanetSelectionHUD
```

## Приложение B. Иконки контроллеров — что есть (и чего нет)

```
Textures/Icons/Controller/Xbox/         ui_icon_xb_pad_{a,b,x,y} (+green/red/blue/yellow),
                                        ui_icon_xb_dpad_{up,down,left,right},
                                        ui_icon_xb_bumper_{left,right},
                                        ui_icon_xb_trigger_{left,right},
                                        ui_icon_xb_thumbstick_{left,right}[_c/_D/_L/_R/_U],
                                        ui_icon_xb_view_{b,menu_b}
Textures/Icons/Controller/Playstation/  ui_icon_ps_pad_{cross,circle,square,triangle}(+colors),
                                        ui_icon_ps5_{create,options,touch_pad},
                                        ui_icon_ps_dpad_*, ui_icon_ps_pad_{l1,l2,r1,r2},
                                        ui_icon_ps_thumbstick_*, ui_icon_ps_share
Textures/Icons/Controller/NX/           ui_icon_switch_button_{b,l,r,zl,zr},
                                        ui_icon_switch_{dpad,dir}_*, ui_icon_switch_bumper_*,
                                        ui_icon_switch_trigger_*, ui_icon_switch_{stick,press_stick}_*,
                                        ui_icon_switch_{plus,minus}
Textures/Icons/Mouse-Kb/ + content/pc/  ui_icon_kb_*, icon_pc_key{A,D,E,Q,R,S,T,W,SHIFT,SPACE,TAB},
                                        icon_pc_mouse_{lc,rc,mw}, pc_{lmb,rmb,mmb}, Mouse_Keyboard
────────────────────────────────────────────────────────────────────────────
VR / Oculus / Quest / OpenXR           ❌ НЕ СУЩЕСТВУЕТ — создать с нуля
```

---

*Документ подготовлен на основе: 2407 UI-ассетов, `script_dependencies.json`, `pak_list_1444.txt`, `astroneer_configs.txt`, `astroneer_vr_prompt.md`. Структура виджетов не проверена инструментально — см. §1.2 и §18.*
