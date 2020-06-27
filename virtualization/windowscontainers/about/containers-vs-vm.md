---
title: Container im Vergleich zu virtuellen Computern
description: 'In diesem Thema werden einige der wichtigsten Gemeinsamkeiten und Unterschiede zwischen Containern und virtuellen Computern erläutert. Container und virtuelle Computer haben jeweils einen eigenen Verwendungszweck: In der Tat verwenden viele Implementierungen von Containern virtuelle Computer als Hostbetriebssystem, anstatt direkt auf der Hardware ausgeführt zu werden, insbesondere wenn Container in der Cloud ausgeführt werden.'
keywords: Docker, Container, VMs, virtuelle Computer
author: jasongerend
ms.author: jgerend
ms.date: 10/21/2019
ms.topic: overview
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 0262fc1c2fe3e050455cad9420cc4f70d0530fa2
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192727"
---
# <a name="containers-vs-virtual-machines"></a>Container im Vergleich zu virtuellen Computern

In diesem Thema werden einige der wichtigsten Gemeinsamkeiten und Unterschiede zwischen Containern und virtuellen Computern (VMs) erläutert. Container und VMs haben jeweils einen eigenen Verwendungszweck: In der Tat verwenden viele Bereitstellungen von Containern VMs als Hostbetriebssystem, anstatt direkt auf der Hardware ausgeführt zu werden, insbesondere wenn Container in der Cloud ausgeführt werden.

Einen Überblick über Container finden Sie unter [Windows und Container](index.md).

## <a name="container-architecture"></a>Containerarchitektur

Ein Container ist ein isoliertes, schlankes Silo zum Ausführen einer Anwendung auf dem Hostbetriebssystem. Container bauen auf dem Kernel des Hostbetriebssystems auf (der als die verborgene Struktur des Betriebssystems betrachtet werden kann) und enthalten nur Apps und einige Lightweight-APIs und -Dienste für Betriebssysteme, die im Benutzermodus ausgeführt werden, wie in dieser Abbildung gezeigt.

![Architekturabbildung, die zeigt, wie Container über dem Kernel ausgeführt werden.](media/container-diagram.svg)

## <a name="virtual-machine-architecture"></a>Architektur des virtuellen Computers

Im Gegensatz zu einem Container führen VMs ein vollständiges Betriebssystem (einschließlich seines eigenen Kernels) aus, wie diese Abbildung zeigt.

![Architekturabbildung, die zeigt, wie VMs ein vollständiges Betriebssystem neben dem Hostbetriebssystem ausführen.](media/virtual-machine-diagram.svg)

## <a name="containers-vs-virtual-machines"></a>Container im Vergleich zu virtuellen Computern

In der folgenden Tabelle werden einige Gemeinsamkeiten und Unterschiede dieser sich ergänzenden Technologien aufgeführt.

|                 | Virtueller Computer  | Container  |
| --------------  | ---------------- | ---------- |
| Isolierung       | Bietet umfassende Isolierung vom Hostbetriebssystem und anderen VMs. Dies ist hilfreich, wenn eine starke Sicherheitsgrenze wichtig ist, z. B. beim Hosten von Apps von konkurrierenden Unternehmen auf demselben Server oder Cluster. | Stellt in der Regel eine vereinfachte Isolierung vom Host und anderen Containern bereit, bietet jedoch keine so starke Sicherheitsgrenze wie eine VM. (Sie können die Sicherheit erhöhen, indem Sie den [Hyper-V-Isolierungsmodus](../manage-containers/hyperv-container.md) verwenden, um jeden Container in einer Lightweight-VM zu isolieren). |
| Betriebssystem | Führt ein vollständiges Betriebssystem einschließlich des Kernels aus und erfordert somit mehr Systemressourcen (CPU, Arbeitsspeicher und Speicherplatz). | Führt den Benutzermodusanteil eines Betriebssystems aus und kann so angepasst werden, dass nur die für Ihre App benötigten Dienste enthalten sind, wobei weniger Systemressourcen verwendet werden. |
| Gastkompatibilität | Führt fast beliebige Betriebssysteme innerhalb des virtuellen Computers aus. | Wird unter der [gleichen Betriebssystemversion wie der Host](../deploy-containers/version-compatibility.md) ausgeführt (Hyper-V-Isolierung ermöglicht Ihnen, frühere Versionen desselben Betriebssystems in einer vereinfachten VM-Umgebung auszuführen)
| Bereitstellung     | Stellen Sie einzelne VMs mithilfe von Windows Admin Center oder Hyper-V-Manager bereit. Stellen Sie mehrere VMs mithilfe von PowerShell oder System Center Virtual Machine Manager bereit. | Stellen Sie einzelne Container mithilfe von Docker über die Befehlszeile bereit. Stellen Sie mehrere Container mithilfe eines Orchestrators wie Azure Kubernetes Service bereit. |
| Betriebssystemupdates und -upgrades | Laden Sie Betriebssystemupdates auf jede VM herunter, und installieren Sie sie. Die Installation einer neuen Betriebssystemversion erfordert ein Upgrade oder häufig nur die Erstellung einer völlig neuen VM. Dies kann zeitaufwändig sein, besonders wenn Sie viele VMs verwenden... | Das Ausführen von Updates oder Upgrades der Betriebssystemdateien innerhalb eines Containers ist identisch: <br><ol><li>Bearbeiten Sie die Builddatei Ihres Containerimages (bekannt als Dockerfile-Datei) so, dass sie auf die neueste Version des Windows-Basisimages verweist. </li><li>Erstellen Sie das Containerimage mit diesem neuen Basisimage neu.</li><li>Pushen Sie das Containerimage in Ihre Containerregistrierung.</li> <li>Stellen Sie es mithilfe eines Orchestrators erneut bereit.<br>Der Orchestrator bietet leistungsstarke Automatisierung im richtigen Maßstab. Weitere Informationen finden Sie unter [Tutorial: Aktualisieren einer Anwendung in Azure Kubernetes Service](https://docs.microsoft.com/azure/aks/tutorial-kubernetes-app-update).</li></ol> |
| Permanenter Speicher | Verwenden Sie eine VHD (Virtual Hard Disk) als lokalen Speicher für eine einzelne VM oder eine SMB-Dateifreigabe für den Speicher, der von mehreren Servern gemeinsam genutzt wird. | Verwenden Sie Azure Disks als lokalen Speicher für einen einzelnen Knoten oder Azure Files (SMB-Freigaben) für Speicher, der von mehreren Knoten oder Servern gemeinsam genutzt wird. |
| Lastenausgleich | Beim Lastenausgleich virtueller Computer werden aktuell ausgeführte VMs oder andere Server in einen Failovercluster verschoben. | Container selbst werden nicht verschoben. Stattdessen kann ein Orchestrator Container auf Clusterknoten automatisch starten oder beenden, um Änderungen bei der Auslastung und Verfügbarkeit zu verwalten. |
| Fehlertoleranz | VMs können ein Failover zu einem anderen Server in einem Cluster ausführen, wobei das Betriebssystem der VM auf dem neuen Server neu gestartet wird.  | Wenn ein Clusterknoten ausfällt, werden alle darauf ausgeführten Container schnell vom Orchestrator auf einem anderen Clusterknoten neu erstellt. |
| Netzwerk     | Verwendet virtuelle Netzwerkadapter. | Verwendet eine isolierte Ansicht eines virtuellen Netzwerkadapters und bietet so etwas weniger Virtualisierung (die Firewall des Hosts wird mit Containern gemeinsam verwendet) bei weniger Ressourcenverbrauch. Weitere Informationen finden sie unter [Windows-Containernetzwerke](../container-networking/architecture.md). |