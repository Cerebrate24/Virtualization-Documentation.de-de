---
title: Kubernetes unter Windows
author: gkudra-msft
ms.author: gekudray
ms.date: 02/09/2018
ms.prod: containers
ms.topic: overview
description: Hinzufügen eines Windows-Knotens zu einem Kubernetes-Cluster mit v 1,14.
keywords: kubernetes, 1,14, Windows, Getting Started
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 43525b86e00ad3d855f98394aa96ffe987fb51d6
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192327"
---
# <a name="kubernetes-on-windows"></a>Kubernetes unter Windows

Diese Seite bietet einen Überblick über die ersten Schritte mit Kubernetes unter Windows, indem Windows-Knoten einem Linux-basierten Cluster hinzufügen. Mit der Veröffentlichung von Kubernetes 1,14 unter Windows Server, [Version 1809](https://docs.microsoft.com/windows-server/get-started/whats-new-in-windows-server-1809#container-networking-with-kubernetes), können Benutzer die folgenden Features in Kubernetes unter Windows nutzen:

- **Überlagerungs Netzwerk**: Verwenden von Flannel im vxlan-Modus zum Konfigurieren eines virtuellen Überlagerungs Netzwerks
    - erfordert entweder Windows Server 2019 mit installiertem [KB4489899](https://support.microsoft.com/help/4489899) oder [Windows Server vNext Insider Preview](https://blogs.windows.com/windowsexperience/tag/windows-insider-program/) -Build 18317 +
    - erfordert Kubernetes v 1.14 (oder höher) mit `WinOverlay` aktiviertem Feature Gate
    - erfordert Flannel v 0.11.0 (oder höher)
- **vereinfachte Netzwerkverwaltung**: Verwenden Sie den Flannel im Host-Gateway-Modus für die automatische Routen Verwaltung zwischen Knoten.
- **Verbesserungen bei der Skalierbarkeit**: schnellere und zuverlässigere Container Startzeiten Dank nicht aufzurufbarer [vNICs für Windows Server-Container](https://techcommunity.microsoft.com/t5/Networking-Blog/Network-start-up-and-performance-improvements-in-Windows-10/ba-p/339716).
- **Hyper-v-Isolation (Alpha)**: orchestrieren Sie die [Hyper-v-Isolation](https://kubernetes.io/docs/getting-started-guides/windows/#hyper-v-containers) mit kernelmodusisolation, um die Sicherheit zu erhöhen. Weitere Informationen finden Sie unter [Windows-Containertypen](https://docs.microsoft.com/virtualization/windowscontainers/about/#windows-container-types).
    - erfordert Kubernetes v 1.10 (oder höher) mit `HyperVContainer` aktiviertem Feature Gate.
- **Storage**-Plug-ins: Verwenden Sie das [flexvolume-Speicher-Plug](https://github.com/Microsoft/K8s-Storage-Plugins) -in für Windows-Container.

>[!TIP]
>Wenn Sie einen Cluster in Azure bereitstellen möchten, macht das Open Source-Tool AKS-Engine dies leicht. Weitere Informationen finden Sie in unserer exemplarischen Vorgehens [Weise.](https://github.com/Azure/aks-engine/blob/master/docs/topics/windows.md)

## <a name="prerequisites"></a>Voraussetzungen

### <a name="plan-ip-addressing-for-your-cluster"></a>Planen der IP-Adressierung für Ihren Cluster

<a name="definitions"></a>Da Kubernetes-Cluster neue Subnetze für Pods und Dienste einführen, ist es wichtig sicherzustellen, dass keine Konflikte mit anderen vorhandenen Netzwerken in Ihrer Umgebung entstehen. Im folgenden finden Sie alle Adressräume, die freigegeben werden müssen, um Kubernetes erfolgreich bereitzustellen:

| Subnetz/Adressbereich | BESCHREIBUNG | Standardwert |
| --------- | ------------- | ------------- |
| <a name="service-subnet-def"></a>**Dienstsubnetz** | Ein nicht Routing fähiges, rein virtuelles Subnetz, das von Pods zum einheitlichen Zugriff auf Dienste verwendet wird, ohne die Netzwerktopologie zu kümmern. Es wird von `kube-proxy` von/auf routingfähige Adressbereiche übersetzt, die auf diesen Knoten ausgeführt werden. | "10.96.0.0/12" |
| <a name="cluster-subnet-def"></a>**Clustersubnetz** |  Hierbei handelt es sich um ein globales Subnetz, das von allen Pods im Cluster verwendet wird. Jedem Knoten wird ein kleineres/24-Subnetz zugewiesen, das von seinen Pods verwendet werden kann. Es muss groß genug sein, um alle im Cluster verwendeten Pods aufnehmen zu können. So wird die *mindestens erforderliche* Subnetzgröße berechnet: `(number of nodes) + (number of nodes * maximum pods per node that you configure)` <p/>Beispiel für einen Cluster mit 5 Knoten für 100 Pods pro Knoten: `(5) + (5 *  100) = 505` .  | "10.244.0.0/16" |
| **Kubernetes DNS-Dienst-IP** | Die IP-Adresse des "Kube-DNS"-Dienstanbieter, der für die DNS-Auflösung & Cluster Dienst Ermittlung verwendet wird. | "10.96.0.10" |

> [!NOTE]
> Bei der Installation von Docker wird standardmäßig ein weiteres docker-Netzwerk (NAT) erstellt, das erstellt wird. Es ist nicht erforderlich, Kubernetes unter Windows zu betreiben, da wir stattdessen IPS aus dem Clustersubnetz zuweisen.

## <a name="what-you-will-accomplish"></a>Was Sie erreichen

Am Ende dieser Anleitung haben Sie:

> [!div class="checklist"]
> * Es wurde ein [Kubernetes-Master](./creating-a-linux-master.md) Knoten erstellt.
> * Es wurde eine [Netzwerklösung](./network-topologies.md)ausgewählt.
> * Sie haben einen [Windows-workerknoten](./joining-windows-workers.md) oder einen Linux- [workerknoten](./joining-linux-workers.md) damit verknüpft.
> * Es wurde eine [Kubernetes-Beispiel Ressource](./deploying-resources.md)bereitgestellt.
> * [Allgemeine Probleme und Fehler ](./common-problems.md) angesprochen.

## <a name="next-steps"></a>Nächste Schritte

In diesem Abschnitt haben wir über wichtige Voraussetzungen gesprochen & Annahmen, die für die erfolgreiche Bereitstellung von Kubernetes unter Windows erforderlich sind. Weitere Informationen zum Einrichten eines Kubernetes Masters finden Sie hier:

>[!div class="nextstepaction"]
>[Erstellen eines Kubernetes-Masters](./creating-a-linux-master.md)