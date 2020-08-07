---
title: Windows-Container Netzwerk
description: Netzwerktreiber und Topologien für Windows-Container.
keywords: Docker, Container
author: jmesser81
ms.date: 03/27/2018
ms.topic: conceptual
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: 1a45bbf5588085df8b740687fd0e889055c84262
ms.sourcegitcommit: 186ebcd006eeafb2b51a19787d59914332aad361
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 08/07/2020
ms.locfileid: "87985124"
---
# <a name="windows-container-network-drivers"></a>Treiber für Windows-Container Netzwerk

Zusätzlich zur Nutzung des standardmäßigen NAT-Netzwerks, das von Docker unter Windows erstellt wurde, können Benutzer benutzerdefinierte Containernetzwerke festlegen. Benutzerdefinierte Netzwerke können mithilfe des Befehls "docker CLI" erstellt werden [`docker network create -d <NETWORK DRIVER TYPE> <NAME>`](https://docs.docker.com/engine/reference/commandline/network_create/) . Unter Windows stehen folgende Netzwerktreibertypen zur Verfügung:

- **NAT** –-Container, die mit einem mit dem NAT-Treiber erstellten Netzwerk verbunden sind, werden mit einem *internen* Hyper-V-Switch verbunden und erhalten eine IP-Adresse vom benutzerdefinierten ``--subnet`` IP-Präfix (). Portweiterleitung/-zuordnung vom Containerhost zu Containerendpunkten wird unterstützt.

  >[!NOTE]
  > Auf Windows Server 2019 (oder höher) erstellte NAT-Netzwerke werden nach dem Neustart nicht mehr beibehalten.

  > Wenn Sie das Windows 10 Creators Update installiert haben (oder höher), werden mehrere NAT-Netzwerke unterstützt.

- **transparente** –-Container, die mit einem mit dem "transparenten"-Treiber erstellten Netzwerk verbunden sind, werden direkt über einen *externen* Hyper-V-Switch mit dem physischen Netzwerk verbunden. IPs aus dem physischen Netzwerk können mithilfe eines externen DHCP-Servers statisch (erfordert eine benutzerdefinierte ``--subnet``-Option) oder dynamisch zugewiesen werden.

  >[!NOTE]
  >Aufgrund der folgenden Anforderung wird das Verbinden Ihrer Container Hosts über ein transparentes Netzwerk auf Azure-VMS nicht unterstützt.

  > Erfordert Folgendes: Wenn dieser Modus in einem Virtualisierungsszenario (Container Host ist eine VM) verwendet wird, _ist das Spoofing von Mac-Adressen erforderlich_.

- **Overlay** : Wenn die Docker-Engine im [Swarm-Modus](../manage-containers/swarm-mode.md)ausgeführt wird, können an ein Überlagerungs Netzwerk angeschlossene Container mit anderen Containern kommunizieren, die über mehrere Container Hosts an dasselbe Netzwerk angeschlossen sind. Jedes Überlagerungsnetzwerk, das für einen Schwarmcluster erstellt wird, wird mit einem eigenen IP-Subnetz erstellt, das durch ein privates IP-Präfix definiert ist. Der Überlagerungsnetzwerktreiber verwendet VXLAN Kapselung. **Kann mit Kubernetes verwendet werden, wenn geeignete Netzwerk steuerungsflächen (z. b. Flannel) verwendet werden.**
  > Erfordert: Stellen Sie sicher, dass Ihre Umgebung diese erforderlichen [Voraussetzungen](https://docs.docker.com/network/overlay/#operations-for-all-overlay-networks) zum Erstellen von Überlagerungs Netzwerken erfüllt.

  > Erfordert: auf Windows Server 2019 ist hierfür [KB4489899](https://support.microsoft.com/help/4489899)erforderlich.

  > Erfordert: auf Windows Server 2016 ist hierfür [KB4015217](https://support.microsoft.com/help/4015217/windows-10-update-kb4015217)erforderlich.

  >[!NOTE]
  >Unter Windows Server 2019 nutzen Überlagerungs Netzwerke, die von Docker Swarm erstellt wurden, VFP-NAT-Regeln für ausgehende Verbindungen. Dies bedeutet, dass ein bestimmter Container 1 IP-Adresse erhält. Dies bedeutet auch, dass ICMP-basierte Tools wie `ping` oder `Test-NetConnection` mit ihren TCP/UDP-Optionen in Debuggingsituationen konfiguriert werden sollten.

- **l2bridge** ähnlich wie beim `transparent` Netzwerkmodus werden Container, die mit einem Netzwerk verbunden sind, das mit dem Treiber "l2bridge" erstellt wurde, über einen *externen* Hyper-V-Switch mit dem physischen Netzwerk verbunden. Der Unterschied in l2bridge besteht darin, dass Container Endpunkte die gleiche Mac-Adresse wie der Host aufweisen, da der Vorgang für die Übersetzung der Layer-2-Adressübersetzung (Mac Re-Write) bei eingehenden und ausgehenden Daten erfolgt. In Clustering-Szenarien trägt dies zu einer Verringerung der Belastung von Switches bei, die Mac-Adressen von manchmal kurzlebigen Containern erlernen müssen. L2bridge-Netzwerke können auf zwei verschiedene Arten konfiguriert werden:
  1. L2bridge Network ist mit dem gleichen IP-Subnetz wie der Container Host konfiguriert.
  2. L2bridge Network ist mit einem neuen benutzerdefinierten IP-Subnetz konfiguriert.

  In Configuration 2 müssen Benutzer einen Endpunkt auf dem Host Netzwerk Depot hinzufügen, das als Gateway fungiert, und Routing Funktionen für das angegebene Präfix konfigurieren.
  > Erfordert: erfordert Windows Server 2016, Windows 10 Creators Update oder eine spätere Version.


- **l2bridge** ähnlich wie beim `transparent` Netzwerkmodus werden Container, die mit einem Netzwerk verbunden sind, das mit dem Treiber "l2bridge" erstellt wurde, über einen *externen* Hyper-V-Switch mit dem physischen Netzwerk verbunden. Der Unterschied in l2bridge besteht darin, dass Container Endpunkte die gleiche Mac-Adresse wie der Host aufweisen, da der Vorgang für die Übersetzung der Layer-2-Adressübersetzung (Mac Re-Write) bei eingehenden und ausgehenden Daten erfolgt. In Clustering-Szenarien trägt dies zu einer Verringerung der Belastung von Switches bei, die Mac-Adressen von manchmal kurzlebigen Containern erlernen müssen. L2bridge-Netzwerke können auf zwei verschiedene Arten konfiguriert werden:
  1. L2bridge Network ist mit dem gleichen IP-Subnetz wie der Container Host konfiguriert.
  2. L2bridge Network ist mit einem neuen benutzerdefinierten IP-Subnetz konfiguriert.

  In Configuration 2 müssen Benutzer einen Endpunkt auf dem Host Netzwerk Depot hinzufügen, das als Gateway fungiert, und Routing Funktionen für das angegebene Präfix konfigurieren.
  >[!TIP]
  >Weitere Informationen zum Konfigurieren und Installieren von l2bridge finden Sie [hier](https://techcommunity.microsoft.com/t5/networking-blog/l2bridge-container-networking/ba-p/1180923).

- **l2tunnel** ähnlich wie bei l2bridge _sollte dieser Treiber jedoch nur in einem Microsoft Cloud Stapel (Azure) verwendet werden_. Aus einem Container stammende Pakete werden an den Virtualisierungshost gesendet, auf dem die Sdn-Richtlinie angewendet wird.


## <a name="network-topologies-and-ipam"></a>Netzwerktopologien und IPAM

Die folgende Tabelle zeigt, wie die Netzwerkkonnektivität für interne (Container-zu-Container) und externe Verbindungen für jeden Netzwerktreiber bereitgestellt wird.

### <a name="networking-modesdocker-drivers"></a>Netzwerk Modi/docker-Treiber

  | Docker-Windows-Netzwerktreiber | Typische Verwendung | Container-zu-Container (einzelner Knoten) | Container-zu-extern (einzelner Knoten + mehrere Knoten) | Container-zu-Container (mehrere Knoten) |
  |-------------------------------|:------------:|:------------------------------------:|:------------------------------------------------:|:-----------------------------------:|
  | **NAT (Standard)** | Gut für Entwickler | <ul><li>Gleiches Subnetz: überbrückt Verbindung über den virtuellen Hyper-V-Switch</li><li> Cross-Subnetz: nicht unterstützt (nur ein internes NAT-Präfix)</li></ul> | Weitergeleitet durch Verwaltungs-vNIC (an winnat gebunden) | Nicht direkt unterstützt: erfordert das verfügbar machen von Ports über den Host |
  | **Transparent** | Gute für Entwickler oder kleine bereit Stellungen | <ul><li>Gleiches Subnetz: überbrückt Verbindung über den virtuellen Hyper-V-Switch</li><li>Cross-Subnetz: weitergeleitet durch den Container Host</li></ul> | Routing durch den Container Host mit direktem Zugriff auf (physischer) Netzwerkadapter | Routing durch den Container Host mit direktem Zugriff auf (physischer) Netzwerkadapter |
  | **Überlagerung** | Gut für mehrere Knoten; erforderlich für docker Swarm, verfügbar in Kubernetes | <ul><li>Gleiches Subnetz: überbrückt Verbindung über den virtuellen Hyper-V-Switch</li><li>Cross-Subnetz: Netzwerk Datenverkehr wird gekapselt und über Mgmt vNIC weitergeleitet.</li></ul> | Nicht direkt unterstützt: erfordert einen zweiten Container Endpunkt, der an das NAT-Netzwerk unter Windows Server 2016 oder VFP-NAT-Regel auf Windows Server 2019 angeschlossen ist.  | Gleiches/Cross-Subnetz: Netzwerk Datenverkehr wird mithilfe von vxlan gekapselt und über Mgmt vNIC weitergeleitet. |
  | **L2Bridge** | Verwendet für Kubernetes und Microsoft Sdn | <ul><li>Gleiches Subnetz: überbrückt Verbindung über den virtuellen Hyper-V-Switch</li><li> Cross-Subnetz: die Mac-Adresse für den Container wurde bei einem eingehenden und ausgehenden Datenverkehr neu geschrieben.</li></ul> | Mac-Adresse für Container bei Eingang und Ausgang neu geschrieben | <ul><li>Gleiches Subnetz: überbrückte Verbindung</li><li>Cross-Subnetz: weitergeleitet über Mgmt vNIC auf WSv1809 und höher</li></ul> |
  | **L2Tunnel**| Nur Azure | Identisch/Cross-Subnetz: an den virtuellen Hyper-V-Switch des physischen Hosts, an den die Richtlinie angewendet wird. | Datenverkehr muss über das Azure Virtual Network-Gateway geleitet werden | Identisch/Cross-Subnetz: an den virtuellen Hyper-V-Switch des physischen Hosts, an den die Richtlinie angewendet wird. |

### <a name="ipam"></a>IPAM

IP-Adressen werden von jedem Netzwerktreiber unterschiedlich belegt und diesem zugewiesen. Windows verwendet den Host-Netzwerkdienst (HNS), um dem NAT-Treiber IPAM bereitzustellen und arbeitet mit dem Docker-Schwarmmodus (interner KVS), um IPAM für die Überlagerung bereitzustellen. Alle anderen Netzwerktreiber verwenden eine externe IPAM.

| Netzwerkmodus/-Treiber | IPAM |
| -------------------------|:----:|
| NAT | Dynamische IP-Zuweisung und Zuweisung durch den Host Netzwerkdienst (HNS) aus dem internen NAT-Subnetzpräfix |
| Transparent | Statischer oder dynamischer (mit externer DHCP-Server) IP-Zuweisung und Zuweisung von IP-Adressen im Netzwerk Präfix des Container Hosts |
| Überlagerung | Dynamische IP-Zuordnung von verwalteten Präfixen und Zuweisungen der Docker-Engine Swarm-Modus durch HNS |
| L2Bridge | Statische IP-Zuweisung und Zuweisung von IP-Adressen im Netzwerk Präfix des Container Hosts (kann auch über HNS zugewiesen werden) |
| L2Tunnel | Nur Azure: dynamische IP-Zuordnung und Zuweisung vom Plug-in |

### <a name="service-discovery"></a>Dienstsuche

Die Dienstermittlung wird nur für bestimmte Windows-Netzwerktreiber unterstützt.

|  | Lokale Dienstermittlung  | Globale Dienstermittlung |
| :---: | :---------------     |  :---                |
| NAT | YES | Ja mit docker EE |
| overlay | YES | Ja mit docker EE oder Kube-DNS |
| transparent | Nein | Nein |
| l2bridge | Nein | Ja mit Kube-DNS |
