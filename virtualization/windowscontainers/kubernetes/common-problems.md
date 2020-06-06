---
title: Problembehandlung für Kubernetes
author: gkudra-msft
ms.author: gekudray
ms.date: 11/02/2018
ms.topic: troubleshooting
ms.prod: containers
description: Lösungen für allgemeine Probleme beim Bereitstellen von Kubernetes und beim Beitritt zu Windows-Knoten.
keywords: kubernetes, 1,14, Linux, kompilieren
ms.openlocfilehash: dfb9be5bb5a5dd3507ee7266346634579df503c0
ms.sourcegitcommit: 7f3d98da46c73e428565268683691f383c72221f
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 06/05/2020
ms.locfileid: "84461608"
---
# <a name="troubleshooting-kubernetes"></a>Problembehandlung für Kubernetes #
Diese Seite führt Sie durch mehrere Probleme beim Setup, Networking oder der Bereitstellung von Kubernetes.

> [!tip]
> Schlagen Sie ein FAQ-Element vor, indem Sie eine Veröffentlichungsanforderung an [unser Dokumentationsrepository](https://github.com/MicrosoftDocs/Virtualization-Documentation/) übermitteln.

Diese Seite ist in die folgenden Kategorien unterteilt:
1. [Allgemeine Fragen](#general-questions)
2. [Allgemeine Netzwerkfehler](#common-networking-errors)
3. [Häufige Windows-Fehler](#common-windows-errors)
4. [Allgemeine Kubernetes-Master Fehler](#common-kubernetes-master-errors)

## <a name="general-questions"></a>Allgemeine Fragen ##

### <a name="how-do-i-know-startps1-on-windows-completed-successfully"></a>Gewusst wie wussten Sie, dass Start. ps1 unter Windows erfolgreich abgeschlossen wurde? ###
Sie sollten kubelet, Kube-Proxy und (wenn Sie "Flannel As Your Network Solution" ausgewählt haben) auf dem Knoten ausgestellte cluneld-Host-Agent-Prozesse ausführen, wobei ausgestellte Protokolle in separaten Posh-Fenstern angezeigt werden. Außerdem sollte der Windows-Knoten in Ihrem Kubernetes-Cluster als "bereit" aufgeführt werden.

### <a name="can-i-configure-to-run-all-of-this-in-the-background-instead-of-posh-windows"></a>Kann ich so konfigurieren, dass all dies anstelle von Posh-Fenstern im Hintergrund ausgeführt wird? ###
Ab Kubernetes, Version 1,11, kann kubelet & Kube-Proxy als Native Windows- [Dienste](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services)ausgeführt werden. Sie können auch immer Alternative Dienst-Manager wie " [nssm. exe](https://nssm.cc/) " verwenden, um diese Prozesse (flanneld, kubelet & Kube-Proxy) im Hintergrund für Sie auszuführen. Weitere Informationen finden Sie [unter Windows-Dienste auf Kubernetes](./kube-windows-services.md) .

### <a name="i-have-problems-running-kubernetes-processes-as-windows-services"></a>Ich habe Probleme beim Ausführen von Kubernetes-Prozessen als Windows-Dienste ###
Zur anfänglichen Problembehandlung können Sie die folgenden Flags in [nssm. exe](https://nssm.cc/) verwenden, um stdout und stderr in eine Ausgabedatei umzuleiten:
```
nssm set <Service Name> AppStdout C:\k\mysvc.log
nssm set <Service Name> AppStderr C:\k\mysvc.log
```
Weitere Informationen finden Sie in der offiziellen [nssm-Verwendungs](https://nssm.cc/usage) Dokumentation.

## <a name="common-networking-errors"></a>Allgemeine Netzwerkfehler ##

### <a name="load-balancers-are-plumbed-inconsistently-across-the-cluster-nodes"></a>Lasten Ausgleichs Module werden inkonsistent über die Cluster Knoten verteilt ###
Unter Windows erstellt Kube-Proxy einen HNS Load Balancer für jeden Kubernetes-Dienst im Cluster. In der (Standard) Kube-Proxy-Konfiguration können Knoten in Clustern, die viele (normalerweise 100 +) Load Balancer enthalten, über verfügbare kurzlebige TCP-Ports verfügen (auch bekannt als der dynamische Port Bereich, der standardmäßig die Ports 49152 bis 65535 abdeckt). Dies liegt an der hohen Anzahl der Ports, die für jeden Knoten (nicht-DSR-Lasten Ausgleichs Modul) reserviert sind. Dieses Problem wird möglicherweise durch Fehler in Kube-Proxy, wie z. b.:
```
Policy creation failed: hcnCreateLoadBalancer failed in Win32: The specified port already exists.
```

Benutzer können dieses Problem erkennen, indem Sie das Skript [collectlogs. ps1](https://github.com/microsoft/SDN/blob/master/Kubernetes/windows/debug/collectlogs.ps1) ausführen und die `*portrange.txt` Dateien konsultieren.

Das `CollectLogs.ps1` imitiert auch die Zuordnungs Logik von HNS, um die Verfügbarkeit von Port Pool Zuordnungen im kurzlebigen TCP-Port Bereich zu testen, und meldet Erfolg/Fehler in `reservedports.txt` . Das Skript reserviert 10 Bereiche von 64 TCP-kurzlebigen Ports (zum Emulieren des HNS-Verhaltens), zählt die Erfolgs & Fehlern und gibt dann die zugewiesenen Port Bereiche frei. Eine Erfolgs Nummer kleiner als 10 gibt an, dass der kurzlebige Pool nicht über genügend freien Speicherplatz verfügt. Eine heuristische Zusammenfassung dazu, wie viele 64-Block-Port Reservierungen ungefähr verfügbar sind, werden auch in generiert `reservedports.txt` .

Um dieses Problem zu beheben, können einige Schritte ausgeführt werden:
1.  Für eine permanente Lösung sollte der Kube-Proxy-Lastenausgleich auf den [DSR-Modus](https://techcommunity.microsoft.com/t5/Networking-Blog/Direct-Server-Return-DSR-in-a-nutshell/ba-p/693710)festgelegt werden. Der DSR-Modus ist vollständig implementiert und nur auf einem neueren [Windows Server-Insider-Build 18945](https://blogs.windows.com/windowsexperience/2019/07/30/announcing-windows-server-vnext-insider-preview-build-18945/#o1bs7T2DGPFpf7HM.97) (oder höher) verfügbar.
2. Um dieses Problem zu umgehen, können Benutzer auch die standardmäßige Windows-Konfiguration der kurzlebigen Ports erhöhen, die mithilfe eines Befehls wie verfügbar sind `netsh int ipv4 set dynamicportrange TCP <start_port> <port_count>` . *Warnung:* Das Überschreiben des dynamischen Standard Port Bereichs kann auf andere Prozesse/Dienste auf dem Host folgen, die sich auf verfügbare TCP-Ports aus dem nicht kurzlebigen Bereich stützen, daher sollte dieser Bereich sorgfältig ausgewählt werden.
3. Es gibt eine Verbesserung der Skalierbarkeit für Lasten Ausgleichs Module ohne DSR-Modus, die eine intelligente Port Pool Freigabe verwenden, die für die Veröffentlichung durch ein kumulatives Update in Q1 2020 geplant ist.

### <a name="hostport-publishing-is-not-working"></a>Die hostport Veröffentlichung funktioniert nicht. ###
Stellen Sie sicher, dass die cni-Plug-Ins [v 0.8.6 installieren](https://github.com/containernetworking/plugins/releases/tag/v0.8.6) Release oder höher sind und dass die cni-Konfigurationsdatei über die folgenden Funktionen verfügt, um die hostport Funktion zu verwenden `portMappings` :
```
"capabilities": {
    "portMappings":  true
}
```

### <a name="i-am-seeing-errors-such-as-hnscall-failed-in-win32-the-wrong-diskette-is-in-the-drive"></a>Ich sehe Fehler wie "hnscall failed in Win32: die falsche Diskette befindet sich im Laufwerk". ###
Dieser Fehler kann auftreten, wenn Sie benutzerdefinierte Änderungen an HNS-Objekten vornehmen oder neue Windows Update installieren, die Änderungen an HNS einleiten, ohne alte HNS-Objekte zu zerreißen. Gibt an, dass ein HNS-Objekt, das zuvor vor einem Update erstellt wurde, nicht mit der aktuell installierten HNS-Version kompatibel ist.

Unter Windows Server 2019 (und niedriger) können Benutzer die HNS-Objekte löschen, indem Sie die Datei HNS. Data löschen. 
```
Stop-Service HNS
rm C:\ProgramData\Microsoft\Windows\HNS\HNS.data
Start-Service HNS
```

Benutzer sollten in der Lage sein, alle inkompatiblen HNS-Endpunkte oder Netzwerke direkt zu löschen:
```
hnsdiag list endpoints
hnsdiag delete endpoints <id>
hnsdiag list networks 
hnsdiag delete networks <id>
Restart-Service HNS
```

Benutzer unter Windows Server, Version 1903, können den folgenden Registrierungs Speicherort aufrufen und alle NICs löschen, die mit dem Netzwerknamen beginnen (z. b. `vxlan0` oder `cbr0` ):
```
\\Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\vmsmp\parameters\NicList
```

### <a name="containers-on-my-flannel-host-gw-deployment-on-azure-cannot-reach-the-internet"></a>Container auf meinem Flannel-Host: die GW-Bereitstellung in Azure kann das Internet nicht erreichen. ###
Wenn Sie den Flannel im Host-GW-Modus in Azure bereitstellen, müssen die Pakete über den physischen Azure-Host-Vswitch geleitet werden. Benutzer sollten [benutzerdefinierte Routen](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-udr-overview#user-defined) vom Typ "virtuelles Gerät" für jedes Subnetz programmieren, das einem Knoten zugewiesen ist. Dies kann über die Azure-Portal erfolgen (siehe [hier](https://docs.microsoft.com/en-us/azure/virtual-network/tutorial-create-route-table-portal)ein Beispiel) oder über `az` Azure CLI. Im folgenden finden Sie ein Beispiel für eine UDR mit dem Namen "MyRoute", wobei AZ Commands für einen Knoten mit IP 10.0.0.4 und das entsprechende Pod-Subnetz 10.244.0.0/24 verwendet wird:
```
az network route-table create --resource-group <my_resource_group> --name BridgeRoute 
az network route-table route create  --resource-group <my_resource_group> --address-prefix 10.244.0.0/24 --route-table-name BridgeRoute  --name MyRoute --next-hop-type VirtualAppliance --next-hop-ip-address 10.0.0.4 
```

>[!TIP]
> Wenn Sie Kubernetes in Azure oder IaaS-VMS von anderen cloudanbietern selbst bereitstellen, können Sie stattdessen auch das [Überlagerungs Netzwerk](./network-topologies.md#flannel-in-vxlan-mode) verwenden.

### <a name="my-windows-pods-cannot-ping-external-resources"></a>Meine Windows-Pods können externe Ressourcen nicht pingen ###
Für Windows-Pods gibt es keine ausgehenden Regeln, die heute für das ICMP-Protokoll programmiert sind. TCP/UDP wird jedoch unterstützt. Wenn Sie versuchen, die Konnektivität mit Ressourcen außerhalb des Clusters zu demonstrieren, ersetzen Sie `ping <IP>` durch die entsprechenden `curl <IP>` Befehle.

Wenn weiterhin Probleme auftreten, verdient die Netzwerkkonfiguration in " [cni. conf](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf) " eine gewisse Aufmerksamkeit. Diese statische Datei kann immer bearbeitet werden, die Konfiguration wird auf alle neu erstellten Kubernetes-Ressourcen angewendet.

Warum?
Eine der Kubernetes-Netzwerk Anforderungen (siehe [Kubernetes-Modell](https://kubernetes.io/docs/concepts/cluster-administration/networking/)) ist, dass die Cluster Kommunikation intern ohne NAT erfolgt. Um diese Anforderung zu erfüllen, haben wir eine [Ausnahmeliste](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf#L20) für die gesamte Kommunikation, bei der wir nicht möchten, dass ausgehende NAT stattfindet. Dies bedeutet jedoch auch, dass Sie die externe IP-Adresse ausschließen müssen, die Sie aus der ExceptionList Abfragen möchten. Nur dann wird der Datenverkehr, der von Ihren Windows-Pods stammt, ordnungsgemäß entfernt, um eine Antwort von der Außenwelt zu erhalten. In dieser Hinsicht sollte die ExceptionList in `cni.conf` wie folgt aussehen:
```conf
"ExceptionList": [
  "10.244.0.0/16",  # Cluster subnet
  "10.96.0.0/12",   # Service subnet
  "10.127.130.0/24" # Management (host) subnet
]
```

### <a name="my-windows-node-cannot-access-a-nodeport-service"></a>Mein Windows-Knoten kann nicht auf einen nodeport-Dienst zugreifen. ###
Der lokale nodeport-Zugriff über den Knoten selbst schlägt aufgrund einer Entwurfs Beschränkung auf Windows Server 2019 fehl. Der nodeport-Zugriff funktioniert von anderen Knoten oder externen Clients.

### <a name="my-windows-node-stops-routing-thourgh-nodeports-after-i-scaled-down-my-pods"></a>Mein Windows-Knoten beendet das Routing von "thourgh"-nodeports nach dem Herunterskalieren der Pods ###
Aufgrund einer Entwurfs Beschränkung muss mindestens ein Pod auf dem Windows-Knoten ausgeführt werden, damit die nodeport-Weiterleitung funktioniert.

### <a name="after-some-time-vnics-and-hns-endpoints-of-containers-are-being-deleted"></a>Nach einiger Zeit werden vNICs und HNS-Endpunkte von Containern gelöscht. ###
Dieses Problem kann verursacht werden, wenn der- `hostname-override` Parameter nicht an [Kube-Proxy](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/)übergeben wird. Um dieses Problem zu beheben, müssen Benutzer den Hostnamen wie folgt an den Kube-Proxy übergeben:
```
C:\k\kube-proxy.exe --hostname-override=$(hostname)
```

### <a name="on-flannel-vxlan-mode-my-pods-are-having-connectivity-issues-after-rejoining-the-node"></a>Im Flannel-Modus (vxlan) treten für meine Pods nach dem erneuten beitreten zum Knoten Konnektivitätsprobleme auf. ###
Wenn ein zuvor gelöschter Knoten dem Cluster erneut hinzugefügt wird, versucht flanneld, dem Knoten ein neues Pod-Subnetz zuzuweisen. Benutzer sollten die alten Pod-subnetzkonfigurationsdateien in den folgenden Pfaden entfernen:
```powershell
Remove-Item C:\k\SourceVip.json
Remove-Item C:\k\SourceVipRequest.json
```

### <a name="after-launching-startps1-flanneld-is-stuck-in-waiting-for-the-network-to-be-created"></a>Nach dem Start von "Start. ps1" bleibt flanneld in "warten auf die Erstellung des Netzwerks". ###
Es gibt zahlreiche Berichte zu diesem Problem, die untersucht werden. höchstwahrscheinlich handelt es sich um ein Zeit Steuerungs Problem, wenn die Verwaltungs-IP des Flannel-Netzwerks festgelegt ist. Eine Problem Umgehung besteht darin, einfach "Start. ps1" neu zu starten oder Sie wie folgt manuell neu zu starten
```
PS C:> [Environment]::SetEnvironmentVariable("NODE_NAME", "<Windows_Worker_Hostname>")
PS C:> C:\flannel\flanneld.exe --kubeconfig-file=c:\k\config --iface=<Windows_Worker_Node_IP> --ip-masq=1 --kube-subnet-mgr=1
```

Es gibt auch einen [PR](https://github.com/coreos/flannel/pull/1042) , der dieses Problem derzeit unter Review behandelt.


### <a name="my-windows-pods-cannot-launch-because-of-missing-runflannelsubnetenv"></a>Meine Windows-Pods können aufgrund fehlender/Run/Flannel/Subnet.env nicht gestartet werden. ###
Dies gibt an, dass der Flannel nicht ordnungsgemäß gestartet wurde. Sie können entweder die Datei "flanneld. exe" neu starten, oder Sie können die Dateien manuell von `/run/flannel/subnet.env` auf dem Kubernetes `C:\run\flannel\subnet.env` -Master auf den Windows-workerknoten kopieren und die `FLANNEL_SUBNET` Zeile in das zugewiesene Subnetz ändern. Beispiel: Wenn node Subnetz 10.244.4.1/24 zugewiesen wurde:
```
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.4.1/24
FLANNEL_MTU=1500
FLANNEL_IPMASQ=true
```
Es ist sicherer, dass "flanneld. exe" diese Datei für Sie generiert.


### <a name="pod-to-pod-connectivity-between-hosts-is-broken-on-my-kubernetes-cluster-running-on-vsphere"></a>Pod-zu-Pod-Konnektivität zwischen Hosts ist auf meinem Kubernetes-Cluster, der auf vSphere ausgeführt wird, beschädigt. 
Da sowohl vSphere als auch der Flannel Port 4789 (vxlan-Standardport) für Überlagerungs Netzwerke reserviert, können Pakete am Ende abgefangen werden. Wenn vSphere für Überlagerungs Netzwerke verwendet wird, sollte es für die Verwendung eines anderen Ports konfiguriert werden, um 4789 freizugeben.  


### <a name="my-endpointsips-are-leaking"></a>Meine Endpunkte/IPS sind nicht freigegeben. ###
Es gibt zwei derzeit bekannte Probleme, die dazu führen können, dass Endpunkte nicht mehr vorhanden sind. 
1.  Das erste [bekannte Problem](https://github.com/kubernetes/kubernetes/issues/68511) ist ein Problem in Kubernetes, Version 1,11. Vermeiden Sie die Verwendung der Kubernetes-Version 1.11.0-1.11.2.
2. Das zweite [bekannte Problem](https://github.com/docker/libnetwork/issues/1950) , das zu einem Fehler bei Endpunkten führen kann, ist ein Parallelitäts Problem beim Speichern von Endpunkten. Um die Behebung zu erhalten, müssen Sie docker EE 18,09 oder höher verwenden.

### <a name="my-pods-cannot-launch-due-to-network-failed-to-allocate-for-range-errors"></a>Meine Pods können aufgrund der Fehler "Netzwerk: Fehler beim Zuordnen für den Bereich" nicht gestartet werden. ###
Dies gibt an, dass der IP-Adressraum auf dem Knoten aufgebraucht ist. Um alle kompromittierten [Endpunkte](#my-endpointsips-are-leaking)zu bereinigen, migrieren Sie alle Ressourcen auf betroffenen Knoten, & führen Sie die folgenden Befehle aus:
```
c:\k\stop.ps1
Get-HNSEndpoint | Remove-HNSEndpoint
Remove-Item -Recurse c:\var
```

### <a name="my-windows-node-cannot-access-my-services-using-the-service-ip"></a>Mein Windows-Knoten kann nicht mithilfe der Dienst-IP auf meine Dienste zugreifen. ###
Dies ist eine bekannte Einschränkung für den aktuellen Netzwerkstapel unter Windows. Windows- *Pods* **sind** jedoch in der Lage, auf die Dienst-IP zuzugreifen.

### <a name="no-network-adapter-is-found-when-starting-kubelet"></a>Beim Start von Kubelet wird kein Netzwerkadapter gefunden. ###
Der Windows-Netzwerkstack benötigt einen virtuellen Adapter, damit das Kubernetes-Netzwerk funktioniert. Wenn die folgenden Befehle keine Ergebnisse (in einer Admin-Shell) zurückgeben, ist die Erstellung eines virtuellen Netzwerks &mdash; eine notwendige Voraussetzung, damit Kubelet funktioniert &mdash; fehlgeschlagen.

```powershell
Get-HnsNetwork | ? Name -ieq "cbr0"
Get-NetAdapter | ? Name -Like "vEthernet (Ethernet*"
```

Häufig ist es sinnvoll, den Parameter " [InterfaceName](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/start.ps1#L6) " des Skripts "Start. ps1" zu ändern, in Fällen, in denen der Netzwerkadapter des Hosts nicht "Ethernet" ist. Überprüfen Sie andernfalls die Ausgabe des `start-kubelet.ps1` Skripts, um zu ermitteln, ob bei der Erstellung des virtuellen Netzwerks Fehler vorliegen. 

### <a name="pods-stop-resolving-dns-queries-successfully-after-some-time-alive"></a>Pods stoppen nach einiger Zeit die erfolgreiche Auflösung von DNS-Abfragen. ###
Es gibt ein bekanntes Problem mit der DNS-Zwischenspeicherung im Netzwerk Stapel von Windows Server, Version 1803 und höher, die mitunter zu Fehlern bei DNS-Anforderungen führen können. Um dieses Problem zu umgehen, können Sie die maximale Gültigkeitsdauer Cache Werte mithilfe der folgenden Registrierungsschlüssel auf 0 (null) festlegen:

```Dockerfile
FROM microsoft/windowsservercore:<your-build>
SHELL ["powershell', "-Command", "$ErrorActionPreference = 'Stop';"]
New-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxCacheTtl -Value 0 -Type DWord 
New-ItemPropery -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxNegativeCacheTtl -Value 0 -Type DWord
```

### <a name="i-am-still-seeing-problems-what-should-i-do"></a>Ich sehe weiterhin Probleme. Wie sollte ich vorgehen? ### 
Möglicherweise gibt es weitere vorhandene Einschränkungen auf Ihrem Netzwerk oder auf Hosts, die bestimmte Kommunikationsarten zwischen Knoten verhindern. Stellen Sie Folgendes sicher:
  - Sie haben die ausgewählte [Netzwerktopologie](./network-topologies.md) ordnungsgemäß konfiguriert.
  - Datenverkehr, der offensichtlich von Pods stammt, ist zulässig.
  - HTTP-Datenverkehr ist zulässig, wenn Sie Webdienste bereitstellen
  - Pakete aus unterschiedlichen Protokollen (d.h. ICMP im Vergleich zu TCP/UDP) werden nicht gelöscht.

>[!TIP]
> Weitere Ressourcen für die Selbsthilfe finden Sie auch in der Anleitung zur Problembehandlung für [Windows Kubernetes](https://techcommunity.microsoft.com/t5/Networking-Blog/Troubleshooting-Kubernetes-Networking-on-Windows-Part-1/ba-p/508648).

## <a name="common-windows-errors"></a>Häufige Windows-Fehler ##

### <a name="my-kubernetes-pods-are-stuck-at-containercreating"></a>Meine Kubernetes-Pods bleiben bei „ContainerCreating” hängen. ###
Dieses Problem kann viele Ursachen haben. Eine der häufigsten ist ein falsch konfiguriertes Pause-Image. Dies ist ein High-Level-Symptom des nächsten Problems.


### <a name="when-deploying-docker-containers-keep-restarting"></a>Bei der Bereitstellung werden Docker-Container immer neu gestartet. ###
Überprüfen Sie, ob das Pause-Image mit Ihrer Betriebssystemversion kompatibel ist. Bei den [Anweisungen](./deploying-resources.md) wird davon ausgegangen, dass das Betriebssystem und die Container Version 1803 sind. Wenn Sie eine neuere Version von Windows besitzen, z. B. eine Insider-Version, müssen Sie die Images entsprechend anpassen. Weitere Informationen finden Sie im Microsoft [Docker-Repository](https://hub.docker.com/u/microsoft/) für Images. Unabhängig davon – sowohl Pause-Image-Dockerfile als auch der Beispieldienst erwarten, dass das Image als `:latest` markiert ist.


## <a name="common-kubernetes-master-errors"></a>Allgemeine Kubernetes-Master Fehler ##
Das Debuggen des Kubernetes Master lässt sich in drei Kategorien einteilen (je nach Wahrscheinlichkeit):

  - Es treten Probleme mit den Kubernetes-Systemcontainern auf.
  - Es treten Probleme mit der Funktionsweise der `kubelet` auf.
  - Es treten Probleme mit dem System auf.

Führen Sie `kubectl get pods -n kube-system` durch, um die durch Kubernetes erstellten Pods anzuzeigen. Dies bietet eventuell Einblicke darin, welche abstürzen oder nicht ordnungsgemäß gestartet werden. Führen Sie dann `docker ps -a` aus, um alle unformatierten Container anzuzeigen, die diese Pods unterstützen. Führen Sie abschließend `docker logs [ID]` auf dem(n) Container(n) aus, die vermutlich das Problem verursachen, um die unformatierte Ausgabe der Prozesse anzuzeigen.


### <a name="cannot-connect-to-the-api-server-at-httpsaddressport"></a>Verbindung mit dem API-Server unter `https://[address]:[port]` ist nicht möglich. ###
In den meisten Fällen deutet dieser Fehler auf Probleme mit dem Sicherheitszertifikat hin. Stellen Sie sicher, dass Sie die Konfigurationsdatei ordnungsgemäß generiert haben, dass die darin enthaltenen IP-Adressen denen Ihres Host entsprechen und dass Sie diese in das Verzeichnis kopiert haben, das vom API-Server bereitgestellt wird.

Wenn Sie die [Anweisungen](./creating-a-linux-master.md)befolgen, finden Sie hier folgende gute Möglichkeiten:   
* `~/kube/kubelet/`
* `$HOME/.kube/config`
*  `/etc/kubernetes/admin.conf`

 Beachten Sie andernfalls die Manifest-Datei des API-Servers, um die Einstellungspunkte zu überprüfen.
