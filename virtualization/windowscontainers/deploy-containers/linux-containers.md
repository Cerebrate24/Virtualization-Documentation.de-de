---
title: Linux-Container unter Windows 10
description: Erfahren Sie mehr über verschiedene Möglichkeiten, wie Sie Hyper-V verwenden können, um Linux-Container unter Windows 10 so auszuführen, als wären sie nativ.
keywords: LCOW, Linux-Container, Docker, Container, Windows 10
author: scooley
ms.date: 09/17/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: edfd11c8-ee99-42d8-9878-efc126fe1826
ms.openlocfilehash: 843bd0ab7ccf3a227482ba3a3d2677e36b395b29
ms.sourcegitcommit: ac923217ee2f74f08df2b71c2a4c57b694f0d7c3
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 03/06/2020
ms.locfileid: "78854014"
---
# <a name="linux-containers-on-windows-10"></a>Linux-Container unter Windows 10

Linux-Container machen einen riesigen Prozentsatz des gesamten Containerökosystems aus und sind sowohl für die Erfahrungen von Entwicklern als auch für Produktionsumgebungen von grundlegender Bedeutung.  Da sich die Container jedoch einen Kernel mit dem Containerhost teilen, ist das direkte Ausführen von Linux-Containern unter Windows keine Option[*](linux-containers.md#other-options-we-considered).  Hier kommt die Virtualisierung ins Spiel.

Im Moment gibt es zwei Möglichkeiten, Linux-Container mit Docker für Windows und Hyper-V auszuführen:

- Ausführen von Linux-Containern in einer vollständigen Linux-VM: Dies erfolgt heute in der Regel durch Docker.
- Ausführen von Linux-Containern mit [Hyper-V-Isolation](../manage-containers/hyperv-container.md) (LCOW): Dies ist eine neue Option in Docker für Windows.

> _Das Ausführen von Linux-Containern unter einem Windows Server-Betriebssystem befindet sich derzeit noch in einer experimentellen Phase. Es sind zusätzliche Lizenzen für das Docker EE-Programm erforderlich, um dies zu testen. **Der Rest dieses Artikels bezieht sich nur auf Windows 10**._

In diesem Artikel wird die Funktionsweise der einzelnen Ansätze beschrieben. Außerdem erhalten Sie Anleitungen dazu, wann welche Lösung ausgewählt wird sowie entsprechende Informationen, welche Aufgaben in Bearbeitung sind.

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

## <a name="linux-containers-with-hyper-v-isolation"></a>Linux-Container mit Hyper-V-Isolation

Um Linux-Container unter Windows 10 (LCOW10) zu testen, folgen Sie den Anweisungen für Linux-Container in [Linux-Container unter Windows 10](../quick-start/quick-start-windows-10-linux.md). 

Linux-Container mit Hyper-V-Isolation führen jeden Linux-Container in einer optimierten Linux-VM mit gerade ausreichenden Betriebssystemfunktionen aus, um Container auszuführen. Im Gegensatz zum Moby VM-Ansatz hat jeder Linux-Container einen eigenen Kernel und eine eigene VM-Sandbox. Sie werden auch direkt von Docker unter Windows verwaltet.

![Linux-Container mit Hyper-V-Isolation (LCOW)](media/lcow-approach.png)

Wenn Sie sich genauer anschauen, wie sich die Containerverwaltung zwischen dem Moby VM-Ansatz und LCOW unterscheidet, bleibt im LCOW-Modell die Containerverwaltung unter Windows und die gesamte LCOW-Verwaltung erfolgt über GRPC und containerd.  Das bedeutet, dass die Linux-Distributionscontainer, die für LCOW verwendet werden, einen viel kleineren Bestand aufweisen können.  Zurzeit verwenden wir LinuxKit für die Verwendung optimierter Distributionscontainer, aber andere Projekte wie Kata erstellen auch ähnliche hochgradig abgestimmte Linux-Distributionen (Clear Linux).

Im Folgenden finden Sie eine genauere LCOW-Betrachtung:

![LCOW-Architektur](media/lcow.png)

Navigieren Sie zu `C:\Program Files\Linux Containers`, um zu prüfen, ob Sie LCOW ausführen. Wenn Docker für die Verwendung von LCOW konfiguriert ist, gibt es hier ein paar Dateien, die die minimale LinuxKit-Distribution enthalten, die in jedem Container ausgeführt wird, der unter Hyper-V-Isolation ausgeführt wird.  Beachten Sie, dass die optimierten VM-Komponenten weniger als 100 MB groß sind, viel kleiner als das LinuxKit-Image in Moby VM.

### <a name="work-in-progress"></a>In Bearbeitung

LCOW befindet sich in der aktiven Entwicklung. Verfolgen der Fortschritte beim Moby-Projekt auf [GitHub](https://github.com/moby/moby/issues/33850)

#### <a name="bind-mounts"></a>Binden von Bereitstellungen

Beim Binden der Bereitstellung von Volumes mit `docker run -v ...` werden die Dateien im Windows NTFS-Dateisystem gespeichert, sodass für POSIX-Vorgänge einige Übersetzung erforderlich ist. Einige Dateisystemvorgänge werden derzeit teilweise oder gar nicht implementiert, was bei einigen Anwendungen zu Inkompatibilitäten führen kann.

Folgende Vorgänge funktionieren momentan nicht für gebunden bereitgestellte Volumes:

* MkNod
* XAttrWalk
* XAttrCreate
* Lock
* Getlock
* Auth
* Leerung
* INotify

Es gibt auch einige Vorgänge, die nicht vollständig implementiert werden:

* GetAttr – die Nlink-Zahl wird immer als 2 gemeldet
* Open – nur ReadWrite-, WriteOnly- und ReadOnly-Flags werden implementiert

Diese Anwendungen erfordern alle eine Volumezuordnung und werden nicht ordnungsgemäß gestartet oder ausgeführt.

* MySQL
* PostgreSQL
* WordPress
* Jenkins
* MariaDB
* RabbitMQ

### <a name="extra-information"></a>Zusätzliche Informationen

[Docker-Blog zur Beschreibung von LCOW](https://blog.docker.com/2017/11/docker-for-windows-17-11/)

[Linux-Containervideo](https://sec.ch9.ms/ch9/1e5a/08ff93f2-987e-4f8d-8036-2570dcac1e5a/LinuxContainer.mp4)

[LinuxKit LCOW-Kernel plus Erstellungsanweisungen](https://github.com/linuxkit/lcow)

## <a name="when-to-use-moby-vm-vs-lcow"></a>Verwenden von Moby VM oder LCOW

### <a name="when-to-use-moby-vm"></a>Verwenden von Moby VM

Zurzeit empfehlen wir die Moby VM-Methode zum Ausführen von Linux-Containern für Personen, für die Folgendes gilt:

- Sie wünschen eine stabile Containerumgebung.  Dies ist der Standard für Docker für Windows.
- Sie führen Windows- oder Linux-Container aus, aber selten beide gleichzeitig.
- Sie verfügen über komplizierte oder benutzerdefinierte Netzwerkanforderungen zwischen Linux-Containern.
- Sie benötigen keine Kernelisolation (Hyper-V-Isolation) zwischen Linux-Containern.

### <a name="when-to-use-lcow"></a>Verwenden von LCOW

Zurzeit wird LCOW für Personen empfohlen, für die Folgendes gilt:

- Sie möchten unsere neueste Technologie testen.
- Sie führen Windows- und Linux-Container gleichzeitig aus.
- Sie benötigen die Kernelisolation (Hyper-V-Isolation) zwischen Linux-Containern.

## <a name="other-options-we-considered"></a>Andere berücksichtigte Optionen

Als wir nach Möglichkeiten suchten, Linux-Container unter Windows auszuführen, haben wir WSL in Erwägung gezogen. Letztendlich haben wir uns für einen auf der Virtualisierung basierten Ansatz entschieden, sodass Linux-Container unter Windows mit Linux-Containern unter Linux konsistent sind. Durch die Verwendung von Hyper-V wird auch LCOW sicherer. Möglicherweise werden wir in Zukunft eine Neubewertung vornehmen, aber im Moment wird LCOW weiterhin Hyper-V verwenden.

Wenn Sie Ideen oder Meinungen hierzu haben, senden Sie Ihr Feedback über GitHub oder UserVoice.  Wir freuen uns besonders über Feedback zu den konkreten Erfahrungen, die Sie erwarten.
