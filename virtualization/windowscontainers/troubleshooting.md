---
title: Problembehandlung für Windows-Container
description: Problembehandlung, Tipps, automatisierte Skripts sowie protokollierte Informationen für Windows-Container und Docker
keywords: Docker, Container, Problembehandlung, Protokolle
author: PatrickLang
ms.date: 12/19/2016
ms.topic: article
ms.assetid: ebd79cd3-5fdd-458d-8dc8-fc96408958b5
ms.openlocfilehash: b408582df4aad340a575029d6f7bb8a997ffadbf
ms.sourcegitcommit: 186ebcd006eeafb2b51a19787d59914332aad361
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 08/07/2020
ms.locfileid: "87984704"
---
# <a name="troubleshooting"></a>Problembehandlung

Haben Sie Probleme beim Einrichten Ihres Computers oder beim Ausführen eines Containers? Wir haben ein PowerShell-Skript entwickelt, das nach häufigen Problemen sucht. Bitte probieren Sie es erst aus, um zu sehen, was es findet, und geben Sie Ihre Ergebnisse frei.

```PowerShell
Invoke-WebRequest https://aka.ms/Debug-ContainerHost.ps1 -UseBasicParsing | Invoke-Expression
```
Eine Liste aller Tests, die das Skript ausführt sowie allgemeine Lösungen, finden Sie in der [Readme-Datei](https://github.com/Microsoft/Virtualization-Documentation/blob/live/windows-server-container-tools/Debug-ContainerHost/README.md) des Skripts.

Wenn das nicht hilft, suchen Sie die Quelle des Problems, posten Sie die Ausgabe Ihres Skripts im [Containerforum](https://social.msdn.microsoft.com/Forums/home?forum=windowscontainers). Dies ist der beste Ort, um Hilfe von der Community zu erhalten, zu der auch Windows Insiders und Entwickler gehören.


### <a name="finding-logs"></a>Suchen von Protokollen
Es gibt mehrere Dienste, die zum Verwalten von Windows-Containern verwendet werden. Im nächsten Abschnitt wird gezeigt, wie Sie Protokolle für jeden Dienst erhalten.

## <a name="docker-container-logs"></a>Docker-Container Protokolle
Der `docker logs` Befehl ruft die Protokolle eines Containers aus "stdout/stderr", den standardmäßigen Anwendungsprotokoll-Speicherorten für Linux-Anwendungen, ab. Windows-Anwendungen melden sich in der Regel nicht bei stdout/stderr an. Stattdessen melden Sie sich unter anderem bei etw, Ereignisprotokollen oder Protokolldateien an.

Der [Protokoll Monitor](https://github.com/microsoft/windows-container-tools/tree/master/LogMonitor), ein von Microsoft unterstütztes Open Source-Tool, ist jetzt auf GitHub verfügbar. Der Protokoll Monitor Bridges Windows-Anwendungsprotokolle auf stdout/stderr. Der Protokoll Monitor wird über eine Konfigurationsdatei konfiguriert.

### <a name="log-monitor-usage"></a>Protokoll Monitor Verwendung

LogMonitor.exe und LogMonitorConfig.json sollten beide in demselben LogMonitor-Verzeichnis enthalten sein.

Der Protokoll Monitor kann entweder in einem shellnutzungsmuster verwendet werden:

```
SHELL ["C:\\LogMonitor\\LogMonitor.exe", "cmd", "/S", "/C"]
CMD c:\windows\system32\ping.exe -n 20 localhost
```

Oder ein entryPoint-Verwendungs Muster:

```
ENTRYPOINT C:\LogMonitor\LogMonitor.exe c:\windows\system32\ping.exe -n 20 localhost
```

Beide Beispiel Verwendungen wrappen die ping.exe Anwendung. Andere Anwendungen (z [. b. IIS). Servicemonitor]( https://github.com/microsoft/IIS.ServiceMonitor)) kann auf ähnliche Weise mit dem Protokoll Monitor (Log Monitor) eingefügt werden:

```
COPY LogMonitor.exe LogMonitorConfig.json C:\LogMonitor\
WORKDIR /LogMonitor
SHELL ["C:\\LogMonitor\\LogMonitor.exe", "powershell.exe"]

# Start IIS Remote Management and monitor IIS
ENTRYPOINT      Start-Service WMSVC; `
                    C:\ServiceMonitor.exe w3svc;
```


Der Protokoll Monitor startet die umschließende Anwendung als untergeordneten Prozess und überwacht die stdout-Ausgabe der Anwendung.

Beachten Sie, dass im shellnutzungsmuster die CMD-/EntryPoint-Anweisung im shellformular und nicht in der exec-Form angegeben werden sollte. Wenn die Exec-Form der cmd-/EntryPoint-Anweisung verwendet wird, wird die Shell nicht gestartet, und das Protokoll Monitor Tool wird nicht innerhalb des Containers gestartet.

Weitere Informationen zur Verwendung finden Sie im [Log Monitor-wiki](https://github.com/microsoft/windows-container-tools/wiki). Beispiel Konfigurationsdateien für wichtige Windows-Container Szenarien (IIS usw.) finden Sie im [GitHub](https://github.com/microsoft/windows-container-tools/tree/master/LogMonitor/src/LogMonitor/sample-config-files)-Repository. Zusätzlichen Kontext finden Sie in diesem [Blogbeitrag](https://techcommunity.microsoft.com/t5/Containers/Windows-Containers-Log-Monitor-Opensource-Release/ba-p/973947).

## <a name="docker-engine"></a>Docker-Engine
Die Docker-Engine protokolliert in das Windows-„Anwendungsereignisprotokoll“, statt in eine Datei. Diese Protokolle können mithilfe von Windows PowerShell einfach gelesen, sortiert und gefiltert werden.

Beispielsweise werden dadurch die Protokolle der Docker-Engine der letzten fünf Minuten angezeigt, angefangen mit dem ältesten.

```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-5) | Sort-Object Time
```

Dies könnte auch einfach in eine CSV-Datei weitergeleitet werden, um dort von einem anderen Tool oder Arbeitsblatt gelesen zu werden.

```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-30)  | Sort-Object Time | Export-CSV ~/last30minutes.CSV
```

### <a name="enabling-debug-logging"></a>Aktivieren der Debugprotokollierung
Sie können auch die Protokollierung auf Debugebene in der Docker-Engine aktivieren. Dies kann für die Problembehandlung hilfreich sein, wenn reguläre Protokolle nicht genügend Informationen enthalten.

Öffnen Sie zunächst eine Eingabeaufforderung mit erhöhten Rechten, führen Sie dann `sc.exe qc docker` aus, um die aktuelle Befehlszeile für den Docker-Dienst zu erhalten.
Beispiel:
```
C:\> sc.exe qc docker
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: docker
        TYPE               : 10  WIN32_OWN_PROCESS
        START_TYPE         : 2   AUTO_START
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : "C:\Program Files\Docker\dockerd.exe" --run-service
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : Docker Engine
        DEPENDENCIES       :
        SERVICE_START_NAME : LocalSystem
```

Modifizieren Sie den aktuellen `BINARY_PATH_NAME`:
- Fügen Sie -D am Ende hinzu
- Fügen Sie für " das Escapezeichen \ hinzu
- Schließen Sie den gesamten Befehl in Anführungszeichen (") ein

Führen Sie anschließend `sc.exe config docker binpath=`, gefolgt von der neuen Zeichenfolge, aus. Zum Beispiel:
```
sc.exe config docker binpath= "\"C:\Program Files\Docker\dockerd.exe\" --run-service -D"
```


Starten Sie nun den Docker-Dienst neu.
```
sc.exe stop docker
sc.exe start docker
```

Dadurch wird viel mehr im Anwendungsereignisprotokoll protokolliert. Deshalb ist es von Vorteil, die Option `-D` zu entfernen, wenn Sie mit der Problembehandlung fertig sind. Führen Sie die gleichen Schritte wie oben aus, jedoch ohne `-D`, und starten Sie den Dienst neu, um die Debugprotokollierung zu deaktivieren.

Eine andere Möglichkeit ist, den Docker Daemon im Debugmodus mithilfe einer PowerShell-Eingabeaufforderung mit erhöhten Rechten auszuführen, die die Ausgabe direkt in einer Datei aufzeichnen.
```PowerShell
sc.exe stop docker
<path\to\>dockerd.exe -D > daemon.log 2>&1
```

### <a name="obtaining-stack-dump"></a>Abrufen von Stapel Abbildern

Im Allgemeinen ist dies nur hilfreich, wenn Sie vom Microsoft Support explizit angefordert werden, oder docker-Entwickler. Sie kann verwendet werden, um die Diagnose einer Situation zu unterstützen, in der Docker anscheinend nicht reagiert hat.

Herunterladen von [Docker-Signal.exe](https://github.com/moby/docker-signal).

Syntax:
```PowerShell
docker-signal --pid=$((Get-Process dockerd).Id)
```

Die Ausgabedatei befindet sich im Daten Stammverzeichnis, in dem docker ausgeführt wird. Das Standardverzeichnis ist `C:\ProgramData\Docker`. Das aktuelle Verzeichnis kann durch Ausführen von `docker info -f "{{.DockerRootDir}}"` bestätigt werden.

Die Datei ist `goroutine-stacks-<timestamp>.log` .

Beachten Sie, dass `goroutine-stacks*.log` keine persönlichen Informationen enthält.


## <a name="host-compute-service"></a>Hostcomputedienst
Das Docker-Modul ist von einem Windows-spezifischen Hostcomputedienst abhängig. Dieser verfügt über separate Protokolle:
- Microsoft-Windows-Hyper-V-Compute-Admin
- Microsoft-Windows-Hyper-V-Compute-Operational

Sie sind in Ereignisanzeige sichtbar und können auch mit PowerShell abgefragt werden.

Zum Beispiel:
```PowerShell
Get-WinEvent -LogName Microsoft-Windows-Hyper-V-Compute-Admin
Get-WinEvent -LogName Microsoft-Windows-Hyper-V-Compute-Operational
```

### <a name="capturing-hcs-analyticdebug-logs"></a>Erfassen von HCS Analyse-/Debug-Protokollen

Um Analyse-/Debug-Protokolle für Hyper-V-Compute zu aktivieren und auf `hcslog.evtx` zu speichern.

```PowerShell
# Enable the analytic logs
wevtutil.exe sl Microsoft-Windows-Hyper-V-Compute-Analytic /e:true /q:true

# <reproduce your issue>

# Export to an evtx
wevtutil.exe epl Microsoft-Windows-Hyper-V-Compute-Analytic <hcslog.evtx>

# Disable
wevtutil.exe sl Microsoft-Windows-Hyper-V-Compute-Analytic /e:false /q:true
```

### <a name="capturing-hcs-verbose-tracing"></a>Aufzeichnen ausführlicher HCS-Protokollierung

In der Regel ist sie nur dann nützlich, wenn sie vom Microsoft Support angefordert wird.

[HcsTraceProfile.wprp](https://github.com/MicrosoftDocs/Virtualization-Documentation/blob/master/windows-server-container-tools/wpr-profiles/HcsTraceProfile.wprp) herunterladen

```PowerShell
# Enable tracing
wpr.exe -start HcsTraceProfile.wprp!HcsArgon -filemode

# <reproduce your issue>

# Capture to HcsTrace.etl
wpr.exe -stop HcsTrace.etl "some description"
```

Bereitstellen von `HcsTrace.etl` an die Supportkontaktperson.
