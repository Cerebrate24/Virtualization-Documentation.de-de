---
title: Verwenden von Containern mit dem Windows-Insider-Programm
description: Erfahren Sie, wie Sie mit dem Windows-Insider-Programm Windows-Container verwenden können.
keywords: Docker, Container, Insider, Windows
author: cwilhit
ms.topic: how-to
ms.openlocfilehash: fe657bb0c9456bda018c60cc8be9032b18d42989
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192157"
---
# <a name="use-containers-with-the-windows-insider-program"></a>Verwenden von Containern mit dem Windows-Insider-Programm

Diese Übung führt Sie durch die Bereitstellung und Verwendung der Windows-Containerfunktionen der neuesten Insider-Build von Windows Server aus dem Windows Insider Preview-Programm. In dieser Übung installieren Sie die Containerrolle und stellen eine Preview-Edition der Basisbetriebssystemimage bereit. Wenn Sie sich mit Containern vertraut machen möchten, finden Sie unter [Windows-Container](../about/index.md) entsprechende Informationen.

## <a name="join-the-windows-insider-program"></a>Nehmen Sie am Windows-Insider-Programm teil

Damit die Insider-Version von Windows-Containern ausgeführt werden kann, benötigen Sie einen Host, auf dem der neueste Build von Windows Server aus dem Windows-Insider-Programm und/oder der neueste Build von Windows 10 aus dem Windows-Insider-Programm ausgeführt wird. Nehmen Sie am [Windows-Insider-Programm](https://insider.windows.com/GettingStarted) teil, und überprüfen Sie die Nutzungsbedingungen.

> [!IMPORTANT]
> Sie müssen einen Build von Windows Server aus dem Windows Server Insider Preview-Programm oder einen Build von Windows 10 aus dem Windows Insider Preview-Programm verwenden, um das unten beschriebene Basisimage verwenden zu können. Wenn Sie keine dieser Builds verwenden, kann der Container nicht mit diesen Basisimages gestartet werden.

## <a name="install-docker"></a>Installieren von Docker

Wenn Sie Docker noch nicht installiert haben, folgen Sie dem Leitfaden [Erste Schritte](../quick-start/set-up-environment.md), um Docker zu installieren.

## <a name="pull-an-insider-container-image"></a>Aufrufen von Insider-Containerimages

Wenn Sie am Windows-Insider-Programms teilnehmen, können Sie unsere neuesten Builds für die Basisimages verwenden.

Zum Aufrufen des Basisimages für Nano Server Insider führen Sie folgenden Befehl aus:

```console
docker pull mcr.microsoft.com/nanoserver/insider
```

Zum Aufrufen des Basisimages für Windows Server Core Insider führen Sie folgenden Befehl aus:

```console
docker pull mcr.microsoft.com/windows/servercore/insider
```

Die Basisimages „Windows“ und „IoTCore“ besitzen auch eine Insider-Version, die zum Abruf zur Verfügung steht. Sie können mehr über die verfügbaren Insider-Basisimages im Dokument zu [Containerbasisimages](../manage-containers/container-base-images.md) nachlesen.

> [!IMPORTANT]
> Lesen Sie dazu die [EULA](../images-eula.md ) für Betriebssystemimages von Windows-Containern und die [Nutzungsbedingungen](https://www.microsoft.com/software-download/windowsinsiderpreviewserver) für das Windows-Insider-Programm.
