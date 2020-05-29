---
title: Orchestrieren von Containern mit einem gMSA
description: Orchestrieren von Windows-Containern mit einem gruppenverwalteten Dienstkonto (gMSA)
keywords: Docker, Container, aktives Verzeichnis, gMSA, Orchestrierung, Kubernetes, gruppenverwaltetes Dienstkonto, gruppenverwaltete Dienstkonten
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 3d102aac45a1becf1879a718bb255d753b215006
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910260"
---
# <a name="orchestrate-containers-with-a-gmsa"></a>Orchestrieren von Containern mit einem gMSA

In Produktionsumgebungen verwenden Sie häufig einen Containerorchestrator, um Ihre Anwendungen und Dienste bereitzustellen und zu verwalten. Jeder Orchestrator hat seine eigenen Verwaltungsparadigmen und ist für die Annahme der Spezifikationen für Anmeldeinformationen für die Windows-Containerplattform verantwortlich.

Stellen Sie beim Orchestrieren von Containern mit gruppenverwalteten Dienstkonten (gMSAs) Folgendes sicher:

> [!div class="checklist"]
> * Alle Containerhosts, die für die Ausführung von Containern mit gMSAs geplant werden können, sind mit einer Domäne verbunden.
> * Die Containerhosts haben Zugriff, um die Kennwörter für alle von Containern verwendeten gMSAs abzurufen.
> * Die Spezifikationsdateien für die Anmeldeinformationen werden erstellt und zum Orchestrator hochgeladen oder auf jeden Containerhost kopiert, je nachdem, wie der Orchestrator sie verarbeiten möchte.
> * Containernetzwerke ermöglichen es den Containern, mit den Active Directory-Domänencontrollern zu kommunizieren, um gMSA-Tickets abzurufen.

## <a name="how-to-use-gmsa-with-service-fabric"></a>Verwenden von gMSA mit Service Fabric

Service Fabric unterstützt das Ausführen von Windows-Containern mit einem gMSA, wenn Sie den Speicherort der Spezifikation für die Anmeldeinformationen in Ihrem Anwendungsmanifest angeben. Sie müssen die Spezifikationsdatei für die Anmeldeinformationen erstellen und im Unterverzeichnis **CredentialSpecs** des Docker-Datenverzeichnisses auf jedem Host speichern, damit sie von Service Fabric gefunden wird. Sie können das Cmdlet **Get-CredentialSpec**, das Teil des [CredentialSpec PowerShell-Moduls](https://aka.ms/credspec) ist, zum Überprüfen ausführen, ob sich Ihre Anmeldeinformationen am richtigen Speicherort befinden.

Weitere Informationen zum Konfigurieren Ihrer Anwendung finden Sie unter [Schnellstart: Bereitstellen von Windows-Containern für Service Fabric](https://docs.microsoft.com/azure/service-fabric/service-fabric-quickstart-containers) und [Einrichten von gMSAs für Windows-Container unter Service Fabric](https://docs.microsoft.com/azure/service-fabric/service-fabric-setup-gmsa-for-windows-containers).

## <a name="how-to-use-gmsa-with-docker-swarm"></a>Verwenden von gMSA mit Docker Swarm

Um ein gMSA mit Containern zu verwenden, die von Docker Swarm verwaltet werden, führen Sie den Befehl [docker service create](https://docs.docker.com/engine/reference/commandline/service_create/) mit dem Parameter `--credential-spec` aus:

```powershell
docker service create --credential-spec "file://contoso_webapp01.json" --hostname "WebApp01" <image name>
```

Weitere Informationen zur Verwendung von Spezifikationen für Anmeldeinformationen mit Docker-Diensten finden Sie im [Docker Swarm-Beispiel](https://docs.docker.com/engine/reference/commandline/service_create/#provide-credential-specs-for-managed-service-accounts-windows-only).

## <a name="how-to-use-gmsa-with-kubernetes"></a>Verwenden von gMSA mit Kubernetes

Unterstützung für die Planung von Windows-Containern mit gMSAs in Kubernetes ist als Alpha-Feature in Kubernetes 1.14 verfügbar. Aktuelle Informationen zu diesem Feature und zum Testen in der Kubernetes-Distribution finden Sie unter [Konfigurieren von gMSA für Windows-Pods und -Container](https://kubernetes.io/docs/tasks/configure-pod-container/configure-gmsa).

## <a name="next-steps"></a>Nächste Schritte

Zusätzlich zum Orchestrieren von Containern können Sie gMSAs auch zu Folgendem verwenden:

- [Konfigurieren von Apps](gmsa-configure-app.md)
- [Ausführen von Containern](gmsa-run-container.md)

Wenn während des Setups Probleme auftreten, überprüfen Sie unseren [Leitfaden zur Problembehandlung](gmsa-troubleshooting.md) auf mögliche Lösungen.
