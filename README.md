# ZMK Studio (patched)

Сборка [ZMK Studio](https://github.com/zmkfirmware/zmk-studio) с двумя вещами:

1. **PR [#135 — "Show Layer-Tap/Mod-Tap information on keys"](https://github.com/zmkfirmware/zmk-studio/pull/135)**
   (ветка `BafS:feat/layer-tap-key-1`, коммит `a177a63`).
2. Локальная доработка: **экспорт / импорт раскладки в JSON** (бэкап и восстановление).

## 1. Подписи на клавишах (PR #135)

В стоковом Studio центр клавиши рисуется **только из `param1`** как HID-скан-код.
У `&mo`, `&sl`, `&to`, `&lt` первый параметр — это *номер слоя*, поэтому клавиша
выглядела пустой и было не видно, на какой слой она ведёт.

С патчем значение рендерится по метаданным поведения:

- `&mo 1` → показывает **имя слоя** ("Mouse");
- `&lt 1 SPACE` → tap-клавиша + имя слоя (hold);
- `&mt LGUI SPACE`, `&hml LGUI A` → модификатор + клавиша.

## 2. Экспорт / импорт JSON (локальная доработка)

В шапке появились две кнопки:

- **Download** — сохраняет текущую раскладку в `.json` (`<device>-keymap.json`).
- **Upload** — читает `.json` и записывает раскладку обратно на клавиатуру.

Идея формата взята из upstream-PR
[#179 «load and export a portable keymap file»](https://github.com/zmkfirmware/zmk-studio/pull/179)
(его файлы `portableKeymap.ts` / `applyPortableKeymap.ts`), но без Tauri-диалога:
используется обычное скачивание файла браузером и `<input type="file">`. Tauri/Cargo
не трогали — работает в веб-сборке.

Формат файла (переносимый между клавиатурами):

```json
{
  "format": "zmk-studio-keymap",
  "version": 1,
  "deviceName": "charybdis",
  "exportedAt": "2026-10-12T…",
  "layers": [
    {
      "name": "Base",
      "bindings": [
        { "behavior": "Key Press", "param1": 458756, "param2": 0 },
        { "behavior": "Momentary Layer", "param1": { "layer": 1 }, "param2": 0 },
        { "behavior": "Home Row Mod - Left", "param1": 458851, "param2": 458756 }
      ]
    }
  ]
}
```

Особенности:

- `behavior` — **display-name поведения** (не числовой id), поэтому файл переживает
  пересборку прошивки и переносится на другой экземпляр с теми же поведениями.
- Ссылки на слои хранятся как `{ "layer": <индекс> }`, а не как числовой id — тоже
  переносимо.
- Импорт проверяет: число слоёв/клавиш, имена слоёв, известность поведений и
  корректность ссылок на слои; при несовпадении файл отклоняется с понятной ошибкой.
- Импорт **сразу сохраняет** раскладку на клавиатуре (`saveChanges`). Вернуться к
  стоку можно через «Restore Stock Settings».
- Требуется **разблокировать** клавиатуру (Unlock) перед экспортом/импортом.

### Известные ограничения

- Кнопки и логика — локальная доработка, не отправленная апстрим.
- Поведения сопоставляются по `displayName`; если два поведения имеют одинаковое
  отображаемое имя, импорт сообщит об этом.
- Слои/клавиши должны совпадать по количеству с целевой клавиатурой.

## Как пересобрать

```
git clone https://github.com/zmkfirmware/zmk-studio
git checkout a177a63e7b332e19896b511104f5df3737dadfee   # PR #135
git apply LOCAL-changes.patch                            # base-path + экспорт/импорт JSON
npm ci && npm run generate-data && npx tsc
npx vite build --base=/zmk-studio-patched/
```

- `PR135.diff` — диф PR #135 на момент сборки.
- `LOCAL-changes.patch` — все локальные изменения поверх PR #135
  (base-aware пути к `zmk.svg` / иконке / шрифту + экспорт/импорт JSON).

Содержимое `dist/` просто опубликовано в этой ветке `gh-pages`.

## Подключение

- HTTPS-контекст обязателен для Web Serial / Web Bluetooth — GitHub Pages его даёт.
- **Компьютер (Chrome/Edge):** по USB (Web Serial) к донглу.
- **Android:** Chrome не поддерживает Web Serial — подключайтесь по **Bluetooth**
  (донгл рекламирует ZMK Studio RPC; в ZMK это включено по умолчанию). iOS/Safari
  Web Bluetooth не умеет.
