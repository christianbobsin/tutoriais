# Fancy Grub

Script para  monitor **16:10 QHD (2560×1600)**, mantendo tudo **seguro, reversível** e com **Windows 11 Pro** na 2ª posição. Ele:

* Clona o tema padrão do Ubuntu e aplica **background 2560×1600**
* Gera **fonte DejaVu** em PF2 e aumenta o tamanho (QHD)
* Ajusta `GRUB_GFXMODE=2560x1600,1920x1200,auto` e `GRUB_GFXPAYLOAD_LINUX=keep`
* Habilita `os-prober` e cria **entrada “Windows 11 Pro”** como **#2** (`11_windows_custom`)
* Mantém **Ubuntu 25.10** como #1 via `GRUB_DISTRIBUTOR`
* Faz **backup** de `/etc/default/grub` e permite **reversão** rápida

> **Antes:** salve sua imagem em `~/Pictures/grub-bg-2560x1600.png` (PNG, ≤ 8 MB).

### Script revisado (cole tudo no terminal)

```bash
cat > ~/setup-grub-theme_qhd16x10.sh <<'EOF'
#!/usr/bin/env bash
set -euo pipefail

THEME_NAME="ub-beauty"
THEME_SRC="/usr/share/grub/themes/ubuntu"
THEME_DST="/boot/grub/themes/${THEME_NAME}"
BG_SRC="${1:-$HOME/Pictures/grub-bg-2560x1600.png}"

# --- helpers ---
fail() { echo "❌ $*" >&2; exit 1; }
info() { echo "➡️  $*"; }

# --- checagens ---
[[ -f "$BG_SRC" ]] || fail "Falta a imagem: $BG_SRC (use PNG 2560x1600, ≤ 8 MB)."
[[ -s "$BG_SRC" ]] || fail "Arquivo de imagem está vazio: $BG_SRC."

# 0) Backup
info "Backup de /etc/default/grub..."
sudo cp -n /etc/default/grub "/etc/default/grub.bak.$(date +%F_%H%M%S)"

# 1) Preparar tema
if [[ -d "$THEME_SRC" ]]; then
  info "Clonando tema padrão para $THEME_DST ..."
  sudo rm -rf "$THEME_DST"
  sudo mkdir -p "$THEME_DST"
  sudo cp -a "$THEME_SRC/." "$THEME_DST/"
else
  info "Tema padrão não encontrado; criando tema minimal em $THEME_DST ..."
  sudo rm -rf "$THEME_DST"
  sudo mkdir -p "$THEME_DST"
  sudo tee "$THEME_DST/theme.txt" >/dev/null <<'MINITHEME'
+ desktop-image: "background.png"
+ title-text: "Selecione o sistema"
+ title-font: "DejaVu Sans 22"
+ message-font: "DejaVu Sans 18"
+ selected_item_color: "#ffffff"
+ item_color: "#d0d0d0"
+ menu_color_normal: "#d0d0d0"
+ menu_color_highlight: "#ffffff"
MINITHEME
fi

# 2) Background no tema
info "Aplicando background..."
sudo cp "$BG_SRC" "$THEME_DST/background.png"

# 3) Fonte (DejaVu Sans) em PF2
info "Gerando fonte PF2 (DejaVu Sans)..."
sudo mkdir -p /boot/grub/fonts
if [[ -f /usr/share/fonts/truetype/dejavu/DejaVuSans.ttf ]]; then
  sudo grub-mkfont -s 22 -o /boot/grub/fonts/DejaVuSans.pf2 /usr/share/fonts/truetype/dejavu/DejaVuSans.ttf
else
  fail "DejaVuSans.ttf não encontrado (instale: sudo apt install fonts-dejavu-core)."
fi

# 4) Se existir theme.txt, ajuste textos/tamanhos
if [[ -f "$THEME_DST/theme.txt" ]]; then
  info "Ajustando theme.txt (títulos e fontes)..."
  sudo sed -i \
    -e 's/^title-text.*/title-text: "Selecione o sistema"/' \
    -e 's/^title-font.*/title-font: "DejaVu Sans 22"/' \
    -e 's/^message-font.*/message-font: "DejaVu Sans 18"/' \
    "$THEME_DST/theme.txt" || true
fi

# 5) /etc/default/grub — timeout, gfxmode, payload, tema, distribuidor, menu visível
info "Configurando /etc/default/grub..."
sudo sed -i 's/^GRUB_TIMEOUT=.*/GRUB_TIMEOUT=5/' /etc/default/grub || echo 'GRUB_TIMEOUT=5' | sudo tee -a /etc/default/grub
sudo sed -i 's/^#\?GRUB_TIMEOUT_STYLE=.*/GRUB_TIMEOUT_STYLE=menu/' /etc/default/grub || echo 'GRUB_TIMEOUT_STYLE=menu' | sudo tee -a /etc/default/grub
sudo sed -i 's/^#\?GRUB_GFXMODE=.*/GRUB_GFXMODE=2560x1600,1920x1200,auto/' /etc/default/grub || echo 'GRUB_GFXMODE=2560x1600,1920x1200,auto' | sudo tee -a /etc/default/grub
sudo sed -i 's/^#\?GRUB_GFXPAYLOAD_LINUX=.*/GRUB_GFXPAYLOAD_LINUX=keep/' /etc/default/grub || echo 'GRUB_GFXPAYLOAD_LINUX=keep' | sudo tee -a /etc/default/grub
grep -q '^GRUB_THEME=' /etc/default/grub \
  && sudo sed -i "s|^GRUB_THEME=.*|GRUB_THEME=\"${THEME_DST}/theme.txt\"|" /etc/default/grub \
  || echo "GRUB_THEME=\"${THEME_DST}/theme.txt\"" | sudo tee -a /etc/default/grub
grep -q '^GRUB_DISTRIBUTOR=' /etc/default/grub \
  && sudo sed -i 's/^GRUB_DISTRIBUTOR=.*/GRUB_DISTRIBUTOR="Ubuntu 25.10"/' /etc/default/grub \
  || echo 'GRUB_DISTRIBUTOR="Ubuntu 25.10"' | sudo tee -a /etc/default/grub

# 6) Habilitar os-prober (detectar outros SOs)
grep -q '^GRUB_DISABLE_OS_PROBER=' /etc/default/grub \
  && sudo sed -i 's/^GRUB_DISABLE_OS_PROBER=.*/GRUB_DISABLE_OS_PROBER=false/' /etc/default/grub \
  || echo 'GRUB_DISABLE_OS_PROBER=false' | sudo tee -a /etc/default/grub

# 7) Entrada Windows 11 Pro (UEFI) em 2º lugar (script 11_*)
EFI_PATH="/boot/efi/EFI/Microsoft/Boot/bootmgfw.efi"
if [[ -f "$EFI_PATH" ]]; then
  info "Criando entrada custom para Windows 11 Pro..."
  sudo tee /etc/grub.d/11_windows_custom >/dev/null <<'EOS'
#!/bin/sh
exec tail -n +3 $0
menuentry "Windows 11 Pro" --class windows --class os {
    insmod part_gpt
    insmod fat
    search --no-floppy --file --set=root /EFI/Microsoft/Boot/bootmgfw.efi
    chainloader /EFI/Microsoft/Boot/bootmgfw.efi
}
EOS
  sudo chmod +x /etc/grub.d/11_windows_custom
else
  info "Windows EFI não encontrado em $EFI_PATH (pulando 11_windows_custom)."
fi

# 8) Atualizar GRUB
info "Atualizando GRUB (update-grub)..."
sudo update-grub

echo "✅ Concluído! Reinicie para conferir o tema."
echo "ℹ️  Ícones (opcional): coloque PNGs (64–128 px) em: ${THEME_DST}/icons/"
echo "    - ubuntu.png, windows.png, submenu.png, recovery.png"
EOF

chmod +x ~/setup-grub-theme_qhd16x10.sh
bash ~/setup-grub-theme_qhd16x10.sh
```

### Reversão (se quiser voltar atrás)

```bash
# veja o backup mais recente
ls -lt /etc/default/grub.bak.*
# escolha um e restaure (substitua pelo nome do seu arquivo)
sudo cp /etc/default/grub.bak.AAAA-MM-DD_HHMMSS /etc/default/grub
# remova a entrada custom do Windows e o tema
sudo rm -f /etc/grub.d/11_windows_custom
sudo sed -i '/^GRUB_THEME=/d' /etc/default/grub
sudo update-grub
```

### Observações finais

* Se o firmware **não anunciar 2560×1600**, o GRUB usa **1920×1200** (fallback) automaticamente.
  (Você pode conferir no console do GRUB com `videoinfo`.)
* **Fonte**: usei **DejaVu Sans** (popular e já presente no Ubuntu).
* **Ícones**: opcionais; o tema funciona sem eles. Se quiser, mando um pacote PNG redondinho.
