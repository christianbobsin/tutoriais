
# 🔐 Trabalhando com arquivos `.axx` no Ubuntu com XecretsCli

## 1. O que é um arquivo `.axx` 🧩

- Arquivos com extensão **`.axx`** são arquivos **criptografados** pelo programa **AxCrypt** (Windows/macOS).
- Eles **não** são arquivos compactados como `.zip` ou `.rar`, e sim arquivos cujo conteúdo foi embaralhado usando **criptografia**.
- Para abrir/descriptografar um `.axx`, você **precisa da senha** usada na criptografia.  
  🔒 Sem a senha correta, o conteúdo é considerado irrecuperável.

---

## 2. Baixando o XecretsCli no Ubuntu 💻

O **XecretsCli** é uma ferramenta de **linha de comando** compatível com o formato dos arquivos AxCrypt (`.axx`), e funciona no Linux.

### 2.1. ⬇️ Baixar o pacote

No terminal:

```bash
cd ~
curl -L https://www.axantum.com/download/cli/linux > XecretsCli-Linux.tar.gz
```

### 2.2. 📦 Descompactar o pacote

```bash
tar -xvf XecretsCli-Linux.tar.gz
cd XecretsCli-Linux-*
```

Agora você estará em uma pasta que contém o binário `XecretsCli`.

### 2.3. (Opcional) ➕ Colocar o XecretsCli no PATH

Para conseguir chamar o comando de qualquer lugar:

```bash
mkdir -p ~/bin
cp -i XecretsCli ~/bin
```

Recarregue o ambiente:

```bash
. ~/.profile
```

Teste:

```bash
XecretsCli -h
```

✅ Se aparecer a ajuda do programa, está tudo certo.

> 💡 Se **não** colocar no `PATH`, use `./XecretsCli` dentro da pasta onde ele foi extraído.

---

## 3. Comando para descriptografar um arquivo `.axx` 🔓

### 3.1. Sintaxe geral

```bash
XecretsCli --password 'SUA_SENHA_AQUI' \
  --decrypt-to ARQUIVO_ENTRADA.axx ARQUIVO_SAIDA
```

* `--password 'SUA_SENHA_AQUI'` → senha usada na criptografia.
* `--decrypt-to` → indica que queremos **descriptografar**.
* `ARQUIVO_ENTRADA.axx` → o arquivo `.axx` de origem.
* `ARQUIVO_SAIDA` → arquivo descriptografado (ex: `.pdf`, `.docx`, `.zip`).

Se estiver dentro da pasta do binário e **não** tiver colocado no PATH, use:

```bash
./XecretsCli --password 'SUA_SENHA_AQUI' \
  --decrypt-to ARQUIVO_ENTRADA.axx ARQUIVO_SAIDA
```

### 3.2. Exemplo prático ✅

Arquivo de entrada:

```text
~/Downloads/Licença PPConecta - Payer-zip.axx
```

Arquivo de saída (conteúdo original, por exemplo um ZIP):

```text
~/Downloads/Licença PPConecta - Payer-zip.zip
```

Comando:

```bash
XecretsCli --password 'SUA_SENHA_AQUI' \
  --decrypt-to \
  "$HOME/Downloads/Licença PPConecta - Payer-zip.axx" \
  "$HOME/Downloads/Licença PPConecta - Payer-zip.zip"
```

> 💡 Sempre use aspas `"` quando o caminho tiver **espaços**.

---

## 4. Comando para criptografar (gerar `.axx`) 🔏

Você também pode **criptografar** arquivos no formato compatível com AxCrypt (`.axx`) usando o XecretsCli.

### 4.1. Sintaxe geral

```bash
XecretsCli --password 'SUA_SENHA_AQUI' \
  --encrypt-to ARQUIVO_ORIGINAL ARQUIVO_ENCRYPTED.axx
```

* `ARQUIVO_ORIGINAL` → arquivo em texto claro (ex: `.pdf`, `.docx`, `.zip`).
* `ARQUIVO_ENCRYPTED.axx` → nome do arquivo criptografado de saída.

### 4.2. Exemplo: criptografar um PDF 📄

```bash
XecretsCli --password 'SUA_SENHA_AQUI' \
  --encrypt-to \
  "$HOME/Documentos/Contrato-Cliente.pdf" \
  "$HOME/Documentos/Contrato-Cliente.pdf.axx"
```

### 4.3. Exemplo: criptografar um ZIP 🗜️

```bash
XecretsCli --password 'SUA_SENHA_AQUI' \
  --encrypt-to \
  "$HOME/Backups/backup-2025-01-01.zip" \
  "$HOME/Backups/backup-2025-01-01.zip.axx"
```

---

## 5. Boas práticas e recomendações ✅

### 5.1. Senhas e histórico de comandos 🔑

* Evite deixar a senha registrada no **histórico do Bash**:

  * Adicione no arquivo `~/.bashrc`:

    ```bash
    export HISTCONTROL="ignoreboth"
    ```
  * Recarregue:

    ```bash
    . ~/.bashrc
    ```
  * Sempre que digitar o comando de criptografia/descriptografia, comece com **um espaço**:

    ```bash
     XecretsCli --password 'SUA_SENHA_AQUI' --decrypt-to ...
    ```

    ➜ Com isso, o comando **não é salvo** no histórico.

* Use senhas fortes:

  * 🔐 Misture maiúsculas, minúsculas, números e símbolos.
  * ❌ Evite datas de nascimento, nomes óbvios ou palavras simples.

### 5.2. Cuidados com caminhos e nomes de arquivos 📁

* Use aspas `"` para caminhos com espaços:

  ```bash
  "$HOME/Downloads/Meu Arquivo Importante.axx"
  ```
* Lembre que o Linux diferencia maiúsculas/minúsculas:

  * `Licenca.axx` é diferente de `licenca.axx`.

### 5.3. Backup e integridade 💾

* Mantenha **backup** dos arquivos `.axx` e, se possível, também do arquivo original (em local seguro).
* ⚠️ Se um `.axx` for corrompido (queda de energia, disco com problema, download incompleto), mesmo com a senha correta pode não ser possível recuperar o conteúdo.

### 5.4. Verificando a ajuda completa 📖

* Para ver todas as opções disponíveis:

  ```bash
  XecretsCli --help
  ```

  ou, se estiver na pasta do binário:

  ```bash
  ./XecretsCli --help
  ```

---

