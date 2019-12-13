---
title: Netzwerktopologien
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: Unterstützte Netzwerktopologien unter Windows und Linux.
keywords: kubernetes, 1,14, Windows, Getting Started
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 6b0e13258b749ad3dfd5c8349200ca8a54908952
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910310"
---
# <a name="network-solutions"></a>Network Solutions #

Nachdem Sie [einen Kubernetes-Master Knoten eingerichtet](./creating-a-linux-master.md) haben, können Sie eine Netzwerklösung auswählen. Es gibt mehrere Möglichkeiten, das [Subnetz](./getting-started-kubernetes-windows.md#cluster-subnet-def) des virtuellen Clusters über Knoten hinweg Routing fähig zu machen. Wählen Sie eine der folgenden Optionen für Kubernetes unter Windows heute aus:

1. Verwenden Sie ein cni-Plug-in, wie z. b. [Flannel](#flannel-in-vxlan-mode) , um ein Überlagerungs Netzwerk
2. Verwenden Sie ein cni-Plug-in, z. b. den [Flannel](#flannel-in-host-gateway-mode) , um Routen für Sie zu programmieren (verwendet den l2bridge
3. Konfigurieren Sie einen intelligenten [Top-of-Rack-Switch (Tor)](#configuring-a-tor-switch) , um das Subnetz weiterzuleiten.

> [!tip]  
> Es gibt eine vierte Netzwerklösung unter Windows, die Open Vswitch (OVS) und Open Virtual Network (OVN) nutzt. Das dokumentieren dieses Dokuments ist nicht im Gültigkeitsbereich, aber Sie können [diese Anweisungen](https://kubernetes.io/docs/getting-started-guides/windows/#for-3-open-vswitch-ovs-open-virtual-network-ovn-with-overlay) lesen, um es einzurichten.

## <a name="flannel-in-vxlan-mode"></a>Flannel im vxlan-Modus

Der Flannel im vxlan-Modus kann verwendet werden, um ein konfigurierbares virtuelles Überlagerungs Netzwerk einzurichten, das vxlan-Tunnelung zum Weiterleiten von Paketen zwischen Knoten verwendet.

### <a name="prepare-kubernetes-master-for-flannel"></a>Vorbereiten des Kubernetes-Masters für den Flannel
Auf dem [Kubernetes-Master](./creating-a-linux-master.md) in unserem Cluster wird eine geringfügige Vorbereitung empfohlen. Es wird empfohlen, bei Verwendung von Flannel einen überbrückten IPv4-Datenverkehr an iptables-Ketten zu aktivieren. Dies kann mithilfe des folgenden Befehls erreicht werden:

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

###  <a name="download--configure-flannel"></a>Herunterladen & Flannel konfigurieren ###
Das aktuellste Flannel-Manifest herunterladen:

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Zum Aktivieren des vxlan-Netzwerk-Back-Ends sollten Sie zwei Abschnitte ändern:

1. Überprüfen Sie im `net-conf.json` Abschnitt Ihrer `kube-flannel.yml`Folgendes:
 * Das Clustersubnetz (z. b. "10.244.0.0/16") wird wie gewünscht festgelegt.
 * Vni 4096 wird im Back-End festgelegt
 * Port 4789 wird im Back-End festgelegt
2. Ändern Sie im `cni-conf.json` Abschnitt Ihrer `kube-flannel.yml`den Netzwerknamen in `"vxlan0"`.

Nachdem Sie die obigen Schritte ausgeführt haben, sollte Ihr `net-conf.json` wie folgt aussehen:
```json
  net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "vxlan",
        "VNI" : 4096,
        "Port": 4789
      }
    }
```

> [!NOTE]  
> Die vni muss auf 4096 und Port 4789 für den Flannel unter Linux festgelegt werden, damit Sie mit dem Flannel unter Windows zusammenarbeiten kann. Die Unterstützung für andere vnis ist demnächst verfügbar. Eine Erläuterung dieser Felder finden Sie unter [vxlan](https://github.com/coreos/flannel/blob/master/Documentation/backends.md#vxlan) .

Die `cni-conf.json` sollte wie folgt aussehen:
```json
cni-conf.json: |
    {
      "name": "vxlan0",
      "plugins": [
        {
          "type": "flannel",
          "delegate": {
            "hairpinMode": true,
            "isDefaultGateway": true
          }
        },
        {
          "type": "portmap",
          "capabilities": {
            "portMappings": true
          }
        }
      ]
    }
```
> [!tip]  
> Weitere Informationen zu den oben aufgeführten Optionen finden Sie in den offiziellen cni- [Flannel](https://github.com/containernetworking/plugins/tree/master/plugins/meta/flannel#network-configuration-reference)-, [portmap](https://github.com/containernetworking/plugins/tree/master/plugins/meta/portmap#port-mapping-plugin)-und [Bridge](https://github.com/containernetworking/plugins/tree/master/plugins/main/bridge#network-configuration-reference) -Plug-in-Dokumentation für Linux.

### <a name="launch-flannel--validate"></a>Flannel starten & validieren ###
Starten Sie den Flannel mithilfe von:

```bash
kubectl apply -f kube-flannel.yml
```

Da die Flannel-Pods auf Linux basieren, wenden Sie den Linux [nodeselector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) -Patch auf `kube-flannel-ds` daemonset nur auf Linux an. (der "flanneld"-Host-Agent-Prozess wird später beim beitreten in Windows gestartet.)

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]  
> Wenn Knoten nicht auf x86-64 basieren, ersetzen Sie `-amd64` oben durch die Prozessorarchitektur.

Nach einigen Minuten sollten alle Pods als ausgeführt angezeigt werden, wenn das Flannel Pod-Netzwerk bereitgestellt wurde.

```bash
kubectl get pods --all-namespaces
```

![Text](media/kube-master.png)

Für das Flannel-daemonset sollte auch das NoDebug-`beta.kubernetes.io/os=linux` angewendet werden.

```bash
kubectl get ds -n kube-system
```

![Text](media/kube-daemonset.png)

> [!tip]  
> Für die verbleibenden Flannel-DS-*-daemonsets können diese entweder ignoriert oder gelöscht werden, da Sie nicht geplant werden, wenn keine Knoten vorhanden sind, die mit der Prozessorarchitektur übereinstimmen.

> [!tip]  
> Lassen Sie sich hierdurch nicht verwirren. Im folgenden finden Sie ein vollständiges [Beispiel Kube-Flannel. yml](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/overlay/manifests/kube-flannel-example.yml) für Flannel v 0.11.0 mit diesen Schritten, die für die standardmäßige Clustersubnetz`10.244.0.0/16`vorab angewendet werden.

Fahren Sie nach der erfolgreichen Ausführung mit den [nächsten Schritten](#next-steps)fort.

## <a name="flannel-in-host-gateway-mode"></a>Flannel im Host-Gateway-Modus

Neben dem [Flannel-vxlan](#flannel-in-vxlan-mode)besteht eine weitere Option für das Flannel-Netzwerk im *hostgatewaymodus* (Host-GW), der die Programmierung statischer Routen auf jedem Knoten in die Pod-Subnetze eines anderen Knotens unter Verwendung der Host Adresse des Ziel Knotens als nächster Hop umfasst.

### <a name="prepare-kubernetes-master-for-flannel"></a>Vorbereiten des Kubernetes-Masters für den Flannel

Auf dem [Kubernetes-Master](./creating-a-linux-master.md) in unserem Cluster wird eine geringfügige Vorbereitung empfohlen. Es wird empfohlen, bei Verwendung von Flannel einen überbrückten IPv4-Datenverkehr an iptables-Ketten zu aktivieren. Dies kann mithilfe des folgenden Befehls erreicht werden:

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```


###  <a name="download--configure-flannel"></a>Herunterladen & Flannel konfigurieren ###
Das aktuellste Flannel-Manifest herunterladen:

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Es gibt eine Datei, die Sie ändern müssen, um Host-GW-Netzwerke über Windows/Linux hinweg zu aktivieren.

Überprüfen Sie im `net-conf.json` Abschnitt der Kube-Flannel. yml Folgendes:
1. Der Typ des verwendeten Netzwerk-Back-Ends wird auf `host-gw` anstatt auf `vxlan`festgelegt.
2. Das Clustersubnetz (z. b. "10.244.0.0/16") wird wie gewünscht festgelegt.

Nachdem Sie die beiden Schritte ausgeführt haben, sollte Ihr `net-conf.json` wie folgt aussehen:
```json
net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "host-gw"
      }
    }
```

### <a name="launch-flannel--validate"></a>Flannel starten & validieren ###
Starten Sie den Flannel mithilfe von:

```bash
kubectl apply -f kube-flannel.yml
```

Da die Flannel-Pods auf Linux basieren, wenden Sie den Linux [nodeselector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) -Patch auf `kube-flannel-ds` daemonset nur auf Linux an. (der "flanneld"-Host-Agent-Prozess wird später beim beitreten in Windows gestartet.)

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]  
> Wenn Knoten nicht auf x86-64 basieren, ersetzen Sie `-amd64` oben durch die gewünschte Prozessorarchitektur.

Nach einigen Minuten sollten alle Pods als ausgeführt angezeigt werden, wenn das Flannel Pod-Netzwerk bereitgestellt wurde.

```bash
kubectl get pods --all-namespaces
```

![Text](media/kube-master.png)

Für das Flannel-daemonset sollte auch "NoDebug Selector" angewendet werden.

```bash
kubectl get ds -n kube-system
```

![Text](media/kube-daemonset.png)

> [!tip]  
> Für die verbleibenden Flannel-DS-*-daemonsets können diese entweder ignoriert oder gelöscht werden, da Sie nicht geplant werden, wenn keine Knoten vorhanden sind, die mit der Prozessorarchitektur übereinstimmen.

> [!tip]  
> Lassen Sie sich hierdurch nicht verwirren. Im folgenden finden Sie ein vollständiges [Beispiel Kube-Flannel. yml](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/manifests/kube-flannel-example.yml) für Flannel v 0.11.0 mit diesen zwei Schritten, die für das standardmäßige Clustersubnetz `10.244.0.0/16`vorangewendet werden.

Fahren Sie nach der erfolgreichen Ausführung mit den [nächsten Schritten](#next-steps)fort.

## <a name="configuring-a-tor-switch"></a>Konfigurieren eines Tor-Schalters ##
> [!NOTE]
> Sie können diesen Abschnitt überspringen, wenn Sie die Option " [Flannel" als Netzwerklösung](#flannel-in-host-gateway-mode)ausgewählt haben.
Die Konfiguration des Tor-Schalters erfolgt außerhalb der eigentlichen Knoten. Weitere Informationen hierzu finden Sie unter [Official Kubernetes docs](https://kubernetes.io/docs/getting-started-guides/windows/#upstream-l3-routing-topology).


## <a name="next-steps"></a>Nächste Schritte ## 
In diesem Abschnitt wird beschrieben, wie Sie eine Netzwerklösung auswählen und konfigurieren. Nun sind Sie bereit für Schritt 4:

> [!div class="nextstepaction"]
> [Beitreten zu Windows-Mitarbeitern](./joining-windows-workers.md)