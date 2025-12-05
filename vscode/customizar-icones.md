# 🎨 Customizando ícones de pastas e arquivos no VS Code

Versão 1.0.0 - 05/12/2025

Este guia mostra **apenas o fluxo simples** para quando uma pasta/arquivo fica com ícone genérico porque o nome foge do padrão do tema, por exemplo:

* `build-debug-x86`
* `build-linux-x86`
* `meu-build`

Vamos ver como:

* ✅ Escolher **qual arquivo `settings.json` editar** (global vs projeto)
* ✅ Usar as chaves de configuração do **Material Icon Theme** para “forçar” um ícone
* ✅ Consultar a **lista de ícones padrão** em PNG para referência
* ⚙️ (Opcional) No final: um resumo de customização avançada com SVG/clones

---

## 🧱 1. Pré-requisito: instalar o Material Icon Theme

Se você já usa o tema, pode pular para a próxima seção.

1. Abra o VS Code.
2. Vá em **View → Extensions** ou use `Ctrl+Shift+X`.
3. Pesquise por: **Material Icon Theme** (autor: `PKief`). ([GitHub][1])
4. Clique em **Install**.
5. Ative o tema:

   * `Ctrl+Shift+P` → digite **“File Icon Theme”**
   * Escolha **“Material Icon Theme”**

Pronto, os ícones do Material Icon Theme estarão ativos.

---

## 🧭 2. Qual `settings.json` editar? (Global x Projeto)

O VS Code tem **três** comandos para abrir configurações em JSON pelo Command Palette (`Ctrl+Shift+P`):

* **Preferences: Open Default Settings (JSON)**
  🔒 Somente leitura, mostra os padrões do VS Code → **não vamos usar.**
* **Preferences: Open User Settings (JSON)**
  🌍 **Global** — vale para **todos os projetos**.
* **Preferences: Open Workspace Settings (JSON)**
  📁 **Por projeto** — vale só para a pasta/workspace atual (cria/edita `.vscode/settings.json`).

👉 Regra prática:

* Se quer que o ícone valha para **todos os projetos**, use
  **`Preferences: Open User Settings (JSON)`**.
* Se é algo específico de **um projeto**, use
  **`Preferences: Open Workspace Settings (JSON)`** (com a pasta do projeto aberta).

---

## 📂 3. Customização básica de ícones de PASTA

### 3.1. Problema clássico

Exemplo: o Material Icon Theme tem ícone para pasta `build`, mas sua pasta se chama:

* `build-debug-x86`
* `build-linux-x86`
* `build-arm64-prod`

Ela fica com ícone genérico. Queremos dizer pro tema:

> “Trata essas pastas como se fossem `build`/`dist`.”

---

### 3.2. Passo a passo (global ou por projeto)

1. Abra o Command Palette: `Ctrl+Shift+P`
2. Escolha:

   * Global: **Preferences: Open User Settings (JSON)**
   * Projeto: **Preferences: Open Workspace Settings (JSON)**
3. No `settings.json` aberto, adicione (ou complete) a seção:

```jsonc
{
  // ...outras configurações...

  "material-icon-theme.folders.associations": {
    // Usa o mesmo ícone da pasta "dist" (build/output)
    "build-debug-x86": "dist",
    "build-linux-x86": "dist",
    "build-arm64-prod": "dist"
  }
}
```

* A **chave** é o **nome exato da pasta** na sua árvore.
* O **valor** é o **nome interno do ícone de pasta** (ex.: `"dist"`, `"src"`, `"admin"`, etc.). ([GitHub][1])

> ⚠️ **Não dá para usar wildcard**, tipo `"build-*": "dist"`.
> A API do VS Code não suporta isso — você precisa listar cada nome que quiser tratar. ([Gist][2])

4. Salve o arquivo.
5. Se o ícone não aparecer na hora:

   * `Ctrl+Shift+P` → **Developer: Reload Window**
     ou
   * Reative: **Material Icons: Activate Icon Theme**

---

## 📄 4. Customização básica de ícones de ARQUIVO (opcional)

Você também pode “forçar” ícones para **extensões** ou **arquivos específicos** com:

```jsonc
"material-icon-theme.files.associations": {
  // Todos os arquivos .configbuild usam o ícone de JSON
  "*.configbuild": "json",

  // Arquivo com nome exato
  "build-debug.config.json": "settings"
}
```

* `*.ext` → associa pela **extensão**.
* Nome sem `*` → arquivo com esse **nome exato**. ([GitHub][1])

---

## 📚 5. Referência: PNG com TODOS os ícones

O próprio projeto mantém imagens PNG com a visão geral dos ícones:

* 📄 **Ícones de arquivos**
  `https://raw.githubusercontent.com/PKief/vscode-material-icon-theme/master/images/fileIcons.png` ([about.gitlab.com][3])
* 📁 **Ícones de pastas**
  `https://raw.githubusercontent.com/PKief/vscode-material-icon-theme/master/images/folderIcons.png` ([DEV Community][4])

Dica de uso:

1. Abra os PNGs no navegador.
2. Dê zoom.
3. Olhe o **nome** do ícone que aparece na imagem (por exemplo, `src`, `dist`, `test`, `admin`).
4. Use esse nome em:

```jsonc
"material-icon-theme.folders.associations": {
  "minha-pasta": "nomeDoIcone"
}
```

ou

```jsonc
"material-icon-theme.files.associations": {
  "*.meuext": "nomeDoIcone"
}
```

---

## ⚙️ 6. (Opcional) Customização avançada: clones e SVGs

Se você quiser ir **além da customização simples**, o Material Icon Theme suporta:

### 6.1. Clonar ícones com outras cores (pastas)

```jsonc
"material-icon-theme.folders.customClones": [
  {
    "name": "build-linux",
    "base": "dist",                      // ícone base
    "color": "light-green-500",          // cor tema escuro
    "lightColor": "light-green-700",     // opcional (tema claro)
    "folderNames": [
      "build-linux-x86",
      "build-linux-arm64"
    ]
  }
]
```

* `base`: ícone existente que você está clonando.
* `color` / `lightColor`: podem ser hex (`"#FF9800"`) ou nomes da paleta Material. ([GitHub][1])

### 6.2. Clonar ícones de arquivo com outras cores

```jsonc
"material-icon-theme.files.customClones": [
  {
    "name": "rust-mod",
    "base": "rust",
    "color": "blue-400",
    "fileNames": ["mod.rs"]
  }
]
```

Cria um ícone novo (`rust-mod`) baseado no ícone `rust`. ([GitHub][1])

### 6.3. Usar SVGs personalizados (arquivos e pastas)

Você pode apontar para **SVGs seus**, desde que estejam dentro da pasta `.vscode/extensions` do usuário:

```text
.vscode
 ┗ extensions
   ┗ icons
     ┣ sample.svg
     ┣ folder-sample.svg
     ┗ folder-sample-open.svg
```

Exemplo para arquivo:

```jsonc
"material-icon-theme.files.associations": {
  "minha-config.ts": "../../icons/sample"
}
```

Exemplo para pasta:

```jsonc
"material-icon-theme.folders.associations": {
  "src": "../../../../icons/folder-sample"
}
```

> Importante: o caminho é dado **sem** `.svg` no final. ([GitHub][1])

---

## ✅ 7. Checklist rápido (versão “só o que importa”)

1. 🧩 **Tema instalado e ativo?**

   * Se não, instale **Material Icon Theme** e ative.
2. 🧭 **Decida o escopo:**

   * 🌍 Global → `Preferences: Open User Settings (JSON)`
   * 📁 Só esse projeto → `Preferences: Open Workspace Settings (JSON)`
3. 🧱 **Adicione associações simples**:

   * Pastas → `"material-icon-theme.folders.associations"`
   * Arquivos → `"material-icon-theme.files.associations"`
4. 🖼️ **Pegue o nome do ícone** nos PNG:

   * `fileIcons.png` e `folderIcons.png`
5. ♻️ **Não apareceu?**

   * `Ctrl+Shift+P` → **Developer: Reload Window**

Com isso você resolve 99% dos casos do tipo **“minha pasta ficou com ícone genérico”**, como o exemplo `build-debug-x86`, sem precisar mexer com SVG nem nada avançado.

[1]: https://github.com/material-extensions/vscode-material-icon-theme?utm_source=chatgpt.com "material-extensions/vscode-material-icon-theme"
[2]: https://gist.github.com/rupeshtiwari/6860fbc1b3e2f6711c780070d6f59748?utm_source=chatgpt.com "Steps to add custom folder icon in Material icons"
[3]: https://gitlab.com/gitlab-org/gitlab/-/issues/226921?utm_source=chatgpt.com "Icon for MATLAB .m files in repository tree is not appropriate"
[4]: https://dev.to/diballesteros/how-to-change-material-icon-theme-folder-association-325k?utm_source=chatgpt.com "How to change Material Icon Theme folder association"
