---
title: GPU-Beschleunigung in Windows-Containern
description: Welche Stufe der GPU-Beschleunigung gibt es in Windows-Containern?
keywords: Docker, Container, Geräte, Hardware
author: cwilhit
ms.openlocfilehash: 8f63c74d7839385e21206188263b9e5d08e7eb60
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909910"
---
# <a name="gpu-acceleration-in-windows-containers"></a>GPU-Beschleunigung in Windows-Containern

Für viele containerisierte Workloads bieten die CPU-Computeressourcen eine ausreichende Leistung. Bei einer bestimmten Klasse von Workloads kann jedoch die enorme parallele Computeleistung von GPUs (Graphics Processing Units) Operationen um ein Vielfaches beschleunigen, die Kosten senken und den Durchsatz immens steigern.

GPUs sind bereits ein gängiges Tool für viele beliebte Workloads, vom herkömmlichen Rendern und Simulieren bis hin zum Trainieren des maschinellen Lernens. Windows-Container unterstützen die GPU-Beschleunigung für DirectX und alle darauf aufsetzenden Frameworks.

> [!NOTE]
> Dieses Feature steht in Docker Desktop, Version 2.1 und dem Docker-Modul – Enterprise, Version 19.03 oder höher, zur Verfügung.

## <a name="requirements"></a>Anforderungen

Damit dieses Feature funktioniert, muss Ihre Umgebung die folgenden Anforderungen erfüllen:

- Auf dem Containerhost muss Windows Server 2019 oder Windows 10, Version 1809 oder höher, ausgeführt werden.
- Das Containerbasisimage muss [mcr.microsoft.com/windows:1809](https://hub.docker.com/_/microsoft-windows) oder höher sein. Windows Server Core- und Nano Server-Containerimages werden derzeit nicht unterstützt.
- Auf dem Containerhost muss das Docker-Modul 19.03 oder höher ausgeführt werden.
- Der Containerhost muss über eine GPU mit Anzeigetreibern der Version WDDM 2.5 oder höher verfügen.

Führen Sie das DirectX-Diagnosetool (dxdiag.exe) auf Ihrem Containerhost aus, um die WDDM-Version Ihrer Anzeigetreiber zu überprüfen. Schauen Sie auf der Registerkarte „Anzeige“ des Tools im Abschnitt „Treiber“ nach, wie unten angegeben.

![Dxdiag](media/dxdiag.png)

## <a name="run-a-container-with-gpu-acceleration"></a>Ausführen eines Containers mit GPU-Beschleunigung

Führen Sie den folgenden Befehl aus, um einen Container mit GPU-Beschleunigung zu starten:

```shell
docker run --isolation process --device class/5B45201D-F2F2-4F3B-85BB-30FF1F953599 mcr.microsoft.com/windows:1809
```

> [!IMPORTANT]
> DirectX (und alle darauf aufsetzenden Frameworks) umfasst die einzigen APIs, die heute mit einer GPU beschleunigt werden können. Frameworks von Drittanbietern werden nicht unterstützt.

## <a name="hyper-v-isolated-windows-container-support"></a>Unterstützung für Windows-Container mit Hyper-V-Isolation

Die GPU-Beschleunigung für Workloads in Windows-Containern mit Hyper-V-Isolation wird derzeit nicht unterstützt.

## <a name="hyper-v-isolated-linux-container-support"></a>Unterstützung von Linux-Containern mit Hyper-V-Isolation

Die GPU-Beschleunigung für Workloads in Linux-Containern mit Hyper-V-Isolation wird derzeit nicht unterstützt.

## <a name="more-information"></a>Weitere Informationen

Ein vollständiges Beispiel für eine DirectX-Container-App, die die GPU-Beschleunigung nutzt, finden Sie unter [DirectX-Containerbeispiel](https://github.com/MicrosoftDocs/Virtualization-Documentation/tree/master/windows-container-samples/directx).
