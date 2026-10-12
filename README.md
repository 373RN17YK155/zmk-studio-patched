# ZMK Studio (patched)

Сборка [ZMK Studio](https://github.com/zmkfirmware/zmk-studio) с PR
[**#135 — "Show Layer-Tap/Mod-Tap information on keys"**](https://github.com/zmkfirmware/zmk-studio/pull/135)
(ветка `BafS:feat/layer-tap-key-1`, коммит `a177a63`).

Причина: в стоковом Studio центр клавиши рисуется **только из `param1`** как HID-скан-код.
У `&mo`, `&sl`, `&to`, `&lt` первый параметр — это *номер слоя*, поэтому клавиша выглядела
пустой, и было не видно, на какой слой она ведёт.

Этот патч вместо `param1`-как-HID рендерит значения по метаданным поведения:
- `&mo 1` → показывает **имя слоя** ("Mouse");
- `&lt 1 SPACE` → нажатие (tap) + имя слоя (hold);
- `&mt LGUI SPACE`, `&hml LGUI A` → модификатор + клавиша.

## Сборка

```
git clone https://github.com/zmkfirmware/zmk-studio
git checkout a177a63e7b332e19896b511104f5df3737dadfee   # PR #135
git apply LOCAL-basepath.patch                            # правка путей под /zmk-studio-patched/
npm ci && npm run generate-data && npx tsc
npx vite build --base=/zmk-studio-patched/
```

`PR135.diff` — диф PR #135 на момент сборки.
`LOCAL-basepath.patch` — единственная локальная правка (3 захардкоженных абсолютных
пути к `zmk.svg`, `zmk-mac-app-icon.webp`, `Inter.woff2` сделаны base-aware, иначе под
подпапкой GitHub Pages ассеты не находятся).

`dist/` просто опубликован в этой ветке `gh-pages`.

## Подключение

- HTTPS-контекст обязателен для Web Serial / Web Bluetooth — GitHub Pages его даёт.
- **Компьютер (Chrome/Edge):** подключение по USB (Web Serial) к донглу.
- **Android:** Chrome не поддерживает Web Serial — подключайтесь по **Bluetooth**
  (донгл должен рекламировать ZMK Studio RPC; в ZMK это включено по умолчанию).
  В браузере на телефоне Web Bluetooth доступен в Chrome.
