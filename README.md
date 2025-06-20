# Gerenciador de Senhas — README

> **Versão:** 2025-06-20
> **Arquivo principal:** `password_manager.html`
> **Plataforma:** Navegador (funciona off-line, todo o código roda no lado-cliente)

---

## 1. Visão geral

Este mini-gerenciador de senhas foi construído **inteiramente em um único arquivo HTML** com JavaScript Vanilla e **tema escuro** inspirado no Discord/ChatGPT.
Ele gera senhas fortes **determinísticas** a partir de:

1. Informações persistentes (URL, usuário, mnemônico, lista de dicas, tamanho, organização hierárquica).
2. Informações transitórias digitadas no momento da recuperação (senha - base e associações das dicas).

Nenhuma senha forte ou associação é armazenada em disco; apenas os metadados necessários são salvos em um arquivo criptografado (`*.pmcfg`).

---

## 2. Funcionalidades

| Categoria          | Recursos principais                                                                                                                                                      |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **UI & UX**        | Interface em colunas, edição inline de ambientes (`✎`), exclusão com confirmação (`×`), atalhos visuais Only-on-Hover.                                                   |
| **Tema**           | Paleta dark (`#202225 / #2f3136`) com acento roxo `--accent`.                                                                                                            |
| **Hierarquia**     | Ambientes ilimitados (carpetas) e sites em ambientes-folha.<br>⇢ Criação rápida: “+ novo ambiente” / “+ novo site”.                                                      |
| **Determinismo**   | Senha forte = `SHA-256` iterado + Base-62 **apenas** sobre dados imutáveis (exceto senha-base & associações, fornecidos na hora).                                        |
| **Segurança**      | • Sugestões de dicas verdadeiramente randômicas via `crypto.getRandomValues`.<br>• Arquivo de configuração = `JSON → AES-GCM (256) + PBKDF2-SHA256 (100 k) + salt 16 B`. |
| **CRUD**           | Edição & remoção de ambientes **e** sites. Campos editáveis: URL, usuário, mnemônico, senha-base, tamanho, lista de dicas.                                               |
| **Exportação**     | Salvar/Carregar configuração (`*.pmcfg`) com senha fornecida na hora.                                                                                                    |
| **Clipboard-only** | A senha forte nunca é exibida—é **copiada** para o clipboard e um “✅” confirma.                                                                                          |

---

## 3. Estrutura de dados

```text
cfg
├── children (objetos-recursivos) ─ ambientes
│     └── default …           # criado por padrão (segurança 0)
└── ... (meta)
```

### Objeto Site

| Campo            | Tipo       | Persistido?   | Descrição                        |
| ---------------- | ---------- | ------------- | -------------------------------- |
| `name`           | `string`   | ✔             | URL/domínio                      |
| `login`          | `string`   | ✔             | Nome de usuário                  |
| `mnemonic`       | `string`   | ✔             | Nominal amigável                 |
| `base`           | `string`   | ✔ (opcional)¹ | Senha-base memorizável           |
| `len`            | `number`   | ✔             | Comprimento desejado (padrão 16) |
| `hints`          | `string[]` | ✔             | Lista de dicas (*cues*)          |
| **Associations** | `string[]` | ✖             | Digitadas só no modal de geração |
| **Senha-forte**  | —          | ✖             | Copiada direto p/ clipboard      |

¹ Armazenar a senha-base é opcional. Se não for salva, o campo fica `""` e será solicitado na geração.

---

## 4. Algoritmo de geração

1. **Seed**

   ```
   seed =
     site.name        +
     site.login       +
     site.mnemonic    +
     <senha-base>     +
     site.hints.join('|') +
     associações.join('|')
   ```
2. **Hash extendido**
   `SHA-256(seed + "#" + i)` até preencher `site.len` caracteres.
3. **Codificação**
   Cada byte → `BASE62[byte % 62]`.

---

## 5. Fluxo de uso

```text
(1) Abrir password_manager.html
(2) Novo (padrão)   ─ ou ─   Carregar config
(3) Navegar/ criar ambientes
(4) + novo site  ➜ preencher campos + dicas
(5) Salvar config (opcional) → *.pmcfg
(6) Recuperar senha
      · Digitar senha-base
      · Preencher associações
      → Senha copiada p/ clipboard ✅
```

### Teclas rápidas

| Tecla                    | Ação          |
| ------------------------ | ------------- |
| `Esc` em modais          | Fecha/cancela |
| `Enter` em edição inline | Confirma      |
| `Esc` em edição inline   | Cancela       |

---

## 6. Dependências

* **Zero** pacotes externos.
* Usa apenas APIs Web Nativas:

  * `Web Crypto`: `crypto.getRandomValues`, `SubtleCrypto.digest`, `encrypt`, `decrypt`.
  * `Clipboard API`: `navigator.clipboard.writeText`.

Compatibilidade mínima: **Chrome 87+ / Firefox 78+ / Edge 88+**.

---

## 7. Instruções de build/execução

1. Clone ou baixe `password_manager.html`.
2. **Abra** o arquivo diretamente em qualquer navegador moderno (funciona off-line).
3. Para atualizar, substitua o HTML por versões posteriores.

> **Observação:** a segurança real depende do navegador, do ambiente físico e da força das senhas-base escolhidas.

---

## 8. Roadmap / ideias futuras

| Status | Item                                             |
| ------ | ------------------------------------------------ |
| ☐      | Sincronização via Web DAV ou Drive               |
| ☐      | Versão PWA (instalável)                          |
| ☐      | Modo-claro togglável                             |
| ☐      | Importação de listas BIP-39 completas para dicas |

---

## 9. Histórico de versões

| Data        | Versão   | Destaques                                                                                                                           |
| ----------- | -------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| 20-jun-2025 | **v1.0** | Primeira release estável: tema dark, hierarquia, determinismo, edição inline, CRUD sites, senha-base on-demand, exportação AES-GCM. |

---

## 10. Licença

Código-fonte liberado em **MIT License**.
Use por sua conta e risco — revise a criptografia antes de produção.
