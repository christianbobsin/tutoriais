# Configuração do Ambiente

Configuração do Avell Ion A70 com Ubuntu 25.10

---
## Gnome Extesion Mananger

```bash
sudo apt update
sudo apt install -y gnome-shell-extension-manager gnome-shell-extension-prefs
```

---
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

---

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

---
## Edge

### 1️⃣ Preparar o keyring

```bash
sudo install -d -m 0755 /etc/apt/keyrings
curl -fsSL https://packages.microsoft.com/keys/microsoft.asc   | sudo gpg --dearmor -o /etc/apt/keyrings/microsoft.gpg
```

### 2️⃣ Adicionar o repositório do Edge

```bash
echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/microsoft.gpg] https://packages.microsoft.com/repos/edge stable main" | sudo tee /etc/apt/sources.list.d/microsoft-edge.list
```

### 3️⃣ Instalar

```bash
sudo apt update
sudo apt install -y microsoft-edge-stable
```

---
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

---
## DataGrip

```bash
cd ~/Downloads
wget -O jetbrains-toolbox.tar.gz "https://data.services.jetbrains.com/products/download?platform=linux&code=TBA"
find . -maxdepth 3 -type f -name 'jetbrains-toolbox' -print -exec chmod +x {} \; -exec {} \;
./jetbrains-toolbox-*/jetbrains-toolbox
```

---
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


---
## Slack

```bash
cd ~/Downloads
wget "$(curl -fsSL https://slack.com/downloads/linux | sed -nE 's/.*href="(https:[^"]+amd64\.deb)".*/\1/p' | head -n1)" -O slack.deb
sudo apt install -y ./slack.deb
```

---
## Postman

```bash
sudo snap install postman
```


## Video Player (Celluloid + mpv)


```bash
sudo apt update
sudo apt install -y celluloid mpv
```

### Habilitar “clique/space = pausa” no mpv

```bash
mkdir -p ~/.config/mpv

cat > ~/.config/mpv/mpv.conf << 'EOF'
vo=gpu-next
profile=gpu-hq
hwdec=auto-safe
EOF

cat > ~/.config/mpv/input.conf << 'EOF'
SPACE cycle pause       # barra de espaço pausa/retoma
MBTN_LEFT cycle pause   # clique esquerdo pausa/retoma
# (duplo clique = tela cheia)
EOF
```

> Dica: abra o **Celluloid** e em *Preferências* mantenha o “Usar configurações do mpv” ativado (padrão). Assim o clique funciona do jeitinho acima.

Defina o Celluloid como player padrão (opcional):

```bash
xdg-mime default io.github.celluloid_player.Celluloid.desktop video/mp4
xdg-mime default io.github.celluloid_player.Celluloid.desktop video/x-matroska
xdg-mime default io.github.celluloid_player.Celluloid.desktop video/x-msvideo
```

---
## Gimp

```bash
sudo apt update
sudo apt install -y gimp
```