---
id: 514
title: Jak uzyskać szablon ARM dla usług w Preview?
date: 2017-08-02T09:28:52+02:00
author: Grabarz
layout: post
guid: https://marekgrabarz.pl/?p=514
permalink: /2017/08/arm-dla-uslug-preview/
image: /wp-content/uploads/2017/08/Template-Microsoft-Azure.png
categories:
  - Tools
tags:
  - ARM
  - Azure
  - Deployment
  - Portal
  - Preview
format: aside
---
Jeden z naszych kolegów w Azurowym community, Kacper Mucha, uruchomił ostatnio swojego <a href="https://kacpermucha.github.io/" target="_blank" rel="noopener">bloga</a>. W związku z tym, że jest on niesamowitym ekspertem PowerShell w wydaniu chmurowym, na pewno będzie o czym czytać w miarę pojawiania się nowych wpisów.

Pierwszy <a href="https://kacpermucha.github.io/azure/arm/2017/08/02/arm-reverse-engineering.html" target="_blank" rel="noopener">wpis </a>zawiera poradę jak wykonać inżynierię wsteczną szablonów ARMowych. W przypadku nowych usług w Azure, będących w Public Preview,  a tym bardziej tych, które udostępniane są tylko wybranym klientom Microsoft bardzo ciężko jest taki szablon samemu stworzyć. Główne powody to oczywiście brak kompletnej dokumentacji, brak przykładów, brak wpisów w sieci na ten temat. Sposób opisany przez Kacpra jest oczywiście jak najbardziej poprawny, wydaje mi się jednak, że przy użyciu dostępnych narzędzi da się operację wykonać w nieco bardziej przystępny sposób.

## Historia wdrożeń dostępna w Portalu

Wdrożenie szablonu dokonywane jest w ramach jednej grupy zasobów (<a href="https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-manager-cross-resource-group-deployment" target="_blank" rel="noopener">chociaż od niedawna</a> mamy nieco większe spektrum możliwości) i każdorazowo składowane jest w historii wdrożeń w tej grupie. Niezależnie czy używamy szablonu z GitHuba lub Marketplace, czy wdrażamy własny szablon, nawet gdy wyklikujemy coś w portalu &#8211; wszystko ląduje w historii.

Dostęp do historii jest stosunkowo prosty. Na portalu wchodzimy do grupy zasobów, w której wykonaliśmy zmiany i otwieramy zakładkę &#8220;Deployments&#8221;. W zakładce mamy pełną historię wdrożeń od samego początku jej istnienia. Historia zawiera nie tylko szablon, ale również wszystkie parametry wejściowe oraz log z wykonanych kroków (niejednokrotnie zakończony błędami). W tym punkcie warto powtórzyć raz jeszcze część powyższego zdania &#8211; wszystkie parametry wejściowe, które zostały przekazane &#8220;plain text&#8217;em&#8221; są tam dostępne, mogą zawierać loginy/hasła, klucze &#8211; warto wtedy pomyśleć o <a href="https://azure.microsoft.com/en-us/services/key-vault/" target="_blank" rel="noopener">KeyVault</a>!

<img class="alignnone wp-image-517 size-large" src="https://marekgrabarz.pl/wp-content/uploads/2017/08/DeploymentsBlade-1024x256.png" alt="" width="730" height="183" srcset="https://marekgrabarz.pl/wp-content/uploads/2017/08/DeploymentsBlade-1024x256.png 1024w, https://marekgrabarz.pl/wp-content/uploads/2017/08/DeploymentsBlade-300x75.png 300w, https://marekgrabarz.pl/wp-content/uploads/2017/08/DeploymentsBlade-768x192.png 768w, https://marekgrabarz.pl/wp-content/uploads/2017/08/DeploymentsBlade.png 1689w" sizes="(max-width: 730px) 100vw, 730px" /> 

Szablon oczywiście możemy skopiować, zmodyfikować, wdrożyć ponownie &#8211; wszystko wewnątrz portalu. Wrażenia zbliżone są do Visual Studio i okienka &#8220;JSON Outline&#8221;, które zwykle jest używane przy tworzeniu szablonów. Kacper przytoczył BotService jak jedna z tych usług, która jest w Preview bez dobrych przykładów szablonów. Z historii uzyskałem taki szablon:

<pre class="EnlighterJSRAW" data-enlighter-language="null">{
    "$schema": "http://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "name": {
            "type": "String"
        },
        "location": {
            "type": "String"
        },
        "subscriptionId": {
            "type": "String"
        }
    },
    "resources": [
        {
            "type": "Microsoft.Web/sites",
            "kind": "functionapp,botapp",
            "name": "[parameters('name')]",
            "apiVersion": "2016-03-01",
            "location": "[parameters('location')]",
            "properties": {
                "name": "[parameters('name')]",
                "siteConfig": {
                    "appSettings": [
                        {
                            "name": "WEBSITE_NODE_DEFAULT_VERSION",
                            "value": "6.5.0"
                        }
                    ]
                }
            }
        }
    ]
}
</pre>

## Jak dobrać się do historii bez użycia Portalu?

Skoro historia leży &#8220;w Azure&#8221;, to na logikę powinna być ona dostępna z poziomu innych narzędzi.

Z PowerShella wydobywamy ją w następujący sposób:

<pre class="EnlighterJSRAW" data-enlighter-language="null">#Deployments
Get-AzureRmResourceGroupDeployment -ResourceGroupName &lt;RGName&gt;

#Failed deployments
Get-AzureRmResourceGroupDeployment -ResourceGroupName &lt;RGName&gt;| Where-Object ProvisioningState -eq Failed

#Deployment details
Get-AzureRmResourceGroupDeploymentOperation -ResourceGroupName &lt;RGName&gt; -DeploymentName &lt;DeplymentName&gt;

#Download template
Save-AzureRmResourceGroupDeploymentTemplate -ResourceGroupName &lt;RGName&gt; -DeploymentName &lt;DeplymentName&gt;</pre>

Z Azure CLI:

<pre class="EnlighterJSRAW" data-enlighter-language="null">#Deployments
az group deployment list --resource-group &lt;RGName&gt;

#Deployment details
az group deployment show --resource-group &lt;RGName&gt; --name &lt;DeploymentName&gt; --json

#Export template
az group deployment export --resource-group &lt;RGName&gt; --name &lt;DeploymentName&gt;</pre>

Podobne dane uzyskamy poprzez REST API i z resources.azure.com. Po prostu wybierz metodę, z którą czujesz się najbardziej komfortowo, zawartość będzie identyczna.

&#8212;

Daj znać jak Ty tworzysz szablony ARM. Może znasz jeszcze inny sposób jak je generować?

&nbsp;