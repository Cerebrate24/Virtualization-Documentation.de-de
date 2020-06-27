---
title: Wartungslebenszyklen von Basisimages
description: Erhalten Sie Informationen zum Lebenszyklus des Windows-Containerbasisimages.
keywords: Windows-Container, Container, Lebenszyklus, Versionsinformationen, Basisimage, Containerontainerbasisimage
author: Heidilohr
ms.author: helohr
ms.date: 05/12/2020
ms.topic: reference
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: b3c519ef3ed93a0c8e20f5b927c34f70cd1677f8
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192077"
---
# <a name="base-image-servicing-lifecycles"></a>Wartungslebenszyklen von Basisimages

> [!Note]
> Microsoft hat das geplante Ende der Support- und Wartungstermine für eine Reihe von Produkten verschoben, um Personen und Organisationen dabei zu unterstützen, ihre Aufmerksamkeit auf die Aufrechterhaltung der Geschäftskontinuität zu richten. Weitere Informationen finden Sie unter dem Eintrag [Lebenszyklusänderungen für das Ende der Support- und Wartungstermine](https://support.microsoft.com/help/4557164/lifecycle-changes-to-end-of-support-and-servicing-dates) vom 14. April 2020.

Windows-Containerbasisimages basieren entweder auf Releases im halbjährlichen Kanal (Semi-Annual Channel) oder des „Long-Term Servicing Channel“ von Windows Server. In diesem Artikel erfahren Sie, wie lange der Support für verschiedene Versionen von Basisimages aus beiden Kanälen andauern wird.

Beim halbjährigen Kanal wird zweimal im Jahr ein Funktionsupdate veröffentlicht, und für jede Veröffentlichung gilt eine 18-monatige Servicefrist. Dadurch können Kunden die Vorteile neuer Betriebssystemfunktionen schneller nutzen, sowohl bei Anwendungen (insbesondere bei Anwendungen, die auf Containern und Microservices aufbauen) als auch im hybriden, softwaredefinierten Rechenzentrum. Weitere Informationen finden Sie unter [Übersicht: Windows Server, Semi-Annual Channel](https://docs.microsoft.com/windows-server/get-started/semi-annual-channel-overview).

Für Server Core-Images können Kunden auch den Long-Term Servicing Channel nutzen, der alle zwei bis drei Jahre eine neue Hauptversion von Windows Server veröffentlicht. Releases des Long-Term Servicing Channel erhalten fünf Jahre grundlegenden Support und fünf Jahre erweiterten Support. Dieser Kanal eignet sich für Systeme, die eine längere Wartungsoption und funktionale Stabilität erfordern.

In der folgenden Tabelle sind die einzelnen Typen von Basisimages, ihr Wartungskanal und die Supportdauer aufgeführt.

|Base image                       |Wartungskanal|Version|Betriebssystembuild|Verfügbarkeit|Enddatum für grundlegenden Support|Erweiterter Support: Datum|
|---------------------------------|-----------------|-------|--------|------------|---------------------------|---------------------|
|Server Core, Nano Server, Windows|Halbjährlich      |2004   |19041   |27.05.2020  |14.12.2021                 |NICHT ZUTREFFEND                  |
|Server Core, Nano Server, Windows|Halbjährlich      |1909   |18363   |12.11.2019  |11.05.2021                 |NICHT ZUTREFFEND                  |
|Server Core, Nano Server, Windows|Halbjährlich      |1903   |18362   |21.05.2019  |08.12.2020                 |NICHT ZUTREFFEND                  |
|Server Core                      |Langfristig        |2019   |17763   |13.11.2018  |09.01.2024                 |09.01.2029           |
|Server Core, Nano Server, Windows|Halbjährlich      |1809   |17763   |13.11.2018  |10.11.2020                 |NICHT ZUTREFFEND                  |
|Server Core, Nano Server         |Halbjährlich      |1803   |17134   |30.04.2018  |12.11.2019                 |NICHT ZUTREFFEND                  |
|Server Core, Nano Server         |Halbjährlich      |1709   |16299   |17.10.2017  |09.04.2019                 |NICHT ZUTREFFEND                  |
|Server Core                      |Langfristig        |2016   |14393   |15.10.2016  |11.01.2022                 |11.01.2027           |
|Nano Server                      |Halbjährlich      |1607   |14393   |15.10.2016  |09.10.2018                 |NICHT ZUTREFFEND                  |

Informationen zu Wartungsanforderungen und weitere zusätzliche Informationen finden Sie in [Windows-Lebenszyklus: Häufig gestellte Fragen (FAQ)](https://support.microsoft.com/help/18581/lifecycle-faq-windows-products), [Windows Server-Versionsinformationen](https://docs.microsoft.com/windows-server/get-started/windows-server-release-info) und in [Windows-Basisbetriebssystemimages: Docker Hub-Repository](https://hub.docker.com/_/microsoft-windows-base-os-images).
