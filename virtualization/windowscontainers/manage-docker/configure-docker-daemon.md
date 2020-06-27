---
title: Konfigurieren von Docker unter Windows
description: Konfigurieren von Docker unter Windows
keywords: Docker, Container
author: PatrickLang
ms.date: 05/03/2019
ms.topic: overview
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 6885400c-5623-4cde-8012-f6a00019fafa
ms.openlocfilehash: b0a89bfcae6a78c28603444a682cecf7c667e477
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192717"
---
# <a name="docker-engine-on-windows"></a>Docker-Modul unter Windows

Die Docker-Engine und der Docker-Client sind nicht im Lieferumfang von Windows enthalten und müssen einzeln installiert und konfiguriert werden. Das Docker-Modul kann zudem zahlreiche verschiedene Konfigurationen akzeptieren. Beispielsweise kann konfiguriert werden, wie der Daemon eingehende Anforderungen akzeptiert, und Sie können standardmäßige Netzwerkoptionen sowie Einstellungen für Debugging und Protokolle konfigurieren. Unter Windows können diese Konfigurationen in einer Konfigurationsdatei oder mit dem Windows-Dienststeuerungs-Manager angegeben werden. In diesem Dokument wird beschrieben, wie Sie die Docker-Engine installieren und konfigurieren, und es werden einige Beispiele für gängige Konfigurationen vorgestellt.

## <a name="install-docker"></a>Installieren von Docker

Für die Arbeit mit Windows-Containern ist Docker erforderlich. Docker besteht aus dem Docker-Modul (dockerd.exe) und dem Docker-Client (docker.exe). Eine Beschreibung der einfachsten Möglichkeit, dies alles zu installieren, finden Sie im Schnellstartleitfaden, der Ihnen hilft, alles einzurichten und ihren ersten Container auszuführen.

- [Installieren von Docker](../quick-start/set-up-environment.md)

Informationen zu skriptbasierten Installationen finden Sie unter [Verwenden eines Skripts zum Installieren von Docker EE](https://docs.docker.com/install/windows/docker-ee/#use-a-script-to-install-docker-ee).

Bevor Sie Docker verwenden können, müssen Sie die Containerimages installieren. Weitere Informationen finden Sie unter [Dokumente zu Containerbasisimages](../manage-containers/container-base-images.md).

## <a name="configure-docker-with-a-configuration-file"></a>Konfigurieren von Docker mit einer Konfigurationsdatei

Die bevorzugte Methode zum Konfigurieren des Docker-Moduls in Windows ist die Verwendung einer Konfigurationsdatei. Die Konfigurationsdatei finden Sie unter „C:\ProgramData\Docker\config\daemon.json”. Sie können diese Datei erstellen, wenn sie noch nicht vorhanden ist.

>[!NOTE]
>Nicht jede verfügbare Docker-Konfigurationsoption ist für Docker unter Windows anwendbar. Im folgenden Beispiel werden die Konfigurationsoptionen gezeigt, die anwendbar sind. Weitere Informationen zur Konfiguration der Docker-Engine finden Sie auf der Docker-Seite unter [Daemon Configuration File (Daemon-Konfigurationsdatei)](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file).

```json
{
    "authorization-plugins": [],
    "dns": [],
    "dns-opts": [],
    "dns-search": [],
    "exec-opts": [],
    "storage-driver": "",
    "storage-opts": [],
    "labels": [],
    "log-driver": "",
    "mtu": 0,
    "pidfile": "",
    "data-root": "",
    "cluster-store": "",
    "cluster-advertise": "",
    "debug": true,
    "hosts": [],
    "log-level": "",
    "tlsverify": true,
    "tlscacert": "",
    "tlscert": "",
    "tlskey": "",
    "group": "",
    "default-ulimits": {},
    "bridge": "",
    "fixed-cidr": "",
    "raw-logs": false,
    "registry-mirrors": [],
    "insecure-registries": [],
    "disable-legacy-registry": false
}
```

Der Konfigurationsdatei müssen nur die gewünschten Konfigurationsänderungen hinzugefügt werden. Im folgenden Beispiel wird die Docker-Engine so konfiguriert, dass eingehende Verbindungen über Port 2375 akzeptiert werden. Für alle weiteren Konfigurationsoptionen werden die Standardwerte verwendet.

```json
{
    "hosts": ["tcp://0.0.0.0:2375"]
}
```

Entsprechend konfiguriert das folgende Beispiel den Docker-Daemon, um Images und Container in einem alternativen Pfad zu speichern. Ohne Angabe wird standardmäßig `c:\programdata\docker` verwendet.

```json
{   
    "data-root": "d:\\docker"
}
```

Ähnlich wird im folgenden Beispiel der Docker-Daemon so konfiguriert, dass nur sichere Verbindungen über Port 2376 akzeptiert werden.

```json
{
    "hosts": ["tcp://0.0.0.0:2376", "npipe://"],
    "tlsverify": true,
    "tlscacert": "C:\\ProgramData\\docker\\certs.d\\ca.pem",
    "tlscert": "C:\\ProgramData\\docker\\certs.d\\server-cert.pem",
    "tlskey": "C:\\ProgramData\\docker\\certs.d\\server-key.pem",
}
```

## <a name="configure-docker-on-the-docker-service"></a>Konfigurieren von Docker über den Docker-Dienst

Die Docker-Engine kann auch konfiguriert werden, indem der Docker-Dienst mit `sc config` geändert wird. Bei Verwendung dieser Methode werden die Flags des Docker-Moduls direkt im Docker-Dienst festgelegt. Führen Sie den folgenden Befehl in einem Eingabeaufforderungsfenster („cmd.exe“, nicht PowerShell) aus:

```cmd
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

>[!NOTE]
>Sie müssen diesen Befehl nicht ausführen, wenn Ihre Datei „daemon.json“ den Eintrag `"hosts": ["tcp://0.0.0.0:2375"]` bereits enthält.

## <a name="common-configuration"></a>Allgemeine Konfiguration

Die folgenden Beispiele für Konfigurationsdateien zeigen allgemeine Docker-Konfigurationen. Diese können in einer einzigen Konfigurationsdatei kombiniert werden.

### <a name="default-network-creation"></a>Erstellen eines Standardnetzwerks

Verwenden Sie die folgende Konfiguration, um die Docker-Engine so zu konfigurieren, dass kein NAT-Standardnetzwerk erstellt wird.

```json
{
    "bridge" : "none"
}
```

Weitere Informationen finden Sie unter [Verwalten von Docker-Netzwerken](../container-networking/network-drivers-topologies.md).

### <a name="set-docker-security-group"></a>Festlegen der Docker-Sicherheitsgruppe

Wenn Sie beim Docker-Host angemeldet sind und Docker-Befehle lokal ausführen, werden diese Befehle über eine Named Pipe ausgeführt. Standardmäßig können nur Mitglieder der Administratorengruppe über die Named Pipe auf das Docker-Modul zugreifen. Verwenden Sie das Flag `group`, um eine Sicherheitsgruppe mit diesem Zugriff festzulegen.

```json
{
    "group" : "docker"
}
```

## <a name="proxy-configuration"></a>Proxykonfiguration

Erstellen Sie eine Windows-Umgebungsvariable namens `HTTP_PROXY` oder `HTTPS_PROXY` und den Wert der Proxyinformationen, um Proxyinformationen für `docker search` und `docker pull` festzulegen. Sie können dies in PowerShell tun, indem Sie einen Befehl ähnlich dem folgenden verwenden:

```powershell
[Environment]::SetEnvironmentVariable("HTTP_PROXY", "http://username:password@proxy:port/", [EnvironmentVariableTarget]::Machine)
```

Starten Sie den Docker-Dienst neu, sobald die Variable festgelegt wurde.

```powershell
Restart-Service docker
```

Weitere Informationen finden Sie unter [Windows Configuration File (Windows-Konfigurationsdatei)](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file) auf Docker.com.

## <a name="how-to-uninstall-docker"></a>Deinstallieren von Docker

In diesem Abschnitt erfahren Sie, wie Sie Docker deinstallieren und eine vollständige Bereinigung der Docker-Systemkomponenten aus dem Windows 10- oder Windows Server 2016-System durchführen.

>[!NOTE]
>Alle Befehle in diesen Anleitungen müssen aus einer PowerShell-Sitzung mit erhöhten Rechten ausgeführt werden.

### <a name="prepare-your-system-for-dockers-removal"></a>Vorbereiten des Systems für das Entfernen von Docker

Stellen Sie vor der Deinstallation von Docker sicher, dass auf Ihrem System aktuell keine Container ausgeführt werden.

Führen Sie die folgenden Cmdlets aus, um die Ausführung von Containern zu überprüfen:

```powershell
# Leave swarm mode (this will automatically stop and remove services and overlay networks)
docker swarm leave --force

# Stop all running containers
docker ps --quiet | ForEach-Object {docker stop $_}
```

Es empfiehlt sich ebenfalls, alle Container, Containerimages, Netzwerke und Volumes aus dem System zu entfernen, bevor Sie Docker entfernen. Verwenden Sie dazu das folgende Cmdlet:

```powershell
docker system prune --volumes --all
```

### <a name="uninstall-docker"></a>Deinstallieren von Docker

Im nächsten Schritt müssen Sie Docker deinstallieren.

So deinstallieren Sie Docker unter Windows 10

- Navigieren Sie zu **Einstellungen** > **Apps** auf Ihrem Windows 10-Computer.
- Suchen Sie unter **Apps und Features** nach **Docker für Windows**.
- Navigieren Sie zu **Docker für Windows** > **Deinstallieren**.

So deinstallieren Sie Docker unter Windows Server 2016:

Verwenden Sie in einer PowerShell-Sitzung mit erhöhten Rechten die Cmdlets **Uninstall-Package** und **Uninstall-Module**, um das Docker-Modul und den zugehörigen Anbieter für die Paketverwaltung von Ihrem System zu entfernen. Das folgende Beispiel zeigt dies:

```powershell
Uninstall-Package -Name docker -ProviderName DockerMsftProvider
Uninstall-Module -Name DockerMsftProvider
```

>[!TIP]
>Sie können den Anbieter des Pakets, den Sie zum Installieren von Docker verwendet haben, mit `PS C:\> Get-PackageProvider -Name *Docker*` finden.

### <a name="clean-up-docker-data-and-system-components"></a>Bereinigen von Docker-Daten und Systemkomponenten

Nachdem Sie Docker deinstalliert haben, müssen Sie die Standardnetzwerke von Docker entfernen, damit deren Konfiguration nach dem Entfernen von Docker nicht auf Ihrem System verbleibt. Verwenden Sie dazu das folgende Cmdlet:

```powershell
Get-HNSNetwork | Remove-HNSNetwork
```

Führen Sie das folgende Cmdlet aus, um die Programmdaten von Docker aus dem System zu entfernen:

```powershell
Remove-Item "C:\ProgramData\Docker" -Recurse
```

Sie können die optionalen Features von Windows ebenfalls entfernen, die Docker/Containern unter Windows zugeordnet sind.

Dies beinhaltet das Feature „Container“, das automatisch unter Windows 10 oder Windows Server 2016 aktiviert wird, wenn Docker installiert ist. Es kann ebenfalls das Feature "Hyper-V" beinhalten, das unter Windows 10 automatisch aktiviert wird, wenn Docker installiert ist. Dies muss jedoch explizit auf Windows Server 2016 aktiviert werden.

>[!IMPORTANT]
>[Das Hyper-V-Feature](https://docs.microsoft.com/virtualization/hyper-v-on-windows/about/) ist ein allgemeines Virtualisierungsfeature, das viel mehr als nur Container aktiviert. Stellen Sie vor dem Deaktivieren des Hyper-V-Features sicher, dass keine anderen virtualisierten Komponenten auf dem System vorhanden sind, die Hyper-V erfordern.

So entfernen Sie Windows-Features unter Windows 10:

- Navigieren Sie zu **Systemsteuerung** > **Programme** > **Programme und Features** > **Windows-Features aktivieren oder deaktivieren**.
- Suchen Sie den Namen der Funktion/en, die Sie deaktivieren möchten. In diesem Fall handelt es sich um **Container** und (optional) **Hyper-V**.
- Deaktivieren Sie das Kontrollkästchen neben dem Namen des Features, das Sie deaktivieren möchten.
- Wählen Sie **OK** aus.

So entfernen Sie Windows-Features unter Windows Server 2016:

Führen Sie die folgenden Cmdlets aus einer PowerShell-Sitzung mit erhöhten Rechten zum Deaktivieren der Features **Container** und (optional) **Hyper-V** im System aus:

```powershell
Remove-WindowsFeature Containers
Remove-WindowsFeature Hyper-V
```

### <a name="reboot-your-system"></a>Neustarten des Systems

Führen Sie das folgende Cmdlet aus einer PowerShell-Sitzung mit erhöhten Rechten aus, um das System neu zu starten und die Installation und Bereinigung abzuschließen:

```powershell
Restart-Computer -Force
```
