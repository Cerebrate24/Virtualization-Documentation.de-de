---
title: Erste Schritte mit dem Schwarmmodus
description: Initialisieren einen Schwarmclusters, Erstellen eines Überlagerungsnetzwerks und Zuordnen eines Diensts zum Netzwerk
keywords: Docker, Container, Schwarm, Orchestrierung
author: kallie-b
ms.date: 02/9/2017
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 5ceb9626-7c48-4d42-81f8-9c936595ad85
ms.openlocfilehash: 560e9ffc92728628268d7d557b8fa8428316c8ec
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909680"
---
# <a name="getting-started-with-swarm-mode"></a>Erste Schritte mit dem Schwarmmodus 

## <a name="what-is-swarm-mode"></a>Was ist der Schwarmmodus?
Der Schwarmmodus ist ein Docker-Feature, das Funktionen zur Orchestrierung von Containern bereitstellt, beispielsweise das systemeigene Clustering von Docker-Hosts und die Planung der Containerarbeitslasten. Eine Gruppe von Docker-Hosts bildet einen „Schwarmcluster“, wenn ihre Docker-Computer zusammen im „Schwarmmodus“ arbeiten. Zusätzliche Informationen zum Schwarmmodus finden auf der [Website für die Docker-Dokumentation](https://docs.docker.com/engine/swarm/).

## <a name="manager-nodes-and-worker-nodes"></a>Verwaltungsknoten und Arbeitsknoten
Ein Schwarm besteht aus zwei Arten von Containerhosts: *Verwaltungsknoten* und *Arbeitsknoten*. Jeder Schwarm wird über einen Verwaltungsknoten initialisiert, und alle Docker-CLI-Befehle zur Steuerung und Überwachung eines Schwarms müssen auf einem seiner Verwaltungsknoten ausgeführt werden. Verwaltungsknoten können als „Wächter“ für den Schwarmstatus betrachtet werden. Sie bilden eine Konsensgruppe, die den Zustand der Dienste überwacht, die im Schwarm ausgeführt werden, und die dafür sorgt, dass der tatsächliche Zustand des Schwarms immer mit seinem beabsichtigten, vom Entwickler oder Admin definierten Zustand übereinstimmt. 

>[!NOTE]
>Jeder Schwarm kann mehrere Verwaltungsknoten besitzen, muss aber über *mindestens einen* verfügen. 

Arbeitsknoten werden vom Docker-Schwarm über Verwaltungsknoten orchestriert. Um einem Schwarm beizutreten, muss ein Arbeitsknoten ein „Beitrittstoken“ verwenden, das vom Verwaltungsknoten bei der Initialisierung des Schwarms generiert wurde. Arbeitsknoten erhalten einfach nur Aufgaben von Verwaltungsknoten und führen sie aus, sodass sie keine Kenntnis vom Schwarmzustand besitzen (und benötigen).

## <a name="swarm-mode-system-requirements"></a>Systemvoraussetzungen für den Schwarmmodus

Mindestens ein physisches oder virtuelles Computersystem (um die volle Funktionalität des Schwarms zu nutzen, werden mindestens zwei Knoten empfohlen), auf dem das **Windows 10 Creators Update** oder **Windows Server 2016** *mit allen aktuellen Updates\** ausgeführt wird und das als Containerhost eingerichtet ist. (Weitere Details über erste Schritte mit Docker-Containern unter Windows 10 finden Sie im Thema [Windows-Container unter Windows 10](https://docs.microsoft.com/virtualization/windowscontainers/quick-start/quick-start-windows-10) oder [Windows-Container unter Windows Server](https://docs.microsoft.com/virtualization/windowscontainers/quick-start/quick-start-windows-server)).

\***Hinweis**: Docker Swarm unter Windows Server 2016 erfordert [KB4015217](https://support.microsoft.com/help/4015217/windows-10-update-kb4015217)

**Docker-Modul v1.13.0 oder höher**

Offene Ports: Die folgenden Ports müssen auf jedem Host verfügbar sein. Auf einigen Systemen sind diese Ports standardmäßig geöffnet.
- TCP-Port 2377 für die Kommunikation zur Clusterverwaltung
- TCP- und UDP-Port 7946 für die Kommunikation zwischen Knoten
- UDP-Port 4789 für Datenverkehr im Überlagerungsnetzwerk

## <a name="initializing-a-swarm-cluster"></a>Initialisieren eines Schwarmclusters

Um einen Schwarm zu initialisieren, führen Sie einfach den folgenden Befehl auf einem der Containerhosts aus (ersetzen Sie dabei \<HOSTIPADDRESS\> durch die lokale IPv4-Adresse des Hostcomputers):

```
# Initialize a swarm 
C:\> docker swarm init --advertise-addr=<HOSTIPADDRESS> --listen-addr <HOSTIPADDRESS>:2377
```
Wenn dieser Befehl auf einem Containerhost ausgeführt wird, beginnt das Docker-Modul auf diesem Host als Verwaltungsknoten im Schwarmmodus zu arbeiten.

## <a name="adding-nodes-to-a-swarm"></a>Hinzufügen von Knoten zu einem Schwarm

Um den Schwarmmodus und die Features des Überlagerungsmodus zu nutzen, sind *nicht* mehrere Knoten erforderlich. Alle Schwarm- und Überlagerungsfeatures können auf einem einzelnen Host verwendet werden, der im Schwarmmodus arbeitet. (Dabei kann es sich beispielsweise um einen Verwaltungsknoten handeln, auf dem der Schwarmmodus mit dem Befehl `docker swarm init` initialisiert wurde.)

### <a name="adding-workers-to-a-swarm"></a>Hinzufügen von Arbeitsknoten zu einem Schwarm

Nachdem ein Schwarm von einem Verwaltungsknoten initialisiert worden ist, können andere Hosts dem Schwarm mit folgendem einfachen Befehl als Arbeitsknoten hinzugefügt werden:

```
C:\> docker swarm join --token <WORKERJOINTOKEN> <MANAGERIPADDRESS>
```

Hierbei ist \<MANAGERIPADDRESS\> die lokale IP-Adresse eines Verwaltungsknotens im Schwarm und \<WORKERJOINTOKEN\> das Beitrittstoken für den Arbeitsknoten, das als Ausgabe des auf dem Verwaltungsknoten ausgeführten Befehls `docker swarm init` bereitgestellt wird. Das Beitrittstoken kann auch mit einem der folgenden Befehle abgerufen werden, wenn der Befehl auf dem Verwaltungsknoten ausgeführt wird, nachdem der Schwarm initialisiert wurde:

```
# Get the full command required to join a worker node to the swarm
C:\> docker swarm join-token worker

# Get only the join-token needed to join a worker node to the swarm
C:\> docker swarm join-token worker -q
```

### <a name="adding-managers-to-a-swarm"></a>Hinzufügen von Verwaltungsknoten zu einem Schwarm
Zusätzliche Verwaltungsknoten können einem Schwarmcluster mit dem folgenden Befehl hinzugefügt werden:

```
C:\> docker swarm join --token <MANAGERJOINTOKEN> <MANAGERIPADDRESS>
```

Auch in diesem Fall ist \<MANAGERIPADDRESS\> die lokale IP-Adresse eines Verwaltungsknotens im Schwarm. Das Beitrittstoken \<MANAGERJOINTOKEN\> ist ein Token eines *Verwaltungsknotens* im Schwarm, das durch die Ausführung eines der folgenden Befehle auf einem Verwaltungsknoten abgerufen werden kann:

```
# Get the full command required to join a **manager** node to the swarm
C:\> docker swarm join-token manager

# Get only the join-token needed to join a **manager** node to the swarm
C:\> docker swarm join-token manager -q
```

## <a name="creating-an-overlay-network"></a>Erstellen eines Überlagerungsnetzwerks

Nach ein Schwarmcluster konfiguriert wurde, können im Schwarm Überlagerungsnetzwerke erstellt werden. Ein Überlagerungsnetzwerk kann durch Ausführen des folgenden Befehls auf einem Schwarmverwaltungsknoten erstellt werden:

```
# Create an overlay network 
C:\> docker network create --driver=overlay <NETWORKNAME>
```

Hierbei ist \<NETWORKNAME\> der Name, den Sie Ihrem Netzwerk zuweisen möchten.

## <a name="deploying-services-to-a-swarm"></a>Bereitstellen von Diensten für einen Schwarm
Nachdem ein Überlagerungsnetzwerk eingerichtet wurde, können Dienste erstellt und dem Netzwerk angeschlossen werden. Ein Service wird mit folgender Syntax erstellt:

```
# Deploy a service to the swarm
C:\> docker service create --name=<SERVICENAME> --endpoint-mode dnsrr --network=<NETWORKNAME> <CONTAINERIMAGE> [COMMAND] [ARGS…]
```

Hierbei ist \<SERVICENAME\> der Name, den Sie dem Dienst zuweisen möchten. Mit diesem Namen referenzieren Sie den Dienst über die Dienstermittlung (die den systemeigenen DNS-Server von Docker verwendet). \<NETWORKNAME\> ist der Name des Netzwerks (z. B. „myOverlayNet“), mit dem Sie diesem Dienst verbinden möchten. \<CONTAINERIMAGE\> ist der Name des Containerimages, das den Dienst definiert.

>[!NOTE]
>Das zweite Argument für diesen Befehl, `--endpoint-mode dnsrr`, ist erforderlich, um das Docker-Modul darüber zu informieren, dass die DNS-Richtlinie Roundrobin verwendet wird, um den Netzwerkdatenverkehr gleichmäßig auf die Containerendpunkte zu verteilen. Derzeit ist DNS-Roundrobin die einzige Lastenausgleichsstrategie, die unter Windows Server 2016 unterstützt wird.[Routing-Mesh](https://docs.docker.com/engine/swarm/ingress/) für Windows-Docker-Hosts wird unter Windows Server 2019 (und höher) unterstützt, aber nicht unter Windows Server 2016. Benutzer, die schon jetzt eine alternative Lastenausgleichsstrategie unter Windows Server 2016 wünschen, können ein externes Lastenausgleichssystem (z. B. NGINX) einrichten und den [publish-Portmodus](https://docs.docker.com/engine/reference/commandline/service_create/#/publish-service-ports-externally-to-the-swarm--p---publish) des Schwarms verwenden, um Containerhostports für den Ausgleich des Datenverkehrs verfügbar zu machen.

## <a name="scaling-a-service"></a>Skalieren eines Diensts
Nachdem ein Dienst für einen Schwarmcluster bereitgestellt worden ist, werden die Containerinstanzen, aus denen dieser Dienst gebildet wird, im Cluster bereitgestellt. Standardmäßig beruht ein Dienst auf nur einer Containerinstanz, „Replikat“ oder „Task“ genannt. Jedoch kann ein Dienst mit mehreren Tasks erstellt werden – entweder mit der Option `--replicas` in der `docker service create`-Anweisung oder durch Skalieren des Diensts nach der Erstellung.

Die Skalierbarkeit von Diensten ist ein wichtiger Vorteil eines Docker-Schwarms. Zum Skalieren genügt ein Docker-Befehl:

```
C:\> docker service scale <SERVICENAME>=<REPLICAS>
```

Hierbei ist \<SERVICENAME\> der Name des zu skalierenden Diensts, und \<REPLICAS\> steht für die Anzahl der Tasks (oder Containerinstanzen), auf die Dienst skaliert wird.


## <a name="viewing-the-swarm-state"></a>Anzeigen des Schwarmzustands

Es gibt mehrere Befehle, mit denen der Zustand des Schwarms und der im Schwarm ausgeführten Dienste ermittelt werden kann.

### <a name="list-swarm-nodes"></a>Liste der Schwarmknoten
Verwenden Sie den folgenden Befehl, um eine Liste der Knoten (einschließlich Statusinformationen) anzuzeigen, die momentan zum Schwarm gehören. Dieser Befehl muss auf einem **Verwaltungsknoten** ausgeführt werden.

```
C:\> docker node ls
```

In der Ausgabe dieses Befehls ist einer der Knoten mit einem Sternchen (*) markiert. Dies ist der aktuelle Knoten, also derjenige, auf dem der `docker node ls`-Befehl ausgeführt wurde.

### <a name="list-networks"></a>Liste der Netzwerke
Verwenden Sie den folgenden Befehl, um eine Liste der Netzwerke anzuzeigen, die auf einem bestimmten Knoten vorhanden sind. Um Überlagerungsnetzwerke anzuzeigen, muss dieser Befehl auf einem **Verwaltungsknoten** ausgeführt werden, der im Schwarmmodus arbeitet.

```
C:\> docker network ls
```

### <a name="list-services"></a>Liste der Dienste
Verwenden Sie den folgenden Befehl, um eine Liste der Dienste (einschließlich Statusinformationen) anzuzeigen, die momentan in einem Schwarm ausgeführt werden.

```
C:\> docker service ls
```

### <a name="list-the-container-instances-that-define-a-service"></a>Liste der Containerinstanzen, die einen Dienst definieren
Verwenden Sie den folgenden Befehl, um Details zu den Containerinstanzen anzuzeigen, die für einen bestimmten Dienst ausgeführt werden. Die Ausgabe dieses Befehl umfasst die Knoten (und IDs), auf denen jeder Container ausgeführt wird, sowie Informationen über den Zustand der Container.  

```
C:\> docker service ps <SERVICENAME>
```
## <a name="linuxwindows-mixed-os-clusters"></a>Vermischte Linux+Windows Betriebssystem-Cluster

Kürzlich hat ein Mitglied unseres Team eine kurze, dreiteilige Demo zum Einrichten einer gemischten Windows+Linux Betriebssystem-Anwendung mit Docker-Swarm veröffentlicht. Das ist ein guter Ausgangspunkt, wenn Sie noch keine Erfahrung mit Docker-Swarm oder beim Ausführen gemischter Betriebssystemanwendungen haben. Entdecken Sie es jetzt:
- [Verwenden von Docker Swarm zum Ausführen einer Containeranwendung unter Windows und Linux (Teil 1/3)](https://www.youtube.com/watch?v=ZfMV5JmkWCY&t=170s)
- [Verwenden von Docker Swarm zum Ausführen einer Containeranwendung unter Windows und Linux (Teil 2/3)](https://www.youtube.com/watch?v=VbzwKbcC_Mg&t=406s)
- [Verwenden von Docker Swarm zum Ausführen einer Containeranwendung unter Windows und Linux (Teil 3/3)](https://www.youtube.com/watch?v=I9oDD78E_1E&t=354s)

### <a name="initializing-a-linuxwindows-mixed-os-cluster"></a>Initialisieren eines vermischten Linux+Windows Betriebssystem-Clusters
Das Initialisieren eines vermischten Linux+Windows Betriebssystem-Swarm-Clusters ist einfach – solange Ihre Firewall-Regeln ordnungsgemäß konfiguriert sind und Ihre Hosts Zugriff aufeinander haben, benötigen Sie zum Hinzufügen eines Linux-Host an einen Swarm nur den Standardbefehl `docker swarm join`:
```
C:\> docker swarm join --token <JOINTOKEN> <MANAGERIPADDRESS>
```
Sie können auch einen Schwarm von einem Linux-Host mit dem gleichen Befehl initialisieren, den Sie ausführen würden, wenn der Schwarm von einem Windows-Host aus initialisiert würde:
```
# Initialize a swarm 
C:\> docker swarm init --advertise-addr=<HOSTIPADDRESS> --listen-addr <HOSTIPADDRESS>:2377
```

### <a name="adding-labels-to-swarm-nodes"></a>Hinzufügen von Beschriftungen zu einem Knoten
Um einen Dockerdienst auf einem vermischten OS-Schwarm-Cluster zu starten, muss unterscheiden werden können, welche Schwarm-Knoten auf dem Betriebssystems ausgeführt werden, das diesen Dienst unterstützten, und welche nicht. [Docker-Objektbeschriftungen](https://docs.docker.com/engine/userguide/labels-custom-metadata/) bieten eine praktische Möglichkeit zum Beschriften von Knoten, damit diese Dienste erstellt und so konfiguriert werden können, um nur auf den Knoten ausgeführt zu werden, die dem Betriebssystem entsprechen. 

>[!NOTE]
>[Docker-Objektbeschriftungen](https://docs.docker.com/engine/userguide/labels-custom-metadata/) werden zum Anwenden von Metadaten auf eine Vielzahl von Docker-Objekten (einschließlich Containerimages, Container, Volumes und Netzwerke) und eine Vielzahl von Zwecken verwendet (z. B. Bezeichnungen können verwendet werden, um Front-End- und Back-End-Komponenten einer Anwendung zu trennen, indem Front-End-Microservices nur auf Knoten mit der Bezeichnung „Front-End” und Back-End-Microservices nur auf Knoten mit der Bezeichnung „Back-End” geplant werden). In diesem Fall verwenden wir Beschriftungen für Knoten, um Knoten auf dem Windows-Betriebssystem von denen des Linux-Betriebssystems zu unterscheiden.

Verwenden Sie die folgende Syntax, um Ihre vorhandenen Schwarm-Knoten zu beschriften:

```
C:\> docker node update --label-add <LABELNAME>=<LABELVALUE> <NODENAME>
```

Dabei ist `<LABELNAME>` der Name der erstellten Beschriftung – beispielsweise unterscheiden wir in diesem Fall Knoten je nach Betriebssystem, also wäre ein logischer Namen für die Bezeichnung hier „Betriebssystem”. `<LABELVALUE>` ist der Wert der Bezeichnung – in diesem Fall können Sie die Werte „Windows“ und „Linux“ verwenden. (Natürlich können Sie die Beschriftung und Label-Werte so benennen, wie Sie möchten, solange Sie dabei konsistent bleiben). `<NODENAME>` ist der Name des Knotens, den Sie beschriften. Sie können die Namen der Knoten durch Ausführen von `docker node ls` aufrufen, wenn Sie sich nicht daran erinnern können. 

**Beispiel**: Wenn Sie über vier Schwarm-Knoten im Cluster verfügen, einschließlich zwei Windows-Knoten und zwei Linux-Knoten, können die Aktualisierungsbefehle für die Beschriftung wie folgt aussehen:

```
# Example -- labeling 2 Windows nodes and 2 Linux nodes in a cluster...
C:\> docker node update --label-add os=windows Windows-SwarmMaster
C:\> docker node update --label-add os=windows Windows-SwarmWorker1
C:\> docker node update --label-add os=linux Linux-SwarmNode1
C:\> docker node update --label-add os=linux Linux-SwarmNode2
```

### <a name="deploying-services-to-a-mixed-os-swarm"></a>Bereitstellen von Diensten für einen vermischten OS-Schwarm
Dank der Beschriftungen für die Schwarm Knoten ist das Bereitstellen von Diensten an den Cluster ganz einfach. Verwenden Sie einfach die Option `--constraint` für den Befehl [`docker service create`](https://docs.docker.com/engine/reference/commandline/service_create/):

```
# Deploy a service with swarm node constraint
C:\> docker service create --name=<SERVICENAME> --endpoint-mode dnsrr --network=<NETWORKNAME> --constraint node.labels.<LABELNAME>=<LABELVALUE> <CONTAINERIMAGE> [COMMAND] [ARGS…]
```

Wenn Sie beispielsweise die Beschriftung und die Beschriftungswertnomenklatur aus dem obigen Beispiel verwenden, sieht eine Reihe der Befehle für die Erstellung der Dienste – eine für einen Windows-basierten Dienst und eine für einen Linux-basierten Dienst – eventuell wie folgt aus:

```
# Example -- using the 'os' label and 'windows'/'linux' label values, service creation commands might look like these...

# A Windows service
C:\> docker service create --name=win_s1 --endpoint-mode dnsrr --network testoverlay --constraint 'node.labels.os==windows' microsoft/nanoserver:latest powershell -command { sleep 3600 }

# A Linux service
C:\> docker service create --name=linux_s1 --endpoint-mode dnsrr --network testoverlay --constraint 'node.labels.os==linux' redis
```

## <a name="limitations"></a>Einschränkungen
Momentan gelten unter Windows noch folgende Einschränkungen für den Schwarmmodus:
- Verschlüsselung auf der Datenschicht wird nicht unterstützt (d. h. Datenverkehr von Container zu Container mit der `--opt encrypted`-Option).
- [Routing-Mesh](https://docs.docker.com/engine/swarm/ingress/) für Windows-Docker-Hosts wird unter Windows Server 2016 nicht unterstützt, sondern erst ab Windows Server 2019. Benutzer, die schon jetzt eine alternative Lastenausgleichsstrategie wünschen, können ein externes Lastenausgleichssystem (z. B. NGINX) einrichten und den [publish-Portmodus](https://docs.docker.com/engine/reference/commandline/service_create/#/publish-service-ports-externally-to-the-swarm--p---publish) des Schwarms verwenden, um Containerhostports für den Lastenausgleich verfügbar zu machen. Weitere Informationen dazu finden Sie weiter unten.

 >[!NOTE]
>Weitere Informationen zur Routing-Mesh-Einrichtung für Docker Swarm finden Sie in diesem [Blogbeitrag](https://docs.microsoft.com/en-us/virtualization/community/team-blog/2017/20170926-docker-s-routing-mesh-available-with-windows-server-version-1709).

## <a name="publish-ports-for-service-endpoints"></a>Veröffentlichen von Ports für Dienstendpunkte
 Benutzer, die Ports für ihre Dienstendpunkte veröffentlichen möchten, können dazu heute den publish-Portmodus oder das Feature [Routing-Mesh](https://docs.docker.com/engine/swarm/ingress/) von Docker Swarm verwenden. 

Um Host-Ports für die einzelnen Aufgaben-/Container-Endpunkte, die einen Dienst definieren, zu veröffentliche, verwenden Sie das `--publish mode=host,target=<CONTAINERPORT>`-Argument für den `docker service create`-Befehl:

```
# Create a service for which tasks are exposed via host port
C:\ > docker service create --name=<SERVICENAME> --publish mode=host,target=<CONTAINERPORT> --endpoint-mode dnsrr --network=<NETWORKNAME> <CONTAINERIMAGE> [COMMAND] [ARGS…]
```

Der folgende Befehl erstellt z. B. einen Dienst, "s1", für die jede einzelne Aufgabe über den Container-Port 80 und einen zufällig ausgewählten Hostport verfügbar gemacht wird.

```
C:\ > docker service create --name=s1 --publish mode=host,target=80 --endpoint-mode dnsrr web_1 powershell -command {echo sleep; sleep 360000;}
```

Nachdem ein Dienstes durch den publish-Portmodus erstellt wurde, kann der Dienst abgefragt werden, um die Zuordnung für jeden Dienstvorgang anzuzeigen:

```
C:\ > docker service ps <SERVICENAME>
```
Der oben genannten Befehl gibt Details für jede Container-Instanz zurück, die für Ihren Dienst ausgeführt wird (für alle Schwarm-Hosts). Eine Spalte der Ausgabe, die Spalte „Ports“, enthält die Portinformationen für jeden Host in der Form \<HOSTPORT\>->\<CONTAINERPORT\>/tcp. Die Werte des \<HOSTPORT\> sind für jede Containerinstanz unterschiedlich, da jeder Container auf seinem eigenen Hostport veröffentlicht wird.


## <a name="tips--insights"></a>Tipps und Einblicke 

#### <a name="existing-transparent-network-can-block-swarm-initializationoverlay-network-creation"></a>*Bereits vorhandene transparente Netzwerke können die Schwarminitialisierung bzw. die Erstellung des Überlagerungsnetzwerks blockieren.* 
Unter Windows erfordern die Überlagerungs- und die transparenten Netzwerktreiber einen externen vSwitch, der an einen (virtuellen) Host des Netzwerkadapters angeschlossen ist. Wenn ein Überlagerungsnetzwerk erstellt wird, wird ein neuer Switch erstellt und einem offenen Netzwerkadapter hinzugefügt. Der transparente Netzwerkmodus verwendet einen Host-Netzwerkadapter. Gleichzeitig kann jeder Netzwerkadapter nur an einen Switch angebunden werden –wenn der Host nur über einen Netzwerkadapter verfügt, kann er jeweils nur an einen externen vSwitch angefügt werden, egal ob der vSwitch für ein Overlay-Netzwerk oder für ein transparentes Netzwerk gilt. 

Wenn ein Containerhost daher nur über einen Netzwerkadapter verfügt, kann ein Problem der transparenten Netzwerkblockierungserstellung eines Überlagerungsnetzwerks (oder umgekehrt) auftreten, da das transparente Netzwerk momentan die virtuelle der Netzwerkschnittstelle des Hosts belegt.

Sie haben zwei Möglichkeiten, dieses Problem zu umgehen:
- *Option 1 – Vorhandenes transparentes Netzwerk löschen:* Stellen Sie vor dem Initialisieren des Schwarm sicher, dass sich kein vorhandenes transparentes Netzwerk auf dem Hostcontainer befindet. Löschen Sie transparent Netzwerke, um sicherzustellen, dass ein kostenloser virtueller Netzwerkadapter auf dem Host existiert, der für die Erstellung eines Überlagerungsnetzwerks verwendet werden kann.
- *Option 2 – Erstellen eines zusätzlichen (virtuellen) Netzwerkadapters auf dem Host:* Sie können einen weiteren Netzwerkadapter auf Ihrem Host erstellen, der für die Erstellung von Überlagerungsnetzwerken verwendet werden kann, anstatt das sich auf Ihrem transparenten Host befindliche Netzwerk zu entfernen. Erstellen Sie dazu einfach einen neuen externen Netzwerkadapter (mithilfe von PowerShell oder dem Hyper-V-Manager), wenn die neue Benutzeroberfläche vorhanden. Wenn Ihr Schwarm initialisiert ist, wird der Host Netzwerk Service (HNS) diesen automatisch auf Ihrem Host erkennen und verwenden, um den externen vSwitch für die Erstellung des Überlagerungsnetzwerks anzubinden.



