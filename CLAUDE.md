# CLAUDE.md

Fork of [piitaya/lovelace-mushroom](https://github.com/piitaya/lovelace-mushroom): a bundle of
Home Assistant Lovelace custom cards/badges built with Lit and shipped as a single
`dist/mushroom.js` (HACS `plugin` category, `hacs.json` -> `filename: mushroom.js`).

## Commands

- `npm run build` - the only check CI runs (`.github/workflows/build.yml`). There is no test
  suite, no lint script and no ESLint config despite `eslint` being a devDependency.
- Vite does not type-check. Run `npx tsc` (uses `tsconfig.json`, `noEmit: true`) to catch type
  errors before relying on a green build.
- `npm run format` - Prettier over the whole repo (only override: `trailingComma: "es5"`).
- Local dev needs two terminals: `npm run start:hass` (Docker HA **beta**; `start:hass-stable` /
  `start:hass-dev` for other channels, mounting `.hass_dev/` as `/config`) and `npm start`
  (`vite build --watch` + `vite preview` on port 4000). `.hass_dev/configuration.yaml` loads the
  card via `frontend.extra_module_url: http://localhost:4000/mushroom.js`, so the port must stay 4000.
- The README mentions `npm run start:hass-cmd`; that script does not exist in `package.json`.

## Gotchas

- Output must stay one file: `vite.config.ts` sets `codeSplitting: false` and a fixed
  `fileName`. HACS and the manual install both expect exactly `mushroom.js`; the release
  workflow uploads `dist/*.js` on `v*.*.*` tags as a draft release.
- Version is read from `package.json` at build time (`src/mushroom.ts` logs it). Releases are
  plain commits named after the version (e.g. `5.2.2`) that bump `package.json` + `package-lock.json`.
- `registerCustomCard` (`src/utils/custom-cards.ts`) derives the documentation URL from the card
  type: `mushroom-foo-card` -> `docs/cards/foo.md`. A new card needs a doc file with that exact
  name, and its module must be imported in `src/mushroom.ts` or it is not bundled.
- Editors are lazy-loaded (`getConfigElement` dynamically imports `*-editor.ts`) and rely on HA's
  own form elements. `loadHaComponents` in `src/utils/loader.ts` is a hack that forces HA to
  define `ha-form`, `ha-entity-picker`, etc. by asking built-in cards for their editors; editors
  call it in `connectedCallback`, and a new editor must do the same.
- Editor configs are validated with `superstruct` (`assert(config, ...Struct)`); a new option
  must be added to the card's struct in `*-card-config.ts` or the editor will reject it.
- `src/ha/` holds code adapted from the Home Assistant frontend (types, helpers, websocket data);
  HA's frontend is not an npm dependency, so missing helpers are copied into `src/ha/`.
- Minimum HA version is `hacs.json` -> `homeassistant`. Behaviour that depends on newer HA
  releases is gated with `atLeastVersion(hass.config.version, ...)` from `src/ha` (see
  `src/utils/compute-entity-name.ts`).
- Translations: only edit `src/translations/en.json` by hand; other languages come from Weblate.
  Adding a language also requires importing the JSON in `src/localize.ts` and adding it to the
  `languages` record (BCP 47 tag as key).
- `browserslist-to-esbuild` reads `.browserslistrc` (`last 7 years`) to set the build target;
  keep syntax that target can down-level.
- Building `culori` emits circular-dependency warnings; `vite.config.ts` deliberately silences
  only those.

## Contributing upstream

The PR template (`.github/PULL_REQUEST_TEMPLATE.md`) states upstream only accepts PRs tied to an
open issue. Card docs live in `docs/cards/*.md` and `docs/badges/*.md`; update them alongside
option changes.
