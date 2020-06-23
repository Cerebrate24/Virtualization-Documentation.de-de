---
title: Windows-Container Netzwerk
description: Sanfte Einführung in die Architektur von Windows-Container Netzwerken.
keywords: Docker, Container
author: jmesser81
ms.date: 03/27/2018
ms.topic: overview
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: 2235ae8b48828535facaa9b2be3dfc7fde450516
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192627"
---
# <a name="windows-container-networking"></a>Windows-Container Netzwerk

>[!IMPORTANT]
>Beachten Sie die Docker- [Container Netzwerke](https://docs.docker.com/engine/userguide/networking/) für allgemeine docker-Netzwerk Befehle, Optionen und Syntax. * * * mit Ausnahme von Fällen, die unter [nicht unterstützte Features und Netzwerkoptionen](#unsupported-features-and-network-options)beschrieben werden, werden alle docker-Netzwerk Befehle unter Windows mit der gleichen Syntax wie unter Linux unterstützt. Allerdings unterscheiden sich die Windows-und Linux-Netzwerk Stapel, und daher werden Sie feststellen, dass einige Linux-Netzwerk Befehle (z. b. ifconfig) unter Windows nicht unterstützt werden.

## <a name="basic-networking-architecture"></a>Grundlegende Netzwerkarchitektur

Dieses Thema bietet einen Überblick darüber, wie docker Host Netzwerke unter Windows erstellt und verwaltet. Windows-Container funktionieren in Bezug auf Netzwerke ähnlich wie virtuelle Computer. Jeder Container verfügt über einen virtuellen Netzwerkadapter (vNIC). der mit einem virtuellen Hyper-V-Switch (vSwitch) verbunden ist. Windows unterstützt fünf verschiedene [Netzwerktreiber oder-Modi](./network-drivers-topologies.md) , die über docker erstellt werden können: *NAT*, *Overlay*, *transparent*, *l2bridge*und *l2tunnel*. Sie sollten den Netzwerktreiber auswählen, der Ihren Bedürfnissen im Hinblick auf Ihre physische Netzwerkinfrastruktur und die Netzwerkanforderungen (Netzwerk mit einem oder mehreren Hosts) am besten gerecht wird.

![Text](media/windowsnetworkstack-simple.png)

Wenn die Docker-Engine das erste Mal ausgeführt wird, wird ein standardmäßiges NAT-Netzwerk erstellt ('nat'), das einen internen vSwitch und eine Windows-Komponente mit dem Namen `WinNAT` verwendet. Wenn bereits vorhandene externe vSwitches auf dem Host vorhanden sind, die über PowerShell oder Hyper-V-Manager erstellt wurden, sind Sie auch für docker mithilfe des *transparenten* Netzwerk Treibers verfügbar und können angezeigt werden, wenn Sie den ``docker network ls`` Befehl ausführen.

![Text](media/docker-network-ls.png)

- Ein **interner** Vswitch ist ein virtueller Switch, der nicht direkt mit einem Netzwerkadapter auf dem Container Host verbunden ist.
- Bei einem externen Vswitch handelt es sich um einen **externen** Vswitch, der direkt mit einem Netzwerkadapter auf dem Container Host verbunden ist.

![Text](media/get-vmswitch.png)

Das NAT-Netzwerk ist das Standardnetzwerk bei Containern, die unter Windows ausgeführt werden. Container, die unter Windows ohne Flags oder Argumente zur Implementierung bestimmter Konfigurationen ausgeführt werden, werden an das standardmäßige NAT-Netzwerk angeschlossen und einer IP-Adresse aus dem NAT-Netzwerk aus dessen internen Präfix-IP-Bereich zugewiesen. Der verwendete interne IP-Standardpräfix für NAT ist 172.16.0.0/16.

## <a name="container-network-management-with-host-network-service"></a>Verwaltung des Containermetzwerks mit dem Host Network Service (HNS)

Der Host Netzwerkdienst (HNS) und der hostcompute-Dienst (HCS) arbeiten zusammen, um Container zu erstellen und Endpunkte an ein Netzwerk anzufügen.

### <a name="network-creation"></a>Erstellen eines Netzwerks

- HNS erstellt einen virtuellen Hyper-V-Switch für jedes Netzwerk
- HNS erstellt NAT-und IP-Pools nach Bedarf.

### <a name="endpoint-creation"></a>Endpunkt Erstellung

- HNS erstellt Netzwerk Namespace pro Container Endpunkt
- HNS/HCS platziert v (m) NIC innerhalb des Netzwerk Namespace
- HNS erstellt (Vswitch)-Ports
- HNS weist dem Endpunkt IP-Adresse, DNS-Informationen, Routen usw. (unter dem Netzwerkmodus) zu.

### <a name="policy-creation"></a>Richtlinienerstellung

- Standard-NAT-Netzwerk: HNS erstellt winnat-Port Weiterleitungs Regeln/-Zuordnungen mit entsprechenden Windows-Firewall-Zulassungsregeln
- Alle anderen Netzwerke: HNS verwendet die Virtual Filtering Platform (VFP) für die Richtlinien Erstellung.
    - Hierzu gehören: Lastenausgleich, ACLs, Kapselung usw.
    - Suchen Sie nach unseren [hier](https://docs.microsoft.com/windows-server/networking/technologies/hcn/hcn-top) veröffentlichten HNS-APIs und-Schemas.

![Text](media/HNS-Management-Stack.png)

## <a name="unsupported-features-and-network-options"></a>Nicht unterstützte Features und Netzwerkoptionen

Die folgenden Netzwerkoptionen werden zurzeit **nicht** unter Windows unterstützt:

- An l2bridge-, NAT-und Überlagerungs Netzwerken angefügte Windows-Container unterstützen die Kommunikation über den IPv6-Stapel nicht.
- Verschlüsselte Container Kommunikation über IPSec.
- HTTP-Proxy Unterstützung für Container.
- [Hostmodusnetzwerk](https://docs.docker.com/ee/ucp/interlock/config/host-mode-networking/)
- Netzwerke in der virtualisierten Azure-Infrastruktur über den transparenten Netzwerktreiber.

| Get-Help        | Nicht unterstützte Option   |
|---------------|:--------------------:|
| ``docker run``|   ``--ip6``, ``--dns-option`` |
| ``docker network create``| ``--aux-address``, ``--internal``, ``--ip-range``, ``--ipam-driver``, ``--ipam-opt``, ``--ipv6``, ``--opt encrypted`` |
