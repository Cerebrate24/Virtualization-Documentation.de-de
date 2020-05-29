---
title: Windows-Containerplattform
description: Erfahren Sie mehr über neue Containerbausteine, die in Windows verfügbar sind.
keywords: LCOW, Linux-Container, Docker, Container, containerd, cri, runhcs, runc
author: scooley
ms.date: 11/19/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: a0e62b32-0c4c-4dd4-9956-8056e9abd9e5
ms.openlocfilehash: 3107eb48dc9c75224b0c9dd9b436af6f0f451871
ms.sourcegitcommit: 16744984ede5ec94cd265b6bff20aee2f782ca88
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 02/18/2020
ms.locfileid: "77439277"
---
# <a name="container-platform-tools-on-windows"></a>Containerplattformtools unter Windows

Die Windows-Containerplattform wird erweitert! Docker war der erste Teil der Containerjourney, jetzt erstellen wir weitere Containerplattformtools.

* [containerd/cri](https://github.com/containerd/cri): Neu in Windows Server 2019/Windows 10 1809.
* [runhcs](https://github.com/Microsoft/hcsshim/tree/master/cmd/runhcs): Eine Entsprechung für runc für Windows-Containerhosts.
* [hcs](https://docs.microsoft.com/virtualization/api/): Der Hostcomputedienst sowie praktische Shims, um die Verwendung einfacher zu gestalten.
  * [hcsshim](https://github.com/microsoft/hcsshim)
  * [dotnet-computevirtualization](https://github.com/microsoft/dotnet-computevirtualization)

In diesem Artikel werden die Windows- und Linux-Containerplattform sowie die einzelnen Containerplattformtools erläutert.

## <a name="windows-and-linux-container-platform"></a>Windows- und Linux-Containerplattform

In Linux-Umgebungen bauen Containerverwaltungstools wie Docker auf einem differenzierteren Satz von Containertools auf: [runc](https://github.com/opencontainers/runc) und [containerd](https://containerd.io/).

![Docker-Architektur unter Linux](media/docker-on-linux.png)

`runc` ist ein Linux-Befehlszeilentool zum Erstellen und Ausführen von Containern gemäß der [Spezifikation der OCI-Containerruntime](https://github.com/opencontainers/runtime-spec).

`containerd` ist ein Daemon, der den Containerlebenszyklus vom Herunterladen und Entpacken des Containerimages bis zur Containerausführung und -überwachung verwaltet.

Unter Windows haben wir einen anderen Ansatz gewählt.  Als wir begannen, mit Docker zur Unterstützung von Windows-Containern zu arbeiten, bauten wir direkt auf dem HCS (Hostcomputedienst) auf.  [Dieser Blogbeitrag](https://techcommunity.microsoft.com/t5/Containers/Introducing-the-Host-Compute-Service-HCS/ba-p/382332) enthält umfassende Informationen darüber, warum wir den HCS erstellt und diesen Ansatz anfänglich für Container gewählt haben.

![Anfängliche Docker-Modularchitektur unter Windows](media/hcs.png)

An diesem Punkt ruft Docker immer noch direkt den HCS auf. Künftig könnten jedoch Containerverwaltungstools, die auf Windows-Container und den Windows-Containerhost erweitert werden, containerd und runhcs so aufrufen, wie sie containerd und runc unter Linux aufrufen.

## <a name="runhcs"></a>runhcs

`runhcs` ist eine Verzweigung von `runc`.  Wie `runc` ist `runhcs` ein Befehlszeilenclient zum Ausführen von Anwendungen, die gemäß dem Format der Open Container Initiative (OCI) gepackt sind, und ist eine konforme Implementierung der Spezifikation der Open Container Initiative.

Zu den funktionalen Unterschieden zwischen runc und runhcs gehören:

* `runhcs` wird unter Windows ausgeführt.  Es kommuniziert mit dem [HCS](containerd.md#hcs), um Container zu erstellen und zu verwalten.
* `runhcs` kann eine Vielzahl verschiedener Containertypen ausführen.

  * [Hyper-V-Isolation](../manage-containers/hyperv-container.md) unter Windows und Linux
  * Windows-Prozesscontainer (Containerimage muss mit dem Containerhost übereinstimmen)

**Syntax:**

``` cmd
runhcs run [ -b bundle ] <container-id>
```

`<container-id>` ist Ihr Name für die Containerinstanz, die Sie starten. Der Name muss auf Ihrem Containerhost eindeutig sein.

Das Paketverzeichnis (mit `-b bundle`) ist optional.  
Wie bei runc werden Container über Bündel konfiguriert. Das Bündel eines Containers ist das Verzeichnis mit der OCI-Spezifikationsdatei des Containers, „config.json“.  Der Standardwert für das Bündel ist das aktuelle Verzeichnis.

Die OCI-Spezifikationsdatei „config.json“ muss zwei Felder besitzen, um ordnungsgemäß ausgeführt werden zu können:

* Ein Pfad zum sicheren Speicherbereich des Containers.
* Ein Pfad zum Ebenenverzeichnis des Containers.

In runhcs verfügbare Containerbefehle umfassen:

* Tools zum Erstellen und Ausführen eines Containers
  * **run** erstellt einen Container und führt ihn aus.
  * **create** erstellt einen Container.

* Tools zum Verwalten von Prozessen, die in einem Container ausgeführt werden:
  * **start** führt den benutzerdefinierten Prozess in einem erstellten Container aus.
  * **exec** führt einen neuen Prozess innerhalb des Containers aus.
  * **pause** hält alle Prozesse innerhalb des Containers an.
  * **resume** setzt alle Prozesse fort, die zuvor angehalten wurden.
  * **ps** zeigt die Prozesse an, die innerhalb eines Containers ausgeführt werden.

* Tools zum Verwalten des Containerzustands
  * **state** gibt den Zustand eines Containers aus.
  * **kill** sendet das angegebene Signal (Standard: SIGTERM) für den Initialisierungsprozess des Containers.
  * **delete** löscht alle Ressourcen, die sich im Container befinden, die häufig mit getrennten Containern verwendet werden.

Der einzige Befehl, der als Befehl für mehrere Container angesehen werden könnte, ist **list**.  Er listet aktive oder angehaltene Container auf, die von runhcs mit dem angegebenen Stamm gestartet wurden.

### <a name="hcs"></a>HCS

Auf GitHub sind zwei Wrapper als Schnittstelle mit dem HCS verfügbar. Da es sich bei dem HCS um eine C-API handelt, erleichtern Wrapper den Aufruf des Hostcomputediensts (HCS) über Sprachen höherer Ebene.  

* [hcsshim](https://github.com/microsoft/hcsshim): HCSShim ist in Go geschrieben und bildet die Grundlage für runhcs.
Holen Sie sich das Neueste von AppVeyor oder erstellen Sie es selbst.
* [dotnet-computevirtualization](https://github.com/microsoft/dotnet-computevirtualization): dotnet-computevirtualization ist ein C#-Wrapper für den HCS.

Wenn Sie den HCS verwenden möchten (entweder direkt oder über einen Wrapper), oder wenn Sie einen Rust/Haskell/InsertYourLanguage-Wrapper um den HCS erstellen möchten, hinterlassen Sie einen Kommentar.

Weitere Informationen zum HCS finden Sie in der [DockerCon-Präsentation von John Stark](https://www.youtube.com/watch?v=85nCF5S8Qok).

## <a name="containerdcri"></a>containerd/cri

> [!IMPORTANT]
> CRI-Unterstützung ist nur in Server 2019/Windows 10 1809 und höher verfügbar.  Außerdem entwickeln wir immer noch aktiv containerd für Windows.
> Nur zu Entwicklungs-/Testzwecken.

Während die OCI-Spezifikationen einen einzelnen Container definieren, beschreibt [CRI](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto) (Container Runtime Interface) Container als Workloads in einer gemeinsam genutzten Sandboxumgebung, die als Pod bezeichnet wird.  Pods können eine oder mehrere Containerworkloads enthalten.  Mit Pods können Containerorchestratoren wie Kubernetes und Service Fabric Mesh gruppierte Workloads bearbeiten, die sich auf demselben Host mit einigen gemeinsamen Ressourcen wie Arbeitsspeicher und vNETs befinden sollten.

containerd/cri ermöglicht die folgende Kompatibilitätsmatrix für Pods:

| Hostbetriebssystem | Containerbetriebssystem | Isolierung | Podunterstützung? |
|:-------------------------------------------------------------------------|:-----------------------------------------------------------------------------|:---------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------|
| <ul><li>Windows Server 2019/1809</ul></li><ul><li>Windows 10 1809</ul></li> | Linux | `hyperv` | Ja – Unterstützt echte Pods für mehrere Container. |
|  | Windows Server 2019/1809 | `process`* oder `hyperv` | Ja – Unterstützt echte Pods für mehrere Container, wenn jedes Betriebssystem des Workloadcontainers mit dem Betriebssystem für den virtuellen Hilfsprogrammcomputer übereinstimmt. |
|  | Windows Server 2016,</br>Windows Server 1709,</br>Windows Server 1803 | `hyperv` | Teilweise – Unterstützt Podsandboxes, die einen einzelnen prozessisolierten Container pro Hilfsprogramm-VM unterstützen können, wenn das Containerbetriebssystem mit dem Betriebssystem der Hilfsprogramm-VM übereinstimmt. |

\*Windows 10-Hosts unterstützen nur Hyper-V-Isolation

Links zur CRI-Spezifikation:

* [RunPodSandbox](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L24): Podspezifikation
* [CreateContainer](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L47): Workloadspezifikation

![Auf containerd basierte Containerumgebungen](media/containerd-platform.png)

Während runHCS und containerd beide auf jedem Windows Server 2016-System oder höher verwaltet werden können, erforderte die Unterstützung von Pods (Containergruppen) Änderungen an den Containertools in Windows.  Die CRI-Unterstützung ist unter Windows Server 2019/Windows 10 1809 und höher verfügbar.
