---
title: Isolationsmodi
description: Hier wird erläutert, wie sich die Hyper-V-Isolation von prozessisolierten Containern unterscheidet.
keywords: Docker, Container
author: crwilhit
ms.date: 09/26/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 42154683-163b-47a1-add4-c7e7317f1c04
ms.openlocfilehash: 362805fa230f461414ccc53643644f6c1b3474a8
ms.sourcegitcommit: ac923217ee2f74f08df2b71c2a4c57b694f0d7c3
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 03/06/2020
ms.locfileid: "78853954"
---
# <a name="isolation-modes"></a>Isolationsmodi

Windows-Container bieten zwei verschiedene Modi der Runtimeisolation: `process`- und `Hyper-V`-Isolation. Container, die unter beiden Isolationsmodi ausgeführt werden, werden identisch erstellt, verwaltet und funktionieren auch identisch. Sie erzeugen und nutzen auch die gleichen Containerimages. Der Unterschied zwischen den Isolationsmodi besteht darin, welcher Grad an Isolation zwischen dem Container, dem Hostbetriebssystem und allen anderen Containern, die auf diesem Host ausgeführt werden, hergestellt wird.

## <a name="process-isolation"></a>Prozessisolation

Dies ist der „herkömmliche“ Isolationsmodus für Container und wird in der [Übersicht über Windows-Container](../about/index.md) beschrieben. Bei der Prozessisolation werden mehrere Containerinstanzen auf einem Host gleichzeitig isoliert ausgeführt, was mithilfe von Technologien zur Isolation von Namespaces, Ressourcensteuerung und Prozessen ermöglicht wird. Bei der Ausführung in diesem Modus teilen sich Container untereinander und mit dem Host denselben Kernel.  Dies entspricht in etwa der Funktionsweise von Linux-Containern.

![](media/container-arch-process.png)

## <a name="hyper-v-isolation"></a>Hyper-V-Isolierung
Dieser Isolationsmodus bietet erhöhte Sicherheit und umfassendere Kompatibilität zwischen Host- und Containerversionen. Bei der Hyper-V-Isolation werden mehrere Containerinstanzen gleichzeitig auf einem Host ausgeführt. Jeder Container wird jedoch innerhalb eines hochgradig optimierten virtuellen Computers ausgeführt und erhält effektiv seinen eigenen Kernel. Das Vorhandensein des virtuellen Computers ermöglicht eine Isolation auf Hardwareebene zwischen den einzelnen Containern und dem Containerhost.

![](media/container-arch-hyperv.png)

## <a name="isolation-examples"></a>Beispiele für Isolation

### <a name="create-container"></a>Erstellen eines Containers

Die Verwaltung von Containern mit Hyper-V-Isolation mit Docker ist nahezu identisch mit der Verwaltung von prozessisolierten Containern. Verwenden Sie zum Erstellen eines Containers mit Hyper-V-Isolation durch Docker den `--isolation`-Parameter, um `--isolation=hyperv` festzulegen.

```cmd
docker run -it --isolation=hyperv mcr.microsoft.com/windows/servercore:ltsc2019 cmd
```

Verwenden Sie zum Erstellen eines Containers mit Prozessisolation durch Docker den `--isolation`-Parameter, um `--isolation=process` festzulegen.

```cmd
docker run -it --isolation=process mcr.microsoft.com/windows/servercore:ltsc2019 cmd
```

Unter Windows Server ausgeführte Windows-Container werden standardmäßig mit Prozessisolation ausgeführt. Unter Windows 10 Pro und Enterprise ausgeführte Windows-Container werden standardmäßig mit Hyper-V-Isolation ausgeführt. Ab Windows 10 Oktober 2018-Update können Benutzer, die einen Windows 10 Pro- oder Enterprise-Host betreiben, einen Windows-Container mit Prozessisolation ausführen. Benutzer müssen die Prozessisolation direkt mithilfe des `--isolation=process`-Flags anfordern.

> [!WARNING]
> Die Ausführung mit Prozessisolation unter Windows 10 Pro und Enterprise ist für die Entwicklung bzw. für Tests gedacht. Auf dem Host muss Windows 10 Build 17763 oder höher ausgeführt werden, und Sie müssen über ein Docker-Modul mit der Version 18.09 oder höher verfügen.
> 
> Sie sollten weiterhin Windows Server als Host für Produktionsbereitstellungen verwenden. Wenn Sie dieses Feature unter Windows 10 Pro und Enterprise verwenden, müssen Sie auch sicherstellen, dass die Host- und Containerversionstags übereinstimmen, da andernfalls der Container ggf. nicht gestartet wird oder nicht definiertes Verhalten aufweist.

### <a name="isolation-explanation"></a>Erläuterung zur Isolation

In diesem Beispiel werden die Unterschiede bei den Isolationsfunktionen zwischen Prozess- und Hyper-V-Isolation veranschaulicht.

Hier wird ein Container mit Prozessisolation bereitgestellt, der einen Pingprozess mit langer Ausführungsdauer hosten wird.

``` cmd
docker run -d mcr.microsoft.com/windows/servercore:ltsc2019 ping localhost -t
```

Mithilfe des `docker top`-Befehls wird der Pingprozess zurückgegeben, wie im Container zu sehen. Der Prozess in diesem Beispiel weist die ID 3964 auf.

``` cmd
docker top 1f8bf89026c8f66921a55e773bac1c60174bb6bab52ef427c6c8dbc8698f9d7a

3964 ping
```

Auf dem Containerhost kann der `get-process`-Befehl verwendet werden, um einen beliebigen ausgeführten Pingprozess vom Host zurückzugeben. In diesem Beispiel ist ein solcher Prozess vorhanden, und die Prozess-ID entspricht der ID im Container. Es handelt sich um den gleichen Prozess, der sowohl im Container als auch auf dem Host sichtbar ist.

```
get-process -Name ping

Handles  NPM(K)    PM(K)      WS(K) VM(M)   CPU(s)     Id  SI ProcessName
-------  ------    -----      ----- -----   ------     --  -- -----------
     67       5      820       3836 ...71     0.03   3964   3 PING
```

Dieses Beispiel startet auch einen Container mit Hyper-V-Isolation mit einem Pingprozess.

```
docker run -d --isolation=hyperv mcr.microsoft.com/windows/servercore:ltsc2019 ping localhost -t
```

Ebenso kann `docker top` verwendet werden, um die ausgeführten Prozesse vom Container zurückzugeben.

```
docker top 5d5611e38b31a41879d37a94468a1e11dc1086dcd009e2640d36023aa1663e62

1732 ping
```

Beim Suchen nach dem Prozess auf dem Containerhost wird jedoch kein Pingprozess gefunden, und es wird ein Fehler ausgegeben.

```
get-process -Name ping

get-process : Cannot find a process with the name "ping". Verify the process name and call the cmdlet again.
At line:1 char:1
+ get-process -Name ping
+ ~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (ping:String) [Get-Process], ProcessCommandException
    + FullyQualifiedErrorId : NoProcessFoundForGivenName,Microsoft.PowerShell.Commands.GetProcessCommand
```

Auf dem Host ist der Prozess `vmwp` sichtbar, bei dem es sich um den ausgeführten virtuellen Computer handelt, der den ausgeführten Container kapselt und die ausgeführten Prozesse vor dem Hostbetriebssystem schützt.

```
get-process -Name vmwp

Handles  NPM(K)    PM(K)      WS(K) VM(M)   CPU(s)     Id  SI ProcessName
-------  ------    -----      ----- -----   ------     --  -- -----------
   1737      15    39452      19620 ...61     5.55   2376   0 vmwp
```
