# Converting **index.html** to an **Angular** application

> This guide is intentionally **very granular** so different team‑members (agents) can divide and conquer without stepping on each other.  Follow the checklist top‑to‑bottom – every sub‑item is *actionable*.

---

## 🗺️ Overall Roadmap

| Phase | Owner (agent)             | Deliverables                                            |
| ----- | ------------------------- | ------------------------------------------------------- |
| 0     | **Lead**                  | Spin‑up repo, copy this guide, open issues per section. |
| 1     | **Scaffolder**            | Angular workspace w/ Tailwind & ESLint.                 |
| 2     | **Theme Guru**            | Global dark theme (SCSS / Tailwind).                    |
| 3     | **State Engineer**        | *ConfigService* (RxJS state + AES‑GCM).                 |
| 4     | **Crypto Engineer**       | Deterministic password generator service.               |
| 5     | **UI Engineer**           | Re‑create components & modals.                          |
| 6     | **Clipboard/FS Engineer** | Clipboard & file import/export.                         |
| 7     | **QA**                    | Cypress e2e + Jest unit tests.                          |

> **Tip:** keep each phase on a dedicated branch – squash‑merge when ✅.

---

## 1  Prerequisites (Phase 0‑1)

1. **Install tooling**

   ```bash
   npm i -g @angular/cli@^17 @angular-eslint/schematics tailwindcss typescript
   ```
2. **Repo** – create *password-manager-angular* on GitHub.
3. **Angular workspace**

   ```bash
   ng new pwd-manager --routing=false --standalone --style=scss --skip-tests
   cd pwd-manager
   ```
4. **ESLint**

   ```bash
   ng add @angular-eslint/schematics
   ```
5. **Tailwind**

   ```bash
   npx tailwindcss init -p
   ```

   * Update *tailwind.config.js* `content` array to include `src/**/*.{html,ts}`.
   * Add dark palette variables in *src/styles.scss* (copy from HTML).

---

## 2  Dark Theme (Phase 2)

1. **Global styles** – migrate CSS variables → `:root` in *styles.scss*.
2. **Utility classes** – Tailwind `@apply` for `.column`, `.item`, etc.
3. **Animations** – optional: use Angular Animations for modal fade.

---

## 3  App skeleton (Phase 3)

### 3.1 Root bootstrap

* Convert to **stand‑alone components**: `AppComponent` only.
* Provide *ConfigService* in root.

### 3.2 Routing

* No routes needed – SPA single view.

---

## 4  Services

### 4.1 ConfigService (Phase 3)

| Task            | Detail                                                                                   |
| --------------- | ---------------------------------------------------------------------------------------- |
| **State**       | `BehaviorSubject<Config>`; initial value = default config.                               |
| **CRUD**        | Methods: `addEnv`, `renameEnv`, `deleteEnv`, `addSite`, `updateSite`, `deleteSite`.      |
| **Persistence** | `export(password)` & `import(file,password)` using WebCrypto (AES‑GCM 256, PBKDF2 100k). |
| **Determinism** | Never store associations or base password – enforce in type definitions.                 |

### 4.2 PasswordService (Phase 4)

* Pure function `derive(seed: string, length: number): Promise<string>` (same algorithm as HTML).
* Expose helper `generate(site, basePwd, associations[])`.

### 4.3 RandomWordService

* Keep list `WORDS[]`, expose `getRandom()`.

---

## 5  Component Tree (Phase 5)

```
<App>
 ├── <TopBar>
 ├── <ColumnContainer>
 │     └── <Column *ngFor="envLevel of levels">
 │              ├── <EnvItem>
 │              ├── <SiteItem>
 │              └── <NewButtons>
 └── <ModalOutlet />  (CDK Overlay)
```

### 5.1 TopBarComponent

* Buttons: New Config, Save, Load.
* Hook into `ConfigService`.

### 5.2 ColumnContainerComponent

* Compute breadcrumb stack via recursive selection in `ConfigService`.
* Scrollable flex row.

### 5.3 EnvItemComponent

* `@Input env`.
* Hover controls ✎ and ×.
* Inline rename via `contenteditable` span or `MatInput`.

### 5.4 SiteItemComponent

* Similar hover controls.
* `click` → open *SiteViewModal*.

### 5.5 Modals (use Angular CDK Dialog or custom)

| Modal           | Purpose                                      |
| --------------- | -------------------------------------------- |
| `SiteFormModal` | Create / edit site (reactive form).          |
| `SiteViewModal` | Show meta +“Gerar senha” botão.              |
| `GenerateModal` | Ask base Pwd + dynamic list of associations. |

---

## 6  Clipboard & File (Phase 6)

* **Clipboard** – Angular CDK `Clipboard` service (`this.clipboard.copy(pwd)`).
* **File export** – create `Blob` → `FileSaver.saveAs` **or** native `<a download>`.
* **File import** – `<input type=file>` + FileReader → `ConfigService.import`.

---

## 7  Forms & Validation

* Reactive Forms (FormBuilder) for all modals.
* Validators: `required`, `minLength(8)` for basePwd, etc.

---

## 8  Theming & Icons

1. **Icons** – Use *Lucide‑Angular* or SVG emojis ⌨️.
2. **Hover controls** – Tailwind `group` + `group-hover:inline`.

---

## 9  Testing (Phase 7)

* **Unit** – Jest for services & pure functions.
* **Component** – Angular Testing Library.
* **E2E** – Cypress: flow → create site → export → reload → import → regenerate → assert clipboard (mock).

---

## 10  CI/CD

* GitHub Actions workflow:

  ```yaml
  steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-node@v4
      with: { node-version: 20 }
    - run: npm ci
    - run: npm run lint && npm run test -- --watch=false && ng build --configuration production
  ```
* Deploy **GitHub Pages** via `angular-cli-ghpages` or Vercel.

---

## 11  Done Definition

* [ ] All functional parity with `index.html`.
* [ ] 100 % unit coverage on critical crypto & state.
* [ ] Lighthouse PWA score ≥ 90.
* [ ] README & AGENTS updated with new commands.