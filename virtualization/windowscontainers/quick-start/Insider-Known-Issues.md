---
title: Bekannte Probleme der Insider-Builds
description: Bekannte Probleme der Insider-Builds
keywords: Docker, Container
ms.topic: quickstart
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 0a7b5a2c7f430babbc7a94f63b150f74b58c6843
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192757"
---
# <a name="known-issues-for-insider-builds"></a>Bekannte Probleme der Insider-Builds

## <a name="build-16237"></a>Build 16237

- Die Hyper-V-Isolation funktioniert nicht ordnungsgemäß. Diese Problemumgehung ist zur Verwendung von Hyper-V-Isolation in 16237 erforderlich. Führen Sie diese Befehle in PowerShell aus:

```PowerShell
Get-ComputeProcess | ? IsTemplate -eq $true | Stop-ComputeProcess -Force
Set-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers\' -Name TemplateVmCount -Type dword -Value 0 -Force
```

- Nano Server wird jetzt als Benutzer ausgeführt, damit Befehle, die Administratorrechte erfordern, fehlschlagen. Eine Befehlszeile wie beispielsweise „RUN setx /M PATH” verursachen einen Fehler beim Build. In diesem Szenario können Sie folgende Alternative verwenden:

```dockerfile
RUN setx PATH <path>
```
