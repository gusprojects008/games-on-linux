> Versão mais estável do BlissOS que encontrei:
> Bliss-v15.9.2-x86_64-OFFICIAL-gapps-20241012.iso

# Criação da máquina virtual

## Crie o disco virtual

> Coloque o nome de arquivo e a quantidade de GB que você quiser, mas o mínimo de GB que recomendo é 20G

```bash
qemu-img create -f qcow2 BlissOS-15.qcow2 64G
```

## Inicie a instalação

Ajuste os paths e parâmetros conforme o necessário.

```bash
qemu-system-x86_64 -enable-kvm -machine q35,accel=kvm -cpu host,kvm=on,hv_relaxed,hv_vapic,hv_spinlocks=0x1fff -smp 8,sockets=1,cores=8,threads=1 -m 4096 -bios /usr/share/edk2/x64/OVMF_CODE.fd -drive file=~/VMs/BlissOS-15.qcow2,if=virtio,cache=writeback,format=qcow2 -cdrom /run/media/gus/Ventoy/Bliss-v15.9.2-x86_64-OFFICIAL-gapps-20241012.iso -boot menu=on -device virtio-vga-gl -display gtk,gl=on,show-cursor=on -device virtio-keyboard -device virtio-tablet -device qemu-xhci -device virtio-net-pci,netdev=net0 -netdev user,id=net0,hostfwd=tcp::5555-:5555 -global ICH9-LPC.disable_s3=1
```

## Inicia a máquina

Para iniciar máquina é só utilizar o mesmo comando da instalação mas com exceção do argumento "-cdrom" ele pode ser removido, não será necessário.

Ajuste os paths para os arquivos conforme o necessário.

```bash
qemu-system-x86_64 -enable-kvm -machine q35,accel=kvm -cpu host,kvm=on,hv_relaxed,hv_vapic,hv_spinlocks=0x1fff -smp 8,sockets=1,cores=8,threads=1 -m 4096 -bios /usr/share/edk2/x64/OVMF_CODE.fd -drive file=~/VMs/BlissOS-15.qcow2,if=virtio,cache=writeback,format=qcow2 -boot menu=on -device virtio-vga-gl -display gtk,gl=on,show-cursor=on -device virtio-keyboard -device virtio-tablet -device qemu-xhci -device virtio-net-pci,netdev=net0 -netdev user,id=net0,hostfwd=tcp::5555-:5555 -global ICH9-LPC.disable_s3=1
```

ou:
qemu-system-x86_64 -enable-kvm -machine q35,accel=kvm -cpu host,kvm=on,hv_relaxed,hv_vapic,hv_spinlocks=0x1fff -smp 8,sockets=1,cores=8,threads=1 -m 4096 -bios /usr/share/edk2/x64/OVMF_CODE.fd -boot menu=on -drive file=~/VMs/BlissOS-15.qcow2,if=virtio,cache=writeback,format=qcow2 -device virtio-vga-gl -display gtk,gl=on,show-cursor=on -device virtio-tablet -device qemu-xhci -global ICH9-LPC.disable_s3=1 -device virtio-net-pci,netdev=net0 -netdev user,id=net0,hostfwd=tcp::5556-:5556

O que fazer após instalar o BlissOS:
Mudar props de device em /system/build.props.
Instalar navegador.
adicionar conta google.
Registrar o dispositivo de google device registration.

# O que é preciso fazer após a instalação

Entrar no boot manager da interface UEFI, e adicionar a entrada de boot para o arquivo .efi do BlissOS. Caso aconteça algum error e você não consiga entrar na interface de firmware, aguarde até ela aparecer, isso pode levar até 5 minutos.

O termux muitas vezes será ineficiente, o melhor é ter um shell root via adb entre a máquina host e a máquina guest (Bliss OS). Para isso é necessário:
No shell do termux:
adb shell
setprop service.adb.tcp.port 5555
exit
adb kill-server
adb start-server

Agora no host:
adb connect ipdamáquinavirtual:5555
adb root
adb shell

Isso define adb props para listar persistentemente na porta 5555.

Ná máquina host:
adb connect 192.168.122.85:5555
Coloque o IP da sua máquina virtual.

Definir porta 5555 através da opção netdev.
Realizar adb connect utilizando o endereço ip do guest e a porta 5555. Dessa forma, o shell do termux é opcional, você pode utilizar o shell do host para controlar o guest.

Agora falta:
Compartilhar aquivos entre guest e host.
Um bom terminal no guest.
Shell root do blissOS no host.

# Dual Boot

Após a instalação do BlissOS, a lista do que é necessário fazer é a mesma de uma máquina virtual.

Mas fique esperto com esses possíveis problemas:
O BlissOS pode simplesmente não conseguir gravar tela, devido à problemas com codecs.

# O que é preciso fazer após a instalação (tanto para Dual Boot quanto para máquina virtual)

Reinstalar o termux, pois ele pode ser antigo e assim não conseguir executar alguns programas.
Executar "pkg upgrade android-tools" no termux.

# Recomendo fazer após a instalação

Configurar testar configurações do desenvolvedor e blisslify para obter maior desempenho.

Utilizar módulo magisk para obter mais desempenho, provavelmente mudando o governor da CPU.

# Links

https://github.com/bryansteiner/gpu-passthrough-tutorial

# Notas

Para aumentar a fonte no termux, dê ctrl + alt + ++
Definir landscape no app de orientation.

# Lista de pacotes, dependencia e utilitários necessários

