---
title: Windows-Containerbasisimages
description: Eine Übersicht über die Windows-Containerbasisimages und wann sie zu verwenden sind.
keywords: Docker, Container, Hashes
author: patricklang
ms.date: 09/25/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 88e6e080-cf8f-41d8-a301-035959dc5ce0
ms.openlocfilehash: 9884cc0ae2d2f398d2dc2fb1997a70493a6de6c0
ms.sourcegitcommit: 16744984ede5ec94cd265b6bff20aee2f782ca88
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 01/28/2020
ms.locfileid: "76764177"
---
# <a name="container-base-images"></a>Containerbasisimages

Windows bietet vier Containerbasisimages, die Benutzer für die Erstellung verwenden können. Jedes Basisimage ist eine andere Variante des Windows-Betriebssystems, hat einen anderen Speicherbedarf und enthält eine andere Menge des Windows-API-Satzes.

<ul class="columns is-multiline has-margin-left-none has-margin-bottom-none has-padding-top-medium">
    <li class="column is-one-quarter has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="https://hub.docker.com/_/microsoft-windows-servercore" data-linktype="external">
            <article class="card has-outline-hover is-relative is-full-height has-padding-none">
                    <div class="cardImageOuter bgdAccent1 has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                        <div class="cardImage centered has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                            <img src="media/Microsoft_logo.svg" alt="" data-linktype="relative-path">
                        </div>
                    </div>
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Windows Server Core</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Unterstützt herkömmliche .NET Framework-Anwendungen.</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-quarter has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="https://hub.docker.com/_/microsoft-windows-nanoserver" data-linktype="external">
            <article class="card has-outline-hover is-relative is-full-height has-padding-none">
                    <div class="cardImageOuter bgdAccent1 has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                        <div class="cardImage centered has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                            <img src="media/Microsoft_logo.svg" alt="" data-linktype="relative-path">
                        </div>
                    </div>
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Nano Server</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Entwickelt für .NET Core-Anwendungen.</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-quarter has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="https://hub.docker.com/_/microsoft-windows" data-linktype="external">
            <article class="card has-outline-hover is-relative is-full-height has-padding-none">
                    <div class="cardImageOuter bgdAccent1 has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                        <div class="cardImage centered has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                            <img src="media/Microsoft_logo.svg" alt="" data-linktype="relative-path">
                        </div>
                    </div>
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Windows</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Stellt den vollständigen Windows-API-Satz bereit.</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-quarter has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="https://hub.docker.com/_/microsoft-windows-iotcore" data-linktype="external">
            <article class="card has-outline-hover is-relative is-full-height has-padding-none">
                    <div class="cardImageOuter bgdAccent1 has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                        <div class="cardImage centered has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                            <img src="media/Microsoft_logo.svg" alt="" data-linktype="relative-path">
                        </div>
                    </div>
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Windows IoT Core</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Speziell für IoT-Anwendungen entwickelt.</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

## <a name="image-discovery"></a>Imageermittlung

Alle Windows-Containerbasisimages können über [Docker Hub](https://hub.docker.com/_/microsoft-windows-base-os-images) ermittelt werden. Die Windows-Containerbasisimages selbst werden von [mcr.microsoft.com](https://azure.microsoft.com/en-us/services/container-registry/), der Microsoft Container Registry (MCR), bereitgestellt. Aus diesem Grund sehen die Pull-Befehle für die Windows-Containerbasisimages wie folgt aus:

```code
docker pull mcr.microsoft.com/windows/servercore:ltsc2019
```

Die MCR verfügt über keine eigene Katalogerfahrung und soll bestehende Kataloge wie Docker Hub unterstützen. Dank der globalen Präsenz von Azure und in Verbindung mit dem Azure CDN liefert die MCR ein konsistentes und schnelles Pull-Erlebnis für Images. Azure-Kunden, die ihre Workloads in Azure ausführen, profitieren von Leistungsverbesserungen im Netzwerk sowie von der engen Integration mit der MCR (der Quelle für Microsoft-Containerimages), dem Azure Marketplace und der wachsenden Zahl von Diensten in Azure, die Container als Bereitstellungspaketformat anbieten.

## <a name="choosing-a-base-image"></a>Auswählen eines Basisimages

Wie wählen Sie das richtige Basisimage als Grundlage für die Erstellung aus? Für die meisten Benutzer werden `Windows Server Core` und `Nanoserver` die am besten geeigneten Images sein.

### <a name="guidelines"></a>Richtlinien

 Obwohl es Ihnen freisteht, welches Image Sie bevorzugen, finden Sie hier einige Richtlinien, die Ihnen bei der Auswahl helfen sollen:

- **Benötigt Ihre Anwendung das vollständige .NET-Framework?** Wenn die Antwort auf diese Frage „Ja“ lautet, sollten Sie `Windows Server Core` als Ziel wählen.
- **Erstellen Sie eine Windows-App auf der Basis von .NET Core?** Wenn die Antwort auf diese Frage „Ja“ lautet, sollten Sie `Nanoserver` als Ziel wählen.
- **Erstellen Sie eine IoT-Anwendung?** Wenn die Antwort auf diese Frage „Ja“ lautet, sollten Sie `IoT Core` als Ziel wählen.
- **Fehlt dem Windows Server Core-Containerimage eine Abhängigkeit, die Ihre App benötigt?** Wenn die Antwort auf diese Frage „Ja“ lautet, sollten Sie versuchen, `Windows` als Ziel auszuwählen. Dieses Image ist viel größer als die anderen Basisimages, aber es enthält viele der Windows-Kernbibliotheken (z. B. die GDI-Bibliothek).
- **Sind Sie ein Windows-Insider?** In diesem Fall sollten Sie erwägen, die Insider-Version der Images zu verwenden. Weitere Informationen finden Sie unten unter „Basisimages für Windows-Insider“.

> [!TIP]
> Viele Windows-Benutzer möchten Anwendungen, die von .NET abhängig sind, containerisieren. Zusätzlich zu den vier hier beschriebenen Basisimages veröffentlicht Microsoft mehrere Windows-Containerimages, die mit gängigen Microsoft-Frameworks vorkonfiguriert sind, z. B. das [.NET-Framework](https://hub.docker.com/_/microsoft-dotnet-framework)-Image und das [ASP.NET](https://hub.docker.com/_/microsoft-dotnet-framework-aspnet/)-Image.

### <a name="base-images-for-windows-insiders"></a>Basisimages für Windows-Insider

Microsoft stellt „Insider“-Versionen von jedem Containerbasisimage zur Verfügung. Diese Insider-Containerimages enthalten die neueste und größte Featureentwicklung in unseren Containerimages. Wenn Sie einen Host ausführen, der eine Insider-Version von Windows ist (entweder Windows-Insider oder Windows Server-Insider), sollten Sie vorzugsweise diese Images verwenden. Die Insider-Images sind auf Docker Hub verfügbar:

- [mcr.microsoft.com/windows/servercore/insider](https://hub.docker.com/_/microsoft-windows-servercore-insider)
- [mcr.microsoft.com/windows/nanoserver/insider](https://hub.docker.com/_/microsoft-windows-nanoserver-insider)
- [mcr.microsoft.com/windows/iotcore/insider](https://hub.docker.com/_/microsoft-windows-iotcore-insider)
- [mcr.microsoft.com/windows/insider](https://hub.docker.com/_/microsoft-windows-insider)

Weitere Informationen finden Sie unter [Verwenden von Containern mit dem Windows-Insider-Programm](../deploy-containers/insider-overview.md).

### <a name="windows-server-core-vs-nanoserver"></a>Windows Server Core im Vergleich zu Nanoserver

`Windows Server Core` und `Nanoserver` sind die am häufigsten verwendeten Basisimages. Der Hauptunterschied zwischen diesen Images besteht darin, dass Nanoserver eine wesentlich kleinere API-Oberfläche hat. PowerShell, WMI und der Windows-Bereitstellungsstapel sind im Nanoserver-Image nicht vorhanden.

Nanoserver wurde entwickelt, um gerade genug API-Oberfläche für die Ausführung von Apps zu bieten, die von .NET Core oder anderen modernen Open-Source-Frameworks abhängig sind. Als Kompromiss zu der kleineren API-Oberfläche hat das Nanoserver-Image einen wesentlich kleineren Speicherbedarf als der Rest der Windows-Basisimages. Aufbauend auf Nano Server können Sie nach Bedarf jederzeit Ebenen hinzufügen. Ein Beispiel hierfür finden Sie unter [.NET Core Nano Server Dockerfile](https://github.com/dotnet/dotnet-docker/blob/master/2.1/sdk/nanoserver-1909/amd64/Dockerfile).
