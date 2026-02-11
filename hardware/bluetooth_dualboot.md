# Compartilhar chave Bluetooth entre Windows e Linux (Dual Boot)

Este procedimento permite que um dispositivo Bluetooth (mouse ou
teclado) funcione em **Windows e Linux sem precisar parear novamente** a
cada troca de sistema.

A ideia é simples:

1.  O Windows gera a chave de pareamento Bluetooth.
2.  Copiamos essa chave.
3.  Inserimos a mesma chave no Linux.
4.  Os dois sistemas passam a usar o mesmo vínculo.

------------------------------------------------------------------------

## ⚠️ Avisos importantes

-   Procedimento testado principalmente com **Ubuntu, Debian e
    derivados**.
-   Funciona melhor com **teclados e mouses Bluetooth simples**.
-   Requer acesso de administrador nos dois sistemas.
-   Faça com cuidado para evitar erros de digitação.


## Visão geral do processo

  Etapa                     Sistema
  ------------------------- ---------
  Parear dispositivo        Windows
  Extrair chave Bluetooth   Windows
  Inserir chave no Linux    Linux
  Reiniciar Bluetooth       Linux

------------------------------------------------------------------------

## 1. Parear o dispositivo no Windows

1.  Inicialize no **Windows**.
2.  Vá em:

Configurações → Bluetooth e dispositivos

3.  Remova registros antigos do dispositivo.
4.  Pareie novamente o mouse ou teclado.
5.  Confirme que está funcionando.

------------------------------------------------------------------------

## 2. Obter a chave Bluetooth no Windows

### Abrir o Editor de Registro

- Pressione:
  - `Win + R`

- Digite:
  - `regedit`

- Pressione
  - `Enter`



### Navegar até a chave Bluetooth

Vá até:

```
HKEY_LOCAL_MACHINE
└── SYSTEM
    └── CurrentControlSet
        └── Services
            └── BTHPORT 
                └── Parameters
                    └── Keys
```
Você verá:

-   Uma pasta com o endereço MAC do adaptador Bluetooth.
-   Dentro dela, outra pasta com o endereço MAC do dispositivo.

Exemplo:
```
Keys
└── a1b2c3d4e5f6
    └── 112233445566
```

Onde:
 - a1b2c3d4e5f6 = adaptador Bluetooth
 - 112233445566 = mouse ou teclado



### Copiar a chave

Dentro da pasta do dispositivo:

1.  Localize o valor: LTK ou Key
2.  Clique duas vezes.
3.  Copie o valor hexadecimal exibido.

Exemplo:

1a2b3c4d5e6f77889900aabbccddeeff

Guarde esse valor.

------------------------------------------------------------------------

## 3. Inicializar o Linux

1.  Reinicie o computador.
2.  Entre no **Linux**.
3.  Não pareie o dispositivo novamente.

------------------------------------------------------------------------

## 4. Localizar o dispositivo no Linux

Abra o terminal e execute:

```bash
bluetoothctl
```
Dentro do prompt:

```bash
devices
```

Você verá algo como:

```
Device AA:BB:CC:DD:EE:FF Logitech Mouse
```
Anote o endereço MAC.

Saia com:

**exit**

------------------------------------------------------------------------

## 5. Editar a chave no Linux

### Parar o Bluetooth

sudo systemctl stop bluetooth


### Localizar a pasta do dispositivo

As chaves ficam em:

/var/lib/bluetooth/

Exemplo:

/var/lib/bluetooth/`<MAC_do_adaptador>`{=html}/`<MAC_do_dispositivo>`{=html}/


### Editar o arquivo info

sudo nano
/var/lib/bluetooth/`<MAC_ADAPTADOR>`{=html}/`<MAC_DISPOSITIVO>`{=html}/info

Procure a seção:

[LongTermKey]

Substitua o valor da chave:

Key=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

Pelo valor copiado do Windows.

------------------------------------------------------------------------

## 6. Reiniciar o Bluetooth

sudo systemctl start bluetooth

Ou reinicie o computador.

------------------------------------------------------------------------

## 7. Testar

1.  Ligue o mouse ou teclado.
2.  Ele deve conectar automaticamente.
3.  Teste nos dois sistemas.

------------------------------------------------------------------------

## Alternativa mais simples

Se você usa dispositivos Logitech:

-   Use um receptor Unifying
-   Pareie mouse e teclado nele
-   Funciona em qualquer sistema sem reconectar

------------------------------------------------------------------------

## Conclusão

Este método:

-   Elimina a necessidade de pareamento a cada troca de sistema
-   Evita duplicações de dispositivos
-   Torna o dual boot mais transparente
