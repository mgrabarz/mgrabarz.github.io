---
title: Hybrid Use Benefit - Obniż koszt maszyn wirtualnych w Azure o nawet 40%
date: 2017-04-22T14:59:42+02:00
header:
  teaser: /assets/images/2017/04/2017-04-22_hub_logo.png
categories:
  - VMs
tags:
  - Azure
  - Cost
  - HUB
  - Hybrid
  - Licensing
  - VMs
---
Jedną z powszechnych metod obniżania kosztów podczas migracji do chmury jest korzystanie z tak zwanego BYOL (Bring Your Own License). W dużym uproszeniu, po zainstalowaniu pożądanego przez nas oprogramowania nie musimy korzystać z licencji, którą daje nam dostawca chmury. Korzystamy z naszych, wcześniej zakupionych licencji (np. importowanych z rozwiązań on-premises). Tego typu model uchroni nas przed ponownymi opłatami licencyjnymi, nie był on jednak przez długi czas możliwy do zastosowania w przypadku systemu operacyjnego maszyny wirtualnej. Dla Windows Server nie da się tego zrobić ani w AWS, ani też w GCP, w Azure jest to natomiast możliwe dzięki Hybrid Use Benefit.
{: style="text-align: justify;"}

### Czym jest Hybrid Use Benefit i kto może z niego skorzystać?

Jeżeli posiadasz zakupione wcześniej licencje Windows Server z aktywnym pakietem Software Assurance przysługuje Ci wiele dodatkowych korzyści. Jedną z nich jest <a href="https://azure.microsoft.com/pl-pl/pricing/hybrid-use-benefit/" target="_blank" rel="noopener noreferrer">Hybrid Use Benefit</a>, umożliwiające użycie posiadanych licencji z serwerów on-premises na serwerach w chmurze. W efekcie cena maszyny wirtualnej uwzględnia jedynie infrastrukturę z pominięciem dodatkowych kosztów na system operacyjny.
{: style="text-align: justify;"}

![img](assets/images/2017/04/2017.04.21_hybridUse.jpg)

Poza powyższą korzyścią <a href="https://www.microsoft.com/en-us/licensing/licensing-programs/faq-software-assurance.aspxhttps://www.microsoft.com/en-us/licensing/licensing-programs/faq-software-assurance.aspx" target="_blank" rel="noopener noreferrer">Software Assurance</a> umożliwia również:
{: style="text-align: justify;"}

- Redukcję kosztu licencjonowania i usług poprzez dostęp do nowego oprogramowania i aktualizacji.
- Dostęp do bezpłatnych konsultacji przy wdrożeniach.
- Poprawienie efektywności licencjonowania poprzez dostęp do wielu benefitów (jak powyższy HUB).
- Dostęp do dodatkowych szkoleń online oraz szkoleń prowadzonych przez instruktorów.
- Wsparcie 24/365 związane z usługami.

### Jak korzystać z Hybrid Use Benefit dla nowych i istniejących maszyn wirtualnych?

W przypadku wdrażania nowych maszyn wirtualnych z Windows Server możemy skorzystać z dedykowanych obrazów w Azure. Możemy je znaleźć poprzez wpisanie frazy "HUB Windows". Maszyny wdrożone z tych obrazów mają automatycznie wyłączone naliczanie kosztów systemu operacyjnego po stronie Azure.
{: style="text-align: justify;"}

![img](assets/images/2017/04/2017-04-22_hub.png)

Podobny efekt możemy uzyskać poprzez użycie klasycznych obrazów dla maszyn wirtualnych i zaznaczenie opcji "Already have a Windows Server license?" podczas podawania właściwości maszyny.
{: style="text-align: justify;"}

![img](2017/04/2017-04-22_saveMoney.png)

Jeżeli posiadamy już maszyny wirtualne, które nie wykorzystują Hybrid Use Benefit nie mamy prostego sposobu na jego włączenie. Jedyną możliwością jest usunięcie maszyny wirtualnej i stworzenie jej od nowa z odpowiednimi ustawieniami. Jeżeli nie mamy wdrożonego configuration management może stanowić to pewien problem, ponieważ w większości przypadków nie chcemy &#8220;ręcznie&#8221; migrować konfiguracji na nową maszynę. W celu poradzenia sobie z problemem przygotowałem bardzo wstępną wersję skryptu migracyjnego. W pierwszym kroku wyłączamy i kasujemy maszynę wirtualną pozostawiając jej interfejsy sieciowe, dyski z systemem operacyjnym i z danymi. Następnie w powershell po zalogowaniu i wybrania naszych subskrypcji uruchamiamy następujący skrypt:
{: style="text-align: justify;"}

```powershell
# !!! Zanim przejdziesz do poniższych kroków upewnij się, że usunąłeś poprzednią maszynę wirtualną z zachowaniem dysków.

$location = "&lt;Lokalizacja_Serwera&gt;"
$resourceGroupName = "&lt;Nazwa_Grupy_Zasobów&gt;"
$newVmName = "&lt;Nazwa_Nowej_Maszyny_Wirtualnej&gt;"
$nicName = "&lt;Nazwa_interfejsu_Sieciowego_Starej_Maszyny&gt;"
$oldOsDiskUri = "&lt;Pełny_Storage_URL_Dysku_Starej_Maszyny&gt;"

#Wyszukujemy istniejącą kartę sieciową
$nic = Get-AzureRmNetworkInterface -Name $nicName -ResourceGroupName $resourceGroupName

#Deklarujemy nową maszynę (tutaj rozmiar Standard_D2_v2)
$vm = New-AzureRmVMConfig -VMName $newVmName -VMSize "Standard_D2_v2"
#Wpinami interfejs seciowy do nowej maszyny
$vm = Add-AzureRmVMNetworkInterface -VM $vm -Id $nic.Id

#Przygotowujemy kopię dysku z systemem operacyjnym
$osDiskName = $vmName + "osDisk"
$vm = Set-AzureRmVMOSDisk -VM $vm -Name $osDiskName -VhdUri $oldOsDiskUri -CreateOption Attach -Windows

#Tworzymy maszynę wirtualną z HUB dzięki przełącznikowi -LicenseType
New-AzureRmVM -ResourceGroupName $resourceGroupName -Location $location -VM $vm -LicenseType "Windows_Server"
```

Powyższy **skrypt jest czysto poglądowy** i powinien być dokładnie przetestowany przed zastosowaniem go na produkcyjnych rozwiązaniach, nie zawiera na przykład dołączania dysków danych, włączania diagnostyki i rozszerzeń.
{: style="text-align: justify;"}

Po przeprowadzonej migracji zachowujemy zawartość naszych maszyn wirtualnych, również adresy IP czy reguły firewall (NSG) pozostają niezmienione. Pozostaje nam jedynie sprawdzenie, że maszyna działa w nowym modelu licencyjnym. Uruchamiamy
{: style="text-align: justify;"}

```powershell
Get-AzureRmVM -ResourceGroupName $resourceGroupName -Name $newVmName
```

i sprawdzamy czy pole LicenseType wynosi Windows_Server.

### Podsumowanie

Powyżej przedstawiona migracja, mająca na celu wdrożenie własnych licencji do maszyn wirtualnych działających z Windows Server w Azure pozwala mocno obniżyć koszty jakie ponosimy w chmurze w modelu IaaS. Przykładowo dla wspomnianej wielkości maszyny wirtualnej (Standard D2 V2), według <a href="https://azure.microsoft.com/en-us/pricing/details/virtual-machines/windows/" target="_blank" rel="noopener noreferrer">cennika </a>na dzień dzisiejszy (22.04.2017) w Europe North koszty przedstawiają się następująco:
{: style="text-align: justify;"}

Koszt maszyny z HUB: ~€82.82/mo

Koszt maszyny z licencją Azure: ~€153.09/mo

Oszczędność, bez uwzględnienia Software Assurance (który pewie już posiadasz) wynosi prawie **46%!**

**Jeżeli masz jakieś uwagi, gorąco zachęcam do dodawania komentarzy pod wpisem!**
