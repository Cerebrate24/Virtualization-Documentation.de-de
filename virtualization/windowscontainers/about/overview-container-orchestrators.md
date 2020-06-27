---
title: Übersicht über die Windows-Containerorchestrierung
description: Erfahren Sie mehr über Windows-Containerorchestratoren.
keywords: Docker, Container
author: Heidilohr
ms.author: helohr
ms.date: 05/22/2019
ms.topic: overview
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: ff9b782814d3efe9c5fc6b537cf79280e784be11
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192566"
---
# <a name="windows-container-orchestration-overview"></a>Übersicht über die Windows-Containerorchestrierung

Aufgrund ihrer geringen Größe und der Ausrichtung der Anwendung sind Container für flexible Übermittlungsumgebungen und Architekturen optimal geeignet, die auf Microservices basieren. Eine Umgebung, in der Container und Microservices verwendet werden, kann jedoch Hunderte oder Tausende von Komponenten enthalten, die nachverfolgt werden müssen. Sie können eventuell einige Dutzend virtuelle Computer oder physische Server manuell verwalten, es ist allerdings unmöglich, eine Containerumgebung der Großserienproduktion ordnungsgemäß zu verwalten, ohne diese zu automatisieren. Diese Aufgabe sollte Ihr Orchestrator übernehmen. Dies ist ein Prozess, der eine große Anzahl von Containern automatisiert und verwaltet und bestimmt, wie sie miteinander interagieren.

Orchestratoren können die folgenden Aufgaben ausführen:

- Planung: Bei Vorliegen eines Containerimages und einer Ressourcenanforderung sucht der Orchestrator nach einem geeigneten Computer, auf dem der Container ausgeführt werden soll.
- Affinität/Antiaffinität: Geben Sie an, ob eine Gruppe von Containern aufgrund von Leistung nah beieinander oder aufgrund von Verfügbarkeit weit entfernt ausgeführt wird.
- Integritätsüberwachung: Achten Sie auf Containerfehler, und planen Sie diese automatisch erneut.
- Failover: Behalten Sie den Überblick über die auf jedem Computer ausgeführten Aufgaben, und planen Sie die Container von fehlerhaften Computern erneut auf fehlerfreien Knoten.
- Skalierung: Fügen Sie Containerinstanzen bei Bedarf hinzu, oder entfernen Sie diese entweder manuell oder automatisch.
- Netzwerk: Stellen Sie ein Überlagerungsnetzwerk für die Koordination der Container bereit, damit diese auf mehreren Hostcomputern miteinander kommunizieren.
- Dienstermittlung: Ermöglichen Sie den Containern, sich auch dann gegenseitig automatisch zu ermitteln, wenn diese zwischen Hostcomputern verschoben werden und sich IP-Adressen ändern.
- Koordinierte Anwendungsupgrades: Verwalten Sie Containerupgrades, um Ausfallzeiten der Anwendung zu vermeiden, und ermöglichen Sie ein Rollback, wenn ein Fehler auftritt.

## <a name="orchestrator-types"></a>Orchestratortypen

Azure bietet zwei Containerorchestratoren: Azure Kubernetes Service (AKS) und Service Fabric.

Mithilfe von [Azure Kubernetes Service (AKS)](/azure/aks/) kann ein Cluster aus virtuellen Computern erstellt, konfiguriert und verwaltet werden, die zum Ausführen von Anwendungen in Containern vorkonfiguriert wurden. Dadurch können Sie Ihre bereits vorhandenen Kenntnisse nutzen sowie vom großen und ständig wachsenden Fachwissen der Community profitieren, um auf Containern basierende Anwendungen in Microsoft Azure bereitzustellen und zu verwalten. Mit AKS können Sie für Unternehmen geeignete Funktionen von Azure nutzen und gleichzeitig die Anwendungsportabilität über Kubernetes und das Docker-Imageformat beibehalten.

[Azure Service Fabric](/azure/service-fabric/) ist eine Plattform für verteilte Systeme, mit der das Verpacken, Bereitstellen und Verwalten von skalierbaren und zuverlässigen Microservices und Containern vereinfacht wird. Service Fabric behandelt die großen Herausforderungen bei der Entwicklung und Verwaltung von systemeigenen Cloudanwendungen. Entwickler und Administratoren können komplexe Infrastrukturprobleme vermeiden und sich auf die Implementierung kritischer, anspruchsvoller Workloads konzentrieren, die skalierbar, zuverlässig und überschaubar sind. Service Fabric ist die Plattform der nächsten Generation für das Erstellen und Verwalten von Anwendungen der Ebene 1, der Unternehmensklasse und der Cloudebene, die im Container ausgeführt werden.

## <a name="getting-started"></a>Erste Schritte

Informationen zu den ersten Schritten bei der Bereitstellung von Azure Kubernetes Service finden Sie im [Kubernetes-Setupleitfaden](../kubernetes/getting-started-kubernetes-windows.md).

Informationen zu den ersten Schritten bei der Bereitstellung von Azure Service Fabric finden Sie im [Service Fabric-Schnellstart](/azure/service-fabric/service-fabric-quickstart-containers.md).
