# Configuração do Ambiente

Configuração do Avell Ion A70 com Ubuntu 25.10

## Gnome Extesion Mananger

```bash
sudo apt update
sudo apt install -y gnome-shell-extension-manager gnome-shell-extension-prefs

```

## Brave

```bash
sudo apt update && sudo apt install -y curl
sudo curl -fsSLo /usr/share/keyrings/brave-browser-archive-keyring.gpg \
  https://brave-browser-apt-release.s3.brave.com/brave-browser-archive-keyring.gpg
sudo curl -fsSLo /etc/apt/sources.list.d/brave-browser-release.sources \
  https://brave-browser-apt-release.s3.brave.com/brave-browser.sources
sudo apt update
sudo apt install -y brave-browser

```

## Chrome

### 1️⃣ Baixar o pacote `.deb` oficial

```bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
```


### 2️⃣ Instalar o pacote

```bash
sudo apt install -y ./google-chrome-stable_current_amd64.deb
```


### 3️⃣ Testar

```bash
google-chrome --version
```


## VS Code


### 1️⃣ Atualize o sistema

```bash
sudo apt update
sudo apt install -y wget gpg apt-transport-https
```

### 2️⃣ Importe a chave GPG da Microsoft

```bash
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor | sudo tee /usr/share/keyrings/microsoft.gpg > /dev/null
```


### 3️⃣ Adicione o repositório oficial do VS Code

```bash
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/microsoft.gpg] https://packages.microsoft.com/repos/code stable main" | sudo tee /etc/apt/sources.list.d/vscode.list
```

### 4️⃣ Instale o Visual Studio Code

```bash
sudo apt update
sudo apt install -y code
```

### 5️⃣ Teste

```bash
code --version
```

Ou abra direto:

```bash
code .
```

## DataGrip

```bash
cd ~/Downloads
wget -O jetbrains-toolbox.tar.gz "https://data.services.jetbrains.com/products/download?platform=linux&code=TBA"
tar -xf jetbrains-toolbox.tar.gz
./jetbrains-toolbox-*/jetbrains-toolbox
```

## Emote

Para acessao os emojis com **(win + .)** como no Windows.


Um pop-up moderno com categorias e busca. Abre por atalho e cola o emoji no app focado.

1. Instale (via Flatpak):

```bash
sudo apt install -y flatpak
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
flatpak install -y flathub com.tomjwatson.Emote
```

2. Crie um atalho global:

* **Configurações → Teclado → Atalhos personalizados → “+”**
  **Nome:** Emote
  **Comando:** `flatpak run com.tomjwatson.Emote`
  **Atalho:** por exemplo **Super+.** ou **Ctrl+Alt+E**

3. Use: aperte o atalho, clique no emoji e pronto (cola no app atual; se algum app não permitir colagem automática, ele já fica no clipboard).
