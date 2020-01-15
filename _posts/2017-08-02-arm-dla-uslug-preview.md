---
title: Jak uzyskać szablon ARM dla usług w Preview?
date: 2017-08-02T09:28:52+02:00
header:
  teaser: /assets/images/2017/08/Template-Microsoft-Azure.png
categories:
  - Tools
tags:
  - ARM
  - Azure
  - Deployment
  - Portal
  - Preview
---
Jeden z naszych kolegów w Azurowym community, Kacper Mucha, uruchomił ostatnio swojego <a href="https://kacpermucha.github.io/" target="_blank" rel="noopener">bloga</a>. W związku z tym, że jest on niesamowitym ekspertem PowerShell w wydaniu chmurowym, na pewno będzie o czym czytać w miarę pojawiania się nowych wpisów.
{: style="text-align: justify;"}

Pierwszy <a href="https://kacpermucha.github.io/azure/arm/2017/08/02/arm-reverse-engineering.html" target="_blank" rel="noopener">wpis </a>zawiera poradę jak wykonać inżynierię wsteczną szablonów ARMowych. W przypadku nowych usług w Azure, będących w Public Preview,  a tym bardziej tych, które udostępniane są tylko wybranym klientom Microsoft bardzo ciężko jest taki szablon samemu stworzyć. Główne powody to oczywiście brak kompletnej dokumentacji, brak przykładów, brak wpisów w sieci na ten temat. Sposób opisany przez Kacpra jest oczywiście jak najbardziej poprawny, wydaje mi się jednak, że przy użyciu dostępnych narzędzi da się operację wykonać w nieco bardziej przystępny sposób.
{: style="text-align: justify;"}

## Historia wdrożeń dostępna w Portalu

Wdrożenie szablonu dokonywane jest w ramach jednej grupy zasobów (<a href="https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-manager-cross-resource-group-deployment" target="_blank" rel="noopener">chociaż od niedawna</a> mamy nieco większe spektrum możliwości) i każdorazowo składowane jest w historii wdrożeń w tej grupie. Niezależnie czy używamy szablonu z GitHuba lub Marketplace, czy wdrażamy własny szablon, nawet gdy wyklikujemy coś w portalu &#8211; wszystko ląduje w historii.
{: style="text-align: justify;"}

Dostęp do historii jest stosunkowo prosty. Na portalu wchodzimy do grupy zasobów, w której wykonaliśmy zmiany i otwieramy zakładkę **Deployments**. W zakładce mamy pełną historię wdrożeń od samego początku jej istnienia. Historia zawiera nie tylko szablon, ale również wszystkie parametry wejściowe oraz log z wykonanych kroków (niejednokrotnie zakończony błędami). W tym punkcie warto powtórzyć raz jeszcze część powyższego zdania - wszystkie parametry wejściowe, które zostały przekazane **plain textem** są tam dostępne, mogą zawierać loginy/hasła, klucze - warto wtedy pomyśleć o <a href="https://azure.microsoft.com/en-us/services/key-vault/" target="_blank" rel="noopener">KeyVault</a>!
{: style="text-align: justify;"}

![img](/assets/images/2017/08/DeploymentsBlade-1024x256.png)

Szablon oczywiście możemy skopiować, zmodyfikować, wdrożyć ponownie &#8211; wszystko wewnątrz portalu. Wrażenia zbliżone są do Visual Studio i okienka **JSON Outline**, które zwykle jest używane przy tworzeniu szablonów. Kacper przytoczył BotService jak jedna z tych usług, która jest w Preview bez dobrych przykładów szablonów. Z historii uzyskałem taki szablon:
{: style="text-align: justify;"}

```json
{
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
```

## Jak dobrać się do historii bez użycia Portalu?

Skoro historia leży w Azure, to na logikę powinna być ona dostępna z poziomu innych narzędzi.
{: style="text-align: justify;"}

Z PowerShella wydobywamy ją w następujący sposób:
{: style="text-align: justify;"}

```powershell
#Deployments
Get-AzureRmResourceGroupDeployment -ResourceGroupName <RGName>

#Failed deployments
Get-AzureRmResourceGroupDeployment -ResourceGroupName <RGName>| Where-Object ProvisioningState -eq Failed

#Deployment details
Get-AzureRmResourceGroupDeploymentOperation -ResourceGroupName <RGName> -DeploymentName <DeplymentName>

#Download template
Save-AzureRmResourceGroupDeploymentTemplate -ResourceGroupName <RGName> -DeploymentName <DeplymentName>
```

Z Azure CLI:

```shell
#Deployments
az group deployment list --resource-group <RGName>

#Deployment details
az group deployment show --resource-group <RGName> --name <DeploymentName> --json

#Export template
az group deployment export --resource-group <RGName> --name <DeploymentName>
```

Podobne dane uzyskamy poprzez REST API i z resources.azure.com. Po prostu wybierz metodę, z którą czujesz się najbardziej komfortowo, zawartość będzie identyczna.
{: style="text-align: justify;"}

Daj znać jak Ty tworzysz szablony ARM. Może znasz jeszcze inny sposób jak je generować?
{: style="text-align: justify;"}