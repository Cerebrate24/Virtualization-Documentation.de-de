---
title: Windows-Container – häufig gestellte Fragen
description: FAQ zu Windows Server-Containern
keywords: Docker, Container
author: PatrickLang
ms.date: 10/25/2019
ms.topic: overview
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 25de368c-5a10-40a4-b4aa-ac8c9a9ca022
ms.openlocfilehash: cd0b4e24cddb434d0a4051888afa3ae3a4a8e096
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192867"
---
# <a name="frequently-asked-questions-about-containers"></a>Häufig gestellte Fragen zu Containern

## <a name="whats-the-difference-between-linux-and-windows-server-containers"></a>Worin besteht der Unterschied zwischen Linux- und Windows Server-Containern?

Linux- und Windows Server-Container implementieren beide ähnliche Technologien in ihrem Kernel und Kernbetriebssystemen. Der Unterschied ergibt sich aus der Plattform und Arbeitslasten, die innerhalb des Containers ausgeführt.

Wenn ein Kunde Windows Server-Container verwendet, ist eine Integration in vorhandene Windows-Technologien wie u. a. .NET, ASP.NET und PowerShell möglich.

## <a name="what-are-the-prerequisites-for-running-containers-on-windows"></a>Was sind die Voraussetzungen für die Ausführung von Containern unter Windows?

Container wurden mit Windows Server 2016 auf der Plattform eingeführt. Zum Verwenden von Containern benötigen Sie entweder Windows Server 2016 oder das Windows 10 Anniversary Update (Version 1607) oder höher. Weitere Informationen finden Sie in den [Systemanforderungen](../deploy-containers/system-requirements.md).

## <a name="what-are-wcow-and-lcow"></a>Was sind WCOW und LCOW?

WCOW ist die Kurzform von „Windows-Container unter Windows“ (Windows Containers On Windows). LCOW ist die Kurzform von „Linux-Container unter Windows“ (Linux Containers On Windows).

## <a name="how-are-containers-licensed-is-there-a-limit-to-the-number-of-containers-i-can-run"></a>Wie werden Container lizenziert? Gibt es eine Einschränkung bezüglich der Anzahl von Containern, die ich ausführen kann?

Die [EULA](../images-eula.md) für Windows Containerimages beschreibt ein Nutzung, die verlangt, dass der Benutzer über ein Hostbetriebssystem mit einer gültigen Lizenz verfügt. Die Anzahl der Container, die ein Benutzer ausführen darf, hängt von der Edition des Hostbetriebssystems und dem [Isolationsmodus](../manage-containers/hyperv-container.md) ab, mit dem ein Container ausgeführt wird, sowie davon, ob diese Container zu Entwicklungs-/Testzwecken oder in der Produktion ausgeführt werden.

|Hostbetriebssystem                                                         |Grenzwert für prozessisolierte Container                   |Grenzwert für Hyper-V-isolierte Container                   |
|----------------------------------------------------------------|---------------------------------------------------|---------------------------------------------------|
|Windows Server Standard                                         |Unbegrenzt                                          |2                                                  |
|Windows Server Datacenter                                       |Unbegrenzt                                          |Unbegrenzt                                          |
|Windows 10 Pro und Enterprise                                   |Unbegrenzt *(nur für Test- oder Entwicklungszwecke)*|Unbegrenzt *(nur für Test- oder Entwicklungszwecke)*|
|Windows 10 IoT Core und Enterprise                             |Unbegrenzt*                                         |Unbegrenzt*                                          |

Die Nutzung von Windows Server-Containerimages wird durch Lesen der Anzahl der Virtualisierungsgäste bestimmt, die für die betreffende [Edition](/windows-server/get-started-19/editions-comparison-19.md) unterstützt werden. <br/>

>[!NOTE]
>\*Die Verwendung von Containern in einer IoT-Edition von Windows ist abhängig davon, ob Sie die kommerziellen Nutzungsbedingungen von Microsoft für Windows 10 Core-Laufzeitimages oder der Windows 10 IoT Enterprise-Gerätelizenz („Kommerzielle Windows IoT-Vereinbarung“) zugestimmt haben. Zusätzliche Bestimmungen und Einschränkungen in den kommerziellen Windows-Vereinbarungen gelten für die Verwendung von Containerimages in einer Produktionsumgebung. Lesen Sie die [Containerimage-EULA](../images-eula.md), um genau zu verstehen, was zulässig und was unzulässig ist.

## <a name="as-a-developer-do-i-have-to-rewrite-my-app-for-each-type-of-container"></a>Muss ich als Entwickler meine App für jeden Containertyp neu schreiben?

Nein Windows-Containerimages sind für Windows Server-Container und Hyper-V-Isolierung gleich. Die Auswahl des Containertyps erfolgt beim Starten des Containers. Aus Sicht des Entwicklers sind Windows Server-Container und Hyper-V-Isolierung zwei Varianten der gleichen Sache. Sie bieten die gleiche Entwicklungs-, Programmier- und Verwaltungserfahrung, sind offen und erweiterbar und beinhalten das gleiche Maß an Integration und Unterstützung wie Docker.

Ein Entwickler kann ein Containerimage unter Verwendung eines Windows Server-Containers erstellen und es in Hyper-V-Isolierung oder umgekehrt bereitstellen, ohne dass irgendwelche Änderungen vorgenommen werden müssen, außer der Angabe des entsprechenden Laufzeitflags.

Windows Server-Container bieten eine größere Dichte und Leistung, wenn es auf Geschwindigkeit ankommt, z. B. eine geringere Hochfahrzeit und eine schnellere Laufzeitleistung im Vergleich zu geschachtelten Konfigurationen. Die Hyper-V-Isolierung bietet, wie der Name schon sagt, eine stärkere Isolierung, die sicherstellt, dass Code, der in einem Container ausgeführt wird, das Hostbetriebssystem oder andere Container, die auf demselben Host ausgeführt werden, nicht kompromittieren oder beeinträchtigen kann. Dies ist hilfreich für mehrinstanzenfähige Szenarien (mit Anforderungen für das Hosten von nicht vertrauenswürdigem Code), einschließlich SaaS-Anwendungen und Computehosting.

## <a name="can-i-run-windows-containers-in-process-isolated-mode-on-windows-10"></a>Kann ich Windows-Container unter Windows 10 im prozessisolierten Modus ausführen?

Ab dem Windows 10-Update vom Oktober 2018 können Sie einen Windows-Container mit Prozessisolierung ausführen. Sie müssen jedoch zunächst Prozessisolierung direkt anfordern, indem Sie das Flag `--isolation=process` verwenden, wenn Sie Ihre Container mit `docker run` ausführen. Prozessisolierung ist mit Windows 10 Pro, Windows 10 Enterprise, Windows 10 IoT Core und Windows 10 IoT Enterprise kompatibel.

Wenn Sie Ihre Windows-Container auf diese Weise ausführen möchten, müssen Sie sicherstellen, dass auf dem Host Windows 10 Build 17763 und höher ausgeführt wird und Sie über eine Docker-Version mit der Engine 18.09 oder höher verfügen.

> [!WARNING]
> Abgesehen von IoT Core- und IoT Enterprise-Hosts (nachdem Sie weitere Bestimmungen und Einschränkungen akzeptiert haben) ist dieses Feature nur für Entwicklung und Tests gedacht. Sie sollten weiterhin Windows Server als Host für Produktionsbereitstellungen verwenden. Wenn Sie diese Funktion verwenden, müssen Sie auch sicherstellen, dass die Host- und Containerversionstags übereinstimmen, da andernfalls der Container ggf. nicht gestartet wird oder nicht definiertes Verhalten aufweist.

## <a name="how-do-i-make-my-container-images-available-on-air-gapped-machines"></a>Wie stelle ich meine Containerimages auf Air-Gap-Computern zur Verfügung?

Windows-Containerbasisimages enthalten Artefakte, deren Weitergabe durch eine Lizenz eingeschränkt ist. Wenn Sie mit diesen Images Builds erstellen und sie in eine private oder öffentliche Registrierung pushen, werden Sie feststellen, dass die Basisebene niemals gepusht wird Stattdessen wird das Konzept einer Fremdebene verwendet, die auf die tatsächliche Basisebene im Azure-Cloudspeicher verweist.

Dies kann die Dinge erschweren, wenn Sie einen Air-Gap-Computer verwenden, der nur Images von der Adresse Ihrer privaten Containerregistrierung abrufen kann. In diesem Fall funktioniert es nicht, der Fremdebene zu folgen, um das Basisimage abzurufen. Um das Verhalten der Fremdebene außer Kraft zu setzen, können Sie das `--allow-nondistributable-artifacts`-Flag im Docker-Daemon verwenden.

> [!IMPORTANT]
> Die Verwendung dieses Flags schließt Ihre Verpflichtung nicht aus, die Bedingungen der Lizenz für Windows-Containerbasisimages einzuhalten. Sie dürfen keinen Windows-Inhalt für die öffentliche oder Drittanbieterverteilung bereitstellen. Die Verwendung in ihrer eigenen Umgebung ist zulässig.

## <a name="additional-feedback"></a>Zusätzliches Feedback

Möchten Sie den FAQ etwas hinzufügen? Öffnen Sie im Abschnitt „Kommentare“ ein neues Issue für Feedback, oder richten Sie einen Pull Request für diese Seite mit GitHub ein.
