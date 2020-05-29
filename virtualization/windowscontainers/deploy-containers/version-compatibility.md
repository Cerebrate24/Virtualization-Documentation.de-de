---
title: Versionskompatibilität von Windows-Containern
description: Hier erfahren Sie, wie Windows Container versionsübergreifend erstellen und ausführen kann.
keywords: Metadaten, Container, Version
author: taylorb-microsoft
ms.openlocfilehash: 917c07e13d6a0ec5b5e73213da4dc4f04ec0d9bb
ms.sourcegitcommit: 8eedfdc1fda9d0abb36e28dc2b5fb39891777364
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 03/10/2020
ms.locfileid: "79027865"
---
# <a name="windows-container-version-compatibility"></a>Versionskompatibilität von Windows-Containern

Windows Server 2016 und Windows 10 Anniversary Update (beide Version 14393) waren die ersten Windows-Versionen, die Windows Server-Container erstellen und ausführen konnten. Container, die mit diesen Versionen erstellt wurden, können zwar mit neueren Versionen ausgeführt werden, es gibt dabei aber ein paar Dinge zu beachten.

Da wir die Windows-Containerfeatures stetig verbessern, mussten wir einige Änderungen vornehmen, die sich auf die Kompatibilität auswirken können. Ältere Container werden unverändert auf neueren Hosts mit [Hyper-V-Isolation](../manage-containers/hyperv-container.md) ausgeführt, und sie verwenden weiterhin die gleiche (ältere) Kernel-Version. Wenn Sie jedoch einen Container basierend auf einem neueren Windows-Build ausführen möchten, kann er nur auf dem neueren Host-Build ausgeführt werden.

## <a name="windows-server-host-os-compatibility"></a>Betriebssystemkompatibilität des Windows Server-Hosts

<!-- start tab view -->
# <a name="windows-server-version-1909"></a>[Windows Server, Version 1909](#tab/windows-server-1909)

|Betriebssystemversion für Containerbasisimage|Unterstützt die Hyper-V-Isolation|Unterstützt die Prozessisolation|
|---|:---:|:---:|
|Windows Server, Version 1909|&#10004;|&#10004;|
|Windows Server, Version 1903|&#10004;|&#10060;|
|Windows Server 2019|&#10004;|&#10060;|
|Windows Server 2016|&#10004;|&#10060;|

# <a name="windows-server-version-1903"></a>[Windows Server, Version 1903](#tab/windows-server-1903)

|Betriebssystemversion für Containerbasisimage|Unterstützt die Hyper-V-Isolation|Unterstützt die Prozessisolation|
|---|:---:|:---:|
|Windows Server, Version 1909|&#10060;|&#10060;|
|Windows Server, Version 1903|&#10004;|&#10004;|
|Windows Server 2019|&#10004;|&#10060;|
|Windows Server 2016|&#10004;|&#10060;|

# <a name="windows-server-2019"></a>[Windows Server 2019](#tab/windows-server-2019)

|Betriebssystemversion für Containerbasisimage|Unterstützt die Hyper-V-Isolation|Unterstützt die Prozessisolation|
|---|:---:|:---:|
|Windows Server, Version 1909|&#10060;|&#10060;|
|Windows Server, Version 1903|&#10060;|&#10060;|
|Windows Server 2019|&#10004;|&#10004;|
|Windows Server 2016|&#10004;|&#10060;|

# <a name="windows-server-2016"></a>[Windows Server 2016](#tab/windows-server-2016)

|Betriebssystemversion für Containerbasisimage|Unterstützt die Hyper-V-Isolation|Unterstützt die Prozessisolation|
|---|:---:|:---:|
|Windows Server, Version 1909|&#10060;|&#10060;|
|Windows Server, Version 1903|&#10060;|&#10060;|
|Windows Server 2019|&#10060;|&#10060;|
|Windows Server 2016|&#10004;|&#10004;|

---
<!-- stop tab view -->

## <a name="windows-10-host-os-compatibility"></a>Betriebssystemkompatibilität des Windows 10-Hosts

<!-- start tab view -->

# <a name="windows-10-version-1909"></a>[Windows 10, Version 1909](#tab/windows-10-1909)

|Betriebssystemversion für Containerbasisimage|Unterstützt die Hyper-V-Isolation|Unterstützt die Prozessisolation|
|---|:---:|:---:|
|Windows Server, Version 1909|&#10004;|&#10060;|
|Windows Server, Version 1903|&#10004;|&#10060;|
|Windows Server 2019|&#10004;|&#10060;|
|Windows Server 2016|&#10004;|&#10060;|

# <a name="windows-10-version-1903"></a>[Windows 10, Version 1903](#tab/windows-10-1903)

|Betriebssystemversion für Containerbasisimage|Unterstützt die Hyper-V-Isolation|Unterstützt die Prozessisolation|
|---|:---:|:---:|
|Windows Server, Version 1909|&#10060;|&#10060;|
|Windows Server, Version 1903|&#10004;|&#10060;|
|Windows Server 2019|&#10004;|&#10060;|
|Windows Server 2016|&#10004;|&#10060;|

# <a name="windows-10-version-1809"></a>[Windows 10, Version 1809](#tab/windows-10-1809)

|Betriebssystemversion für Containerbasisimage|Unterstützt die Hyper-V-Isolation|Unterstützt die Prozessisolation|
|---|:---:|:---:|
|Windows Server, Version 1909|&#10060;|&#10060;|
|Windows Server, Version 1903|&#10060;|&#10060;|
|Windows Server 2019|&#10004;|&#10060;|
|Windows Server 2016|&#10004;|&#10060;|

---
<!-- stop tab view -->

## <a name="matching-container-host-version-with-container-image-versions"></a>Abgleichen der Containerhostversion und der Containerimageversionen

### <a name="windows-server-containers"></a>Windows Server-Container

Da sich Windows Server-Container und der zugrunde liegende Host einen einzelnen Kernel teilen, muss die Betriebssystemversion des Basisimages des Containers mit dem des Hosts übereinstimmen. Wenn die Versionen nicht übereinstimmen, ist es möglich, dass der Container dennoch startet. Allerdings kann der volle Funktionsumfang nicht gewährleistet werden. Das Windows-Betriebssystem hat vier Versionierungsgrade: die Hauptversion, die Nebenversion, den Build und die Revision. Die Version 10.0.14393.103 hat z. B. die Hauptversion 10, die Nebenversion 0, die Buildnummer 14393 und die Revisionsnummer 103. Die Buildnummer wird nur geändert, wenn neue Versionen des Betriebssystems veröffentlicht werden, z. B. Version 1709, 1903 usw. Die Revisionsnummer wird aktualisiert, wenn Windows-Updates angewendet werden.

#### <a name="build-number-new-release-of-windows"></a>Buildnummer (neue Version von Windows)

Der Start eines Windows Server-Containers wird verhindert, wenn die Buildnummer zwischen dem Containerhost und dem Containerimage unterschiedlich ist. Wenn der Containerhost z. B. die Version 10.0.14393.* (Windows Server 2016) und das Containerimage die Version 10.0.16299.* (Windows Server Version 1709) aufweist, kann der Container nicht gestartet werden.  

#### <a name="revision-number-patching"></a>Revisionsnummer (Patches)

Windows Server-Container unterstützen derzeit keine Szenarien, in denen Windows Server 2016-basierte Container in einem System ausgeführt werden, in dem die Revisionsnummer für Containerhost und Containerimage unterschiedlich ist. Wenn der Containerhost z. B. Version 10.0.14393.**1914** aufweist (Windows Server 2016 mit angewendeter KB4051033) und das Containerimage entspricht Version 10.0.14393.**1944** (Windows Server 2016 mit angewendeter KB4053579), dann startet das Image möglicherweise nicht.

Für Hosts oder Images, die Windows Server Version 1809 und höher verwenden, gilt diese Regel jedoch nicht, und das Host- und das Containerimage müssen keine übereinstimmenden Revisionen aufweisen.

Wir empfehlen Ihnen, Ihre Systeme (Host und Container) mit den neuesten Patches und Updates auf dem neuesten Stand zu halten, um geschützt zu bleiben.

>[!NOTE]
>Bei der Verwendung von Windows Server-Containern mit der Sicherheitsupdateversion vom 11. Februar 2020 (auch „2B“ genannt) oder späteren monatlichen Sicherheitsupdateversionen können Probleme auftreten. Weitere Informationen finden Sie in [diesem Artikel](https://support.microsoft.com/help/4542617/you-might-encounter-issues-when-using-windows-server-containers-with-t).  
>
>Wir empfehlen Ihnen dringend, sowohl Ihren Host als auch Ihre Container mit den neuesten Patches und Updates zu aktualisieren, um geschützt und kompatibel zu bleiben. Wichtige Hinweise zum Aktualisieren von Windows-Containern finden Sie unter [Aktualisieren von Windows Server-Containern](update-containers.md).

#### <a name="practical-application"></a>Praktische Anwendung

Beispiel 1:  Der Containerhost führt Windows Server 2016 mit KB4041691 aus. Alle Windows Server-Container, die auf diesem Host bereitgestellt werden, müssen auf der Version 10.0.14393.1770 für Containerbasisimages basieren. Wenn Sie KB4053579 auf den Hostcontainer anwenden, müssen Sie auch die Images aktualisieren, um sicherzustellen, dass der Hostcontainer sie unterstützt.

Beispiel 2: Der Containerhost führt Windows Server Version 1809 mit KB4534273 aus. Alle auf diesem Host bereitgestellten Windows Server-Container müssen auf einem Containerbasisimage von Windows Server-Version 1809 (10.0.17763) basieren, müssen allerdings nicht mit der Host-KB übereinstimmen. Wenn KB4534273 auf den Host angewendet wird, werden die Containerimages weiterhin unterstützt, aber wir empfehlen Ihnen, sie zu aktualisieren, um mögliche Sicherheitsprobleme zu behandeln.

#### <a name="querying-version"></a>Abfrageversion

Methode 1: Die in Version 1709 eingeführte Cmd-Eingabeaufforderung und der **ver**-Befehl geben jetzt die Revisionsdetails zurück.

```batch
Microsoft Windows [Version 10.0.16299.125]
(c) 2017 Microsoft Corporation. All rights reserved.

C:\>ver

Microsoft Windows [Version 10.0.16299.125]
```

Methode 2: Fragen Sie den folgenden Registrierungsschlüssel ab: HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion

Beispiel:

```batch
C:\>reg query "HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion" /v BuildLabEx
```

```powershell
Windows PowerShell
Copyright (C) 2016 Microsoft Corporation. All rights reserved.

PS C:\Users\Administrator> (Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\').BuildLabEx
14393.321.amd64fre.rs1_release_inmarket.161004-2338
```

Prüfen Sie die Tags im Docker Hub oder die Image-Hashtabelle in der Beschreibung des Images, um zu überprüfen, welche Version Ihr Basisimage verwendet. Auf der Seite [Windows 10-Updateverlauf](https://support.microsoft.com/help/12387/windows-10-update-history) wird aufgeführt, wann die einzelnen Builds und Revisionen veröffentlicht wurden.

### <a name="hyper-v-isolation-for-containers"></a>Hyper-V-Isolation für Container

Sie können Windows-Container mit oder ohne Hyper-V-Isolation ausführen. Hyper-V-Isolation erstellt mithilfe eines optimierten VMs eine Sicherheitsbegrenzung um den Container herum. Im Gegensatz zu Windows Standard-Containern, bei denen sich die Container und der Host den Kernel teilen, verwendet jeder isolierte Hyper-V-Container eine eigene Instanz des Windows-Kernels. Das bedeutet, dass Sie unterschiedliche Betriebssystemversionen im Containerhost und im Image verwenden können (weitere Informationen finden Sie in der folgenden Kompatibilitätsmatrix).  

Um einen Container mit Hyper-V-Isolierung auszuführen, fügen Sie einfach das Tag `--isolation=hyperv` zu Ihrem Docker-Ausführungsbefehl hinzu.

## <a name="errors-from-mismatched-versions"></a>Fehler bei nicht übereinstimmenden Versionen

Wenn Sie versuchen, eine nicht unterstützte Kombination auszuführen, erhalten Sie den folgenden Fehler:

```dockerfile
docker: Error response from daemon: container b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839 encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839","Owner":"docker","IsDummy":false,"VolumePath":"\\\\?\\Volume{2443d38a-1379-4bcf-a4b7-fc6ad4cd7b65}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839","Layers":[{"ID":"1532b584-8431-5b5a-8735-5e1b4fe9c2a9","Path":"C:\\ProgramData\\docker\\windowsfilter\\b2b88bc2a47abcc682e422507abbba9c9b6d826d34e67b9e4e3144cc125a1f80"},{"ID":"a64b8da5-cd6e-5540-bc73-d81acae6da54","Path":"C:\\ProgramData\\docker\\windowsfilter\\5caaedbced1f546bccd01c9d31ea6eea4d30701ebba7b95ee8faa8c098a6845a"}],"HostName":"b81ed896222e","MappedDirectories":[],"HvPartition":false,"EndpointList":["002a0d9e-13b7-42c0-89b2-c1e80d9af243"],"Servicing":false,"AllowUnqualifiedDNSQuery":true}.
```

Es gibt drei Möglichkeiten, diesen Fehler zu beheben:

- Erstellen Sie den Container neu basierend auf der richtigen Version von `mcr.microsoft.com/windows/nanoserver` oder `mcr.microsoft.com/windows/servercore`.
- Wenn der Host neuer ist, führen Sie **docker run --isolation=hyperv ...** aus.
- Versuchen Sie, den Container auf einem anderen Host mit derselben Windows-Version auszuführen.

## <a name="choose-which-container-os-version-to-use"></a>Auswählen der zu verwendenden Containerbetriebssystemversion

>[!NOTE]
>Seit dem 16. April 2019 wird das „latest“-Tag für die [Windows-Basisbetriebssystem-Containerimages](https://hub.docker.com/_/microsoft-windows-base-os-images) nicht mehr veröffentlicht oder unterstützt. Geben Sie ein bestimmtes Tag an, wenn Sie Images aus diesen Repositorys mithilfe von Pull übertragen oder auf diese verweisen.

Sie müssen wissen, welche Version Sie für Ihren Container verwenden müssen. Wenn Sie z. B. Windows Server Version 1809 als Ihr Containerbetriebssystem wünschen und die neuesten Patches dafür abrufen möchten, sollten Sie das Tag `1809` verwenden, wenn Sie angeben, welche Version der Basisbetriebssystem-Containerimages Sie möchten, also:

```dockerfile
FROM mcr.microsoft.com/windows/nanoserver:1809
...
```

Wenn Sie ein bestimmtes Patch von Windows Server Version 1809 verwenden möchten, können Sie die KB-Nummer im Tag angeben. Wenn Sie z. B. das Basisbetriebssystem-Containerimage für Nano Server von Windows Server Version 1809 mit angewandtem KB4493509 abrufen möchten, würden Sie dies wie folgt angeben:

```dockerfile
FROM mcr.microsoft.com/windows/nanoserver:1809-KB4493509
...
```

Sie können auch genau die benötigten Patches mit dem zuvor verwendeten Schema angeben (Angabe der Betriebssystemversion im Tag):

```dockerfile
FROM mcr.microsoft.com/windows/nanoserver:10.0.17763.437
...
```

Bei den Server Core-Basisimages, die auf Windows Server 2019 und Windows Server 2016 basieren, handelt es sich um [LTSC-Versionen (Long-Term Servicing Channel)](https://docs.microsoft.com/windows-server/get-started-19/servicing-channels-19#long-term-servicing-channel-ltsc). Wenn Sie z. B. Windows Server 2019 als Containerbetriebssystem für Ihr Server Core-Image verwenden und die neuesten Patches dafür abrufen möchten, können Sie die LTSC-Versionen wie folgt angeben:

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019
...
```

## <a name="matching-versions-using-docker-swarm"></a>Versionsabgleich mit Docker-Schwarm

Docker-Schwarm verfügt derzeit nicht über eine integrierte Möglichkeit, die Windows-Version, die ein Container verwendet, mit einem Host mit derselben Version abzugleichen. Wenn Sie den Dienst aktualisieren, um einen neueren Container zu verwenden, wird er erfolgreich ausgeführt.

Wenn Sie mehrere Windows-Versionen über einen längeren Zeitraum ausführen müssen, gibt es zwei Möglichkeiten: Entweder konfigurieren Sie die Windows-Hosts so, dass sie immer die Hyper-V-Isolation verwenden, oder Sie verwenden Labeleinschränkungen.

### <a name="finding-a-service-that-wont-start"></a>Suchen eines Diensts, der nicht gestartet wird

Wenn ein Dienst nicht startet, sehen Sie, dass `MODE` dem Wert `replicated` entspricht, `REPLICAS` jedoch bei „0“ bleibt. Um festzustellen, ob dies an der Version des Betriebssystems liegt, führen Sie die folgenden Befehle aus:

Führen Sie **docker service ls** aus, um den Dienstnamen zu finden:

```dockerfile
ID                  NAME                MODE                REPLICAS            IMAGE                                             PORTS
xh6mwbdq2uil        angry_liskov        replicated          0/1                 microsoft/iis:windowsservercore-10.0.14393.1715
```

Führen Sie **docker service ps (Dienstname)** aus, um den Status und die letzten Versuche abzurufen:

```dockerfile
C:\Program Files\Docker>docker service ps angry_liskov
ID                  NAME                 IMAGE                                             NODE                DESIRED STATE       CURRENT STATE               ERROR                              PORTS
klkbhn742lv0        angry_liskov.1       microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Ready               Ready 3 seconds ago
y5blbdum70zo         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed 24 seconds ago       "starting container failed: co…"
yjq6zwzqj8kt         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed 31 seconds ago       "starting container failed: co…"

ytnnv80p03xx         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed about a minute ago   "starting container failed: co…"
xeqkxbsao57w         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed about a minute ago   "starting container failed: co…"
```

Wenn `starting container failed: ...` angezeigt wird, können Sie den vollständigen Fehler mit **docker service ps --no-trunc (Containername)** anzeigen:

```dockerfile
C:\Program Files\Docker>docker service ps --no-trunc angry_liskov
ID                          NAME                 IMAGE                                                                                                                     NODE                DESIRED STATE       CURRENT STATE                     ERROR                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          PORTS
dwsd6sjlwsgic5vrglhtxu178   angry_liskov.1       microsoft/iis:windowsservercore-10.0.14393.1715@sha256:868bca7e89e1743792e15f78edb5a73070ef44eae6807dc3f05f9b94c23943d5   WIN-BSTMQDRQC2E     Running             Starting less than a second ago                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              
y5blbdum70zoh1f6uhx5nxsfv    \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715@sha256:868bca7e89e1743792e15f78edb5a73070ef44eae6807dc3f05f9b94c23943d5   WIN-BSTMQDRQC2E     Shutdown            Failed 39 seconds ago             "starting container failed: container e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0 encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0","Owner":"docker","VolumePath":"\\\\?\\Volume{2443d38a-1379-4bcf-a4b7-fc6ad4cd7b65}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0","Layers":[{"ID":"bcf2630f-ea95-529b-b33c-e5cdab0afdb4","Path":"C:\\ProgramData\\docker\\windowsfilter\\200235127f92416724ae1d53ed3fdc86d78767132d019bdda1e1192ee4cf3ae4"},{"ID":"e3ea10a8-4c2f-5b93-b2aa-720982f116f6","Path":"C:\\ProgramData\\docker\\windowsfilter\\0ccc9fa71a9f4c5f6f3bc8134fe3533e454e09f453de662cf99ab5d2106abbdc"},{"ID":"cff5391f-e481-593c-aff7-12e080c653ab","Path":"C:\\ProgramData\\docker\\windowsfilter\\a49576b24cd6ec4a26202871c36c0a2083d507394a3072186133131a72601a31"},{"ID":"499cb51e-b891-549a-b1f4-8a25a4665fbd","Path":"C:\\ProgramData\\docker\\windowsfilter\\fdf2f52c4323c62f7ff9b031c0bc3af42cf5fba91098d51089d039fb3e834c08"},{"ID":"1532b584-8431-5b5a-8735-5e1b4fe9c2a9","Path":"C:\\ProgramData\\docker\\windowsfilter\\b2b88bc2a47abcc682e422507abbba9c9b6d826d34e67b9e4e3144cc125a1f80"},{"ID":"a64b8da5-cd6e-5540-bc73-d81acae6da54","Path":"C:\\ProgramData\\docker\\windowsfilter\\5caaedbced1f546bccd01c9d31ea6eea4d30701ebba7b95ee8faa8c098a6845a"}],"HostName":"e7b5d3adba7e","HvPartition":false,"EndpointList":["298bb656-8800-4948-a41c-1b0500f3d94c"],"AllowUnqualifiedDNSQuery":true}"
```

Dies ist derselbe Fehler wie `CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101)`.

### <a name="fix---update-the-service-to-use-a-matching-version"></a>Problembehebung: Aktualisieren des Diensts für die Verwendung einer übereinstimmenden Version

Bei Docker-Schwarm gibt es zwei Dinge zu berücksichtigen. Für den Fall, dass Sie über eine Compose-Datei mit einem Dienst verfügen, der ein Image verwendet, das Sie nicht erstellt haben, sollten Sie den Verweis entsprechend aktualisieren. Beispiel:

``` yaml
version: '3'

services:
  YourServiceName:
    image: microsoft/windowsservercore:1709
...
```

Wenn Sie auf ein Image verweisen, das Sie selbst erstellt haben (z. B. contoso/myimage):

```yaml
version: '3'

services:
  YourServiceName:
    image: contoso/myimage
...
```

In diesem Fall sollten Sie die in [Fehler bei nicht übereinstimmenden Versionen](#errors-from-mismatched-versions) beschriebene Methode verwenden, um diese Dockerfile anstelle der Zeile docker-compose zu ändern.

### <a name="mitigation---use-hyper-v-isolation-with-docker-swarm"></a>Lösung: Hyper-V-Isolierung mit Docker-Schwarm

Es ist eine Unterstützung mithilfe der Hyper-V-Isolation pro Container möglich; der Code ist jedoch noch nicht fertig entwickelt. Unter [GitHub](https://github.com/moby/moby/issues/31616) können Sie den Fortschritt verfolgen. Bis dahin müssten die Hosts so konfiguriert werden, dass sie immer mit Hyper-V-Isolierung ausgeführt werden.

Dies erfordert eine Änderung der Docker-Dienstkonfiguration und einen anschließenden Neustart des Docker-Moduls.

1. Bearbeiten von `C:\ProgramData\docker\config\daemon.json`
2. Hinzufügen einer Zeile mit `"exec-opts":["isolation=hyperv"]`

    >[!NOTE]
    >Die Datei „daemon.json“ ist standardmäßig nicht vorhanden. Wenn Sie im Verzeichnis feststellen, dass dies der Fall ist, müssen Sie die Datei erstellen. Kopieren Sie anschließend Folgendes hinein:

    ```JSON
    {
        "exec-opts":["isolation=hyperv"]
    }
    ```

3. Schließen und speichern Sie die Datei, und starten Sie dann das Docker-Modul neu, indem Sie die folgenden Cmdlets in PowerShell ausführen:

    ```powershell
    Stop-Service docker
    Start-Service docker
    ```

4. Nachdem Sie den Dienst neu gestartet haben, starten Sie Ihre Container. Sobald sie ausgeführt werden, können Sie die Isolationsstufe eines Containers durch Überprüfen des betreffenden Containers mit dem folgenden Cmdlet anzeigen:

    ```powershell
    docker inspect --format='{{json .HostConfig.Isolation}}' $instanceNameOrId
    ```

Es wird entweder "process" oder "hyperv" zurückgegeben. Wenn Sie "daemon.json" wie oben beschrieben geändert und gespeichert haben, sollte Letzteres angezeigt werden.

### <a name="mitigation---use-labels-and-constraints"></a>Lösung: Verwenden von Labels und Einschränkungen

Hier erfahren Sie, wie Sie Label und Einschränkungen verwenden, um Versionen abzugleichen:

1. Fügen Sie jedem Knoten Label hinzu.

    Fügen Sie auf jedem Knoten zwei Label hinzu: `OS` und `OsVersion`. Es wird hier davon ausgegangen, dass sie lokal ausgeführt werden, es kann aber auch so geändert werden, dass sie auf einem Remote-Host ausgeführt werden.

    ```powershell
    docker node update --label-add OS="windows" $ENV:COMPUTERNAME
    docker node update --label-add OsVersion="$((Get-ComputerInfo).OsVersion)" $ENV:COMPUTERNAME
    ```

    Anschließend können Sie diese überprüfen, indem Sie den Befehl **docker node inspect** ausführen, der die neu hinzugefügten Label anzeigen sollte:

    ```yaml
           "Spec": {
                "Labels": {
                   "OS": "windows",
                   "OsVersion": "10.0.16296"
               },
                "Role": "manager",
                "Availability": "active"
            }
    ```

2. Fügen Sie eine Diensteinschränkung hinzu.

    Nachdem Sie die einzelnen Knoten bezeichnet haben, können Sie Einschränkungen aktualisieren, die die Platzierung von Diensten bestimmen. Ersetzen Sie im folgenden Beispiel „contoso_service“ durch den Namen Ihres eigentlichen Diensts:

    ```powershell
    docker service update \
        --constraint-add "node.labels.OS == windows" \
        --constraint-add "node.labels.OsVersion == $((Get-ComputerInfo).OsVersion)" \
        contoso_service
    ```

    Dies erzwingt und schränkt den Ort ein, an dem ein Knoten ausgeführt werden kann.

Weitere Informationen zur Verwendung von Diensteinschränkungen finden Sie in der [Referenz zur Diensterstellung](https://docs.docker.com/engine/reference/commandline/service_create/#specify-service-constraints-constraint).

## <a name="matching-versions-using-kubernetes"></a>Versionsabgleich mit Kubernetes

Dasselbe Problem, das in [Versionsabgleich mit Docker-Schwarm](#matching-versions-using-docker-swarm) beschrieben wird, kann auftreten, wenn Pods in Kubernetes geplant sind. Dieses Problem kann mit ähnlichen Strategien vermieden werden:

- Erstellen Sie den Container auf der Grundlage derselben Betriebssystemversion in der Entwicklungs- und Produktionsumgebung neu. Wie das funktioniert, erfahren Sie unter [Auswählen der zu verwendenden Containerbetriebssystemversion](#choose-which-container-os-version-to-use).
- Verwenden Sie Labels und Knotenselektoren, um sicherzustellen, dass die Pods auf kompatiblen Knoten geplant werden, wenn sich die Knoten von Windows Server 2016 und Windows Server Version 1709 im gleichen Cluster befinden.
- Verwenden von separaten Clustern basierend auf der Betriebssystemversion

### <a name="finding-pods-failed-on-os-mismatch"></a>Suchen der Pods, die wegen Betriebssysteminkonsistenz fehlgeschlagen sind

Hier enthielt eine Bereitstellung einen Pod, der auf einem Knoten mit einer nicht übereinstimmenden Betriebssystemversion und ohne aktivierter Hyper-V-Isolation geplant wurde.

Der gleiche Fehler wird in den Ereignissen angezeigt, die mit `kubectl describe pod <podname>` abgerufen werden. Nach mehreren Versuchen lautet der Podstatus vermutlich `CrashLoopBackOff`.

```
$ kubectl -n plang describe pod fabrikamfiber.web-789699744-rqv6p

Name:           fabrikamfiber.web-789699744-rqv6p
Namespace:      plang
Node:           38519acs9011/10.240.0.6
Start Time:     Mon, 09 Oct 2017 19:40:30 +0000
Labels:         io.kompose.service=fabrikamfiber.web
                pod-template-hash=789699744
Annotations:    kubernetes.io/created-by={"kind":"SerializedReference","apiVersion":"v1","reference":{"kind":"ReplicaSet","namespace":"plang","name":"fabrikamfiber.web-789699744","uid":"b5062a08-ad29-11e7-b16e-000d3a...
Status:         Running
IP:             10.244.3.169
Created By:     ReplicaSet/fabrikamfiber.web-789699744
Controlled By:  ReplicaSet/fabrikamfiber.web-789699744
Containers:
  fabrikamfiberweb:
    Container ID:       docker://eab0151378308315ed6c31adf4ad9e31e6d65fd300e56e742757004a969a803a
    Image:              patricklang/fabrikamfiber.web:latest
    Image ID:           docker-pullable://patricklang/fabrikamfiber.web@sha256:562741016ce7d9a232a389449a4fd0a0a55aab178cf324144404812887250ead
    Port:               80/TCP
    State:              Waiting
      Reason:           CrashLoopBackOff
    Last State:         Terminated
      Reason:           ContainerCannotRun
      Message:          container eab0151378308315ed6c31adf4ad9e31e6d65fd300e56e742757004a969a803a encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"eab0151378308315ed6c31adf4ad9e31e6d65fd300e56e742757004a969a803a","Owner":"docker","IsDummy":false,"VolumePath":"\\\\?\\Volume{037b6606-bc9c-461f-ae02-829c28410798}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\eab0151378308315ed6c31adf4ad9e31e6d65fd300e56e742757004a969a803a","Layers":[{"ID":"f8bc427f-7aa3-59c6-b271-7331713e9451","Path":"C:\\ProgramData\\docker\\windowsfilter\\e206d2514a6265a76645b9d6b3dc6a78777c34dbf5da9fa2d564651645685881"},{"ID":"a6f35e41-a86c-5e4d-a19a-a6c2464bfb47","Path":"C:\\ProgramData\\docker\\windowsfilter\\0f21f1e28ef13030bbf0d87cbc97d1bc75f82ea53c842e9a3250a2156ced12d5"},{"ID":"4f624ca7-2c6d-5c42-b73f-be6e6baf2530","Path":"C:\\ProgramData\\docker\\windowsfilter\\4d9e2ad969fbd74fd58c98b5ab61e55a523087910da5200920e2b6f641d00c67"},{"ID":"88e360ff-32af-521d-9a3f-3760c12b35e2","Path":"C:\\ProgramData\\docker\\windowsfilter\\9e16a3d53d3e9b33344a6f0d4ed34c8a46448ee7636b672b61718225b8165e6e"},{"ID":"20f1a4e0-a6f3-5db3-9bf2-01fd3e9e855a","Path":"C:\\ProgramData\\docker\\windowsfilter\\7eec7f59f9adce38cc0a6755da58a3589287d920d37414b5b21b5b543d910461"},{"ID":"c2b3d728-4879-5343-a92a-b735752a4724","Path":"C:\\ProgramData\\docker\\windowsfilter\\8ed04b60acc0f65f541292a9e598d5f73019c8db425f8d49ea5819eab16a42f3"},{"ID":"2973e760-dc59-5800-a3de-ab9d93be81e5","Path":"C:\\ProgramData\\docker\\windowsfilter\\cc71305d45f09ce377ef497f28c3a74ee027c27f20657d2c4a5f157d2457cc75"},{"ID":"454a7d36-038c-5364-8a25-fa84091869d6","Path":"C:\\ProgramData\\docker\\windowsfilter\\9e8cde1ce8f5de861a5f22841f1ab9bc53d5f606d06efeacf5177f340e8d54d0"},{"ID":"9b748c8c-69eb-55fb-a1c1-5688cac4efd8","Path":"C:\\ProgramData\\docker\\windowsfilter\\8e02bf5404057055a71d542780a2bb2731be4b3707c01918ba969fb4d83b98ec"},{"ID":"bfde5c26-b33f-5424-9405-9d69c2fea4d0","Path":"C:\\ProgramData\\docker\\windowsfilter\\77483cedfb0964008c33d92d306734e1fab3b5e1ebb27e898f58ccfd108108f2"},{"ID":"bdabfbf5-80d1-57f1-86f3-448ce97e2d05","Path":"C:\\ProgramData\\docker\\windowsfilter\\aed2ebbb31ad24b38ee8521dd17744319f83d267bf7f360bc177e27ae9a006cf"},{"ID":"ad9b34f2-dcee-59ea-8962-b30704ae6331","Path":"C:\\ProgramData\\docker\\windowsfilter\\d44d3a675fec1070b61d6ea9bacef4ac12513caf16bd6453f043eed2792f75d8"}],"HostName":"fabrikamfiber.web-789699744-rqv6p","MappedDirectories":[{"HostPath":"c:\\var\\lib\\kubelet\\pods\\b50f0027-ad29-11e7-b16e-000d3afd2878\\volumes\\kubernetes.io~secret\\default-token-rw9dn","ContainerPath":"c:\\var\\run\\secrets\\kubernetes.io\\serviceaccount","ReadOnly":true,"BandwidthMaximum":0,"IOPSMaximum":0}],"HvPartition":false,"EndpointList":null,"NetworkSharedContainerName":"586870f5833279678773cb700db3c175afc81d557a75867bf39b43f985133d13","Servicing":false,"AllowUnqualifiedDNSQuery":false}
      Exit Code:        128
      Started:          Mon, 09 Oct 2017 20:27:08 +0000
      Finished:         Mon, 09 Oct 2017 20:27:08 +0000
    Ready:              False
    Restart Count:      10
    Environment:        <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-rw9dn (ro)
Conditions:
  Type          Status
  Initialized   True
  Ready         False
  PodScheduled  True
Volumes:
  default-token-rw9dn:
    Type:       Secret (a volume populated by a Secret)
    SecretName: default-token-rw9dn
    Optional:   false
QoS Class:      BestEffort
Node-Selectors: beta.kubernetes.io/os=windows
Tolerations:    <none>
Events:
  FirstSeen     LastSeen        Count   From                    SubObjectPath                           Type            Reason                  Message
  ---------     --------        -----   ----                    -------------                           --------        ------                  -------
  49m           49m             1       default-scheduler                                               Normal          Scheduled               Successfully assigned fabrikamfiber.web-789699744-rqv6p to 38519acs9011
  49m           49m             1       kubelet, 38519acs9011                                           Normal          SuccessfulMountVolume   MountVolume.SetUp succeeded for volume "default-token-rw9dn"
  29m           29m             1       kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Warning         Failed                  Failed to pull image "patricklang/fabrikamfiber.web:latest": rpc error: code = 2 desc = Error response from daemon: {"message":"Get https://registry-1.docker.io/v2/: dial tcp: lookup registry-1.docker.io: no such host"}
  49m           3m              12      kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Normal          Pulling                 pulling image "patricklang/fabrikamfiber.web:latest"
  33m           3m              11      kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Normal          Pulled                  Successfully pulled image "patricklang/fabrikamfiber.web:latest"
  33m           3m              11      kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Normal          Created                 Created container
  33m           2m              11      kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Warning         Failed                  Error: failed to start container "fabrikamfiberweb": Error response from daemon: {"message":"container fabrikamfiberweb encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {\"SystemType\":\"Container\",\"Name\":\"fabrikamfiberweb\",\"Owner\":\"docker\",\"IsDummy\":false,\"VolumePath\":\"\\\\\\\\?\\\\Volume{037b6606-bc9c-461f-ae02-829c28410798}\",\"IgnoreFlushesDuringBoot\":true,\"LayerFolderPath\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\fabrikamfiberweb\",\"Layers\":[{\"ID\":\"f8bc427f-7aa3-59c6-b271-7331713e9451\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\e206d2514a6265a76645b9d6b3dc6a78777c34dbf5da9fa2d564651645685881\"},{\"ID\":\"a6f35e41-a86c-5e4d-a19a-a6c2464bfb47\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\0f21f1e28ef13030bbf0d87cbc97d1bc75f82ea53c842e9a3250a2156ced12d5\"},{\"ID\":\"4f624ca7-2c6d-5c42-b73f-be6e6baf2530\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\4d9e2ad969fbd74fd58c98b5ab61e55a523087910da5200920e2b6f641d00c67\"},{\"ID\":\"88e360ff-32af-521d-9a3f-3760c12b35e2\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\9e16a3d53d3e9b33344a6f0d4ed34c8a46448ee7636b672b61718225b8165e6e\"},{\"ID\":\"20f1a4e0-a6f3-5db3-9bf2-01fd3e9e855a\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\7eec7f59f9adce38cc0a6755da58a3589287d920d37414b5b21b5b543d910461\"},{\"ID\":\"c2b3d728-4879-5343-a92a-b735752a4724\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\8ed04b60acc0f65f541292a9e598d5f73019c8db425f8d49ea5819eab16a42f3\"},{\"ID\":\"2973e760-dc59-5800-a3de-ab9d93be81e5\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\cc71305d45f09ce377ef497f28c3a74ee027c27f20657d2c4a5f157d2457cc75\"},{\"ID\":\"454a7d36-038c-5364-8a25-fa84091869d6\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\9e8cde1ce8f5de861a5f22841f1ab9bc53d5f606d06efeacf5177f340e8d54d0\"},{\"ID\":\"9b748c8c-69eb-55fb-a1c1-5688cac4efd8\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\8e02bf5404057055a71d542780a2bb2731be4b3707c01918ba969fb4d83b98ec\"},{\"ID\":\"bfde5c26-b33f-5424-9405-9d69c2fea4d0\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\77483cedfb0964008c33d92d306734e1fab3b5e1ebb27e898f58ccfd108108f2\"},{\"ID\":\"bdabfbf5-80d1-57f1-86f3-448ce97e2d05\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\aed2ebbb31ad24b38ee8521dd17744319f83d267bf7f360bc177e27ae9a006cf\"},{\"ID\":\"ad9b34f2-dcee-59ea-8962-b30704ae6331\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\d44d3a675fec1070b61d6ea9bacef4ac12513caf16bd6453f043eed2792f75d8\"}],\"HostName\":\"fabrikamfiber.web-789699744-rqv6p\",\"MappedDirectories\":[{\"HostPath\":\"c:\\\\var\\\\lib\\\\kubelet\\\\pods\\\\b50f0027-ad29-11e7-b16e-000d3afd2878\\\\volumes\\\\kubernetes.io~secret\\\\default-token-rw9dn\",\"ContainerPath\":\"c:\\\\var\\\\run\\\\secrets\\\\kubernetes.io\\\\serviceaccount\",\"ReadOnly\":true,\"BandwidthMaximum\":0,\"IOPSMaximum\":0}],\"HvPartition\":false,\"EndpointList\":null,\"NetworkSharedContainerName\":\"586870f5833279678773cb700db3c175afc81d557a75867bf39b43f985133d13\",\"Servicing\":false,\"AllowUnqualifiedDNSQuery\":false}"}
  33m           11s             151     kubelet, 38519acs9011                                           Warning         FailedSync              Error syncing pod
  32m           11s             139     kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Warning         BackOff                 Back-off restarting failed container
```

### <a name="mitigation---using-node-labels-and-nodeselector"></a>Lösung: Verwenden von Knotenlabels und -selektoren

Führen Sie **kubectl get node** aus, um eine Liste aller Knoten abzurufen. Danach können Sie **kubectl describe node (Knotenname)** ausführen, um weitere Details zu erhalten.

Im folgenden Beispiel werden auf zwei Windows-Knoten unterschiedliche Versionen ausgeführt:

```
$ kubectl get node

NAME                        STATUS    AGE       VERSION
38519acs9010                Ready     21h       v1.7.7-7+e79c96c8ff2d8e
38519acs9011                Ready     4h        v1.7.7-25+bc3094f1d650a2
k8s-linuxpool1-38519084-0   Ready     21h       v1.7.7
k8s-master-38519084-0       Ready     21h       v1.7.7

$ kubectl describe node 38519acs9010

Name:                   38519acs9010
Role:
Labels:                 beta.kubernetes.io/arch=amd64
                        beta.kubernetes.io/instance-type=Standard_D2_v2
                        beta.kubernetes.io/os=windows
                        failure-domain.beta.kubernetes.io/region=westus2
                        failure-domain.beta.kubernetes.io/zone=0
                        kubernetes.io/hostname=38519acs9010
Annotations:            node.alpha.kubernetes.io/ttl=0
                        volumes.kubernetes.io/controller-managed-attach-detach=true
Taints:                 <none>
CreationTimestamp:      Fri, 06 Oct 2017 01:41:02 +0000

...
  
System Info:
 Machine ID:                    38519acs9010
 System UUID:
 Boot ID:
 Kernel Version:                10.0 14393 (14393.1715.amd64fre.rs1_release_inmarket.170906-1810)
 OS Image:
 Operating System:              windows
 Architecture:                  amd64
 ...
 
$ kubectl describe node 38519acs9011
Name:                   38519acs9011
Role:
Labels:                 beta.kubernetes.io/arch=amd64
                        beta.kubernetes.io/instance-type=Standard_DS1_v2
                        beta.kubernetes.io/os=windows
                        failure-domain.beta.kubernetes.io/region=westus2
                        failure-domain.beta.kubernetes.io/zone=0
                        kubernetes.io/hostname=38519acs9011
Annotations:            node.alpha.kubernetes.io/ttl=0
                        volumes.kubernetes.io/controller-managed-attach-detach=true
Taints:                 <none>
CreationTimestamp:      Fri, 06 Oct 2017 18:13:25 +0000
Conditions:
...

System Info:
 Machine ID:                    38519acs9011
 System UUID:
 Boot ID:
 Kernel Version:                10.0 16299 (16299.0.amd64fre.rs3_release.170922-1354)
 OS Image:
 Operating System:              windows
 Architecture:                  amd64
...

```

Anhand dieses Beispiels möchten wir zeigen, wie die Versionen abgeglichen werden:

1. Notieren Sie sich die Namen der Knoten sowie die `Kernel Version` aus den Systeminformationen.

    In unserem Beispiel sehen die Informationen wie folgt aus:

    Name         | Version
    -------------|--------------------------------------------------------
    38519acs9010 | 14393.1715.amd64fre.rs1_release_inmarket.170906-1810
    38519acs9011 | 16299.0.amd64fre.rs3_release.170922-1354

2. Fügen Sie jedem Knoten namens `beta.kubernetes.io/osbuild` ein Label hinzu. Für Windows Server 2016 müssen sowohl Haupt- als auch Nebenversionen (in diesem Beispiel 14393.1715) ohne Hyper-V-Isolation unterstützt werden. Für Windows Server Version 1709 muss nur die Hauptversion (16299 in diesem Beispiel) übereinstimmen.

    In diesem Beispiel sieht der Befehl zum Hinzufügen der Label wie folgt aus:

    ```
    $ kubectl label node 38519acs9010 beta.kubernetes.io/osbuild=14393.1715


    node "38519acs9010" labeled
    $ kubectl label node 38519acs9011 beta.kubernetes.io/osbuild=16299

    node "38519acs9011" labeled

    ```

3. Überprüfen Sie, ob die Label vorhanden sind, indem Sie **kubectl get nodes --show-labels** ausführen.

    In diesem Beispiel sieht die Ausgabe wie folgt aus:

    ```
    $ kubectl get nodes --show-labels

    NAME                        STATUS                     AGE       VERSION                    LABELS
    38519acs9010                Ready,SchedulingDisabled   3d        v1.7.7-7+e79c96c8ff2d8e    beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=windows,beta.kubernetes.io/osbuild=14393.1715,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=38519acs9010
    38519acs9011                Ready                      3d        v1.7.7-25+bc3094f1d650a2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_DS1_v2,beta.kubernetes.io/os=windows,beta.kubernetes.io/osbuild=16299,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=38519acs9011
    k8s-linuxpool1-38519084-0   Ready                      3d        v1.7.7                     agentpool=linuxpool1,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=k8s-linuxpool1-38519084-0,kubernetes.io/role=agent
    k8s-master-38519084-0       Ready                      3d        v1.7.7                     beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=k8s-master-38519084-0,kubernetes.io/role=master
    ```

4. Fügen Sie den Bereitstellungen Knotenselektoren hinzu. In diesem Beispiel fügen Sie einen `nodeSelector` der Containerspezifikation mit `beta.kubernetes.io/os` = windows und `beta.kubernetes.io/osbuild` = 14393.* oder 16299 hinzu, um die Basisbetriebsversion des Containers abzugleichen.

    Hier ein vollständiges Beispiel für die Ausführung eines Containers, der für Windows Server 2016 erstellt wurde:

    ```yaml
    apiVersion: extensions/v1beta1
    kind: Deployment
    metadata:
      annotations:
        kompose.cmd: kompose convert -f docker-compose-combined.yml
        kompose.version: 1.2.0 (99f88ef)
      creationTimestamp: null
      labels:
        io.kompose.service: fabrikamfiber.web
      name: fabrikamfiber.web
    spec:
      replicas: 1
      strategy: {}
      template:
        metadata:
          creationTimestamp: null
          labels:
            io.kompose.service: fabrikamfiber.web
        spec:
          containers:
          - image: patricklang/fabrikamfiber.web:latest
            name: fabrikamfiberweb
            ports:
            - containerPort: 80
            resources: {}
          restartPolicy: Always
          nodeSelector:
            "beta.kubernetes.io/os": windows
            "beta.kubernetes.io/osbuild": "14393.1715"
    status: {}
    ```

    Der Pod kann nun mit der aktualisierten Bereitstellung starten. Die Knotenselektoren werden ebenfalls in `kubectl describe pod <podname>` angezeigt, damit Sie diesen Befehl ausführen und prüfen können, ob sie hinzugefügt wurden.

    Die Ausgabe für unser Beispiel lautet wie folgt:

    ```
    $ kubectl -n plang describe po fa

    Name:           fabrikamfiber.web-1780117715-5c8vw
    Namespace:      plang
    Node:           38519acs9010/10.240.0.4
    Start Time:     Tue, 10 Oct 2017 01:43:28 +0000
    Labels:         io.kompose.service=fabrikamfiber.web
                    pod-template-hash=1780117715
    Annotations:    kubernetes.io/created-by={"kind":"SerializedReference","apiVersion":"v1","reference":{"kind":"ReplicaSet","namespace":"plang","name":"fabrikamfiber.web-1780117715","uid":"6a07aaf3-ad5c-11e7-b16e-000d3...
    Status:         Running
    IP:             10.244.1.84
    Created By:     ReplicaSet/fabrikamfiber.web-1780117715
    Controlled By:  ReplicaSet/fabrikamfiber.web-1780117715
    Containers:
      fabrikamfiberweb:
        Container ID:       docker://c94594fb53161f3821cf050d9af7546991aaafbeab41d333d9f64291327fae13
        Image:              patricklang/fabrikamfiber.web:latest
        Image ID:           docker-pullable://patricklang/fabrikamfiber.web@sha256:562741016ce7d9a232a389449a4fd0a0a55aab178cf324144404812887250ead
        Port:               80/TCP
        State:              Running
          Started:          Tue, 10 Oct 2017 01:43:42 +0000
        Ready:              True
        Restart Count:      0
        Environment:        <none>
        Mounts:
          /var/run/secrets/kubernetes.io/serviceaccount from default-token-rw9dn (ro)
    Conditions:
      Type          Status
      Initialized   True
      Ready         True
      PodScheduled  True
    Volumes:
      default-token-rw9dn:
        Type:       Secret (a volume populated by a Secret)
        SecretName: default-token-rw9dn
        Optional:   false
    QoS Class:      BestEffort
    Node-Selectors: beta.kubernetes.io/os=windows
                    beta.kubernetes.io/osbuild=14393.1715
    Tolerations:    <none>
    Events:
      FirstSeen     LastSeen        Count   From                    SubObjectPath                           Type            Reason                  Message
      ---------     --------        -----   ----                    -------------                           --------        ------                  -------
      5m            5m              1       default-scheduler                                               Normal          Scheduled               Successfully assigned fabrikamfiber.web-1780117715-5c8vw to 38519acs9010
      5m            5m              1       kubelet, 38519acs9010                                           Normal          SuccessfulMountVolume   MountVolume.SetUp succeeded for volume "default-token-rw9dn"
      5m            5m              1       kubelet, 38519acs9010   spec.containers{fabrikamfiberweb}       Normal          Pulling                 pulling image "patricklang/fabrikamfiber.web:latest"
      5m            5m              1       kubelet, 38519acs9010   spec.containers{fabrikamfiberweb}       Normal          Pulled                  Successfully pulled image "patricklang/fabrikamfiber.web:latest"
      5m            5m              1       kubelet, 38519acs9010   spec.containers{fabrikamfiberweb}       Normal          Created                 Created container
      5m            5m              1       kubelet, 38519acs9010   spec.containers{fabrikamfiberweb}       Normal          Started                 Started container
    ```
