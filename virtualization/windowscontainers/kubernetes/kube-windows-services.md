---
title: Ausführen von Kubernetes als Windows-Dienst
author: daschott
ms.author: daschott
ms.date: 02/12/2019
ms.topic: how-to
description: Ausführen von Kubernetes-Komponenten als Windows-Dienste
keywords: kubernetes, 1,14, Windows, Getting Started
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5c18
ms.openlocfilehash: a4f29626ce51714e9313d56cc6558677506e75e5
ms.sourcegitcommit: 186ebcd006eeafb2b51a19787d59914332aad361
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 08/07/2020
ms.locfileid: "87985274"
---
# <a name="kubernetes-components-as-windows-services"></a>Kubernetes-Komponenten als Windows-Dienste

Einige Benutzer möchten möglicherweise Prozesse wie flanneld.exe, kubelet.exe kube-proxy.exe oder andere so konfigurieren, dass Sie als Windows-Dienste ausgeführt werden. Dies führt zu zusätzlichen Fehlern bei der Fehlertoleranz, z. b., wenn die Prozesse bei einem unerwarteten Prozess oder Knoten Absturz automatisch neu gestartet werden.


## <a name="prerequisites"></a>Voraussetzungen
1. Sie haben [nssm.exe](https://nssm.cc/download) in das Verzeichnis heruntergeladen. `c:\k`
2. Sie haben den Knoten zu Ihrem Cluster hinzugefügt und die [install.ps1](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/install.ps1) oder [start.ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1) Skript auf dem Knoten ausgeführt.

## <a name="registering-windows-services"></a>Registrieren von Windows-Diensten
Sie können [ein Beispielskript](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/register-svc.ps1) ausführen, das nssm.exe verwendet, `kubelet` mit dem `kube-proxy` , und `flanneld.exe` als Windows-Dienste im Hintergrund ausgeführt werden:

```
C:\k\register-svc.ps1 -NetworkMode <Network mode> -ManagementIP <Windows Node IP> -ClusterCIDR <Cluster subnet> -KubeDnsServiceIP <Kube-dns Service IP> -LogDir <Directory to place logs>
```

# <a name="managementip"></a>[Managementip](#tab/ManagementIP)
Die IP-Adresse, die dem Windows-Knoten zugewiesen ist. Sie können verwenden `ipconfig` , um diese zu suchen.

|  |  |
|---------|---------|
|Parameter     | `-ManagementIP`        |
|Standardwert    | Niederländische Antillen        |


# <a name="networkmode"></a>[Network Mode](#tab/NetworkMode)
Der Netzwerkmodus `l2bridge` (Flannel Host-GW) oder `overlay` (Flannel vxlan), der als [Netzwerklösung](./network-topologies.md)ausgewählt wurde.

> [!Important]
> `overlay`der Netzwerkmodus (Flannel vxlan) erfordert Kubernetes v 1.14-Binärdateien oder höher.

|  |  |
|---------|---------|
|Parameter     | `-NetworkMode`        |
|Standardwert    | `l2bridge`        |


# <a name="clustercidr"></a>[Clustercidr](#tab/ClusterCIDR)
Der [clustersubnetzbereich](./getting-started-kubernetes-windows.md#cluster-subnet-def).

|  |  |
|---------|---------|
|Parameter     | `-ClusterCIDR`        |
|Standardwert    | `10.244.0.0/16`        |


# <a name="kubednsserviceip"></a>[Kubednsserviceip](#tab/KubeDnsServiceIP)
Die [DNS-IP-Adresse des Kubernetes](./getting-started-kubernetes-windows.md#kube-dns-def).

|  |  |
|---------|---------|
|Parameter     | `-KubeDnsServiceIP`        |
|Standardwert    | `10.96.0.10`        |


# <a name="logdir"></a>[LOGDIR](#tab/LogDir)
Das Verzeichnis, in das kubelet-und Kube-Proxy-Protokolle in ihre jeweiligen Ausgabedateien umgeleitet werden.

|  |  |
|---------|---------|
|Parameter     | `-LogDir`        |
|Standardwert    | `C:\k`        |

---


> [!TIP]
> Sollte etwas schief gehen, lesen Sie den [Abschnitt zur Problem](./common-problems.md#i-have-problems-running-kubernetes-processes-as-windows-services) Behandlung.

## <a name="manual-approach"></a>Manueller Ansatz
Wenn das [obige Skript](#registering-windows-services) , auf das verwiesen wird, nicht für Sie geeignet ist, enthält dieser Abschnitt einige *Beispiel Befehle* , die zum manuellen Registrieren dieser Dienste verwendet werden können.

> [!TIP]
> Weitere Informationen zum Konfigurieren und Ausführen von als Native Windows-Dienste über finden Sie unter [kubelet und Kube-Proxy können jetzt als Windows-Dienste ausgeführt](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services) werden `kubelet` `kube-proxy` `sc` .

### <a name="register-flanneldexe"></a>flanneld.exe registrieren
```
nssm install flanneld C:\flannel\flanneld.exe
nssm set flanneld AppParameters --kubeconfig-file=c:\k\config --iface=<ManagementIP> --ip-masq=1 --kube-subnet-mgr=1
nssm set flanneld AppEnvironmentExtra NODE_NAME=<hostname>
nssm set flanneld AppDirectory C:\flannel
nssm start flanneld
```

### <a name="register-kubeletexe"></a>kubelet.exe registrieren
```
nssm install kubelet C:\k\kubelet.exe
nssm set kubelet AppParameters --hostname-override=<hostname> --v=6 --pod-infra-container-image=kubeletwin/pause --resolv-conf="" --allow-privileged=true --enable-debugging-handlers --cluster-dns=<DNS-service-IP> --cluster-domain=cluster.local --kubeconfig=c:\k\config --hairpin-mode=promiscuous-bridge --image-pull-progress-deadline=20m --cgroups-per-qos=false  --log-dir=<log directory> --logtostderr=false --enforce-node-allocatable="" --network-plugin=cni --cni-bin-dir=c:\k\cni --cni-conf-dir=c:\k\cni\config
nssm set kubelet AppDirectory C:\k
nssm start kubelet
```

### <a name="register-kube-proxyexe-l2bridge--host-gw"></a>Register kube-proxy.exe (l2bridge/Host-GW)
```
nssm install kube-proxy C:\k\kube-proxy.exe
nssm set kube-proxy AppDirectory c:\k
nssm set kube-proxy AppParameters --v=4 --proxy-mode=kernelspace --hostname-override=<hostname>--kubeconfig=c:\k\config --enable-dsr=false --log-dir=<log directory> --logtostderr=false
nssm.exe set kube-proxy AppEnvironmentExtra KUBE_NETWORK=cbr0
nssm set kube-proxy DependOnService kubelet
nssm start kube-proxy
```

### <a name="register-kube-proxyexe-overlay--vxlan"></a>Registrieren kube-proxy.exe (Overlay/vxlan)
```
PS C:\k> nssm install kube-proxy C:\k\kube-proxy.exe
PS C:\k> nssm set kube-proxy AppDirectory c:\k
PS C:\k> nssm set kube-proxy AppParameters --v=4 --proxy-mode=kernelspace --feature-gates="WinOverlay=true" --hostname-override=<hostname> --kubeconfig=c:\k\config --network-name=vxlan0 --source-vip=<source-vip> --enable-dsr=false --log-dir=<log directory> --logtostderr=false
nssm set kube-proxy DependOnService kubelet
nssm start kube-proxy
```