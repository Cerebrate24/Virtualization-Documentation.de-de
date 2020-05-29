---
title: Implementieren von Ressourcensteuerungen
description: Details zur Ressourcensteuerung für Windows-Container
keywords: Docker, Container, cpu, Arbeitsspeicher, Datenträger, Ressourcen
author: taylorb-microsoft
ms.date: 11/21/2017
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 8ccd4192-4a58-42a5-8f74-2574d10de98e
ms.openlocfilehash: 3e9f7e3208222cd6c0f512c5f892453ac6e6980c
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910170"
---
# <a name="implementing-resource-controls-for-windows-containers"></a>Implementieren von Ressourcensteuerungen für Windows-Container
Pro Container und pro Ressource können diverse Steuerungen implementiert werden.  Standardmäßig unterliegen ausgeführte Container einer typischen Windows-Ressourcenverwaltung, die im Allgemeinen auf einer gleichberechtigten Verteilung basiert. Ein Entwickler oder Administrator kann die Ressourcennutzung jedoch durch die Konfiguration der einzelnen Steuerungselemente begrenzen oder beeinflussen.  Zu den steuerbaren Ressourcen gehören: CPU/Prozessor, Arbeitsspeicher/RAM, Datenträger/Speicher und Netzwerk/Durchsatz.

Windows-Container nutzen [Auftragsobjekte](https://docs.microsoft.com/windows/desktop/ProcThread/job-objects) zum Gruppieren und Nachverfolgen der Prozesse, die jedem Container zugeordnet sind.  Steuerungselemente für Ressourcen werden für das dem Container zugeordnete übergeordnete Auftragsobjekt implementiert. 

Im Fall der [Hyper-V-Isolation](./hyperv-container.md) werden die Ressourcensteuerelemente automatisch sowohl auf den virtuellen Computer als auch auf das Auftragsobjekt des Containers innerhalb der virtuellen Maschine angewendet. Dies stellt sicher, dass selbst dann, wenn ein im Container ausgeführter Prozess die Steuerelemente des Auftragsobjekts umgeht, der virtuelle Computer dafür sorgen kann, dass die definierten Ressourcenbegrenzungen nicht überschritten werden.

## <a name="resources"></a>Ressourcen
Dieser Abschnitt enthält für jede Ressource eine Zuordnung zwischen der Docker-Befehlszeilenschnittstelle (als Beispiel für die Verwendung der Ressourcensteuerung, die von einem Orchestrator oder per Tool konfiguriert sein kann) und der entsprechenden Windows Host Compute Service (HCS)-API. Zudem wird allgemein angegeben, wie die Ressourcensteuerung von Windows implementiert wurde (bitte beachten Sie, dass diese Beschreibung auf einer höheren Ebene erfolgt, deren zugrundeliegende Implementierung sich ändern kann).

|  | |
| ----- | ------|
| *Arbeitsspeicher* ||
| Docker-Schnittstelle | [--memory](https://docs.docker.com/engine/admin/resource_constraints/#memory) |
| HCS-Schnittstelle | [MemoryMaximumInMB](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| Gemeinsamer Kernel | [JOB_OBJECT_LIMIT_JOB_MEMORY](https://docs.microsoft.com/windows/desktop/api/winnt/ns-winnt-_jobobject_basic_limit_information) |
| Hyper-V-Isolierung | Arbeitsspeicher des virtuellen Computers |
| _Hinweis zur Hyper-V-Isolation in Windows Server 2016: Wenn Sie eine Memory-Cap verwenden, werden Sie bemerken, dass der Container den durch die Cap festgelegten Speicher zunächst zuweist und dann beginnt, ihn an den Containerhost zurückzugeben.  In späteren Versionen (ab 1709) wurde dieses Verhalten optimiert._ |
| ||
| *CPU (Anzahl)* ||
| Docker-Schnittstelle | [--cpus](https://docs.docker.com/engine/admin/resource_constraints/#cpu) |
| HCS-Schnittstelle | [ProcessorCount](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| Gemeinsamer Kernel | Simuliert mit [JOB_OBJECT_CPU_RATE_CONTROL_HARD_CAP](https://docs.microsoft.com/windows/desktop/api/winnt/ns-winnt-_jobobject_cpu_rate_control_information)* |
| Hyper-V-Isolierung | Anzahl verfügbarer virtueller Prozessoren |
| ||
| *CPU (Prozent)* ||
| Docker-Schnittstelle | [--cpu-percent](https://docs.docker.com/engine/admin/resource_constraints/#cpu) |
| HCS-Schnittstelle | [ProcessorMaximum](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| Gemeinsamer Kernel | [JOB_OBJECT_CPU_RATE_CONTROL_HARD_CAP](https://docs.microsoft.com/windows/desktop/api/winnt/ns-winnt-_jobobject_cpu_rate_control_information) |
| Hyper-V-Isolierung | Hypervisor-Limits für virtuelle Prozessoren |
| ||
| *CPU (Freigaben)* ||
| Docker-Schnittstelle | [--cpu-shares](https://docs.docker.com/engine/admin/resource_constraints/#cpu) |
| HCS-Schnittstelle | [ProcessorWeight](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| Gemeinsamer Kernel | [JOB_OBJECT_CPU_RATE_CONTROL_WEIGHT_BASED](https://docs.microsoft.com/windows/desktop/api/winnt/ns-winnt-_jobobject_cpu_rate_control_information) |
| Hyper-V-Isolierung | Hypervisor-Gewichtungen für virtuelle Prozessoren |
| ||
| *Speicher (Image)* ||
| Docker-Schnittstelle | [--io-maxbandwidth/--io-maxiops](https://docs.docker.com/edge/engine/reference/commandline/run/#usage) |
| HCS-Schnittstelle | [StorageIOPSMaximum und StorageBandwidthMaximum](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| Gemeinsamer Kernel | [JOBOBJECT_IO_RATE_CONTROL_INFORMATION](https://docs.microsoft.com/windows/desktop/api/jobapi2/ns-jobapi2-jobobject_io_rate_control_information) |
| Hyper-V-Isolierung | [JOBOBJECT_IO_RATE_CONTROL_INFORMATION](https://docs.microsoft.com/windows/desktop/api/jobapi2/ns-jobapi2-jobobject_io_rate_control_information) |
| ||
| *Speicher (Volumes)* ||
| Docker-Schnittstelle | [--storage-opt size=](https://docs.docker.com/edge/engine/reference/commandline/run/#set-storage-driver-options-per-container) |
| HCS-Schnittstelle | [StorageSandboxSize](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| Gemeinsamer Kernel | [JOBOBJECT_IO_RATE_CONTROL_INFORMATION](https://docs.microsoft.com/windows/desktop/api/jobapi2/ns-jobapi2-jobobject_io_rate_control_information) |
| Hyper-V-Isolierung | [JOBOBJECT_IO_RATE_CONTROL_INFORMATION](https://docs.microsoft.com/windows/desktop/api/jobapi2/ns-jobapi2-jobobject_io_rate_control_information) |

## <a name="additional-notes-or-details"></a>Zusätzliche Hinweise oder Details

### <a name="memory"></a>Memory

Windows-Container führen in jedem Container einige Systemprozesse aus, typischerweise solche, die Funktionalität pro Container bereitstellen wie beispielsweise Benutzerverwaltung, Networking usw. Und da ein großer Teil des von diesen Prozessen benötigten Speichers unter Containern geteilt wird, muss die Speicherkapazität hoch genug sein, um die Prozessausführung zu ermöglichen.  Eine Tabelle für jeden Basisimagetyp mit und ohne Hyper-V-Isolation finden Sie im Dokument [Systemanforderungen](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/system-requirements#memory-requirments).

### <a name="cpu-shares-without-hyper-v-isolation"></a>CPU-Freigaben (ohne Hyper-V-Isolation)

Bei Nutzung von CPU-Freigaben wird [JOBOBJECT_CPU_RATE_CONTROL_INFORMATION](https://docs.microsoft.com/windows/desktop/api/winnt/ns-winnt-_jobobject_cpu_rate_control_information) von der zugrunde liegenden Implementierung konfiguriert (wenn keine Hyper-V-Isolation verwendet wird). Insbesondere wird das Steuerungsflag auf JOB_OBJECT_CPU_RATE_CONTROL_WEIGHT_BASED gesetzt und eine entsprechende Gewichtung angegeben.  Die gültigen Gewichtungen für das Auftragsobjekt sind die Werte 1 bis 9 mit dem Standardwert 5, was eine geringere Genauigkeit bedeutet als die HCS-Werte 1 bis 10000.  Beispiel: Eine Freigabegewichtung 7500 würde eine Gewichtung 7 und eine Freigabegewichtung 2500 einen Wert 2 ergeben.
