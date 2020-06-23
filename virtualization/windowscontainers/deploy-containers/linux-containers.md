---
title: Linux-Container unter Windows 10
description: Erfahren Sie mehr über verschiedene Möglichkeiten, wie Sie Hyper-V verwenden können, um Linux-Container unter Windows 10 so auszuführen, als wären sie nativ.
keywords: Linux-Container, Docker, Container, Windows 10
author: scooley
ms.date: 09/17/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: edfd11c8-ee99-42d8-9878-efc126fe1826
ms.openlocfilehash: 6b737129692ec8e56bebf290ad8064f010f78ea3
ms.sourcegitcommit: 6a5c237bff2c953fec2ce1e09424375a7c615010
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 06/09/2020
ms.locfileid: "84632970"
---
# <a name="linux-containers-on-windows-10"></a>Linux-Container unter Windows 10

Linux-Container machen einen riesigen Prozentsatz des gesamten Containerökosystems aus und sind sowohl für die Erfahrungen von Entwicklern als auch für Produktionsumgebungen von grundlegender Bedeutung.  Da sich die Container jedoch einen Kernel mit dem Containerhost teilen, ist das direkte Ausführen von Linux-Containern unter Windows keine Option. Hier kommt die Virtualisierung ins Spiel.

## <a name="linux-containers-in-a-moby-vm"></a>Linux-Container in einer Moby VM

Zum Ausführen von Linux-Containern auf einem virtuellen Linux-Computer befolgen Sie die Anweisungen im [Leitfaden zu den ersten Schritten mit Docker](https://docs.docker.com/docker-for-windows/).

Docker kann Linux-Container auf dem Windows-Desktop ausführen, seit es 2016 veröffentlicht wurde (bevor Hyper-V-Isolation oder Linux-Container unter Windows verfügbar waren), wobei ein [LinuxKit](https://github.com/linuxkit/linuxkit)-basierter virtueller Computer unter Hyper-V verwendet wird.

Bei diesem Modell wird der Docker-Client auf dem Windows-Desktop ausgeführt, ruft aber den Docker-Daemon auf dem virtuellen Linux-Computer auf.

![Moby VM als Containerhost](media/MobyVM.png)

Bei diesem Modell teilen sich alle Linux-Container einen einzelnen Linux-basierten Containerhost und für alle Linux-Container gilt Folgendes:

* Sie Teilen sich einen Kernel untereinander und mit der Moby VM, aber nicht mit dem Windows-Host.
* Sie verfügen über konsistente Speicher- und Netzwerkeigenschaften für Linux-Container, die unter Linux ausgeführt werden (da sie auf einer Linux-VM ausgeführt werden).

Das bedeutet auch, dass auf dem Linux-Containerhost (Moby VM) Docker-Daemon und alle Abhängigkeiten von Docker-Daemon ausgeführt werden müssen.

Um festzustellen, ob Sie mit Moby VM arbeiten, aktivieren Sie Hyper-V-Manager für Moby VM entweder über die Hyper-V-Manager-Benutzeroberfläche oder durch Ausführen von `Get-VM` in einem PowerShell-Fenster mit erhöhten Rechten.
