---
title: Druckspooler in Windows-Containern
description: Erläutert das aktuelle Arbeitsverhalten für den Druckspoolerdienst in Windows-Containern
keywords: Docker, Container, Drucker, Spooler
author: cwilhit
ms.openlocfilehash: e104a87046545b90d244783aafb62ad9d151e14b
ms.sourcegitcommit: 16744984ede5ec94cd265b6bff20aee2f782ca88
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 02/18/2020
ms.locfileid: "77439537"
---
# <a name="print-spooler-in-windows-containers"></a>Druckspooler in Windows-Containern

Anwendungen mit einer Abhängigkeit von Druckdiensten können erfolgreich mit Windows-Containern containerisiert werden. Es gibt spezielle Anforderungen, die erfüllt werden müssen, um die Funktionalität des Druckerdienstes erfolgreich zu aktivieren. Dieser Leitfaden erklärt, wie Sie Ihre Bereitstellung ordnungsgemäß konfigurieren.

> [!IMPORTANT]
> Der Zugriff auf Druckdienste in Containern funktioniert zwar erfolgreich, die Funktionalität ist jedoch eingeschränkt. Einige druckbezogene Aktionen funktionieren möglicherweise nicht. Beispielsweise können Anwendungen, die von der Installation von Druckertreibern auf dem Host abhängig sind, nicht containerisiert werden, da die **Installation von Treibern innerhalb eines Containers nicht unterstützt wird**. Verfassen Sie unten ein Feedback, wenn Sie ein nicht unterstütztes Druckfeature finden, das in Containern unterstützt werden soll.

## <a name="setup"></a>Setup

* Der Host sollte Windows Server 2019 oder Windows 10 Pro/Enterprise October 2018 Update oder höher sein.
* Das [mcr.microsoft.com/windows](https://hub.docker.com/_/microsoft-windowsfamily-windows)-Image sollte das angestrebte Basisimage sein. Andere Windows-Containerbasisimages (wie Nano Server und Windows Server Core) verfügen nicht über die Druckserverrolle.

### <a name="hyper-v-isolation"></a>Hyper-V-Isolierung

Es wird empfohlen, Ihren Container mit Hyper-V-Isolation auszuführen. Wenn Sie in diesem Modus arbeiten, können Sie beliebig viele Container mit Zugriff auf die Druckdienste ausführen. Sie müssen den Spoolerdienst auf dem Host nicht ändern.

Sie können die Funktionalität mit der folgenden PowerShell-Abfrage überprüfen:

```PowerShell
PS C:\Users\Administrator> docker run -it --isolation hyperv mcr.microsoft.com/windows:1809 powershell.exe
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

PS C:\> Get-Service spooler

Status   Name               DisplayName
------   ----               -----------
Running  spooler            Print Spooler


PS C:\> Get-Printer

Name                           ComputerName    Type         DriverName                PortName        Shared   Published
----                           ------------    ----         ----------                --------        ------   --------
Microsoft XPS Document Writer                  Local        Microsoft XPS Document... PORTPROMPT:     False    False
Microsoft Print to PDF                         Local        Microsoft Print To PDF    PORTPROMPT:     False    False
Fax                                            Local        Microsoft Shared Fax D... SHRFAX:         False    False


PS C:\>
```

### <a name="process-isolation"></a>Prozessisolation

Aufgrund der Tatsache, dass prozessisolierte Container einen gemeinsamen Kernel verwenden, beschränkt das aktuelle Verhalten den Benutzer darauf, nur **eine Instanz** des Druckerspoolerdienstes auf dem Host und allen seinen untergeordneten Containern auszuführen. Wenn der Druckerspooler auf dem Host ausgeführt wird, müssen Sie den Dienst auf dem Host beenden, bevor Sie versuchen, den Druckerdienst im Gast zu starten.

> [!TIP]
> Wenn Sie einen Container und eine Abfrage für den Spoolerdienst gleichzeitig sowohl im Container als auch im Host starten, melden beide ihren Status als „Wird ausgeführt“. Aber lassen Sie sich nicht täuschen – der Container wird nicht in der Lage sein, eine Liste der verfügbaren Drucker abzufragen. Der Spoolerdienst des Hosts darf nicht ausgeführt werden. 

Verwenden Sie die unten stehende Abfrage in PowerShell, um zu prüfen, ob der Host den Druckerdienst ausführt:

```PowerShell
PS C:\Users\Administrator> Get-Service spooler

Status   Name               DisplayName
------   ----               -----------
Running  spooler            Print Spooler

PS C:\Users\Administrator>
```

Verwenden Sie die folgenden Befehle in PowerShell, um den Spoolerdienst auf dem Host zu beenden:

```PowerShell
Stop-Service spooler
Set-Service spooler -StartupType Disabled
```

Starten Sie den Container, und überprüfen Sie den Zugriff auf die Drucker.

```PowerShell
PS C:\Users\Administrator> docker run -it --isolation process mcr.microsoft.com/windows:1809 powershell.exe
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.


PS C:\> Get-Service spooler

Status   Name               DisplayName
------   ----               -----------
Running  spooler            Print Spooler


PS C:\> Get-Printer

Name                           ComputerName    Type         DriverName                PortName        Shared   Published
----                           ------------    ----         ----------                --------        ------   --------
Microsoft XPS Document Writer                  Local        Microsoft XPS Document... PORTPROMPT:     False    False
Microsoft Print to PDF                         Local        Microsoft Print To PDF    PORTPROMPT:     False    False
Fax                                            Local        Microsoft Shared Fax D... SHRFAX:         False    False


PS C:\>
```