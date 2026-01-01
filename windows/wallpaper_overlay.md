# 🖼️ Automação de Wallpaper com Identificação do PC (Help Desk)

## 🎯 Objetivo
Esta automação **gera um novo papel de parede** a partir de uma imagem base (referência) e **desenha informações úteis do computador** por cima — como **IP**, **interface de rede**, **nome do PC** e **data/hora do login**.

Isso é especialmente útil para **Help Desk / Suporte** porque, ao bater o olho no desktop do usuário, já dá para identificar rapidamente:
- Qual máquina é
- Qual IP ela está usando
- Qual interface está ativa
- Quando foi o último logon (momento em que o script rodou)

---

## 🧩 Script que gera o wallpaper com os dados


```powershell
#requires -version 5.1
param(
    [Parameter(Mandatory = $false)]
    [string]$ReferenceWallpaperPath
)

Add-Type -AssemblyName System.Drawing

function Get-PrimaryIPv4 {
    $route = Get-NetRoute -DestinationPrefix "0.0.0.0/0" -ErrorAction SilentlyContinue |
             Sort-Object -Property RouteMetric, InterfaceMetric |
             Select-Object -First 1

    if ($route) {
        $ip = Get-NetIPAddress -AddressFamily IPv4 -InterfaceIndex $route.InterfaceIndex -ErrorAction SilentlyContinue |
              Where-Object { $_.IPAddress -notlike '169.254.*' -and $_.IPAddress -ne '127.0.0.1' } |
              Select-Object -First 1

        $ifName = (Get-NetAdapter -InterfaceIndex $route.InterfaceIndex -ErrorAction SilentlyContinue).Name
        if ($ip) {
            return [pscustomobject]@{ IP = $ip.IPAddress; IfName = $ifName }
        }
    }

    $upIf = Get-NetAdapter -ErrorAction SilentlyContinue |
            Where-Object { $_.Status -eq 'Up' } |
            Sort-Object -Property InterfaceMetric |
            Select-Object -First 1

    if ($upIf) {
        $ip2 = Get-NetIPAddress -AddressFamily IPv4 -InterfaceIndex $upIf.IfIndex -ErrorAction SilentlyContinue |
               Where-Object { $_.IPAddress -notlike '169.254.*' -and $_.IPAddress -ne '127.0.0.1' } |
               Select-Object -First 1

        if ($ip2) {
            return [pscustomobject]@{ IP = $ip2.IPAddress; IfName = $upIf.Name }
        }
    }

    return [pscustomobject]@{ IP = 'IP não encontrado'; IfName = 'N/A' }
}

function Get-CurrentWallpaperPath {
    $regPath = 'HKCU:\Control Panel\Desktop'
    $wp = (Get-ItemProperty -Path $regPath -Name WallPaper -ErrorAction SilentlyContinue).WallPaper
    if ($wp -and (Test-Path $wp)) { return $wp }

    $fallback = Join-Path $env:APPDATA 'Microsoft\Windows\Themes\TranscodedWallpaper'
    if (Test-Path $fallback) { return $fallback }

    return $null
}

function Load-Image([string]$Path) {
    # Carrega via stream para evitar lock e ficar robusto para PNG/JPG/etc
    $fs = [System.IO.File]::Open($Path, [System.IO.FileMode]::Open, [System.IO.FileAccess]::Read, [System.IO.FileShare]::ReadWrite)
    try {
        $img = [System.Drawing.Image]::FromStream($fs)
        return $img
    }
    catch {
        throw "Falha ao carregar imagem: $Path. Detalhes: $($_.Exception.Message)"
    }
    finally {
        $fs.Dispose()
    }
}

function Resolve-SourceWallpaper([string]$RefPath, [string]$GeneratedPath) {
    # 1) Preferência: referência
    if ($RefPath) {
        $full = $RefPath
        try { $full = (Resolve-Path -Path $RefPath -ErrorAction Stop).Path } catch { }
        if ($full -and (Test-Path $full)) { return $full }
    }

    # 2) Fallback: wallpaper atual
    $cur = Get-CurrentWallpaperPath
    if (-not $cur) { return $null }

    # Se o wallpaper atual é o arquivo gerado, tenta usar o último wallpaper base salvo
    try {
        $curFull = (Resolve-Path -Path $cur -ErrorAction Stop).Path
        $genFull = $null
        try { $genFull = (Resolve-Path -Path $GeneratedPath -ErrorAction Stop).Path } catch { }

        if ($genFull -and ($curFull -eq $genFull)) {
            $reg = 'HKCU:\Software\WallpaperIPOverlay'
            $lastBase = (Get-ItemProperty -Path $reg -Name LastBaseWallpaper -ErrorAction SilentlyContinue).LastBaseWallpaper
            if ($lastBase -and (Test-Path $lastBase)) { return $lastBase }
        }
    } catch { }

    return $cur
}

function Set-Wallpaper([string]$Path) {
    $code = @"
using System;
using System.Runtime.InteropServices;
public class Wallpaper {
  [DllImport("user32.dll", SetLastError=true)]
  public static extern bool SystemParametersInfo(int uAction, int uParam, string lpvParam, int fuWinIni);
}
"@
    Add-Type $code -ErrorAction SilentlyContinue | Out-Null

    # Estilo: Fill
    Set-ItemProperty -Path 'HKCU:\Control Panel\Desktop' -Name WallpaperStyle -Value '10' -ErrorAction SilentlyContinue
    Set-ItemProperty -Path 'HKCU:\Control Panel\Desktop' -Name TileWallpaper  -Value '0'  -ErrorAction SilentlyContinue

    $ok = [Wallpaper]::SystemParametersInfo(20, 0, $Path, 3)
    if (-not $ok) {
        $err = [Runtime.InteropServices.Marshal]::GetLastWin32Error()
        throw "Falha ao aplicar wallpaper. Win32Error=$err Path=$Path"
    }
}

# ---------------- MAIN ----------------
try {
    $net = Get-PrimaryIPv4

    $outDir  = Join-Path $env:TEMP "WallpaperIP"
    $outPath = Join-Path $outDir "wallpaper_ip.png"
    New-Item -ItemType Directory -Path $outDir -Force | Out-Null

    $src = Resolve-SourceWallpaper -RefPath $ReferenceWallpaperPath -GeneratedPath $outPath
    if (-not $src) { throw "Não consegui localizar nem a imagem de referência nem o wallpaper atual." }

    # Salva o wallpaper base escolhido (se não for o gerado)
    $reg = 'HKCU:\Software\WallpaperIPOverlay'
    New-Item -Path $reg -Force | Out-Null
    try {
        $srcFull = (Resolve-Path -Path $src -ErrorAction Stop).Path
        $genFull = $null
        try { $genFull = (Resolve-Path -Path $outPath -ErrorAction Stop).Path } catch { }
        if (-not $genFull -or ($srcFull -ne $genFull)) {
            Set-ItemProperty -Path $reg -Name LastBaseWallpaper -Value $srcFull -Force
        }
    } catch { }

    $pcName = $env:COMPUTERNAME
    $login  = (Get-Date).ToString("yyyy-MM-dd HH:mm:ss")

    $lineIP     = "IP: $($net.IP)"
    $lineIf     = "IF: $($net.IfName)"
    $linePc     = "PC: $pcName"
    $lineLogin  = "Login: $login"

    $img = Load-Image -Path $src
    $bmp = New-Object System.Drawing.Bitmap $img.Width, $img.Height
    $g   = [System.Drawing.Graphics]::FromImage($bmp)
    $g.SmoothingMode = [System.Drawing.Drawing2D.SmoothingMode]::AntiAlias
    $g.TextRenderingHint = [System.Drawing.Text.TextRenderingHint]::ClearTypeGridFit
    $g.DrawImage($img, 0, 0, $img.Width, $img.Height)

    # ===== Escala =====
    $mainPx = [int]($img.Height * 0.0525)
    $subPx  = [int]($img.Height * 0.0225)
    if ($mainPx -lt 32) { $mainPx = 32 }
    if ($subPx  -lt 14) { $subPx  = 14 }

    $mainPt = ($mainPx * 72.0) / $g.DpiY
    $subPt  = ($subPx  * 72.0) / $g.DpiY

    $fontMain = New-Object System.Drawing.Font("Consolas", [single]$mainPt, [System.Drawing.FontStyle]::Bold)
    $fontSub  = New-Object System.Drawing.Font("Consolas", [single]$subPt,  [System.Drawing.FontStyle]::Regular)

    $pad = [int]($img.Height * 0.020)
    if ($pad -lt 16) { $pad = 16 }

    $lineGap = [int]($img.Height * 0.006)
    if ($lineGap -lt 4) { $lineGap = 4 }

    $lines = @(
        @{ Text = $lineIP;    Font = $fontMain; Shadow = 3 },
        @{ Text = $lineIf;    Font = $fontSub;  Shadow = 2 },
        @{ Text = $linePc;    Font = $fontSub;  Shadow = 2 },
        @{ Text = $lineLogin; Font = $fontSub;  Shadow = 2 }
    )

    $maxW = 0.0
    $totalH = 0.0
    foreach ($l in $lines) {
        $sz = $g.MeasureString($l.Text, $l.Font)
        if ($sz.Width -gt $maxW) { $maxW = $sz.Width }
        $totalH += $sz.Height
    }
    $totalH += ($lineGap * ($lines.Count - 1))

    $boxW = [int]([Math]::Ceiling($maxW + ($pad * 2)))
    $boxH = [int]([Math]::Ceiling($totalH + ($pad * 2)))

    $x = $pad
    $y = $pad

    $bgColor = [System.Drawing.Color]::FromArgb(160, 0, 0, 0)
    $brushBg = New-Object System.Drawing.SolidBrush($bgColor)
    $white   = New-Object System.Drawing.SolidBrush([System.Drawing.Color]::White)
    $shadow  = New-Object System.Drawing.SolidBrush([System.Drawing.Color]::FromArgb(220, 0, 0, 0))

    $g.FillRectangle($brushBg, $x, $y, $boxW, $boxH)

    $tx = $x + $pad
    $ty = $y + $pad

    foreach ($l in $lines) {
        $sz = $g.MeasureString($l.Text, $l.Font)
        $so = [int]$l.Shadow
        $g.DrawString($l.Text, $l.Font, $shadow, ($tx + $so), ($ty + $so))
        $g.DrawString($l.Text, $l.Font, $white,  $tx,        $ty)
        $ty += [Math]::Ceiling($sz.Height + $lineGap)
    }

    $g.Dispose()
    $img.Dispose()

    $bmp.Save($outPath, [System.Drawing.Imaging.ImageFormat]::Png)
    $bmp.Dispose()

    Set-Wallpaper $outPath

    Write-Host "OK! Wallpaper aplicado."
    Write-Host "Fonte: $src"
    Write-Host "IP: $($net.IP) | IF: $($net.IfName) | PC: $pcName | Login: $login"
    Write-Host "Arquivo: $outPath"
}
catch {
    Write-Error $_
    exit 1
}
```

Salve como: `C:\Scripts\WallpaperIP.ps1`

**Parâmetro principal**
- `-ReferenceWallpaperPath` (opcional): caminho para a imagem de referência (ex.: `C:\Wallpapers\ref.png`).  
  - Se não existir, o script usa o **papel de parede atual**.
  - Para evitar “texto sobre texto” (efeito de sombra duplicada), o script guarda o último wallpaper base em:
    - `HKCU:\Software\WallpaperIPOverlay\LastBaseWallpaper`

**Formatos de imagem suportados**
- O carregamento é feito via `.NET (System.Drawing)`, o que suporta **PNG, JPG/JPEG, BMP, GIF, TIFF** no Windows.


> Observação: o arquivo final gerado é sempre salvo como PNG:  
> `%TEMP%\WallpaperIP\wallpaper_ip.png`

---

## ⚙️ Script de configuração


```powershell
# Config.ps1
# Cria/atualiza a tarefa "WallpaperIPOverlay"

$script = "C:\Scripts\WallpaperIP.ps1"
$ref    = "C:\Wallpapers\ref.png"   # <--- pode ser PNG
$userId = "$env:USERDOMAIN\$env:USERNAME"

if (-not (Test-Path $script)) {
    throw "Não encontrei o script principal em: $script"
}

$action = New-ScheduledTaskAction -Execute "powershell.exe" -Argument "-NoProfile -ExecutionPolicy Bypass -File `"$script`" -ReferenceWallpaperPath `"$ref`""
$trigger = New-ScheduledTaskTrigger -AtLogOn -User $userId
$principal = New-ScheduledTaskPrincipal -UserId $userId -LogonType Interactive -RunLevel Limited

$settings = New-ScheduledTaskSettingsSet `
    -AllowStartIfOnBatteries `
    -DontStopIfGoingOnBatteries `
    -StartWhenAvailable `
    -ExecutionTimeLimit (New-TimeSpan -Minutes 2) `
    -MultipleInstances IgnoreNew

$task = New-ScheduledTask -Action $action -Trigger $trigger -Principal $principal -Settings $settings `
    -Description "Aplica wallpaper com IP/IF/PC e data/hora do login"

Register-ScheduledTask -TaskName "WallpaperIPOverlay" -InputObject $task -Force | Out-Null

Write-Host "OK! Tarefa criada/atualizada: WallpaperIPOverlay"
Write-Host "Usuário: $userId"
Write-Host "Script:  $script"
Write-Host "Ref:     $ref"

```

Salve como: `C:\Scripts\Config.ps1`

Este script cria/atualiza a tarefa agendada `WallpaperIPOverlay` para rodar **no logon do usuário** e chamar o script principal.

---

## ▶️ Como executar o script

### 🧪 1) Executar manualmente (teste)
```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -File "C:\Scripts\WallpaperIP.ps1" -ReferenceWallpaperPath "C:\Wallpapers\ref.png"
```

### 🗓️ 2) Criar/atualizar a tarefa agendada
```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -File "C:\Scripts\Config.ps1"
```

### ✅ 3) Testar a tarefa
```powershell
Start-ScheduledTask -TaskName "WallpaperIPOverlay"
Get-ScheduledTaskInfo -TaskName "WallpaperIPOverlay" | Select-Object LastRunTime, LastTaskResult
```

`LastTaskResult = 0` significa sucesso.

---

## 🖼️ Como atualizar o Wallpaper de referência
Você tem duas opções:

### 📝 Opção A (simples): editar o `Config.ps1`
1. Abra `C:\Scripts\Config.ps1`
2. Troque o valor de `$ref` para o novo caminho:
   ```powershell
   $ref = "C:\Wallpapers\nova.png"
   ```
3. Execute o `Config.ps1` novamente:
   ```powershell
   powershell.exe -NoProfile -ExecutionPolicy Bypass -File "C:\Scripts\Config.ps1"
   ```

✅ Como a tarefa é registrada com `-Force`, ela é **atualizada**, não duplicada.

### ♻️ Opção B: manter o mesmo caminho e apenas substituir o arquivo
Se você sempre usar, por exemplo, `C:\Wallpapers\ref.png`, basta **trocar o arquivo** (substituir o conteúdo) mantendo o mesmo nome/caminho.

---

## 🛡️ Observações sobre execução de scripts sem assinatura (ExecutionPolicy)

### ⚠️ Erro comum
> “a execução de scripts foi desabilitada neste sistema…”

Isso é controlado por **Execution Policy**.

### 🔐 Solução recomendada (somente para seu usuário)
```powershell
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
```
- Permite scripts locais sem assinatura
- Scripts baixados da internet continuam exigindo assinatura (mais seguro)

### 🚀 Rodar apenas uma vez sem alterar o sistema
```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -File "C:\Scripts\Config.ps1"
```

### 🔓 Desbloquear arquivo marcado como “baixado da internet”
Se o arquivo tiver “Mark of the Web”, rode:
```powershell
Unblock-File "C:\Scripts\Config.ps1"
Unblock-File "C:\Scripts\WallpaperIP.ps1"
```

### 🔎 Ver políticas ativas (todas as scopes)
```powershell
Get-ExecutionPolicy -List
```

> Se uma política vier de **Group Policy** (GPO), pode aparecer como `MachinePolicy`/`UserPolicy` e você pode precisar do administrador da máquina/domínio para ajustar.

---
