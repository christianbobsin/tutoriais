# 🧭 Guia simples: Git + SSH + Bitbucket no **Linux (Ubuntu e Debian)**

Versão 1.0.1 - 16/04/2026

Este passo-a-passo foi feito para quem **está começando** com:

- 💻 Terminal no Linux
- 🔐 SSH
- 🪣 Git e Bitbucket

No final, você vai ter:

1. ✅ Git instalado e funcionando
2. ✅ Seu **nome** e **e-mail** configurados no Git (global)
3. ✅ Uma **chave SSH** configurada e testada com o Bitbucket

---

## 0. 🧩 Pré-requisitos

Antes de começar, você precisa:

1. 🌐 Ter uma conta no **Bitbucket Cloud** (se não tiver, crie em `https://bitbucket.org`).
2. 🐧 Estar usando um Linux comum de desktop.  
   Este guia assume **Ubuntu Desktop 24.04 LTS** como base, mas também aponta as diferenças mais comuns no **Debian**.
3. 👮 Ter permissão para instalar pacotes com `sudo`  
   ou acesso ao usuário `root`.

> 💡 Sempre que eu falar “**terminal**”, estou falando do aplicativo de terminal padrão do seu sistema, como **Terminal** no Ubuntu ou **Console / Terminal** no Debian com GNOME.

## 0.1 🚨 Antes de tudo: em qual usuário você deve rodar cada comando?

Esta parte é **muito importante**:

- 👤 **Comandos de Git do seu usuário** devem ser rodados como **seu usuário normal**
- 🔐 **Comandos da pasta `~/.ssh`** devem ser rodados como **seu usuário normal**
- 🗝️ **`ssh-keygen` e `ssh-add`** devem ser rodados como **seu usuário normal**
- 📦 **`sudo` ou `root`** devem ser usados **somente** para instalar pacotes ou mexer em arquivos do sistema

### ✅ Regra simples para não errar

Se o comando mexe em uma destas coisas:

- `~/.ssh`
- `git config --global`
- sua chave SSH
- seu nome e seu e-mail no Git

então rode como **usuário normal**, sem `sudo`.

Se o comando for instalar programa com `apt`, aí sim pode usar:

- `sudo ...`
- ou `su -` / `root`, se seu Debian não tiver `sudo`

### 🔎 Como conferir em qual usuário você está

Rode:

```bash
whoami
```

- Se aparecer seu usuário normal, por exemplo `christian`, pode continuar ✅
- Se aparecer `root`, **pare** para os passos de Git e SSH ❌

> 🚨 Resumo direto:
> `git config --global`, `mkdir ~/.ssh`, `chmod ~/.ssh`, `ssh-keygen`, `ssh-add` e `cat ~/.ssh/id_ed25519.pub` **não** devem ser executados como `root`.

### 🆚 Ubuntu x Debian neste guia

- 🟠 **Ubuntu** é a referência principal porque costuma ser o desktop Linux mais comum.
- 🔴 **Debian** aparece nas diferenças práticas, para você não travar se estiver usando ele.
- 📦 Nos dois, os comandos principais são quase iguais porque ambos usam `apt`.
- 👮 Se no seu Debian o comando `sudo` não existir, entre como `root` com `su -` e rode os mesmos comandos **sem** `sudo`.
- 🚨 Essa dica de usar `root` vale para **instalação de pacotes com `apt`**. Ela **não vale** para configurar `git --global` nem para criar sua chave em `~/.ssh`.

---

## 1. ▶️ Abrir o terminal no Linux

No Ubuntu, a forma mais simples é:

1. ⌨️ Pressione `Ctrl + Alt + T`

Ou:

1. 🟠 Clique em **Mostrar aplicativos**
2. 🔎 Procure por **Terminal**
3. 🖱️ Abra o aplicativo

No Debian com GNOME, normalmente funciona do mesmo jeito:

1. ⌨️ Tente `Ctrl + Alt + T`
2. Se não abrir, clique em **Atividades**
3. 🔎 Procure por **Terminal** ou **Console**

🔍 **Como saber se deu certo?**  
Você verá algo parecido com:

```text
seuusuario@seu-pc:~$
```

Se apareceu isso, está tudo certo ✅

---

## 2. 🧰 Verificar se o Git está instalado

No terminal, digite:

```bash
git --version
```

- ✅ Se aparecer algo como `git version 2.x.x`, o Git já está instalado.
- ❌ Se aparecer `git: command not found`, o Git ainda não está instalado.

### 🛠️ Se o Git NÃO estiver instalado

Rode:

```bash
sudo apt update
sudo apt install -y git
```

> 🆚 **Ubuntu x Debian**
> Nos dois sistemas, o pacote é o mesmo: `git`.
> Se no Debian o `sudo` não funcionar, use:
>
> ```bash
> su -
> apt update
> apt install -y git
> ```

Depois, teste novamente:

```bash
git --version
```

Se aparecer a versão, o Git está pronto 🎉

---

## 3. 🔎 Verificar se o SSH está disponível

No mesmo terminal, rode:

```bash
ssh -V
```

- ✅ Se aparecer algo como:

  ```text
  OpenSSH_9.xp1 Ubuntu-...
  ```

  então o cliente **OpenSSH** está instalado.

- ❌ Se aparecer erro dizendo que `ssh` não foi encontrado, instale com:

  ```bash
  sudo apt update
  sudo apt install -y openssh-client
  ```

> 🆚 **Ubuntu x Debian**
> Nos dois sistemas, o pacote do cliente SSH é `openssh-client`.
> Em instalações desktop do Ubuntu ele quase sempre já vem instalado.
> No Debian isso também é comum, mas depende mais do perfil de instalação escolhido.

Depois disso, teste novamente:

```bash
ssh -V
```

Se mostrar a versão, tudo certo ✅

---

## 4. ✍️ Configurar nome e e-mail globais no Git

Isso define **quem é você** nos commits.

> 🚨 Rode estes comandos como **usuário normal**.
> Não use `sudo` aqui.
> Não rode isso logado como `root`.

No terminal, troque pelos seus dados:

```bash
git config --global user.name "Seu Nome Completo"
git config --global user.email "seu_email_do_bitbucket@example.com"
```

> ℹ️ Dica: use o e-mail que você quer associar aos commits no Bitbucket.
> Não precisa ser necessariamente o e-mail de login, mas precisa ser o e-mail que você quer ver nos commits.

### ✅ Verificar se ficou certo

Rode:

```bash
git config --global --list
```

Você deve ver algo como:

```text
user.name=Seu Nome Completo
user.email=seu_email_do_bitbucket@example.com
```

Se aparecer isso, o Git já está com seu nome e e-mail configurados corretamente 😎

### ♻️ E se já existir `user.name` e `user.email`?

Se já aparecerem valores antigos no `git config --global --list`, você tem duas opções:

- ✏️ **Substituir direto** pelos valores corretos
- 🗑️ **Apagar primeiro** e depois configurar de novo

### ✅ Jeito mais simples: substituir direto

Rode novamente:

```bash
git config --global user.name "Seu Nome Completo"
git config --global user.email "seu_email_do_bitbucket@example.com"
```

Isso sobrescreve os valores globais do seu usuário normal.

### 🧹 Se você quiser apagar antes

```bash
git config --global --unset-all user.name
git config --global --unset-all user.email
```

Depois configure de novo:

```bash
git config --global user.name "Seu Nome Completo"
git config --global user.email "seu_email_do_bitbucket@example.com"
```

### 🚨 Como desfazer se você adicionou o usuário do Git no lugar errado

Se você rodou como `root`, os dados foram para o Git do `root`.

Para apagar do `root`:

```bash
sudo git config --global --unset-all user.name
sudo git config --global --unset-all user.email
```

Depois, no seu usuário normal:

```bash
git config --global user.name "Seu Nome Completo"
git config --global user.email "seu_email_do_bitbucket@example.com"
```

### 🧠 O que significa `--global` de verdade?

No Git, `--global` quer dizer:

- “salvar para o **usuário atual**”
- **não** quer dizer “salvar para o sistema todo”

Então:

- se você rodar como `christian`, ele grava em `/home/christian/.gitconfig`
- se você rodar como `root`, ele grava em `/root/.gitconfig`

Ou seja: se você fizer isso como `root`, configurou o Git do `root`, não o seu.

---

## 5. 📁 Verificar (e criar) a pasta `.ssh`

No Linux, o local **padrão** para guardar chaves SSH é:

```text
~/.ssh
```

Neste guia, vamos usar **somente esse local padrão**.  
Nada de salvar chave em pasta personalizada.

> 🚨 Todos os comandos desta seção devem ser executados como **usuário normal**.
> Se você estiver como `root`, a pasta criada será `/root/.ssh`, e não a sua.

### 5.1 👀 Verificar se a pasta `.ssh` existe

No terminal, rode:

```bash
ls -ld ~/.ssh
```

- Se a pasta existir, você verá algo começando com `drwx------` ou parecido.
- Se aparecer `No such file or directory`, ela ainda não existe.

### 5.2 🏗️ Criar a pasta `.ssh` (se necessário)

Se a pasta ainda não existir, rode:

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
```

> ❌ Não use:
>
> ```bash
> sudo mkdir -p ~/.ssh
> sudo chmod 700 ~/.ssh
> ```
>
> Isso pode criar ou alterar a pasta com dono errado.

Você pode conferir com:

```bash
ls -ld ~/.ssh
```

Se aparecer a pasta, está pronta 👌

### ♻️ E se a pasta `~/.ssh` já existir?

Isso é normal.

Se ela já existir:

- ✅ **não precisa apagar**
- ✅ normalmente basta manter e continuar
- ✅ só confirme se ela pertence ao seu usuário normal

Para conferir dono e permissões:

```bash
ls -ld ~/.ssh
```

Você quer algo parecido com:

```text
drwx------ 2 seuusuario seuusuario ... /home/seuusuario/.ssh
```

### 🔧 Se a pasta existe, mas está com permissões erradas

Rode como seu usuário normal:

```bash
chmod 700 ~/.ssh
```

### 🚨 Se a pasta foi criada com dono errado

Se você usou `sudo` e a pasta ficou como dona do `root`, corrija assim:

```bash
sudo chown -R "$USER:$USER" ~/.ssh
chmod 700 ~/.ssh
```

### 🗑️ Como desfazer a criação da pasta `.ssh`

Só apague a pasta se você tiver certeza de que ela **não tem nenhuma chave ou arquivo importante**.

Primeiro confira o conteúdo:

```bash
ls -la ~/.ssh
```

Se ela estiver vazia e você quiser remover:

```bash
rmdir ~/.ssh
```

Se ela tiver sido criada errada no `root`, confira:

```bash
sudo ls -la /root/.ssh
```

Se você tiver certeza de que quer remover a pasta do `root`:

```bash
sudo rm -rf /root/.ssh
```

> 🚨 Use `rm -rf` com cuidado.
> Esse comando apaga a pasta e tudo dentro dela.

---

## 6. 🔐 Gerar uma nova chave SSH para o Bitbucket

A chave SSH permite que o Bitbucket reconheça seu computador **sem pedir usuário e senha toda hora** 🗝️

Vamos usar o tipo **ed25519**, recomendado hoje.

No terminal, trocando pelo **SEU e-mail**:

```bash
ssh-keygen -t ed25519 -C "seu_email_do_bitbucket@example.com"
```

> 🚨 Rode como **usuário normal**.
> Se você fizer isso como `root`, a chave vai parar em `/root/.ssh/id_ed25519`, e isso não é o que você quer.

Você verá algo como:

```text
Enter file in which to save the key (/home/SEUUSUARIO/.ssh/id_ed25519):
```

- Aperte **Enter** ⏎ para aceitar o padrão `~/.ssh/id_ed25519`.
- Não digite outro caminho. Assim a chave fica no local padrão do Linux, como você pediu.

Depois:

```text
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
```

Você pode:

- 🔓 Deixar em branco e apertar Enter duas vezes → mais simples, menos seguro.
- 🔐 Definir uma passphrase → mais seguro; pode ser solicitada em alguns momentos.

### ✅ Conferir se a chave foi criada

Rode:

```bash
ls -la ~/.ssh
```

Você deve ver arquivos como:

- `id_ed25519` → **chave privada** (fica só com você, não compartilhe)
- `id_ed25519.pub` → **chave pública** (essa vamos colocar no Bitbucket)

Se esses dois arquivos aparecerem, sucesso 🎉

---

## 7. 🤝 Carregar a chave no `ssh-agent`

No Ubuntu Desktop, normalmente o ambiente gráfico já lida com isso bem.  
Mesmo assim, o fluxo mais simples é este:

> 🆚 **Ubuntu x Debian**
> No Ubuntu com GNOME, o `ssh-agent` costuma estar pronto com mais frequência.
> No Debian, especialmente em instalações mais enxutas ou sessões diferentes do GNOME padrão, é mais comum precisar iniciar o agente manualmente.

### 7.1 ➕ Tentar adicionar a chave

Rode:

```bash
ssh-add ~/.ssh/id_ed25519
```

> 🚨 Também rode este comando como **usuário normal**.

- Se pedir a passphrase, digite a senha da chave.
- Se terminar sem erro, ótimo.

🔍 **Como saber se deu certo?**  
Você verá algo como:

```text
Identity added: /home/SEUUSUARIO/.ssh/id_ed25519 (seu_email_do_bitbucket@example.com)
```

### 7.2 🚀 Se o `ssh-agent` não estiver rodando

Se aparecer algo como:

```text
Could not open a connection to your authentication agent
```

então rode:

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

Depois disso, a chave deve ser carregada normalmente ✅

### 7.3 👀 Verificar se a chave foi carregada

Rode:

```bash
ssh-add -l
```

Se aparecer uma linha com sua chave, está tudo certo.

---

## 8. 📋 Copiar a chave pública para usar no Bitbucket

Agora vamos mostrar a chave pública para copiar.

No terminal:

```bash
cat ~/.ssh/id_ed25519.pub
```

> 🚨 Esse arquivo deve estar no seu `~/.ssh`, não no `/root/.ssh`.

Você verá uma linha longa parecida com:

```text
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI...resto_da_chave... seu_email_do_bitbucket@example.com
```

Copie a linha inteira.

> 💡 No Terminal do Ubuntu, normalmente o atalho para copiar é `Ctrl + Shift + C`.
> No Debian com GNOME, esse atalho costuma ser o mesmo.

---

## 9. 🪣 Adicionar a chave SSH na sua conta do Bitbucket

1. Abra o navegador 🌍 e faça login em `https://bitbucket.org`.
2. Acesse diretamente: `https://bitbucket.org/account/settings/ssh-keys/`
3. Clique em **Add key**.
4. Em **Label**, coloque um nome fácil de reconhecer, por exemplo:
   `Notebook Ubuntu 24.04`
5. Em **Key**, cole a chave pública copiada no passo anterior.
6. Clique em **Add SSH key** ✅

> 💡 Se você estiver no Debian, pode trocar o rótulo por algo como `Notebook Debian 12`.

🔍 **Como saber se deu certo?**  
Você verá a nova chave listada na página de **SSH keys**.

---

## 10. 🧪 Testar a conexão SSH com o Bitbucket

De volta ao terminal, rode:

```bash
ssh -T git@bitbucket.org
```

Na primeira vez, pode aparecer algo como:

```text
The authenticity of host 'bitbucket.org (...)' can't be established.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

1. Digite `yes` e aperte **Enter** ⏎

Se tudo estiver certo, você verá uma mensagem parecida com:

```text
authenticated via ssh key.
You can use git to connect to Bitbucket. Shell access is disabled.
```

🎉 Isso quer dizer:

- A chave SSH foi aceita
- O Bitbucket reconheceu sua conta
- A conexão SSH está funcionando corretamente

Se aparecer `Permission denied (publickey)`, normalmente significa que:

- a chave não foi adicionada no Bitbucket, ou
- a chave não foi carregada no `ssh-agent`, ou
- você gerou a chave em outro nome/local e não está usando o arquivo padrão

---

## 11. 🧪 (Opcional) Testar clonando um repositório via SSH

Se você já tem algum repositório no Bitbucket:

1. Abra a página do repositório.
2. Clique em **Clone**.
3. Escolha a opção **SSH**.
4. Copie a URL SSH, algo como:

   ```text
   git@bitbucket.org:seu-workspace/seu-repo.git
   ```

No terminal, na pasta onde você quer baixar o projeto:

```bash
git clone git@bitbucket.org:seu-workspace/seu-repo.git
```

Se aparecer algo como:

```text
Cloning into 'seu-repo'...
```

e surgir uma pasta `seu-repo`, parabéns 🥳  
Seu ambiente **Git + SSH + Bitbucket** está redondinho.

---

## 12. ✅ Checklist final

Use esta lista para conferir se está tudo certo:

- ✅ `git --version` mostra uma versão do Git
- ✅ `ssh -V` mostra a versão do OpenSSH
- ✅ `git config --global --list` mostra `user.name` e `user.email` corretos
- ✅ `ls -ld ~/.ssh` mostra a pasta `.ssh`
- ✅ `ls -la ~/.ssh` mostra `id_ed25519` e `id_ed25519.pub`
- ✅ Sua chave aparece em **Bitbucket → Personal settings → SSH keys**
- ✅ `ssh -T git@bitbucket.org` mostra uma mensagem dizendo que você foi autenticado por chave SSH

Se tudo isso estiver ok:

- Você já consegue **clonar** repositórios via SSH 🪣
- Consegue dar **git pull** / **git push** sem digitar senha o tempo todo 🚀
- Seus commits aparecem com **seu nome** e **seu e-mail** certinhos 🧾

---

## 13. 🧹 Se você fez como `root` por engano, como desfazer?

Se você rodou os passos de Git e SSH como `root`, o problema mais comum é este:

- o Git foi salvo em `/root/.gitconfig`
- a chave SSH foi criada em `/root/.ssh`
- o seu usuário normal continua sem configuração

### 13.1 👀 Ver se você está como `root`

Rode:

```bash
whoami
```

Se aparecer `root`, saia da sessão de root:

```bash
exit
```

Depois abra o terminal normalmente no seu usuário.

### 13.2 🔎 Conferir se o Git foi configurado no lugar errado

Como usuário normal:

```bash
git config --global --list
```

Se não aparecer `user.name` e `user.email`, é sinal de que você provavelmente configurou no `root`.

Para conferir o `root`, use:

```bash
sudo git config --global --list
```

Se ali aparecer seu nome e seu e-mail, então foi gravado em `/root/.gitconfig`.

### 13.3 🗑️ Remover a configuração errada do Git no `root`

Se você quer apagar a configuração do `root`, rode:

```bash
sudo git config --global --unset-all user.name
sudo git config --global --unset-all user.email
```

Depois configure de novo no seu usuário normal:

```bash
git config --global user.name "Seu Nome Completo"
git config --global user.email "seu_email_do_bitbucket@example.com"
```

### 13.4 🔎 Conferir se a chave SSH foi criada no lugar errado

Veja se ela foi parar no `root`:

```bash
sudo ls -la /root/.ssh
```

Se aparecer `id_ed25519` e `id_ed25519.pub`, a chave foi criada no lugar errado.

### 13.5 🔁 Corrigir a chave SSH

O caminho mais simples para iniciantes é:

1. apagar a chave errada do `root`
2. gerar tudo de novo no seu usuário normal

Para apagar a chave do `root`:

```bash
sudo rm -f /root/.ssh/id_ed25519 /root/.ssh/id_ed25519.pub
```

Depois, já no seu usuário normal, gere novamente:

```bash
ssh-keygen -t ed25519 -C "seu_email_do_bitbucket@example.com"
```

### 13.6 🧯 E se eu já adicionei a chave errada no Bitbucket?

Sem problema.

1. Vá em `https://bitbucket.org/account/settings/ssh-keys/`
2. Remova a chave antiga
3. Gere a nova no seu usuário normal
4. Adicione a nova chave pública no Bitbucket

### ✅ Resumo do conserto

Se você fez tudo como `root`, faça assim:

1. saia do `root`
2. apague a configuração errada do Git em `/root/.gitconfig`
3. apague a chave errada de `/root/.ssh`
4. refaça os passos no seu usuário normal
