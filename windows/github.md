
# 🧭 Guia simples: Git + SSH + GitHub no **Windows 11**

Versão 1.0.0 - 01/12/2025

Este passo-a-passo foi feito para quem **está começando** com:

- 💻 PowerShell / terminal  
- 🔐 SSH  
- 🐙 Git e GitHub  

No final, você vai ter:

1. ✅ Git instalado e funcionando  
2. ✅ Seu **nome** e **e-mail** configurados no Git (global)  
3. ✅ Uma **chave SSH** configurada e testada com o GitHub  

---

## 0. 🧩 Pré-requisitos

Antes de começar, você precisa:

1. 🌐 Ter uma conta no **GitHub** (se não tiver, crie em `https://github.com`).
2. 🪟 Estar usando **Windows 11**.
3. 👮 Ter permissão para instalar programas (em PC corporativo, talvez precise de admin).

> 💡 Sempre que eu falar “**terminal**”, é a janela do **PowerShell** no Windows 11.

---

## 1. ▶️ Abrir o PowerShell no Windows 11

1. 🪟 Clique no botão **Iniciar** (ícone do Windows).
2. ⌨️ Digite: `powershell`
3. 🖱️ Clique em **Windows PowerShell** ou **Windows Terminal**  
   (se abrir o Terminal, por padrão ele já usa PowerShell).

🔍 **Como saber se deu certo?**  
Você verá algo como:

```text
PS C:\Users\SeuUsuario>
```

Se aparecer isso, está tudo certo ✅

---

## 2. 🧰 Verificar se o Git está instalado

No PowerShell, digite:

```powershell
git --version
```

* ✅ Se aparecer algo como `git version 2.x.x`, o Git já está instalado.
* ❌ Se aparecer algo como `git : O termo 'git' não é reconhecido`, o Git **ainda não está instalado**.

### 🛠️ Se o Git NÃO estiver instalado

1. Abra o navegador 🌍 e acesse **git-scm.com** → **Download for Windows**.
2. Baixe o instalador e execute 🧑‍🏫.
3. No instalador, pode aceitar as opções padrão (**Next, Next, Finish**).

Depois da instalação:

1. ❌ Feche o PowerShell.
2. ✅ Abra de novo o PowerShell.
3. Rode novamente:

```powershell
git --version
```

Se aparecer a versão, o Git está pronto 🎉

---

## 3. 🔎 Verificar se o SSH está disponível

No mesmo PowerShell, rode:

```powershell
ssh -V
```

* ✅ Se aparecer algo como:

  ```text
  OpenSSH_for_Windows_9.xp1, LibreSSL ...
  ```

  então o **OpenSSH** está instalado (normal no Windows 11).

* ❌ Se der erro dizendo que `ssh` não foi encontrado:

  1. Abra ⚙️ **Configurações** → **Aplicativos** → **Recursos opcionais**.
  2. Procure por **Cliente OpenSSH**.
  3. Se não estiver instalado, clique em **Adicionar recurso** ➕ e instale **Cliente OpenSSH**.

Depois disso, feche e abra o PowerShell novamente e teste:

```powershell
ssh -V
```

Se mostrar a versão, tudo certo ✅

---

## 4. ✍️ Configurar nome e e-mail globais no Git

Isso define **quem é você** nos commits que aparecerão no GitHub.

No PowerShell, troque pelos seus dados:

```powershell
git config --global user.name "Seu Nome Completo"
git config --global user.email "seu_email_do_github@example.com"
```

> ℹ️ Dica: use o **mesmo e-mail** que você usa no GitHub
> (ou o e-mail *noreply* do GitHub, se quiser mais privacidade).

### ✅ Verificar se ficou certo

Rode:

```powershell
git config --global --list
```

Você deve ver algo como:

```text
user.name=Seu Nome Completo
user.email=seu_email_do_github@example.com
```

Se aparecer isso, o Git já está com seu nome e e-mail configurados corretamente 😎

---

## 5. 📁 Verificar (e criar) a pasta `.ssh`

A pasta `.ssh` fica dentro da sua pasta de usuário, geralmente:

```text
C:\Users\SeuUsuario\.ssh
```

Ela **pode não existir ainda**, e está tudo bem.
Vamos checar e criar, se precisar. 🧱

### 5.1 👀 Verificar se a pasta `.ssh` existe

No PowerShell, rode:

```powershell
Test-Path $env:USERPROFILE\.ssh
```

* Se o resultado for `True` ✅ → a pasta já existe.
* Se o resultado for `False` ❌ → vamos criar a pasta.

### 5.2 🏗️ Criar a pasta `.ssh` (se necessário)

Se o passo anterior retornou `False`, rode:

```powershell
New-Item -ItemType Directory -Path $env:USERPROFILE\.ssh -Force
```

Se não aparecer erro, a pasta foi criada 👌

Você pode conferir com:

```powershell
ls $env:USERPROFILE\.ssh
```

Se estiver vazia, não tem problema — vamos gerar a chave no próximo passo.

---

## 6. 🔐 Gerar uma nova chave SSH para o GitHub

A chave SSH permite que o GitHub reconheça seu computador **sem pedir login e senha toda hora** 🗝️

Vamos usar o tipo **ed25519**, recomendado hoje.

No PowerShell, trocando pelo **SEU e-mail do GitHub**:

```powershell
ssh-keygen -t ed25519 -C "seu_email_do_github@example.com"
```

Você verá algo como:

```text
Enter a file in which to save the key (C:\Users\SEUUSUARIO\.ssh\id_ed25519):
```

* Aperte **Enter** ⏎ para aceitar o padrão (`C:\Users\SEUUSUARIO\.ssh\id_ed25519`).

Depois:

```text
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
```

Você pode:

* 🔓 Deixar em branco (apertar Enter duas vezes) → mais simples, menos seguro.
* 🔐 Colocar uma senha (passphrase) → mais seguro; pode ser pedida às vezes.

### ✅ Conferir se a chave foi criada

Rode:

```powershell
ls $env:USERPROFILE\.ssh
```

Você deve ver arquivos como:

* `id_ed25519` → **chave privada** (fica só com você, não compartilhe).
* `id_ed25519.pub` → **chave pública** (essa vamos colocar no GitHub).

Se esses dois arquivos aparecerem, sucesso! 🎉

---

## 7. 🤝 Ativar o `ssh-agent` e adicionar sua chave (Windows 11)

O `ssh-agent` é um “guardinha” que guarda sua chave na memória, pra você não ter que digitar senha toda hora.

### 7.1 🚀 Iniciar o serviço `ssh-agent` (uma vez só)

1. Feche o PowerShell normal.
2. Abra o **PowerShell como Administrador**:

   * Clique em **Iniciar** → digite `powershell`.
   * Clique com o botão direito → **Executar como administrador**.

No PowerShell (Admin), rode:

```powershell
Get-Service -Name ssh-agent | Set-Service -StartupType Manual
Start-Service ssh-agent
```

Isso liga o serviço e configura para ele poder ser usado.

Depois pode **fechar** esse PowerShell de administrador.

### 7.2 ➕ Adicionar sua chave ao `ssh-agent`

Abra um PowerShell **normal** (sem admin) e rode:

```powershell
ssh-add $env:USERPROFILE\.ssh\id_ed25519
```

* Se você colocou passphrase, o terminal pode pedir essa senha 🔑.
* Se não colocou, o comando só termina em silêncio (sem erro).

🔍 **Como saber se deu certo?**

Você deve ver algo como:

```text
Identity added: C:/Users/SEUUSUARIO/.ssh/id_ed25519 (seu_email_do_github@example.com)
```

Se aparecer `Could not open a connection to your authentication agent`, o `ssh-agent` não está rodando → volte na etapa 7.1.

---

## 8. 📋 Copiar a chave pública para usar no GitHub

Agora vamos copiar o conteúdo do arquivo `id_ed25519.pub` para a área de transferência (Ctrl+C automático 😄).

No PowerShell:

```powershell
Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub | Set-Clipboard
```

🔍 **Conferindo o conteúdo**

1. Abra o **Bloco de Notas** 📝.
2. Pressione **Ctrl+V**.

Você deve ver algo como:

```text
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI...resto_da_chave... seu_email_do_github@example.com
```

Se aparecer uma linha longa começando com `ssh-ed25519`, está perfeito ✅

---

## 9. 🐙 Adicionar a chave SSH na sua conta do GitHub

1. Abra o navegador 🌍 e vá em `https://github.com`.
2. Faça login na sua conta.
3. No canto superior direito, clique na **sua foto** → **Settings** ⚙️.
4. No menu da esquerda, clique em **SSH and GPG keys** 🔑.
5. Clique em **New SSH key** ou **Add SSH key**.
6. Em **Title**, coloque algo fácil de identificar, por exemplo:
   `Notebook Windows 11` 💻
7. Em **Key**, cole (Ctrl+V) a chave que copiamos no passo anterior.
8. Clique em **Add SSH key** ✅

Se o GitHub pedir sua senha de novo, confirme.

🔍 **Como saber se deu certo?**
Você verá a nova chave listada na página **SSH and GPG keys** com o título que escolheu.

---

## 10. 🧪 Testar a conexão SSH com o GitHub

De volta ao **PowerShell**, rode:

```powershell
ssh -T git@github.com
```

Na primeira vez, pode aparecer:

```text
The authenticity of host 'github.com (IP ADDRESS)' can't be established.
ED25519 key fingerprint is SHA256:...
Are you sure you want to continue connecting (yes/no)?
```

1. Digite: `yes` e aperte **Enter** ⏎.

Se tudo estiver certo, você verá:

```text
Hi SEUUSUARIO! You've successfully authenticated, but GitHub does not provide shell access.
```

🎉 Isso quer dizer:

* A chave SSH foi aceita.
* O GitHub reconheceu o seu usuário.
* A conexão SSH está funcionando corretamente.

Se aparecer `Permission denied (publickey)`, algo deu errado (chave errada, não cadastrada, etc.).

---

## 11. 🧪 (Opcional) Testar clonando um repositório via SSH

Se você já tem algum repositório seu no GitHub:

1. Vá à página do repositório.
2. Clique no botão verde **Code**.
3. Selecione a aba **SSH**.
4. Copie a URL SSH, algo como:

   ```text
   git@github.com:seuusuario/seu-repo.git
   ```

No PowerShell, na pasta onde você quer o projeto:

```powershell
git clone git@github.com:seuusuario/seu-repo.git
```

Se aparecer algo como:

```text
Cloning into 'seu-repo'...
```

e surgir uma pasta `seu-repo`, parabéns 🥳
Seu ambiente **Git + SSH + GitHub** está redondinho.

---

## 12. ✅ Checklist final

Use esta lista para conferir se está tudo certo:

* ✅ `git --version` mostra uma versão do Git
* ✅ `ssh -V` mostra a versão do OpenSSH
* ✅ `git config --global --list` mostra `user.name` e `user.email` corretos
* ✅ `Test-Path $env:USERPROFILE\.ssh` retornou `True`
* ✅ `ls $env:USERPROFILE\.ssh` mostra `id_ed25519` e `id_ed25519.pub`
* ✅ Sua chave aparece em **GitHub → Settings → SSH and GPG keys**
* ✅ `ssh -T git@github.com` mostra
  `Hi SEUUSUARIO! You've successfully authenticated, but GitHub does not provide shell access.`

Se tudo isso estiver ok:

* Você já consegue **clonar** repositórios via SSH 🐙
* Consegue dar **git pull** / **git push** sem digitar senha o tempo todo 🚀
* Seus commits aparecem com **seu nome** e **seu e-mail** certinhos 🧾



