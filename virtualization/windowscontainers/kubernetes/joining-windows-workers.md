---
title: Beitreten zu Windows-Knoten
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: how-to
ms.prod: containers
description: Hinzufügen eines Windows-Knotens zu einem Kubernetes-Cluster mit v 1,14.
keywords: kubernetes, 1,14, Windows, Getting Started
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: f808428547a0134e6fea2d9165a4b5cee35b6cfb
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192647"
---
# <a name="joining-windows-server-nodes-to-a-cluster"></a>Hinzufügen von Windows Server-Knoten zu einem Cluster #
Nachdem Sie [einen Kubernetes-Master Knoten eingerichtet](./creating-a-linux-master.md) und [die gewünschte Netzwerklösung ausgewählt](./network-topologies.md)haben, können Sie Windows Server-Knoten zum bilden eines Clusters hinzufügen. Dies erfordert vor [dem beitreten eine Vorbereitung auf den Windows-Knoten](#preparing-a-windows-node) .

## <a name="preparing-a-windows-node"></a>Vorbereiten eines Windows-Knotens ##
> [!NOTE]
> Alle Codeausschnitte in den Windows-Abschnitten müssen in PowerShell mit _erhöhten Rechten_ ausgeführt werden.

### <a name="install-docker-requires-reboot"></a>Installieren von docker (Neustart erforderlich) ###
Kubernetes verwendet [docker](https://www.docker.com/) als Container-Engine, daher müssen wir es installieren. Folgen Sie den [offiziellen Dokumentanweisungen](../manage-docker/configure-docker-daemon.md#install-docker), den [Docker-Anweisungen](https://store.docker.com/editions/enterprise/docker-ee-server-windows) oder gehen Sie folgendermaßen vor:

```powershell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
Install-Package -Name Docker -ProviderName DockerMsftProvider
Restart-Computer -Force
```

Wenn Sie einen Proxyserver verwenden, müssen die folgenden PowerShell-Umgebungsvariablen definiert werden:
```powershell
[Environment]::SetEnvironmentVariable("HTTP_PROXY", "http://proxy.example.com:80/", [EnvironmentVariableTarget]::Machine)
[Environment]::SetEnvironmentVariable("HTTPS_PROXY", "http://proxy.example.com:443/", [EnvironmentVariableTarget]::Machine)
```

Wenn nach dem Neustart der folgende Fehler angezeigt wird:

![Text](media/docker-svc-error.png)

Starten Sie den docker-Dienst dann manuell:

```powershell
Start-Service docker
```

### <a name="create-the-pause-infrastructure-image"></a>Erstellen des Abbilds (Infrastruktur) ###
> [!Important]
> Es ist wichtig, dass in Konflikt stehende Container Images berücksichtigt werden. Wenn das erwartete Tag nicht vorhanden ist, kann dies zu einem nicht `docker pull` kompatiblen Container Image führen, was [Bereitstellungs Probleme](./common-problems.md#when-deploying-docker-containers-keep-restarting) wie den unbegrenzten `ContainerCreating` Status verursacht.

Nachdem Sie nun `docker` installiert haben, müssen Sie ein „pause”-Image vorbereiten, das von Kubernetes zum Vorbereiten der Infrastruktur-Pods verwendet wird. Hierfür sind drei Schritte erforderlich:
  1. [Ziehen des Bilds](#pull-the-image)
  2. [Tagging](#tag-the-image) als Microsoft/NanoServer: Latest
  3. und [Ausführen](#run-the-container)


#### <a name="pull-the-image"></a>Pull des Bilds ####
 Ziehen Sie das Image für Ihre bestimmte Windows-Version. Wenn Sie z. b. Windows Server 2019 ausführen:

 ```powershell
docker pull mcr.microsoft.com/windows/nanoserver:1809
 ```

#### <a name="tag-the-image"></a>Markieren des Bilds ####
Die dockerfiles-Dateien, die Sie später in dieser Anleitung verwenden, suchen nach dem `:latest` imagetag. Markieren Sie das von Ihnen soeben gezogene NanoServer-Abbild wie folgt:

```powershell
docker tag mcr.microsoft.com/windows/nanoserver:1809 microsoft/nanoserver:latest
```

#### <a name="run-the-container"></a>Ausführen des Containers ####
Überprüfen Sie, ob der Container tatsächlich auf dem Computer ausgeführt wird:

```powershell
docker run microsoft/nanoserver:latest
```

Die Ausgabe sollte folgendermaßen aussehen:

![Text](./media/docker-run-sample.png)

> [!tip]
> Wenn Sie den Container nicht ausführen können, finden Sie [entsprechende Informationen unter: übereinstimmendes Container Host Version mit Container Image](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/version-compatibility#matching-container-host-version-with-container-image-versions)


#### <a name="prepare-kubernetes-for-windows-directory"></a>Vorbereiten von Kubernetes für Windows-Verzeichnis ####
Erstellen Sie ein "Kubernetes for Windows"-Verzeichnis, um Kubernetes-Binärdateien sowie Bereitstellungs Skripts und Konfigurationsdateien zu speichern.

```powershell
mkdir c:\k
```

#### <a name="copy-kubernetes-certificate"></a>Kubernetes Zertifikat kopieren ####
Kopieren Sie die Kubernetes-Zertifikat Datei ( `$HOME/.kube/config` ) [vom Master](./creating-a-linux-master.md#collect-cluster-information) in dieses neue `C:\k` Verzeichnis.

> [!tip]
> Sie können Tools wie [xcopy](https://docs.microsoft.com/windows-server/administration/windows-commands/xcopy) oder [WinSCP](https://winscp.net/eng/download.php) verwenden, um die Konfigurationsdatei zwischen Knoten zu übertragen.

#### <a name="download-kubernetes-binaries"></a>Kubernetes-Binärdateien herunterladen ####
Um Kubernetes ausführen zu können, müssen Sie zunächst die `kubectl` `kubelet` Binärdateien, und herunterladen `kube-proxy` . Sie können diese von den Links in der- `CHANGELOG.md` Datei der [neuesten Releases](https://github.com/kubernetes/kubernetes/releases/)herunterladen.
 - Hier sind z. b. die [v 1.14-Knoten Binärdateien](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.14.md#node-binaries).
 - Verwenden Sie ein Tool wie [Expand-Archive](https://docs.microsoft.com/powershell/module/microsoft.powershell.archive/expand-archive?view=powershell-6) , um das Archiv zu extrahieren und die Binärdateien in zu platzieren `C:\k\` .

#### <a name="optional-setup-kubectl-on-windows"></a>Optionale Einrichten von kubectl unter Windows ####
Wenn Sie den Cluster von Windows aus steuern möchten, können Sie dazu den Befehl verwenden `kubectl` . Ändern Sie zunächst `kubectl` `C:\k\` die Umgebungsvariable, um Sie außerhalb des Verzeichnisses verfügbar zu machen `PATH` :

```powershell
$env:Path += ";C:\k"
```

Wenn Sie diese Änderung permanent machen möchten, ändern Sie die Variable im Computerziel:

```powershell
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\k", [EnvironmentVariableTarget]::Machine)
```

Als nächstes überprüfen wir, ob das [Cluster Zertifikat](#copy-kubernetes-certificate) gültig ist. Um den Speicherort `kubectl` für die Konfigurationsdatei festzulegen, können Sie den `--kubeconfig` Parameter übergeben oder die `KUBECONFIG` Umgebungsvariable ändern. Wenn die Konfiguration beispielsweise unter `C:\k\config` gespeichert ist:

```powershell
$env:KUBECONFIG="C:\k\config"
```

Um diese Einstellung für den aktuellen Benutzerbereich permanent zu machen:

```powershell
[Environment]::SetEnvironmentVariable("KUBECONFIG", "C:\k\config", [EnvironmentVariableTarget]::User)
```

Zum Schluss können Sie Folgendes verwenden, um zu überprüfen, ob die Konfiguration ordnungsgemäß erkannt wurde:

```powershell
kubectl config view
```

Falls ein Verbindungsfehler auftritt,

```
Unable to connect to the server: dial tcp [::1]:8080: connectex: No connection could be made because the target machine actively refused it.
```

Sie sollten den kubeconfig-Speicherort überprüfen oder erneut versuchen, ihn zu kopieren.

Wenn keine Fehler angezeigt werden, kann der Knoten jetzt dem Cluster beitreten.

## <a name="joining-the-windows-node"></a>Beitreten zum Windows-Knoten ##
Abhängig von der ausgewählten [Netzwerklösung](./network-topologies.md)haben Sie folgende Möglichkeiten:
1. [Verknüpfen von Windows Server-Knoten mit einem Flannel-Cluster (vxlan oder Host-GW)](#joining-a-flannel-cluster)
2. [Hinzufügen von Windows Server-Knoten zu einem Cluster mit einem Tor-Switch](#joining-a-tor-cluster)

### <a name="joining-a-flannel-cluster"></a>Beitreten zu einem Flannel-Cluster ###
Es gibt eine Sammlung von Flanken Bereitstellungs Skripts für [dieses Microsoft-Repository](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/overlay) , mit dem Sie diesen Knoten dem Cluster hinzufügen können.

Laden Sie den [Flannel start.ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1) Skripts herunter, dessen Inhalt extrahiert werden soll `C:\k` :

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/start.ps1 -o c:\k\start.ps1
```

Wenn Sie den [Windows-Knoten vorbereitet](#preparing-a-windows-node)haben und Ihr `c:\k` Verzeichnis wie unten aussieht, sind Sie bereit, dem Knoten beizutreten.

![Text](./media/flannel-directory.png)

#### <a name="join-node"></a>Knoten beitreten ####
Um den Prozess der Verknüpfung eines Windows-Knotens zu vereinfachen, müssen Sie nur ein einzelnes Windows-Skript ausführen, um `kubelet` `kube-proxy` `flanneld` den Knoten zu starten.

> [!Note]
> [start.ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1) verweist auf [install.ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/install.ps1), bei der zusätzliche Dateien heruntergeladen werden, z `flanneld` . b. die ausführbare Datei und die [dockerfile-Datei für die Infrastruktur Pod](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/Dockerfile) *and install those for you*. Für den Überlagerungs Netzwerkmodus wird die [Firewall](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/helper.psm1#L111) für den lokalen UDP-Port 4789 geöffnet. Möglicherweise werden mehrere PowerShell-Fenster geöffnet oder geschlossen, sowie ein paar Sekunden Netzwerkausfall, während der neue externe Vswitch für das Pod-Netzwerk erstmalig erstellt wird.

```powershell
cd c:\k
.\start.ps1 -ManagementIP <Windows Node IP> -NetworkMode <network mode>  -ClusterCIDR <Cluster CIDR> -ServiceCIDR <Service CIDR> -KubeDnsServiceIP <Kube-dns Service IP> -LogDir <Log directory>
```
# <a name="managementip"></a>[Managementip](#tab/ManagementIP)
Die IP-Adresse, die dem Windows-Knoten zugewiesen ist. Sie können verwenden `ipconfig` , um diese zu suchen.

|  |  |
|---------|---------|
|Parameter     | `-ManagementIP`        |
|Standardwert    | Niederländische Antillen **erforderlich**        |

# <a name="networkmode"></a>[Network Mode](#tab/NetworkMode)
Der Netzwerkmodus `l2bridge` (Flannel Host-GW) oder `overlay` (Flannel vxlan), der als [Netzwerklösung](./network-topologies.md)ausgewählt wurde.

> [!Important]
> `overlay`der Netzwerkmodus (Flannel vxlan) erfordert Kubernetes v 1.14-Binärdateien (oder höher) und [KB4489899](https://support.microsoft.com/help/4489899).

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


# <a name="servicecidr"></a>[Servicecidr](#tab/ServiceCIDR)
Der [dienstsubnetzbereich](./getting-started-kubernetes-windows.md#service-subnet-def).

|  |  |
|---------|---------|
|Parameter     | `-ServiceCIDR`        |
|Standardwert    | `10.96.0.0/12`        |


# <a name="kubednsserviceip"></a>[Kubednsserviceip](#tab/KubeDnsServiceIP)
Die [DNS-IP-Adresse des Kubernetes](./getting-started-kubernetes-windows.md#plan-ip-addressing-for-your-cluster).

|  |  |
|---------|---------|
|Parameter     | `-KubeDnsServiceIP`        |
|Standardwert    | `10.96.0.10`        |


# <a name="interfacename"></a>[Schnittstellenname](#tab/InterfaceName)
Der Name der Netzwerkschnittstelle des Windows-Hosts. Sie können verwenden `ipconfig` , um diese zu suchen.

|  |  |
|---------|---------|
|Parameter     | `-InterfaceName`        |
|Standardwert    | `Ethernet`        |


# <a name="logdir"></a>[LOGDIR](#tab/LogDir)
Das Verzeichnis, in das kubelet-und Kube-Proxy-Protokolle in ihre jeweiligen Ausgabedateien umgeleitet werden.

|  |  |
|---------|---------|
|Parameter     | `-LogDir`        |
|Standardwert    | `C:\k`        |


---

> [!tip]
> Sie haben sich bereits [zuvor](./creating-a-linux-master.md#collect-cluster-information) das Clustersubnetz, das dienstsubnetz und die Kube-DNS-IP aus dem Linux-Master notiert.

Nachdem Sie diese ausgeführt haben, sollten Sie folgende Aktionen ausführen können:
  * Anzeigen von verbundenen Windows-Knoten mithilfe von`kubectl get nodes`
  * Siehe 3 PowerShell-Fenster geöffnet, eine für `kubelet` , eine für `flanneld` und eine weitere für`kube-proxy`
  * Siehe Host-Agent-Prozesse für `flanneld` , `kubelet` und, `kube-proxy` die auf dem Knoten ausgeführt werden.

Wenn Sie erfolgreich sind, fahren Sie mit den [nächsten Schritten](#next-steps)fort.

## <a name="joining-a-tor-cluster"></a>Beitreten zu einem Tor-Cluster ##
> [!NOTE]
> Sie können diesen Abschnitt überspringen, wenn Sie [zuvor](./network-topologies.md#flannel-in-host-gateway-mode)die Option "Flannel" als Netzwerklösung ausgewählt haben.

Befolgen Sie hierzu die Anweisungen zum [Einrichten von Windows Server-Containern auf Kubernetes für die upstreaml3-Routing Topologie](https://kubernetes.io/docs/getting-started-guides/windows/#for-1-upstream-l3-routing-topology-and-2-host-gateway-topology). Dies umfasst auch die Sicherstellung, dass Sie Ihren upstreamrouter so konfigurieren, dass das einem Knoten zugewiesene Pod-CIDR-Präfix der jeweiligen Knoten-IP zugeordnet wird.

Vorausgesetzt, dass der neue Knoten als "bereit" aufgeführt ist `kubectl get nodes` , dass kubelet + Kube-Proxy ausgeführt wird und Sie den Upstream-Tor-Router konfiguriert haben, sind Sie für die nächsten Schritte bereit.

## <a name="next-steps"></a>Nächste Schritte ##
In diesem Abschnitt wird beschrieben, wie Sie Windows-Worker mit unserem Kubernetes-Cluster verknüpfen. Nun sind Sie bereit für Schritt 5:

> [!div class="nextstepaction"]
> [Beitreten zu Linux-Worker](./joining-linux-workers.md)

Wenn Sie noch keine Linux-Worker haben, können Sie mit Schritt 6 fortfahren:

> [!div class="nextstepaction"]
> [Bereitstellen von Kubernetes Ressourcen](./deploying-resources.md)
