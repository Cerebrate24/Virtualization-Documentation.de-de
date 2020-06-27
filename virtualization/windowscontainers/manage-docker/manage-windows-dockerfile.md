---
title: Dockerfile und Windows-Container
description: Erstellen Sie Dockerfile-Dateien für Windows-Container.
keywords: Docker, Container
author: PatrickLang
ms.date: 05/03/2019
ms.topic: tutorial
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 75fed138-9239-4da9-bce4-4f2e2ad469a1
ms.openlocfilehash: cda05088f632cee1f5e902bf958fd85f4eddaeec
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192557"
---
# <a name="dockerfile-on-windows"></a>Dockerfile unter Windows

Die Docker-Engine umfasst Tools, die das Erstellen von Containerimages automatisieren. Containerimages können zwar manuell mit dem Befehl `docker commit` erstellt werden, doch die Verwendung eines automatisierten Imageerstellungsprozesses bietet viele Vorteile, unter anderem:

- Speichern von Containerimages als Code.
- Schnelle und präzise Neuerstellung von Containerimages für Wartung und Aktualisierung.
- Fortlaufende Integration von Containerimages in den Entwicklungszyklus.

Die Docker-Komponenten, die diese Automatisierung steuern, sind die Dockerfile-Datei und der `docker build`-Befehl.

Die Dockerfile-Datei ist eine Textdatei mit Anweisungen, die erforderlich sind, um ein neues Containerimage zu erstellen. Diese Anweisungen beinhalten die Identifikation eines vorhandenen Images, das als Basis verwendet werden kann, Befehle, die während des Erstellungsprozesses ausgeführt werden, und einen Befehl, der bei der Bereitstellung neuer Instanzen des Containerimages ausgeführt wird.

„Docker Build“ ist der Befehl der Docker-Engine, der eine Dockerfile-Datei nutzt und den Imageerstellungsprozess auslöst.

In diesem Thema erfahren Sie, wie Sie Dockerfile-Dateien mit Windows-Containern verwenden, die grundlegende Syntax verstehen und die gängigsten Dockerfile-Anweisungen verwenden.

In diesem Dokument wird das Konzept von Containerimages und Containerimageebenen erörtert. Weitere Informationen zu Images und Imageebenen finden Sie unter [Containerbasisimages](../manage-containers/container-base-images.md).

Umfassende Informationen zu Dockerfile-Dateien finden Sie in der [Dockerfile-Referenz](https://docs.docker.com/engine/reference/builder/).

## <a name="basic-syntax"></a>Grundlegende Syntax

In ihrer grundlegendsten Form kann eine Dockerfile-Datei sehr einfach sein. Das folgende Beispiel erstellt ein neues Image, das IIS und eine „Hello World“-Website beinhaltet. Dieses Beispiel enthält Kommentare (mit `#` gekennzeichnet), die jeden Schritt erläutern. Die nachfolgenden Abschnitte dieses Artikels gehen ausführlicher auf Dockerfile-Syntaxregeln und -Anweisungen ein.

>[!NOTE]
>Eine Dockerfile-Datei muss ohne Erweiterung erstellt werden. In Windows erstellen Sie die Datei mit einem Editor Ihrer Wahl und speichern sie dann als "Dockerfile" (einschließlich der Anführungszeichen).

```dockerfile
# Sample Dockerfile

# Indicates that the windowsservercore image will be used as the base image.
FROM mcr.microsoft.com/windows/servercore:ltsc2019

# Metadata indicating an image maintainer.
LABEL maintainer="jshelton@contoso.com"

# Uses dism.exe to install the IIS role.
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart

# Creates an HTML file and adds content to this file.
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html

# Sets a command or process that will run each time a container is run from the new image.
CMD [ "cmd" ]
```

Weitere Beispiele für Dockerfile-Dateien für Windows finden Sie im [Repository „Dockerfile for Windows“](https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-container-samples).

## <a name="instructions"></a>Anweisungen

Dockerfile-Anweisungen informieren die Docker-Engine über die erforderlichen Schritte zum Erstellen eines Containerimages. Diese Anweisungen werden einzeln der Reihe nach ausgeführt. In den folgenden Beispielen finden Sie die am häufigsten verwendeten Anweisungen in Dockerfile-Dateien. Eine umfassende Liste der Dockerfile-Anweisungen finden Sie in der [Dockerfile-Referenz](https://docs.docker.com/engine/reference/builder/).

### <a name="from"></a>FROM

Die `FROM`-Anweisung legt das Containerimage fest, das während der Erstellung des neuen Images verwendet wird. Beispielsweise wird bei Verwendung von Anweisung `FROM mcr.microsoft.com/windows/servercore` das resultierende Image vom Windows Server Core-Basisbetriebssystemimage abgeleitet und ist davon abhängig. Ist das angegebene Image nicht auf dem System vorhanden, auf dem der „Docker Build“-Prozess ausgeführt wird, versucht das Docker-Modul, das Image von einer öffentlichen oder privaten Imageregistrierung herunterzuladen.

Das Format der FROM-Anweisung sieht wie folgt aus:

```dockerfile
FROM <image>
```

Hier sehen Sie ein Beispiel für den FROM-Befehl:

Laden Sie die Version ltsc2019 von Windows Server Core aus Microsoft Container Registry (MCR) herunter:
```
FROM mcr.microsoft.com/windows/servercore:ltsc2019
```

Weitere Informationen finden Sie in der [FROM-Referenz](https://docs.docker.com/engine/reference/builder/#from).

### <a name="run"></a>RUN

Die `RUN`-Anweisung gibt Befehle an, die ausgeführt und im neuen Containerimage erfasst werden sollen. Diese Befehle können Elemente wie das Installieren von Software sowie das Erstellen von Dateien, Verzeichnissen und Umgebungskonfigurationen enthalten.

Die RUN-Anweisung sieht wie folgt aus:

```dockerfile
# exec form

RUN ["<executable>", "<param 1>", "<param 2>"]

# shell form

RUN <command>
```

Das Exec- und Shell-Format unterscheiden sich darin, wie die `RUN`-Anweisung ausgeführt wird. Bei Verwendung des Exec-Formats wird das angegebene Programm explizit ausgeführt.

Das folgende Beispiel zeigt das Exec-Format:

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN ["powershell", "New-Item", "c:/test"]
```

Das sich ergebende Image führt den Befehl `powershell New-Item c:/test` aus:

```dockerfile
docker history doc-exe-method

IMAGE               CREATED             CREATED BY                    SIZE                COMMENT
b3452b13e472        2 minutes ago       powershell New-Item c:/test   30.76 MB
```

Im Gegensatz dazu wird im folgenden Beispiel der gleiche Vorgang im Shell-Format ausgeführt:

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell New-Item c:\test
```

Das sich ergebende Image weist eine Ausführungsanweisung `cmd /S /C powershell New-Item c:\test` auf.

```dockerfile
docker history doc-shell-method

IMAGE               CREATED             CREATED BY                              SIZE                COMMENT
062a543374fc        19 seconds ago      cmd /S /C powershell New-Item c:\test   30.76 MB
```

### <a name="considerations-for-using-run-with-windows"></a>Überlegungen zur Verwendung von RUN mit Windows

Wenn unter Windows die `RUN`-Anweisung mit dem Exec-Format verwendet wird, müssen umgekehrte Schrägstriche mit Escapezeichen versehen werden.

```dockerfile
RUN ["powershell", "New-Item", "c:\\test"]
```

Wenn das Zielprogramm ein Windows-Installer ist, müssen Sie das Setup über das `/x:<directory>`-Flag extrahieren, bevor Sie das eigentliche (unbeaufsichtigte) Installationsverfahren starten können. Sie müssen außerdem warten, bis der Befehl beendet wird, bevor Sie eine andere Aktion ausführen. Andernfalls endet der Vorgang vorzeitig, ohne dass etwas installiert wurde. Weitere Informationen finden Sie im folgenden Beispiel.

#### <a name="examples-of-using-run-with-windows"></a>Beispiele für die Verwendung von RUN mit Windows

Die folgende Dockerfile-Beispieldatei verwendet DISM zum Installieren von IIS im Containerimage:

```dockerfile
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart
```

In diesem Beispiel wird das verteilbare Paket von Visual Studio installiert. `Start-Process` und der Parameter `-Wait` werden verwendet, um das Installationsprogramm auszuführen. Dadurch wird gewährleistet, dass die Installation abgeschlossen wird, bevor in der Dockerfile-Datei mit der nächsten Anweisung begonnen wird.

```dockerfile
RUN powershell.exe -Command Start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait
```

Ausführliche Informationen zur RUN-Anweisung finden Sie in der [RUN-Referenz](https://docs.docker.com/engine/reference/builder/#run).

### <a name="copy"></a>KOPIEREN

Die `COPY`-Anweisung kopiert Dateien und Verzeichnisse in das Dateisystem des Containers. Die Dateien und Verzeichnisse müssen sich in einem zur Dockerfile-Datei relativen Pfad befinden.

Das Format der `COPY`-Anweisung sieht wie folgt aus:

```dockerfile
COPY <source> <destination>
```

Wenn die Quelle oder das Ziel Leerzeichen enthalten, schließen Sie den Pfad in eckige Klammern und doppelte Anführungszeichen ein. Das folgende Beispiel zeigt dies:

```dockerfile
COPY ["<source>", "<destination>"]
```

#### <a name="considerations-for-using-copy-with-windows"></a>Überlegungen zur Verwendung von COPY mit Windows

Unter Windows müssen im Zielformat Schrägstriche verwendet werden. Dies sind z. B. gültige `COPY`-Anweisungen:

```dockerfile
COPY test1.txt /temp/
COPY test1.txt c:/temp/
```

Folgende Formate mit umgekehrten Schrägstrichen funktionieren jedoch nicht:

```dockerfile
COPY test1.txt c:\temp\
```

#### <a name="examples-of-using-copy-with-windows"></a>Beispiele für die Verwendung von COPY mit Windows

Im folgenden Beispiel wird der Inhalt des Quellverzeichnisses in ein Verzeichnis namens `sqllite` kopiert, das sich im Containerimage befindet:

```dockerfile
COPY source /sqlite/
```

Im folgenden Beispiel werden alle Dateien, die mit „config“ beginnen, dem Verzeichnis `c:\temp` des Containerimages hinzugefügt:

```dockerfile
COPY config* c:/temp/
```

Ausführliche Informationen zur `COPY`-Anweisung finden Sie in der [COPY-Referenz](https://docs.docker.com/engine/reference/builder/#copy).

### <a name="add"></a>HINZUFÜGEN

Die ADD-Anweisung ähnelt der COPY-Anweisung, weist aber noch mehr Funktionen auf. Die `ADD`-Anweisung kann nicht nur Dateien vom Host in das Containerimage kopieren, sondern auch von einem Remotestandort aus mit einer URL-Spezifikation.

Das Format der `ADD`-Anweisung sieht wie folgt aus:

```dockerfile
ADD <source> <destination>
```

Wenn die Quelle oder das Ziel Leerzeichen enthalten, schließen Sie den Pfad in eckige Klammern und doppelte Anführungszeichen ein:

```dockerfile
ADD ["<source>", "<destination>"]
```

#### <a name="considerations-for-running-add-with-windows"></a>Überlegungen zur Ausführung von ADD mit Windows

Unter Windows müssen im Zielformat Schrägstriche verwendet werden. Dies sind z. B. gültige `ADD`-Anweisungen:

```dockerfile
ADD test1.txt /temp/
ADD test1.txt c:/temp/
```

Folgende Formate mit umgekehrten Schrägstrichen funktionieren jedoch nicht:

```dockerfile
ADD test1.txt c:\temp\
```

Außerdem erweitert die `ADD`-Anweisung unter Linux komprimierte Pakete beim Kopieren. Diese Funktionalität ist in Windows nicht verfügbar.

#### <a name="examples-of-using-add-with-windows"></a>Beispiele für die Verwendung von ADD mit Windows

Im folgenden Beispiel wird der Inhalt des Quellverzeichnisses in ein Verzeichnis namens `sqllite` kopiert, das sich im Containerimage befindet:

```dockerfile
ADD source /sqlite/
```

Im folgenden Beispiel werden alle Dateien, die mit „config“ beginnen, dem Verzeichnis `c:\temp` des Containerimages hinzugefügt.

```dockerfile
ADD config* c:/temp/
```

In diesem Beispiel wird Python für Windows in das Verzeichnis `c:\temp` des Containerimages heruntergeladen.

```dockerfile
ADD https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe /temp/python-3.5.1.exe
```

Ausführlichere Informationen zur `ADD`-Anweisung finden Sie in der [ADD-Referenz](https://docs.docker.com/engine/reference/builder/#add).

### <a name="workdir"></a>WORKDIR

Die `WORKDIR`-Anweisung legt ein Arbeitsverzeichnis für andere Dockerfile-Anweisungen wie z.B. `RUN` und `CMD` sowie das Arbeitsverzeichnis für ausgeführte Instanzen des Containerimages fest.

Das Format der `WORKDIR`-Anweisung sieht wie folgt aus:

```dockerfile
WORKDIR <path to working directory>
```

#### <a name="considerations-for-using-workdir-with-windows"></a>Überlegungen zur Verwendung von WORKDIR mit Windows

Wenn das Arbeitsverzeichnis unter Windows einen umgekehrten Schrägstrich enthält, muss es mit Escapezeichen versehen werden.

```dockerfile
WORKDIR c:\\windows
```

**Beispiele**

```dockerfile
WORKDIR c:\\Apache24\\bin
```

Ausführliche Informationen zur `WORKDIR`-Anweisung finden Sie in der [WORKDIR-Referenz](https://docs.docker.com/engine/reference/builder/#workdir).

### <a name="cmd"></a>CMD

Die `CMD`-Anweisung legt den Standardbefehl fest, der bei der Bereitstellung einer Instanz des Containerimages ausgeführt werden soll. Wenn der Container beispielsweise einen NGINX-Webserver hostet, könnte `CMD` Anweisungen zum Starten des Webservers mit einem Befehl wie `nginx.exe` enthalten. Wenn mehrere `CMD`-Anweisungen in einer Dockerfile-Datei angegeben sind, wird nur die letzte ausgewertet.

Das Format der `CMD`-Anweisung sieht wie folgt aus:

```dockerfile
# exec form

CMD ["<executable", "<param>"]

# shell form

CMD <command>
```

#### <a name="considerations-for-using-cmd-with-windows"></a>Überlegungen zur Verwendung von CMD mit Windows

Unter Windows müssen in Dateipfaden, die in der `CMD`-Anweisung angegeben werden, Schrägstriche oder mit Escapezeichen versehene umgekehrte Schrägstriche `\\` verwendet werden. Die folgenden Anweisungen sind gültige `CMD`-Anweisungen:

```dockerfile
# exec form

CMD ["c:\\Apache24\\bin\\httpd.exe", "-w"]

# shell form

CMD c:\\Apache24\\bin\\httpd.exe -w
```

Das folgende Format ohne die entsprechenden Schrägstriche funktioniert jedoch nicht:

```dockerfile
CMD c:\Apache24\bin\httpd.exe -w
```

Ausführlichere Informationen zur `CMD`-Anweisung finden Sie in der [CMD-Referenz](https://docs.docker.com/engine/reference/builder/#cmd).

## <a name="escape-character"></a>Escapezeichen

Eine Dockerfile-Anweisung muss häufig mehrere Zeilen umfassen. Zu diesem Zweck können Sie ein Escapezeichen verwenden. In einer Dockerfile-Anweisung wird als Escapezeichen standardmäßig ein umgekehrter Schrägstrich verwendet: `\`. Da der umgekehrte Schrägstrich jedoch auch ein Trennzeichen für Dateipfade in Windows ist, kann seine Verwendung zur Angabe von mehreren Zeilen zu Problemen führen. Als Problemumgehung können Sie eine Parserdirektive zum Ändern des Standardescapezeichens verwenden. Weitere Informationen zu Parserdirektiven finden Sie unter [Parserdirektiven](https://docs.docker.com/engine/reference/builder/#parser-directives).

Das folgende Beispiel zeigt eine einzelne RUN-Anweisung, die mehrere Zeilen umfasst und für die das Standardescapezeichen verwendet wird:

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
    $ErrorActionPreference = 'Stop'; \
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
    Remove-Item c:\python-3.5.1.exe -Force
```

Platzieren Sie zum Ändern des Escapezeichens eine Escape-Parser-Anweisung in der ersten Zeile der Dockerfile-Datei. Dies wird im folgenden Beispiel veranschaulicht.

>[!NOTE]
>Nur zwei Werte können als Escapezeichen verwendet werden: `\` und `` ` ``.

```dockerfile
# escape=`

FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command `
    $ErrorActionPreference = 'Stop'; `
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; `
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; `
    Remove-Item c:\python-3.5.1.exe -Force
```

Weitere Informationen zur Escape-Parserdirektive finden Sie unter [Escape-Parserdirektive](https://docs.docker.com/engine/reference/builder/#escape).

## <a name="powershell-in-dockerfile"></a>PowerShell in Dockerfile

### <a name="powershell-cmdlets"></a>PowerShell-Cmdlets

PowerShell-Cmdlets können mit dem `RUN`-Vorgang in einer Dockerfile-Datei ausgeführt werden.

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -command Expand-Archive -Path c:\apache.zip -DestinationPath c:\
```

### <a name="rest-calls"></a>REST-Aufrufe

Das PowerShell-Cmdlet `Invoke-WebRequest` kann beim Erfassen von Informationen oder Dateien von einem Webdienst hilfreich sein. Wenn Sie z. B. ein Image erstellen, das Python enthält, können Sie `$ProgressPreference` auf `SilentlyContinue` festlegen, um schnellere Downloads zu erreichen, wie im folgenden Beispiel gezeigt wird.

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  $ProgressPreference = 'SilentlyContinue'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

>[!NOTE]
>`Invoke-WebRequest` funktioniert auch in Nano Server.

Eine weitere Möglichkeit zur Verwendung von PowerShell zum Herunterladen von Dateien während der Imageerstellung ist die Verwendung der .NET WebClient-Bibliothek. Dies kann die Downloadleistung verbessern. Im folgenden Beispiel wird die Python-Software mit der WebClient-Bibliothek heruntergeladen.

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  (New-Object System.Net.WebClient).DownloadFile('https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe','c:\python-3.5.1.exe') ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

>[!NOTE]
>Nano Server unterstützt WebClient zurzeit nicht.

### <a name="powershell-scripts"></a>PowerShell-Skripts

In einigen Fällen kann es hilfreich sein, ein Skript in die Container zu kopieren, die Sie während des Vorgangs der Imageerstellung verwende, und das Skript dann aus dem Container heraus auszuführen.

>[!NOTE]
>Hierdurch wird das Zwischenspeichern von Imageebenen eingeschränkt und die Lesbarkeit der Dockerfile-Datei verringert.

In diesem Beispiel wird ein Skript vom Buildcomputer mithilfe der `ADD`-Anweisung in den Container kopiert. Dieses Skript wird dann mit der RUN-Anweisung ausgeführt.

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019
ADD script.ps1 /windows/temp/script.ps1
RUN powershell.exe -executionpolicy bypass c:\windows\temp\script.ps1
```

## <a name="docker-build"></a>Docker build

Sobald Sie eine Dockerfile-Datei erstellt und auf der Festplatte gespeichert haben, können Sie `docker build` ausführen, um das neue Image zu erstellen. Der `docker build`-Befehl unterstützt verschiedene optionale Parameter und einen Pfad zur Dockerfile-Datei. Eine vollständige Dokumentation zu Docker Build einschließlich einer Liste aller Buildoptionen finden Sie in der [Build-Referenz](https://docs.docker.com/engine/reference/commandline/build/#build).

Das Format des Befehls `docker build` sieht wie folgt aus:

```dockerfile
docker build [OPTIONS] PATH
```

Der folgende Befehl erstellt z. B. ein Image mit dem Namen „iis“.

```dockerfile
docker build -t iis .
```

Wenn der Buildprozess eingeleitet wurde, wird der Status ausgegeben, und ggf. werden Fehlermeldungen zurückgegeben.

```dockerfile
C:\> docker build -t iis .

Sending build context to Docker daemon 2.048 kB
Step 1 : FROM mcr.microsoft.com/windows/servercore:ltsc2019
 ---> 6801d964fda5

Step 2 : RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
 ---> Running in ae8759fb47db

Deployment Image Servicing and Management tool
Version: 10.0.10586.0

Image Version: 10.0.10586.0

Enabling feature(s)
The operation completed successfully.

 ---> 4cd675d35444
Removing intermediate container ae8759fb47db

Step 3 : RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
 ---> Running in 9a26b8bcaa3a
 ---> e2aafdfbe392
Removing intermediate container 9a26b8bcaa3a

Successfully built e2aafdfbe392
```

Das Ergebnis ist ein neues Containerimage, das in diesem Beispiel den Namen „iis“ trägt.

```dockerfile
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
iis                 latest              e2aafdfbe392        About a minute ago   207.8 MB
windowsservercore   latest              6801d964fda5        4 months ago         0 B
```

## <a name="further-reading-and-references"></a>Weitere Informationen und Referenzen

- [Optimieren von Dockerfile-Dateien und Docker build für Windows](optimize-windows-dockerfile.md)
- [Dockerfile-Referenz](https://docs.docker.com/engine/reference/builder/)
