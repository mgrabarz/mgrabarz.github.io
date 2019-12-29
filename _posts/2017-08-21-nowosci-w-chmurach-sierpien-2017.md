---
id: 520
title: 'Nowości w chmurach &#8211; sierpień 2017'
date: 2017-08-21T23:35:16+02:00
author: Grabarz
layout: post
guid: https://marekgrabarz.pl/?p=520
permalink: /2017/08/nowosci-w-chmurach-sierpien-2017/
image: /wp-content/uploads/2017/08/16100681368_59c91ee0e1_c.jpg
categories:
  - News
tags:
  - AppService
  - Azure
  - AzureStack
  - Backup
  - Containers
  - DataFactory
  - EventGrid
  - Functions
  - Java
  - Jenkins
  - News
  - VMs
---
W zeszły czwartek miałem okazję po raz pierwszy gościć, tym razem po drugiej stronie ekranu, na Webinarze organizowanym przez <a href="https://chmurowisko.pl" target="_blank" rel="noopener">chmurowisko.pl</a> i <a href="https://twitter.com/miroburn" target="_blank" rel="noopener">Mirka Burnejko</a>. Celem było przedstawienie nowości, jakie pojawiły się u trzech wiodących dostawców chmurowych.

Część związana z Google Cloud Platform została przedstawiona przez <a href="https://www.linkedin.com/in/michal-szafranski-0490a015/" target="_blank" rel="noopener">Michała Szafrańskiego</a>, nowości z Amazon Web Services omówił <a href="https://www.linkedin.com/in/piotrowskipawel/" target="_blank" rel="noopener">Paweł Piotrowski</a>, mi przypadł w udziale Azure. Całe nagranie wyszło bardzo fajnie mimo kilku wpadek technicznych, ale te pewnie zostaną wyeliminowane przy następnej okazji. Panowie, jeśli kiedyś traficie na mój wpis, chciałem Wam pogratulować &#8211; wykonaliście kawał dobrej roboty!

Zapraszam do oglądania, naprawdę warto.[embedyt] https://www.youtube.com/watch?v=3HosGSFOW1U[/embedyt]

## Linki do materiałów na temat nowości w Microsoft Azure

#### Azure gov in Australia:

<a href="https://azure.microsoft.com/en-us/blog/microsoft-azure-expands-with-two-new-regions-for-australia/" target="_blank" rel="noopener">https://azure.microsoft.com/en-us/blog/microsoft-azure-expands-with-two-new-regions-for-australia/</a>

<a href="https://azure.microsoft.com/en-us/regions/" target="_blank" rel="noopener">https://azure.microsoft.com/en-us/regions/</a>

#### Azure Stack

<a href="https://azure.microsoft.com/pl-pl/overview/azure-stack/" target="_blank" rel="noopener">https://azure.microsoft.com/pl-pl/overview/azure-stack/</a>

Dostępność od września przez wybranych partnerów (IBM, HPE, Lenovo), kolejni partnerzy np Cisco i Huawei zaczną dostarczać w późniejszym terminie.

Nowe, hybrydowe podejście w modelu PaaS. Cykl wydań w Azure Stack będzie zbliżony do głównych wydań Microsoft Azure.

<a href="https://azure.microsoft.com/pl-pl/overview/azure-stack/development-kit" target="_blank" rel="noopener">https://azure.microsoft.com/pl-pl/overview/azure-stack/development-kit</a>

#### Nested Virtualization

<a href="https://azure.microsoft.com/pl-pl/blog/nested-virtualization-in-azure/" target="_blank" rel="noopener">https://azure.microsoft.com/pl-pl/blog/nested-virtualization-in-azure/</a>

Umożliwia realizację następujących scenariuszy:

  * Budowanie własnej infrastruktury kontenerów na Azure VMs
  * Wspiera uruchamianie środowisk dev/test oraz lab.
  * Uruchamianie emulatorów w oparciu o Hyper-V
  * Wsparcie dla własnych obrazów z on-premises

#### Instant File Recovery

<a href="https://azure.microsoft.com/en-us/blog/instant-file-recovery-from-azure-vm-backups-is-now-generally-available/" target="_blank" rel="noopener">https://azure.microsoft.com/en-us/blog/instant-file-recovery-from-azure-vm-backups-is-now-generally-available/</a>

  * Umożliwia szybkie odzyskiwanie plików z kopii zapasowej bez potrzeby użycia dodatkowej infrastruktury. Odzyskiwanie może się odbyć bezpośrednio z Azure VM.
  * Umożliwia montowanie danych bezpośrednio z backupów, bez potrzeby odtwarzania całej maszyny wirtualnej, to z kolei znacząco przyspiesza proces testowania i analizy przy utraconych danych.

#### JIT VM Access

<a href="https://azure.microsoft.com/pl-pl/blog/announcing-the-just-in-time-vm-access-public-preview/" target="_blank" rel="noopener">https://azure.microsoft.com/pl-pl/blog/announcing-the-just-in-time-vm-access-public-preview/</a>

Tymczasowe umożliwienie dostępu do maszyn wirtualnych. Zarządzane uprawnieniami, monitorowane i logowane.

#### Azure Container Instances

<a href="https://azure.microsoft.com/en-us/blog/announcing-azure-container-instances/" target="_blank" rel="noopener">https://azure.microsoft.com/en-us/blog/announcing-azure-container-instances/</a>

Nowa usługa chmurowa, w pewnym sensie działająca w modelu serverless. Płatność w oparciu o czas i ilość zasobów używanych przez instancję kontenera.

Przeznaczona dla prostych rozwiązań oraz takich obrazów, które nie mają złożonych wymagań co do sieci lub storage (mimo, że umożliwia montowanie Azure Storage Files po SMB 3.0).

Jednocześnie zostało ogłoszone, że Microsoft stał się platynowym członkiem CNCF. Opublikował OSS konektor do ACI dla Kubernetes.

#### AppService Environment V2 oraz AppService Isolated

<a href="https://azure.microsoft.com/en-us/blog/announcing-app-service-environment-v2/" target="_blank" rel="noopener">https://azure.microsoft.com/en-us/blog/announcing-app-service-environment-v2/</a>

<a href="https://azure.microsoft.com/pl-pl/blog/announcing-app-service-isolated-more-power-scale-and-ease-of-use/" target="_blank" rel="noopener">https://azure.microsoft.com/pl-pl/blog/announcing-app-service-isolated-more-power-scale-and-ease-of-use/</a>

Do tej pory ASE wymagało zadeklarowania infrastruktury z wyprzedzeniem. Obecnie w wersji v2 mamy pełne skalowanie zarówno horyzontalne, jak i wertykalne.

#### Azure Event Grid

<a href="https://azure.microsoft.com/pl-pl/resources/videos/azure-friday-azure-event-grid-banisadr/" target="_blank" rel="noopener">https://azure.microsoft.com/pl-pl/resources/videos/azure-friday-azure-event-grid-banisadr/</a>

IFTTT/Zappier dla zasobów Azure. Poza wbudowanymi zdarzeniami dla zasobów umożliwia definiowanie własnych zdarzeń w aplikacjach. Wspiera model jeden do wielu (topic-subscription).

#### Azure Durable Functions

<a href="https://blogs.msdn.microsoft.com/appserviceteam/2017/07/06/alpha-preview-for-durable-functions/" target="_blank" rel="noopener">https://blogs.msdn.microsoft.com/appserviceteam/2017/07/06/alpha-preview-for-durable-functions/</a>

Alpha preview – nie używać na produkcji. Trochę przeciwstawne podejście do powyższego Event Grid’a.

#### VS 2017.3 Release

<a href="https://www.visualstudio.com/en-us/news/releasenotes/vs2017-relnotes" target="_blank" rel="noopener">https://www.visualstudio.com/en-us/news/releasenotes/vs2017-relnotes</a>

Długo oczekiwany release VS, który wprowadza wsparcie dla Azure Functions i pełną integrację z Azure, Azure China, Azure Gov i Azure Stack.

#### Data Management Gateway

<a href="https://azure.microsoft.com/pl-pl/blog/data-management-gateway-high-availability-and-scalability-preview/" target="_blank" rel="noopener">https://azure.microsoft.com/pl-pl/blog/data-management-gateway-high-availability-and-scalability-preview/</a>

Wysoka dostępność I skalowalność dla bramki Azure Data Factory.

#### Managed Applications

<a href="https://azure.microsoft.com/pl-pl/blog/azure-managed-applications/" target="_blank" rel="noopener">https://azure.microsoft.com/pl-pl/blog/azure-managed-applications/</a>

Pozwalają opublikować gotowe rozwiązania do klientów. Nie wymagają od klientów wchodzenia w technikalia danego rozwiązania lub platformy. Są monolitycznymi black-boxami. Resource group będzie zamknięta dla klienta.

Mogą być publikowane w „Service Catalog” – czyli zaakceptowane przez nas, zgodne ze standardami rozwiązania, lub w „Marketplace Catalog” – typowe rozwiązania od ISV.

#### Java in Azure

<a href="https://docs.microsoft.com/en-us/java/azure" target="_blank" rel="noopener">https://docs.microsoft.com/en-us/java/azure</a>

Nowa strona dla programistów Java używających Azure. Jednocześnie pojawiło się wsparcie Azure w Maven.

#### Jenkins in Azure

<a href="https://docs.microsoft.com/en-us/azure/jenkins/" target="_blank" rel="noopener">https://docs.microsoft.com/en-us/azure/jenkins/</a>

Powrót do Marketplace. Azure AppService Plugin, Azure Storage Plugin + managed disks.

#### Azure Batch Rendering

<a href="https://rendering.azure.com/" target="_blank" rel="noopener">https://rendering.azure.com/</a>

Renderowanie na żądanie: Autodesk, 3DS Max, Solidangle Arnold, V-Ray.

#### Azure Media Redactor

<a href="https://azure.microsoft.com/pl-pl/blog/general-availability-azure-media-redactor/" target="_blank" rel="noopener">https://azure.microsoft.com/pl-pl/blog/general-availability-azure-media-redactor/</a>

Rozmywanie twarzy w Twoim wideo lub nawet w live stream.

&#8212;

Photo credit: [Nicolas Alejandro Street Photography](https://www.flickr.com/photos/nalejandro/16100681368/) via [Visualhunt.com](https://visualhunt.com/re/3ea3a1) /  [CC BY](http://creativecommons.org/licenses/by/2.0/)