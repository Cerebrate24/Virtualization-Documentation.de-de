---
title: Windows-Containernetzwerk
description: Erweiterte Netzwerke für Windows-Container.
keywords: Docker, Container
author: jmesser81
ms.date: 03/27/2018
ms.topic: how-to
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: aed798cf7444cc8ec67aeb663693911ce20247c6
ms.sourcegitcommit: 186ebcd006eeafb2b51a19787d59914332aad361
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 08/07/2020
ms.locfileid: "87985374"
---
# <a name="advanced-network-options-in-windows"></a>Erweiterte Netzwerkoptionen für Windows

Es werden verschiedene Netzwerktreiberoptionen unterstützt, um die Vorteile der Windows-spezifischen Funktionen und Features zu nutzen.

## <a name="switch-embedded-teaming-with-docker-networks"></a>Switch Embedded Teaming und Docker-Netzwerke

> Gilt für alle Netzwerktreiber

Nutzen Sie [Switch Embedded Teaming](https://docs.microsoft.com/windows-server/virtualization/hyper-v-virtual-switch/RDMA-and-Switch-Embedded-Teaming#a-namebkmksswitchembeddedaswitch-embedded-teaming-set), wenn Sie Containerhost-Netzwerke für die Verwendung von Docker durch Angabe mehrerer Netzwerkadapter (durch Kommas getrennt) mit der `-o com.docker.network.windowsshim.interface`-Option erstellen.

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2", "Ethernet 3" TeamedNet
```

## <a name="set-the-vlan-id-for-a-network"></a>Legen Sie die VLAN-ID für ein Netzwerk fest

> Gilt für transparente und l2bridge-Netzwerktreiber

Wenn Sie eine VLAN-ID für ein Netzwerk festlegen möchten, verwenden Sie die Option `-o com.docker.network.windowsshim.vlanid=<VLAN ID>`für den Befehl `docker network create`. Sie können z. B. folgenden Befehl verwenden, um ein transparentes Netzwerk mit einer ID VLAN 11 zu erstellen:

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.vlanid=11 MyTransparentNetwork
```
Wenn Sie die VLAN-ID für ein Netzwerk festlegen, wird damit die VLAN-Isolation für alle Containerendpunkte festgelegt, die mit dem Netzwerk verbunden sind.

> Stellen Sie sicher, dass sich der Hostnetzwerkadapter (physisch) im Trunk-Modus befindet, damit alle markierten (tagged) Datenpakte vom vSwitch mit dem vNIC-Port (Containerendpunkt) im Zugriffsmodus auf dem richtigen VLAN verarbeitet werden können.

## <a name="specify-outboundnat-policy-for-a-network"></a>Angeben der outboundnat-Richtlinie für ein Netzwerk

> Gilt für l2bridge-Netzwerke

Wenn Sie ein `l2bridge` Container Netzwerk mithilfe von erstellen `docker network create` , wird für Container Endpunkte normalerweise nicht die Richtlinie HNS-outboundnat angewendet, was dazu führt, dass Container die Außenwelt nicht erreichen können. Wenn Sie ein Netzwerk erstellen, können Sie die `-o com.docker.network.windowsshim.enable_outboundnat=<true|false>` Option zum Anwenden der outboundnat HNS-Richtlinie verwenden, um Containern den Zugriff auf die Außenwelt zu gestatten:

```
C:\> docker network create -d l2bridge -o com.docker.network.windowsshim.enable_outboundnat=true MyL2BridgeNetwork
```

Wenn eine Gruppe von Zielen (z. b. eine Container-zu-Container-Konnektivität erforderlich) für den Speicherort ist, an dem wir nicht möchten, müssen wir auch eine ExceptionList angeben:

```
C:\> docker network create -d l2bridge -o com.docker.network.windowsshim.enable_outboundnat=true -o com.docker.network.windowsshim.outboundnat_exceptions=10.244.10.0/24
```

## <a name="specify-the-name-of-a-network-to-the-hns-service"></a>Festlegen des Netzwerknamens für den HNS-Dienst

> Gilt für alle Netzwerktreiber

Wenn Sie mit `docker network create` ein Containernetzwerk erstellen, wird der Name des Netzwerks, den Sie bereitstellen, normalerweise vom Docker-Dienst verwendet, aber nicht vom HNS-Dienst. Wenn Sie ein Netzwerk erstellen, können Sie den Namen, den es vom HNS-Dienst erhält, mit der Option `-o com.docker.network.windowsshim.networkname=<network name>` in der Anweisung `docker network create` angeben. Beispielsweise können Sie den folgenden Befehl verwenden, um ein transparentes Netzwerk mit einem Namen zu erstellen, den der HNS-Dienst verwenden soll:

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.networkname=MyTransparentNetwork MyTransparentNetwork
```

## <a name="bind-a-network-to-a-specific-network-interface"></a>Binden eines Netzwerks an eine bestimmte Netzwerkschnittstelle

> Gilt für alle Netzwerktreiber mit Ausnahme von "NAT"

Verwenden Sie zum Binden eines Netzwerks (angeschlossen über den virtuellen Hyper-V-Switch) an eine bestimmte Netzwerkschnittstelle die Option `-o com.docker.network.windowsshim.interface=<Interface>` für den Befehl `docker network create`. Sie können z. B. die folgende Anweisung verwenden, um ein transparentes Netzwerk zu erstellen, das an die Netzwerkschnittstelle „Ethernet 2“ angeschlossen ist:

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2" TransparentNet2
```

> Hinweis: Der Wert für *com.docker.network.windowsshim.interface* ist der *Name* des Netzwerkadapters und kann mit folgender Anweisung gefunden werden:

```
PS C:\> Get-NetAdapter
```

## <a name="specify-the-dns-suffix-andor-the-dns-servers-of-a-network"></a>Angeben von DNS-Suffix und/oder der DNS-Server eines Netzwerks

> Gilt für alle Netzwerktreiber

Verwenden Sie die Option `-o com.docker.network.windowsshim.dnssuffix=<DNS SUFFIX>`, um das DNS-Suffix eines Netzwerks anzugeben, und die Option `-o com.docker.network.windowsshim.dnsservers=<DNS SERVER/S>` zur Angabe der DNS-Server eines Netzwerks. Beispielsweise können Sie den folgenden Befehl verwenden, um das DNS-Suffix eines Netzwerks auf „abc.com“ und die DNS-Server eines Netzwerks auf 4.4.4.4 und 8.8.8.8 festzulegen:

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.dnssuffix=abc.com -o com.docker.network.windowsshim.dnsservers=4.4.4.4,8.8.8.8 MyTransparentNetwork
```

## <a name="vfp"></a>VFP

Weitere Informationen finden Sie in [diesem Artikel](https://www.microsoft.com/research/project/azure-virtual-filtering-platform/).

## <a name="tips--insights"></a>Tipps und Einblicke
Hier ist eine Liste mit nützlichen Tipps und Einblicken, die von den häufig von uns gehörten Fragen der Community über Windows-Container-Netzwerke inspiriert sind...

#### <a name="hns-requires-that-ipv6-is-enabled-on-container-host-machines"></a>HNS erfordert, dass IPv6 auf den Containerhostgeräten aktiviert ist
Als Teil des [KB4015217](https://support.microsoft.com/help/4015217/windows-10-update-kb4015217) ist es für HNS erforderlich, dass IPv6 auf den Windows-Containerhosts aktiviert ist. Wenn wie unten beschrieben ein Fehler auftritt, ist IPv6 wahrscheinlich auf dem Hostcomputer deaktiviert.
```
docker: Error response from daemon: container e15d99c06e312302f4d23747f2dfda4b11b92d488e8c5b53ab5e4331fd80636d encountered an error during CreateContainer: failure in a Windows system call: Element not found.
```
Wir arbeiten an Plattform Änderungen, um dieses Problem automatisch zu erkennen/zu vermeiden. Derzeit kann die folgende Problemumgehung verwendet werden, um sicherzustellen, dass IPv6 auf dem Hostcomputer aktiviert ist:

```
C:\> reg delete HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip6\Parameters  /v DisabledComponents  /f
```


#### <a name="linux-containers-on-windows"></a>Linux-Container unter Windows

**Neu:** Wir arbeiten daran, Linux-und Windows-Container parallel ohne den virtuellen Computer " _Moby Linux_" auszuführen. Weitere Informationen finden Sie [in diesem Blogbeitrag zu Linux-Containern unter Windows (lkuh)](https://blog.docker.com/2017/11/docker-for-windows-17-11/) . Hier finden [Sie](https://docs.microsoft.com/virtualization/windowscontainers/quick-start/quick-start-windows-10-linux)Informationen zu den ersten Schritten.
> Hinweis: lkuh ist veraltet für den virtuellen Computer "Moby Linux" und verwendet den standardmäßigen internen Vswitch "NAT".

#### <a name="moby-linux-vms-use-dockernat-switch-with-docker-for-windows-a-product-of-docker-ce"></a>Moby-Linux-VMs verwenden den dockernat-Switch mit docker für Windows (ein Produkt von [docker CE](https://www.docker.com/community-edition)).

Docker für Windows (der Windows-Treiber für das docker CE-Modul) unter Windows 10 verwendet einen internen Vswitch mit dem Namen "dockernat", um die Verbindung mit den virtuellen Computern mit dem Container "-Host" herzustellen. Entwickler, die unter Windows mit der Verwendung von Moby-Linux-VMs arbeiten, sollten beachten, dass Ihre Hosts den dockernat-Vswitch verwenden und nicht den "NAT"-Vswitch, der vom HNS-Dienst erstellt wird (der Standard Switch, der für Windows-Container verwendet wird).



#### <a name="to-use-dhcp-for-ip-assignment-on-a-virtual-container-host-enable-macaddressspoofing"></a>Aktivieren Sie MACAddressSpoofing, um DHCP für die IP-Adresszuweisung auf einem virtuellen Containerhost zu verwenden.

Wenn der Containerhost virtualisiert ist und Sie DHCP für die IP-Zuweisung verwenden möchten, müssen Sie MACAddressSpoofing im Netzwerkadapter der virtuellen Computer aktivieren. Andernfalls blockiert der Hyper-V-Host Netzwerkdatenverkehr von den Containern auf dem virtuellen Computer mit mehreren MAC-Adressen. Aktivieren Sie MACAddressSpoofing mit folgendem PowerShell-Befehl:
```
PS C:\> Get-VMNetworkAdapter -VMName ContainerHostVM | Set-VMNetworkAdapter -MacAddressSpoofing On
```
Wenn VMware als Hypervisor ausgeführt wird, müssen Sie hierfür den promisken Modus aktivieren. Details finden Sie [hier.](https://kb.vmware.com/s/article/1004099)


#### <a name="creating-multiple-transparent-networks-on-a-single-container-host"></a>Erstellen mehrerer transparenter Netzwerke auf einem einzelnen Containerhost
Wenn Sie mehr als ein transparentes Netzwerk erstellen möchten, müssen Sie angeben, an welchen (virtuellen) Netzwerkadapter der Hyper-V Virtual Switch gebunden werden soll. Um die Benutzeroberfläche für ein Netzwerk anzugeben, verwenden Sie folgende Syntax:
```
# General syntax:
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface=<INTERFACE NAME> <NETWORK NAME>

# Example:
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2" myTransparent2
```

#### <a name="remember-to-specify---subnet-and---gateway-when-using-static-ip-assignment"></a>Denken Sie daran, bei der Verwendung einer statischen IP-Zuordnung *‑‑Subnetz* und *‑‑Gateway* anzugeben
Wenn Sie die statische IP-Zuweisung verwenden, müssen Sie zunächst sicherstellen, dass die Parameter *--subnet* und *--gateway* beim Erstellen des Netzwerks festgelegt sind. Subnetz- und Gateway-IP-Adresse sollten den Netzwerkeinstellungen für den Containerhost (d. h. für das physische Netzwerk) entsprechen. Hier sehen Sie, wie Sie z. B., wie Sie ein transparentes Netzwerk erstellen und dann einen Endpunkt auf diesem Netzwerk mithilfe der statischen IP-Adresszuweisung ausführen können:

```
# Example: Create a transparent network using static IP assignment
# A network create command for a transparent container network corresponding to the physical network with IP prefix 10.123.174.0/23
C:\> docker network create -d transparent --subnet=10.123.174.0/23 --gateway=10.123.174.1 MyTransparentNet
# Run a container attached to MyTransparentNet
C:\> docker run -it --network=MyTransparentNet --ip=10.123.174.105 windowsservercore cmd
```

#### <a name="dhcp-ip-assignment-not-supported-with-l2bridge-networks"></a>Die DHCP-IP-Adresszuweisung wird nicht von L2Bridge-Netzwerken unterstützt
Es wird nur eine statische IP-Adresszuweisung mit Container-Netzwerken unterstützt, die mithilfe des l2bridge-Treibers erstellt wurden. Wie bereits erwähn: denken Sie daran, die *‑‑Subnetz* und *‑‑Gateway*-Parameter zu verwenden, um ein Netzwerk zu erstellen, das für statische IP-Adresszuweisungen konfiguriert ist.

#### <a name="networks-that-leverage-external-vswitch-must-each-have-their-own-network-adapter"></a>Netzwerke, die externe vSwitches nutzen müssen jeweils über eigene Netzwerkadapter verfügen.
Wenn mehrere Netzwerke, die einen externen vSwitch für die Konnektivität verwenden (z. B. Transparent, L2-Brücke, L2 Transparent), auf dem gleichen Containerhost erstellt werden, muss jedes über einen eigenen Netzwerkadapter verfügen.

#### <a name="ip-assignment-on-stopped-vs-running-containers"></a>IP-Adresszuweisung auf Containern im angehaltenen Zustand im Vergleich zu ausgeführten Containern
Die statische IP-Zuweisung erfolgt direkt im Netzwerkadapter des Containers und darf nur dann durchgeführt werden, wenn sich der Container im Zustand BEENDET befindet. Das Hinzufügen von Containernetzwerkadaptern im laufenden Betrieb (Hot-Add) sowie Änderungen am Netzwerkstapel werden während der Containerausführung nicht unterstützt (auf Windows Server 2016).

#### <a name="existing-vswitch-not-visible-to-docker-can-block-transparent-network-creation"></a>Ein bereits vorhandener vSwitch (nicht für Docker sichtbar) kann die Erstellung eines transparenten Netzwerks blockieren
Sollten Fehler beim Erstellen eines transparenten Netzwerks auftreten, ist es möglich, dass ein externer vSwitch in Ihrem System existiert, der nicht automatisch von Docker erkannt wurde, und der verhindert, dass das transparente Netzwerk an den externen Netzwerkadapter Ihres Containerhosts gebunden wird.

Wenn Sie ein transparentes Netzwerk erstellen, erstellt Docker einen externen vSwitch für dieses und versucht dann den Switch mit einem (externen) Netzwerkadapter zu verbinden. Der Adapter kann ein VM-Netzwerkadapter oder der physische Netzwerkadapter sein. Wenn bereits ein vSwitch auf dem Containerhost erstellt wurde, *und von Docker erkannt wird*, wird die Docker-Engine unter Windows diesen Switch verwenden, anstatt einen neuen zu erstellen. Wenn jedoch der vSwitch Out-of-Band erstellt wurde (z.B. auf dem Containerhost mithilfe des Hyper-V-Managers oder PowerShell) und noch nicht von Docker erkannt wird, versucht die Docker-Engine unter Windows einen neuen vSwitch zu erstellen, kann den neuen Switch dann jedoch nicht mit dem externen Netzwerkadapter des Containerhosts verbinden (da der Netzwerkadapter bereits mit dem Switch verbunden ist, der Out-of-Band erstellt wurde).

Das Problem taucht z.B. auf, wenn Sie zuerst einen neuen vSwitch auf Ihrem Host erstellen würden, während der Docker-Dienst ausgeführt wird, und dann versuchen würden, ein transparentes Netzwerk zu erstellen. In diesem Fall würde Docker den von Ihnen erstellen Switch nicht erkennen und würde einen neuen vSwitch für das transparente Netzwerk erstellen.

Es gibt drei Ansätze zum Lösen dieses Problems:

* Sie können natürlich den vSwitch löschen, der Out-of-Band erstellt wurde. Daraufhin kann Docker einen neuen vSwitch erstellen und ihn dann ohne Probleme mit dem Netzwerkadapter des Hosts verbinden. Bevor Sie sich für diesen Ansatz entscheiden, stellen Sie sicher dass Ihr Out-of-Band-vSwitch nicht von anderen Diensten verwendet wird (z.B. Hyper-V).
* Wenn Sie sich alternativ dafür entscheiden, einen externen Out-of-Band-vSwitch zu verwenden, starten Sie Docker und die HNS-Dienste neu, um *den Switch für Docker erkennbar zu machen.*
```
PS C:\> restart-service hns
PS C:\> restart-service docker
```
* Eine andere Möglichkeit ist die Verwendung der Option „-o com.docker.network.windowsshim.interface“, um den externen vSwitch des transparenten Netzworks an einen bestimmten Netzwerkadapter zu binden, der noch nicht auf dem Containerhost in Gebrauch ist (z.B. ein anderer Netzwerkadapter als der, der vom vSwitch verwendet wird, der Out-of-Band erstellt wurde). Die Option "-o" wird weiter unten im Abschnitt [Erstellen mehrerer transparenter Netzwerke auf einem einzelnen Container Host](advanced.md#creating-multiple-transparent-networks-on-a-single-container-host) in diesem Dokument beschrieben.


## <a name="windows-server-2016-work-arounds"></a>Windows Server 2016-Problemumgehungen

Obwohl wir auch weiterhin neue Funktionen hinzufügen und die Entwicklung vorantreiben, können einige dieser Funktionen nicht auf ältere Plattformen rückportiert werden. Der beste Aktionsplan ist das neueste Updates für Windows 10 und Windows Server.  Im folgenden Abschnitt werden einige Problemumgehungen und die Vorsichtsmaßnahmen erklärt, die für ältere Versionen von Windows 10 (d. h. vor dem 1704 Creators Update) und Windows Server 2016 gelten.

### <a name="multiple-nat-networks-on-ws2016-container-host"></a>Mehrere NAT-Netzwerke auf WS2016-Containerhost

Die Partitionierung für neue NAT-Netzwerke muss unter dem größeren internen NAT-Netzwerkpräfix erfolgen. Das Präfix kann durch die Ausführung des folgenden PowerShell-Befehls gefunden werden und bezieht sich auf das Feld „InternalIPInterfaceAddressPrefix“.

```
PS C:\> Get-NetNAT
```

Angenommen, das interne NAT-Netzwerkpräfix des Hosts ist 172.16.0.0/16. In diesem Fall kann Docker verwendet werden, um zusätzliche NAT-Netzwerke zu erstellen *solange sie eine Teilmenge des Präfix 172.16.0.0/16 sind.* Beispielsweise können zwei NAT-Netzwerke mit den IP-Präfixen 172.16.1.0/24 (Gateway, 172.16.1.1) und 172.16.2.0/24 (Gateway, 172.16.2.1) erstellt werden.

```
C:\> docker network create -d nat --subnet=172.16.1.0/24 --gateway=172.16.1.1 CustomNat1
C:\> docker network create -d nat --subnet=172.16.2.0/24 --gateway=172.16.1.1 CustomNat2
```

Die neu erstellten Netzwerke können mithilfe folgender Optionen aufgelistet werden:
```
C:\> docker network ls
```

### <a name="docker-compose"></a>Docker Compose

[Docker Compose](https://docs.docker.com/compose/overview/) kann verwendet werden, um Containernetzwerke neben den Containern/Diensten zu definieren und konfigurieren, die diese Netzwerke verwenden werden. Der Compose „Netzwerke“-Schlüssel wird als Schlüssel der obersten Ebene verwendet, um die Netzwerke zu definieren, mit denen die Container verbunden werden. Die folgende Syntax definiert beispielsweise das bereits vorhandene NAT-Netzwerk, das von Docker als „Standardnetzwerk“ für alle Container/Dienste erstellt wurde, die in einer bestimmten Compose-Datei definiert sind.

```
networks:
 default:
  external:
   name: "nat"
```

Die folgende Syntax kann auf ähnliche Weise zum Definieren eines benutzerdefinierten NAT-Netzwerks verwendet werden.

> Hinweis: Das „benutzerdefinierte NAT-Netzwerk“, das im nachfolgenden Beispiel definiert ist, ist als Partition des bereits vorhandenen NAT-internen Präfixes des Containerhosts definiert. Weiteren Kontext finden Sie im Abschnitt „Mehrere NAT-Netzwerke“ weiter oben.

```
networks:
  default:
    driver: nat
    ipam:
      driver: default
      config:
      - subnet: 172.16.3.0/24
```

Weitere Informationen über das Definieren/Konfigurieren von Containernetzwerken mit Docker Compose finden Sie unter [Compose File reference (Referenz zur Compose-Datei)](https://docs.docker.com/compose/compose-file/).
