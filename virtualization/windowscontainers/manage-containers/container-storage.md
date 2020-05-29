---
title: Übersicht über Containerspeicher
description: So können Windows Server-Container Host- und andere Speichertypen verwenden
keywords: Container, Volume, Speicher, Mount, Binden von Bereitstellungen
author: cwilhit
ms.openlocfilehash: f758877f1131813fe4637a01c03b49d7a18a83c4
ms.sourcegitcommit: db085db8a54664184a2f7cfa01d00598a1c66992
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 03/04/2020
ms.locfileid: "78288672"
---
# <a name="container-storage-overview"></a>Übersicht über Containerspeicher

<!-- Great diagram would be great! -->

Dieses Thema bietet eine Übersicht über die verschiedenen Arten, wie Container den Speicher unter Windows verwenden. Container verhalten sich bei der Speicherung anders als virtuelle Computer. Container werden von Natur aus so erstellt, dass eine App, die in ihnen ausgeführt wird, den Zustand nicht in das gesamte Dateisystem des Hosts schreiben kann. Container verwenden standardmäßig einen „sicheren“ Speicherplatz, aber Windows bietet auch eine Möglichkeit zur permanenten Speicherung.

## <a name="scratch-space"></a>Sicherer Speicherbereich

Windows-Container verwenden standardmäßig kurzlebigen Speicher. Alle E/A-Vorgänge des Containers erfolgen ein einem „sicheren Speicherbereich“ und jeder Container erhält einen eigenen „sicheren Speicher“. Die Dateierstellung und das Schreiben von Dateien werden im sicheren Speicherbereich erfasst und gelangen nicht zum Host. Wenn eine Containerinstanz beendet wird, werden alle Änderungen, die im sicheren Speicherbereich aufgetreten sind, verworfen. Wenn eine neue Containerinstanz gestartet wird, wird ein neuer sicherer Speicherbereich für die Instanz bereitgestellt.

## <a name="layer-storage"></a>Schichtspeicher

Wie in der [Containerübersicht](../about/index.md) beschrieben, sind Containerimages ein Bündel von Dateien, die als Folge von Schichten dargestellt werden. Beim Schichtspeicher handelt es sich um die Dateien, die im Container integriert sind. Bei jedem `docker pull` und `docker run` des Containers sind diese identisch.

### <a name="where-layers-are-stored-and-how-to-change-it"></a>Wo Schichten gespeichert werden und wie Sie diese ändern

Bei einer Standardinstallation werden die Schichten unter `C:\ProgramData\docker` gespeichert und auf die Verzeichnisse "Image" und "Windowsfilter" verteilt. Sie können den Speicherort der Schichten mithilfe der `docker-root`-Konfiguration ändern, wie in der Dokumentation [Docker-Modul unter Windows](../manage-docker/configure-docker-daemon.md) erläutert.

> [!NOTE]
> Für die Schichtspeicher wird nur NTFS unterstützt. ReFS wird nicht unterstützt.

Sie sollten keine Dateien der Schichtverzeichnisse ändern – diese werden sorgfältig verwaltet mithilfe von Befehlen wie:

- [docker images](https://docs.docker.com/engine/reference/commandline/images/)
- [docker rmi](https://docs.docker.com/engine/reference/commandline/rmi/)
- [Docker Abruf](https://docs.docker.com/engine/reference/commandline/pull/)
- [Docker laden](https://docs.docker.com/engine/reference/commandline/load/)
- [docker save](https://docs.docker.com/engine/reference/commandline/save/)

### <a name="supported-operations-in-layer-storage"></a>Unterstützte Vorgänge im Schichtspeicher

Beim Ausführen von Containern können die meisten NTFS-Vorgänge, mit Ausnahme der Transaktionen, verwendet werden. Dies beinhaltet das Festlegen von ACLs, wobei alle ACLs innerhalb des Containers geprüft werden. Wenn Prozesse mit mehreren Benutzern in einem Container ausgeführt werden sollen, können Sie Benutzer in Ihrer `Dockerfile`mit `RUN net user /create ...` erstellen, Dateizugriffssteuerungslisten festlegen und dann Vorgänge für diesen Benutzer mithilfe der [Dockerfile-USER-Direktive](https://docs.docker.com/engine/reference/builder/#user) konfigurieren.

## <a name="persistent-storage"></a>Permanenter Speicher

Windows-Container unterstützen Mechanismen zur Bereitstellung von permanentem Speicher über das Binden von Bereitstellungen und Volumes. Weitere Informationen finden Sie unter [Permanente Speicherung in Containern](./persistent-storage.md).

## <a name="storage-limits"></a>Speichergrenzwerte

Ein gängiges Muster für Windows-Anwendungen ist das Abfragen des Speicherplatzes vor der Installation oder vor dem Erstellen neuer Dateien oder als Auslöser für das Bereinigen temporärer Dateien.  Zur Maximierung der Anwendungskompatibilität stellt das Laufwerk „C:“ in einem Windows-Container eine virtuelle Größe von 20 GB bereit.

Einige Benutzer möchten diesen Standardwert möglicherweise außer Kraft setzen und für den freien Speicherplatz einen kleineren oder größeren Wert konfigurieren. Dies kann durch die „size“-Option innerhalb der „storage-opt“-Konfiguration erreicht werden.

### <a name="examples"></a>Beispiele

Befehlszeile: `docker run --storage-opt "size=50GB" mcr.microsoft.com/windows/servercore:ltsc2019 cmd`

Oder Sie können die Docker-Konfigurationsdatei direkt ändern:

```Docker Configuration File
"storage-opt": [
    "size=50GB"
  ]
```

> [!TIP]
> Diese Methode funktioniert auch für Docker Build. Weitere Informationen zum Ändern der Docker-Konfigurationsdatei finden Sie im Dokument [Konfigurieren von Docker](https://docs.microsoft.com/virtualization/windowscontainers/manage-docker/configure-docker-daemon#configure-docker-with-configuration-file).
