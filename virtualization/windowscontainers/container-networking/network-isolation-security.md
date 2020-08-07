---
title: Windows-Containernetzwerk
description: Netzwerk Isolation und-Sicherheit in Windows-Containern.
keywords: Docker, Container
author: jmesser81
ms.date: 03/27/2018
ms.topic: conceptual
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: 5c60406c0cc839a84e25ff12abf53439c6a208cb
ms.sourcegitcommit: 186ebcd006eeafb2b51a19787d59914332aad361
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 08/07/2020
ms.locfileid: "87985384"
---
# <a name="network-isolation-and-security"></a>Netzwerkisolation und Sicherheit

## <a name="isolation-with-network-namespaces"></a>Isolation mit Netzwerknamespaces

Jeder Container Endpunkt wird in seinem eigenen __Netzwerk Namespace__platziert. Die Verwaltungs Host-vNIC und der Host Netzwerk Stapel befinden sich im standardmäßigen Netzwerk Namespace. Um die Netzwerk Isolation zwischen Containern auf demselben Host zu erzwingen, wird für jeden Windows Server-Container ein Netzwerk Namespace erstellt, und Container werden unter Hyper-V-Isolation ausgeführt, in der der Netzwerkadapter für den Container installiert ist. Windows Server-Container verwenden eine Host-vNIC für die Verbindung mit dem virtuellen Switch. Die Hyper-V-Isolation verwendet eine synthetische VM-NIC (die nicht für die VM des-Hilfsprogramms verfügbar gemacht wird), um Sie an den virtuellen Switch

![text](media/network-compartment-visual.png)

```powershell
Get-NetCompartment
```

## <a name="network-security"></a>Netzwerksicherheit

Abhängig vom verwendeten Container und Netzwerktreiber werden Port-ACLs durch eine Kombination aus Windows-Firewall und [VFP](https://www.microsoft.com/research/project/azure-virtual-filtering-platform/)erzwungen.

### <a name="windows-server-containers"></a>Windows Server-Container

Diese verwenden die Firewall der Windows-Hosts (mit Netzwerknamespaces) und VFP

* Standard ausgehender Datenverkehr: Alle zulassen
* Standard eingehender: alle (TCP, UDP, ICMP, IGMP) angeforderten Netzwerk Datenverkehr zulassen
  * Verweigern des gesamten anderen Netzwerk Datenverkehrs nicht aus diesen Protokollen

  >[!NOTE]
  >Vor Windows Server, Version 1709 und Windows 10 Fall Creators Update, lautete die Standardregel "Alle ablehnen". Benutzer, die diese älteren Releases ausführen, können eingehende Zulassungsregeln mit ``docker run -p`` (Port Weiterleitung) erstellen.

### <a name="hyper-v-isolation"></a>Hyper-V-Isolierung

Container, die in Hyper-V-Isolation ausgeführt werden, verfügen über einen eigenen isolierten Kernel und führen daher eine eigene Instanz der Windows-Firewall mit der folgenden Konfiguration aus:

* Standardmäßige allow all in der Windows-Firewall (wird auf dem virtuellen Computer des-Hilfsprogramms ausgeführt) und VFP

![text](media/windows-firewall-containers.png)

### <a name="kubernetes-pods"></a>Kubernetes-Pods

In einem [Kubernetes Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod/)wird zuerst ein Infrastruktur Container erstellt, an den ein Endpunkt angefügt wird. Container, die zum selben Pod gehören, einschließlich Infrastruktur-und workercontainern, nutzen einen gemeinsamen Netzwerk Namespace (gleicher IP-und Port-Speicherplatz).

![text](media/pod-network-compartment.png)

### <a name="customizing-default-port-acls"></a>Anpassen von Standardport-ACLs

Wenn Sie die Standardport-ACLs ändern möchten, lesen Sie zuerst unsere Dokumentation zum Host Netzwerkdienst (der Link wird in Kürze hinzugefügt). Sie müssen Richtlinien in den folgenden Komponenten aktualisieren:

>[!NOTE]
>Für die Hyper-V-Isolation im transparenten und NAT-Modus können Sie die Standardport-ACLs zurzeit nicht neu programmieren. Dies wird durch ein "X" in der Tabelle reflektiert.

| Netzwerktreiber | Windows Server-Container | Hyper-V-Isolierung  |
| -------------- |-------------------------- | ------------------- |
| Transparent | Windows-Firewall | X |
| NAT | Windows-Firewall | X |
| L2Bridge | Beide | VFP |
| L2Tunnel | Beide | VFP |
| Überlagerung  | Beide | VFP |