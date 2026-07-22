# Free fire no waydroid

## Vantagens

## Desvantagens

## Erros/Bugs comuns e como concerta-los

### "ERROR][2026-07-21 00:39:29][MainProcess:15214][sdk.py:139] - Failed to get persist property sys.boot_completed: Command failed with return code -9: [00:39:19] Waiting for binder Service Manager... [00:39:19] Failed to get service waydroidplatform, trying again..."

Esse é um problema de APIs gráficas que o waydroid está tentnado utilizar para inicializar o boot com interface gráfica. Você pode definir drivers básico para manter a estabilidade/compatibilidade com o waydroid e assim, continuar a inicialização:

faça backup do waydroid_base.props:

```bash
sudo cp /var/lib/waydroid/waydroid_base.props /var/lib/waydroid/waydroid_base.props.bak
```
Susbtitua e adicione essas props em /var/lib/waydroid/waydroid_base.props :
ro.hardware.gralloc=default
ro.hardware.egl=swiftshader

Isso resolve temporáriamente, mas a causa mais provável é que o waydroid está selecionando forçadamente apenas uma interface de dispositivo DRM de vídeo específica, provavelmente a da placa de vídeo, o que pode causar erros. O ideal é que primeiro ela utiliza a interface DRM da placa de vídeo integrada primeir durante a inicialização, e depois utilize a placa de video caso seja necessário. então você precisa tincorreta, então liste suas interfaces DRM de dispositivos de vídeo:

```bash
sudo waydroid shell
ls -lah /dev/dri/
```

Depois execute no host:

```bash
ls -lah /dev/dri/
```

Identifique a interface DRM da placa de vídeo integrada e depois da dedicada.

Faça o backup:

```bash
sudo cp /var/lib/waydroid/lxc/waydroid/config_nodes /var/lib/waydroid/lxc/waydroid/config_nodes.bak
```

```bash
sudo vim /var/lib/waydroid/lxc/waydroid/config_nodes
```

Troque a ordem de uso das interface (o nome delas pode variar, então utilize o nome correpondente à sua interface DRM da placa de vídeo integrada ou dedicada):
lxc.mount.entry = /dev/dri/renderD128 dev/dri/renderD128 none bind,create=file,optional 0 0

Troque por (exemplo):
lxc.mount.entry = /dev/dri/renderD129 dev/dri/renderD128 none bind,create=file,optional 0 0

renderD129 é a interface DRM da minha placa de vídeo integrada, e agora a inicialização está funcionando.

No fim, se ainda der error, teste a variação da ordem de uso das interfaces DRM das placas de vídeo, alterando o mesmo local /var/lib/waydroid/lxc/waydroid/config_node .

### Panda mouse pro não reconhece teclado e mouse:

Ative a prop uevents para true no waydroid, para permitir ele capturar eventos de teclado e mouse diretamente, isso pode ser feito atrav
és do waydroid-helper.

Após isso, reinicia o container, abra o panda mouse pro, ative ele, e desconecte e conecte o teclado e mouse.

Ao abrir seu keymapper (panda mouse pro ou outro), desconecte e conecte o teclado e o mouse.

### Erros relacionado à falta de uma sessão wayland "WAYLAND_DISPLAY is not set".

Inicialize o waydroid através do utilitário "cage" ou através do weston, ou através da opção "Launch waydroid with weston" no waydroid-helper.

### Erros de crash ou jogo não estar abrindo

Isso pode estar acontecendo por diversos motivos, mas algumas soluções são:

##### Realizar spoof de dispositivo específico

Defina um dispositivo específico em /system/build.props

##### Utilize o waydroid com gapps, faça login e registre seu dispositivo.

Registre seu dispositivo através das instruções que o google recomenda (não esqueça de fazer isso utilizando a mesma conta google que você utiliza no waydroid): https://www.google.com/android/uncertified

##### Teste diferentes libs de tradução de instruções para jogos

Os principais motivos de crash ou jogo são abrindo é por falta da libhoudini ou ndk, ou necessidade delas em uma versão específica. Por isso é necessário testa-las.

O free fire depende da libhoudini específica . A que funcionou foi instalada através do waydroid_scripts.

##### Erro ao abrir waydroid-helper

Instale a libsecret para não dar error ao abrir appimages.

##### failed to commit changes to dconf: Error spawning command line “dbus-launch --autolaunch=ae1cbd179e7... --binary-syntax --close-stderr”: Child process exited with code 1

A sessão gráfica deve estar dentro de dbus-run-session, isso depende do seu ambiente gráfico e como ele é inicializado, se estiver em um ambiente minmalista, configure o ~/.xinitrc para algo como:

exec dbus-run-session i3

No lugar do "i3" pode ser outro ambiente gráfico.

# Rodar o waydroid e jogos a 120 FPS

Crie o arquivo de configuração weston em ~/.config/weston/weston.ini com o conteúdo:
[core]
backend=x11-backend.so

[output]
name=eDP-1
mode=1920x1080@120

Coloque sua resolução e taxa de FPS normal.

Inicie o weston com essa configuração:

weston -c ~/.config/weston/weston.ini

Abra o terminal e execute o waydroid.

# Conceitos a entender
Weston.

# Necessário ou recomendado fazer

Instale o MockGPS através do waydroid-helper.


# Para resolver problemas

Dados sobre a libhoudini que o waydroid_script obtém:

[gus@void ~/Documents/games-on-linux/waydroid/houdini]$ readelf -p .comment libhoudini.so

String dump of section '.comment':
  [     0]  clang version 13.0.0 (531fdb0190bc6f15636a9d7f3b4685f9f1cdd5d5)
  [    40]  clang version 14.0.6 (https://github.com/llvm/llvm-project.git f28c006a5895fc0e329fe15fead81e37457cb1d1)

[gus@void ~/Documents/games-on-linux/waydroid/houdini]$ readelf -p .comment libhoudini64.so

String dump of section '.comment':
  [     0]  clang version 13.0.0 (531fdb0190bc6f15636a9d7f3b4685f9f1cdd5d5)
  [    40]  clang version 14.0.6 (https://github.com/llvm/llvm-project.git f28c006a5895fc0e329fe15fead81e37457cb1d1)

[gus@void ~/Documents/games-on-linux/waydroid/houdini]$ file libhoudini.so
libhoudini.so: ELF 32-bit LSB shared object, Intel i386, version 1 (SYSV), dynamically linked, BuildID[sha1]=477847ceec6fe4f9bba47c79fb4511cea1861ebf, stripped
[gus@void ~/Documents/games-on-linux/waydroid/houdini]$ file libhoudini64.so
libhoudini64.so: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, BuildID[sha1]=a8f9eb16786fb84cf49ef2a32078627220de72bb, stripped

[gus@void ~/Documents/games-on-linux/waydroid/houdini]$ waydroid prop get ro.product.cpu.abilist
x86_64,x86,arm64-v8a,armeabi-v7a,armeabi

Links de onde o waydroid_scripts obtém a libhoudini:
Para android 11:
["https://github.com/supremegamers/vendor_intel_proprietary_houdini/archive/81f2a51ef539a35aead396ab7fce2adf89f46e88.zip", "fbff756612b4144797fbc99eadcb6653"]

Para android 13:
["https://github.com/supremegamers/vendor_intel_proprietary_houdini/archive/9e77896350caccd228b36b2e1b4a994aa4bd48da.zip", "3807fe029559db3037efe245d9e74270"]

Tudo incluindo as props que ele injeta estão em:
waydroid_script/stuff/houdini.py

# Links úteis
https://github.com/1999AZZAR/use-waydroid-on-x11
https://github.com/waydroid-helper/waydroid-helper
https://github.com/Quackdoc/waydroid-scripts/tree/main
https://www.reddit.com/r/waydroid/comments/1bnglzb/failed_to_get_waydroidplatform_trying_again/

# Agora

Conseguir utilizar a placa de vídeo e 120fps no waydroid.

Sair da black list.

# Minha visão atualmente

Atualmente o waydroid é muito instável, principalmente para jogos, por mais que tenha uma vantagem técnica teórica de desemepnho em relação a uma máquina virtual, ele ainda facilmente quebra e possui muitos bugs de compatibilidade, ou seja, inviável e ineficiente atualmente.

# Lista de pacotes, dependencia e utilitários necessários


