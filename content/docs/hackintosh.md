---
title: "Hackintosh"
date: 2024-03-10T22:20:55-03:00
draft: false
---

Um hackintosh é um computador que usa o sistema operacional macOS, desenvolvido pela Apple, mas que não é um produto da marca. Isso significa instalar o sistema em um hardware que não foi projetado para ele, como um PC ou laptop comum. Essa prática não é ilegal por si mesma, mas pode violar os termos de serviço da Apple e resultar em problemas legais -- além de não haver suporte técnico (fora você mesmo 😉).

Justamente por tratar-se da instalação de um OS numa máquina para a qual ele não foi projetado, não será uma tarefa simples como baixar um .iso e gravá-lo num pendrive e "bootar" na BIOS.

Recomendo que façam o download do programa [AIDA64]([Downloads | AIDA64](https://www.aida64.com/downloads)):
![a](/aida.png)
Procure e anote as seguintes informações:

- CPU
- Chipset (placa-mãe)
- GPU
- Rede
  No meu caso:
- CPU: RYZEN 5 2600
- CHIPSET: AMD A320
- GPU: RX 570
- REDE: Realtek(R) PCI(e) Ethernet Controller

> Atenção: caso você tenha uma placa de vídeo NVIDEA, não será possível fazer o processo de hackintosh pois não há suporte para esse hardware. Apenas se tiver uma APU Intel (APUs da AMD não têm vídeo suportado) será possível utilizar o macOS.

## EFI

Agora que temos as informações necessárias para começar o processo de EFI; faremos o download de dois arquivos:

- uma EFI base;
- `macrecovery.py` (recuperação do macOS a partir dos servidores da Apple)

1. Criaremos uma pasta, que chamarei de `pendrive` (aqui, o nome é irrelevante)
2. Dentro dessa pasta, jogaremos nossa EFI base e macrecovery-1.0.0:
   ![[pasta.png]]
3. Certifique-se de que o Python está instalado no seu sistema.
4. Abra o diretório do macrecovery-1.0.0 no Terminal apertando SHIFT + Botão direito do mouse _ou_ abra o prompt de comando clicando no caminho do diretório e digitando `cmd`:
   ![[cmd.png]] 5. Escolha a versão do macOS que deseja baixar no arquivo `recovery_urls.txt`:
   ![[versao_recovery.png]] 6. Copie o comando e cole-o clicando com o botão direito no terminal/prompt de comando:
   ![[download_recovery.png]] 7. Remova todos os arquivos na pasta com exceção dos dois baixados, `BaseSystem.dmg` e `BaseSystem.chunklist`. 8. Renomeie essa mesma pasta para `com.apple.recovery.boot`.
   Agora, podemos partir para a modificação do arquivo `config.plist` através do [OCAT](https://github.com/ic005k/OCAuxiliaryTools/releases/tag/20240001) ou do [ProperTree](https://github.com/corpnewt/ProperTree). Eu, particularmente, recomendo o uso do OCAT.
5. No OCAT, vá em `File -> Open` e selecione o arquivo `config.plist` no caminho `...\EFI\OC\config.plist`.
6. Vá em `Kernel -> Add` e clique no ícone `+`, no canto inferior direito, para adicionar os kexts condizentes com o seu hardware:

### Áudio

| Kext                                                              | Descrição                                                                                                                                                                                                               |
| :---------------------------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [AppleALC.kext](https://github.com/acidanthera/AppleALC/releases) | Usado para correção AppleHDA, permitindo suporte para a maioria dos controladores de som integrados. <br>AMD 15h/16h pode ter problemas com isso e os sistemas Ryzen/Threadripper raramente têm suporte para microfone. |
| [VoodooHDA.kext](https://sourceforge.net/projects/voodoohda/)     | Áudio para sistemas FX e suporte Mic+Audio do painel frontal para sistema Ryzen, não misture com AppleALC. <br>A qualidade do áudio é visivelmente pior do que AppleALC em CPUs Zen.                                    |

### Ethernet

| Kext                                                                                               | Description                                                                                                                                                                                           |
| :------------------------------------------------------------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [IntelMausi.kext](https://github.com/acidanthera/IntelMausi/releases)                              | As NICs 82578, 82579, I217, I218 e I219 da Intel são oficialmente suportadas.                                                                                                                         |
| [AtherosE2200Ethernet.kext](https://github.com/Mieze/AtherosE2200Ethernet/releases)                | Necessário para Atheros e Killer NICs. <br>**Nota**: Os modelos Atheros Killer E2500 são na verdade baseados em Realtek. Para esses sistemas, use RealtekRTL8111.                                     |
| [RealtekRTL8111.kext](https://github.com/Mieze/RTL8111_driver_for_OS_X/releases)                   | Para Gigabit Ethernet da Realtek. <br>Às vezes, a versão mais recente do kext pode não funcionar corretamente com sua Ethernet. Se você vir esse problema, tente versões mais antigas.                |
| [LucyRTL8125Ethernet.kext](https://www.insanelymac.com/forum/files/file/1004-lucyrtl8125ethernet/) | Para Ethernet de 2,5 Gb da Realtek.                                                                                                                                                                   |
| [SmallTreeIntel82576.kext](https://github.com/khronokernel/SmallTree-I211-AT-patch/releases)       | Necessário para NICs I211, baseado no kext SmallTree, mas corrigido para suportar I211. <br>Necessário para a maioria das placas AMD que executam NICs Intel.                                         |
| [AppleIGB.kext](https://github.com/donatengit/AppleIGB/releases)                                   | Necessário para NICs I211 em execução no macOS Monterey e acima. Pode ter problemas de instabilidade em algumas NICs, recomendado permanecer em Big Sur e usar SmallTree. Requer macOS 12 e superior. |

### Wi-fi e Bluetooth

| Kext                                                                                           | Descrição                                                                                                                                     |
| :--------------------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------- |
| [AirportItlwm](https://github.com/OpenIntelWireless/itlwm/releases)                            | Adiciona suporte para uma grande variedade de cartões INTEL WIRELESS e funciona nativamente na recuperação graças à integração IO80211Family. |
| [IntelBluetoothFirmware](https://github.com/OpenIntelWireless/IntelBluetoothFirmware/releases) | Adiciona suporte Bluetooth ao macOS quando emparelhado com uma placa sem fio Intel.                                                           |
| [AirportBrcmFixup](https://github.com/acidanthera/AirportBrcmFixup/releases)                   | Usado para corrigir cartões Broadcom não Apple/não Fenvi, não funcionará em Intel, Killer, Realtek, etc.                                      |
| [BrcmPatchRAM](https://github.com/acidanthera/BrcmPatchRAM/releases)                           | Usado para carregar firmware no chipset Broadcom Bluetooth, necessário para todos os cartões não Apple/não Fenvi Airport.                     |

### USB

| Kext                                                                                               | Description                                                                                                                                                                                                                                                                   |
| :------------------------------------------------------------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [USBInjectAll](https://github.com/daliansky/OS-X-USB-Inject-All/releases)                          | Usado para injetar controladores Intel USB em sistemas sem portas USB definidas no ACPI. <br>Todas as séries de chipsets Intel. <br>Requer OS X 10.11 ou mais recente.                                                                                                        |
| [XHCI-unsupported](https://github.com/daliansky/OS-X-USB-Inject-All/archive/refs/heads/master.zip) | Necessário para controladores USB não nativos. <br>Chipsets comuns que precisam disso: H370, B360, H310, Z390 (não necessário no Mojave e mais recentes), X79, X99, placas AsRock (em placas-mãe Intel especificamente, B460/No entanto, as placas Z490+ não precisam disso). |

### Outros

| Kext                                                                                                                  | Description                                                                                                                                                                                                    |
| :-------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [NVMeFix](https://github.com/acidanthera/NVMeFix/releases)                                                            | Usado para fixar o gerenciamento de energia e a inicialização no NVMe não-Apple.                                                                                                                               |
| [SATA-Unsupported](https://github.com/khronokernel/Legacy-Kexts/blob/master/Injectors/Zip/SATA-unsupported.kext.zip)  | Adiciona suporte para uma grande variedade de controladores SATA, principalmente relevantes para laptops que apresentam problemas para ver a unidade SATA no macOS. <br>Recomendamos testar sem isso primeiro. |
| [AppleMCEReporterDisabler](https://github.com/acidanthera/bugtracker/files/3703498/AppleMCEReporterDisabler.kext.zip) | É útil começar com Catalina para desativar o kext AppleMCEReporter, o que causará pânico no kernel em CPUs AMD. <br>Recomendado para sistemas de soquete duplo (ou seja. Intel Xeons).                         |
| [RestrictEvents](https://github.com/acidanthera/RestrictEvents/releases)                                              | Melhor experiência com processadores não suportados como AMD, Desative os avisos de memória MacPro7,1 e forneça atualização para o macOS Monterey por meio de atualizações de software quando disponíveis.     |
| [CpuTscSync](https://github.com/acidanthera/CpuTscSync/releases)                                                      | É um plugin Lilu, combinando funcionalidade do VoodooTSCSync e desativando xcpm_urgency se o TSC não estiver sincronizado. Deve resolver alguns pânicos do kernel após o wake.                                 |
| [SMDRadeonGPU](https://github.com/ChefKissInc/RadeonSensor)                                                           | Usado para monitorar a temperatura da GPU em sistemas de GPU AMD. Requer RadeonSensor do mesmo repositório. Requer macOS 11 ou mais recente.                                                                   |

### ACPI Tables

Esses arquivos **DEVEM** estar inclusos no diretório `ACPI` da sua pasta EFI. Recomendo jogar esses arquivos `prebuilt` lá, ao invés de um processo manual.

- [SSDT-PLUG](https://github.com/dortania/Getting-Started-With-ACPI/raw/master/extra-files/compiled/SSDT-PLUG-DRTNIA.aml)
- [SSDT-EC-USBX](https://github.com/dortania/Getting-Started-With-ACPI/raw/master/extra-files/compiled/SSDT-EC-USBX-DESKTOP.aml)
- [SSDT-AWAC](https://github.com/dortania/Getting-Started-With-ACPI/raw/master/extra-files/compiled/SSDT-AWAC.aml)
- [SSDT-RHUB](https://github.com/luchina-gabriel/youtube-files/raw/main/SSDT-RHUB.aml)

3. Agora que temos todos os `kexts` e as `tabelas` propriamente colocados na nossa EFI via OCAT, acesse `PI -> Generic`.
4. Você deverá alterar os seguintes valores:
   - _MLB (Board Serial)_
     - *ROM (Mac Address)*
   - _SystemSerialNumber (Serial)_
   - _SystemUUID (SmUUID)_
5. Baixe o arquivo [genSMBIOS](https://github.com/corpnewt/GenSMBIOS/archive/refs/heads/master.zip) e rode seu script `.bat` (Windows) ou `.py`.
   ![[sm_bios1.png]]
   Selecione um modelo mac para seu sistema. Você provavelmente deverá procurar essa informação online; no meu caso trata-se do iMacPro1,1 :
   ![[sm_bios2.png]]
   Preencha os valores no OCAT com seus correspondentes novos como acima.
6. Salve suas modificações.
   > Importante: nunca use os números de serial de outra pessoa que, por ventura, tenha compartilhado sua `config.plist`. Isso causará diversos problemas, incluindo (e severamente) com o iCloud.

## Pronto!

Agora você já pode transferir as duas pastas de dentro de `pendrive` (ou o que quer que você tenha escolhido nomear) para o seu dispositivo USB e bootá-lo na BIOS.

> Importante: você deverá formatar seu disco atual ou particioná-lo com o Disk Utility no menu de instalação do macOS antes de iniciar o processo de instalação/recuperação.

## Só mais uma coisinha...

Depois que o macOS estiver instalado, certifique-se que ele será acessível sem o seu USB ao particionar o disco do sistema novamente criando uma partição de cerca de 250 MB do tipo `MS-DOS` chamado `EFI` através da ferramenta de disco do sistema. Na BIOS, selecione `OpenCore` como primeira opção de boot.
![[disk_utility.png]]

# Aproveite seu Hackintosh!

![[Screenshot 2024-03-10 at 22.35.18.png]]
