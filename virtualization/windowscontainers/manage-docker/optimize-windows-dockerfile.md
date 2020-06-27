---
title: Optimieren von Windows-Dockerfile-Dateien
description: Optimieren Sie Dockerfile-Dateien für Windows-Container.
keywords: Docker, Container
author: cwilhit
ms.date: 05/03/2019
ms.topic: tutorial
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb2848ca-683e-4361-a750-0d1d14ec8031
ms.openlocfilehash: 25297432a3e515a130c9710431fd550f2efd8226
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192357"
---
# <a name="optimize-windows-dockerfiles"></a>Optimieren von Windows-Dockerfile-Dateien

Mehrere Methoden können zur Optimierung des Docker Build-Prozesses sowie der sich ergebenden Docker-Images verwendet werden. In diesem Artikel wird erläutert, wie der Docker Build-Prozess funktioniert und wie Images für Windows-Container optimal erstellt werden.

## <a name="image-layers-in-docker-build"></a>Imageebenen in Docker build

Bevor Sie Ihren Docker build-Prozess optimieren können, müssen Sie wissen, wie Docker build funktioniert. Während des Docker Build-Prozesses wird eine Dockerfile-Datei genutzt, und die ausführbaren Anweisungen werden nacheinander ausgeführt, jede in einem eigenen temporären Container. Das Ergebnis ist eine neue Imageebene für jede ausführbare Anweisung.

Beispielsweise verwendet die folgende Dockerfile-Beispieldatei das `mcr.microsoft.com/windows/servercore:ltsc2019`-Basisbetriebssystem-Image, installiert IIS und erstellt dann eine einfache Website.

```dockerfile
# Sample Dockerfile

FROM mcr.microsoft.com/windows/servercore:ltsc2019
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

Sie erwarten möglicherweise, dass diese Dockerfile-Datei ein Image mit zwei Ebenen generiert: eines für das Containerbetriebssystem-Image und ein zweites, das IIS und die Website enthält. Das tatsächliche Image weist jedoch viele Ebenen auf, und jede Ebene hängt von der jeweils vorherigen Ebene ab.

Um dies zu verdeutlichen, führen wir den Befehl `docker history` für das Image aus, das von der Dockerfile-Beispieldatei erstellt wurde.

```dockerfile
docker history iis

IMAGE               CREATED              CREATED BY                                      SIZE                COMMENT
f4caf476e909        16 seconds ago       cmd /S /C REM (nop) CMD ["cmd"]                 41.84 kB
f0e017e5b088        21 seconds ago       cmd /S /C echo "Hello World - Dockerfile" > c   6.816 MB
88438e174b7c        About a minute ago   cmd /S /C dism /online /enable-feature /all /   162.7 MB
6801d964fda5        4 months ago                                                         0 B
```

Die Ausgabe zeigt, dass dieses Image vier Ebenen besitzt: die Basisebene und drei zusätzliche Ebenen, die jeder Anweisung in der Dockerfile-Datei zugeordnet sind. Die unterste Ebene (`6801d964fda5` in diesem Beispiel) stellt das Basis-Betriebssystemimage dar. Eine Ebene darüber findet die IIS-Installation statt. Die nächste Ebene enthält die neue Website usw.

Dockerfile-Dateien können geschrieben werden, um Imageebenen zu minimieren, die Buildleistung zu optimieren und Barrierefreiheit durch bessere Lesbarkeit zu verbessern. Schließlich stehen viele Alternativen zur Durchführung einer bestimmten Buildaufgabe zur Verfügung. Das Wissen darüber, wie sich das Format einer Dockerfile-Datei auf die Builddauer und das generierte Image auswirkt, verbessert die Automatisierungserfahrung.

## <a name="optimize-image-size"></a>Optimieren der Imagegröße

Abhängig von Ihren Speicherplatzanforderungen kann die Imagegröße ein wichtiger Faktor bei der Erstellung von Docker-Containerimages sein. Containerimages werden zwischen Registrierungen und Host verschoben, exportiert und importiert und verbrauchen so letztendlich Speicherplatz. In diesem Abschnitt erfahren Sie, wie Sie die Imagegröße während des Docker build-Prozesses für Windows-Container minimieren.

Weitere Informationen zu bewährten Methoden für Dockerfile-Dateien finden Sie unter [Bewährte Methoden für das Schreiben von Dockerfile-Dateien auf Docker.com](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/).

### <a name="group-related-actions"></a>Gruppieren von verwandten Aktionen

Da jede `RUN`-Anweisung eine neue Ebene in dem Containerimage erstellt, kann die Gruppierung von Aktionen in einer `RUN`-Anweisung die Anzahl der Ebenen in einer Dockerfile-Datei reduzieren. Das Minimieren der Ebenen wirkt sich möglicherweise nicht auf die Imagegröße aus, aber das Gruppieren verwandter Aktionen kann eine Einfluss haben, wie die nachfolgenden Beispiele zeigen.

In diesem Abschnitt werden zwei Docker-Beispieldateien verglichen, die dieselben Aktionen ausführen. Allerdings weist eine Dockerfile-Datei eine Anweisung pro Aktion auf, während bei der anderen zusammengehörige Aktionen gruppiert wurden.

In der folgenden nicht gruppierten Dockerfile-Beispieldatei wird Python für Windows heruntergeladen, installiert und die heruntergeladene Setup Datei entfernt, nachdem die Installation abgeschlossen ist. In dieser Dockerfile-Datei erhält jede Aktion eine eigene `RUN`-Anweisung.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command Invoke-WebRequest "https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe" -OutFile c:\python-3.5.1.exe
RUN powershell.exe -Command Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait
RUN powershell.exe -Command Remove-Item c:\python-3.5.1.exe -Force
```

Das resultierende Image besteht aus drei zusätzliche Ebenen, einer für jede `RUN`-Anweisung.

```dockerfile
docker history doc-example-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
a395ca26777f        15 seconds ago      cmd /S /C powershell.exe -Command Remove-Item   24.56 MB
6c137f466d28        28 seconds ago      cmd /S /C powershell.exe -Command Start-Proce   178.6 MB
957147160e8d        3 minutes ago       cmd /S /C powershell.exe -Command Invoke-WebR   125.7 MB
```

Das zweite Beispiel ist eine Dockerfile-Datei, die genau denselben Vorgang ausführt. Alle zusammengehörigen Aktionen wurden jedoch unter einer einzigen `RUN`-Anweisung gruppiert. Jeder Schritt in der `RUN`-Anweisung befindet sich in einer neuen Zeile der Dockerfile-Datei, wobei der Zeilenumbruch mit dem Zeichen „\\“ erfolgt.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

Das sich ergebende Image besitzt nur eine zusätzlichen Ebene für die `RUN`-Anweisung.

```dockerfile
docker history doc-example-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
69e44f37c748        54 seconds ago      cmd /S /C powershell.exe -Command   $ErrorAct   216.3 MB
```

### <a name="remove-excess-files"></a>Entfernen überflüssiger Dateien

Wenn eine Datei in ihrer Dockerfile-Datei vorhanden ist (z. B. ein Installer), die Sie nicht mehr benötigen, nachdem sie verwendet wurde, können Sie sie entfernen, um die Imagegröße zu verringern. Dies muss in dem gleichen Schritt erfolgen, in dem die Datei in die Imageebene kopiert wurde. So wird verhindert, dass die Datei auf einer niedrigeren Imageebene beibehalten wird.

In der folgenden Dockerfile-Beispieldatei wird das Python-Paket heruntergeladen, ausgeführt und dann entfernt. Dies alles wird in einem einzigen `RUN`-Vorgang ausgeführt und resultiert in einer einzelnen Imageebene.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

## <a name="optimize-build-speed"></a>Optimieren der Buildgeschwindigkeit

### <a name="multiple-lines"></a>Mehrere Zeilen

Sie können Vorgänge in mehrere einzelne Anweisungen aufteilen, um die Docker build-Geschwindigkeit zu optimieren. Mehrere `RUN`-Vorgänge erhöhen die Zwischenspeicherungseffektivität, weil einzelne Ebenen für jede `RUN`-Anweisung erstellt werden. Wenn eine identische Anweisung bereits in einem anderen Docker build-Vorgang ausgeführt wurde, wird dieser zwischengespeicherte Vorgang (Imageebene) erneut verwendet und führt zu einer verringerten Docker build-Laufzeit.

Im folgenden Beispiel werden sowohl Apache als auch die verteilbaren Pakete von Visual Studio heruntergeladen, installiert und dann bereinigt, indem nicht benötigte Dateien entfernt werden. Dies alles erfolgt mit einer einzigen `RUN`-Anweisung. Wenn eine dieser Aktionen aktualisiert wird, werden alle Aktionen erneut ausgeführt.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -Command \

  # Download software ; \

  wget https://www.apachelounge.com/download/VC11/binaries/httpd-2.4.18-win32-VC11.zip -OutFile c:\apache.zip ; \
  wget "https://download.microsoft.com/download/1/6/B/16B06F60-3B20-4FF2-B699-5E9B7962F9AE/VSU_4/vcredist_x86.exe" -OutFile c:\vcredist.exe ; \
  wget -Uri http://windows.php.net/downloads/releases/php-5.5.33-Win32-VC11-x86.zip -OutFile c:\php.zip ; \

  # Install Software ; \

  Expand-Archive -Path c:\php.zip -DestinationPath c:\php ; \
  Expand-Archive -Path c:\apache.zip -DestinationPath c:\ ; \
  start-Process c:\vcredist.exe -ArgumentList '/quiet' -Wait ; \

  # Remove unneeded files ; \

  Remove-Item c:\apache.zip -Force; \
  Remove-Item c:\vcredist.exe -Force; \
  Remove-Item c:\php.zip
```

Das sich ergebende Image weist zwei Ebenen auf: eine Ebene für das Betriebssystem-Basisimage und eine Ebene, die alle Vorgänge aus der einzelnen `RUN`-Anweisung enthält.

```dockerfile
docker history doc-sample-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
9bdf3a21fd41        8 minutes ago       cmd /S /C powershell -Command     Invoke-WebR   205.8 MB
6801d964fda5        5 months ago                                                        0 B
```

Zum Vergleich sehen Sie hier die gleichen Aktionen auf drei `RUN`-Anweisungen verteilt. In diesem Fall wird jede `RUN`-Anweisung in einer Containerimageebene zwischengespeichert, und nur solche, die geändert wurden, müssen in nachfolgenden Dockerfile-Builds neu ausgeführt werden.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    wget https://www.apachelounge.com/download/VC11/binaries/httpd-2.4.18-win32-VC11.zip -OutFile c:\apache.zip ; \
    Expand-Archive -Path c:\apache.zip -DestinationPath c:\ ; \
    Remove-Item c:\apache.zip -Force

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    wget "https://download.microsoft.com/download/1/6/B/16B06F60-3B20-4FF2-B699-5E9B7962F9AE/VSU_4/vcredist_x86.exe" -OutFile c:\vcredist.exe ; \
    start-Process c:\vcredist.exe -ArgumentList '/quiet' -Wait ; \
    Remove-Item c:\vcredist.exe -Force

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    wget http://windows.php.net/downloads/releases/php-5.5.33-Win32-VC11-x86.zip -OutFile c:\php.zip ; \
    Expand-Archive -Path c:\php.zip -DestinationPath c:\php ; \
    Remove-Item c:\php.zip -Force
```

Das sich ergebende Image besteht aus vier Ebenen: einer Ebene für das Betriebssystem-Basisimage und jede der drei `RUN`-Anweisungen. Da jede `RUN`-Anweisung in einer eigenen Ebene ausgeführt wurde, verwenden alle nachfolgenden Ausführungen dieser Dockerfile-Datei oder eines identischen Satzes von Anweisungen in einer anderen Dockerfile-Datei die zwischengespeicherte Imageebene, wodurch die Buildzeit reduziert wird.

```dockerfile
docker history doc-sample-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
ddf43b1f3751        6 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   127.2 MB
d43abb81204a        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   66.46 MB
7a21073861a1        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   115.8 MB
6801d964fda5        5 months ago
```

Es ist wichtig, wie Sie die Anweisungen anordnen, wenn Sie mit Imagecaches arbeiten, wie Sie im nächsten Abschnitt sehen werden.

### <a name="ordering-instructions"></a>Anordnen von Anweisungen

Eine Dockerfile-Datei wird von oben nach unten verarbeitet und jede Anweisung mit den zwischengespeicherten Ebenen verglichen. Wenn eine Anweisung gefunden wird, der keine zwischengespeicherte Ebene zugeordnet ist, werden diese Anweisung und alle nachfolgenden Anweisungen in neuen Containerimageebenen verarbeitet. Aus diesem Grund ist die Reihenfolge wichtig, in der die Anweisungen platziert werden. Platzieren Sie Anweisungen, die konstant bleiben werden, im oberen Teil der Dockerfile-Datei. Platzieren Sie Anweisungen, die sich ändern könnten, im unteren Teil der Dockerfile-Datei. Dies reduziert die Wahrscheinlichkeit, dass der vorhandene Cache negiert wird.

Die folgenden Beispiele zeigen, wie die Reihenfolge von Dockerfile-Anweisungen die Wirksamkeit des Zwischenspeichers beeinflussen kann. Diese einfache Dockerfile-Beispieldatei weist vier nummerierte Ordner auf.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-3
RUN mkdir test-4
```

Das sich ergebende Image besteht aus fünf Ebenen: einer Ebene für das Betriebssystem-Basisimage und jede `RUN`-Anweisung.

```dockerfile
docker history doc-sample-1

IMAGE               CREATED              CREATED BY               SIZE                COMMENT
afba1a3def0a        38 seconds ago       cmd /S /C mkdir test-4   42.46 MB
86f1fe772d5c        49 seconds ago       cmd /S /C mkdir test-3   42.35 MB
68fda53ce682        About a minute ago   cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        About a minute ago   cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                  0 B
```

Diese nächste Dockerfile-Datei wurde nun geringfügig geändert, wobei die dritte `RUN`-Anweisung in eine neue Datei geändert wurde. Wenn Docker Build mit dieser Dockerfile-Datei ausgeführt wird, verwenden die ersten drei Anweisungen, die mit denen im letzten Beispiel identisch sind, zwischengespeicherte Imageebenen. Aber da die geänderte `RUN`-Anweisung nicht zwischengespeichert wurde, wird eine neue Ebene für die geänderte Anweisung und alle nachfolgenden Anweisungen erstellt.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-5
RUN mkdir test-4
```

Wenn Sie die Image-IDs des neuen Images mit denen im ersten Beispiel dieses Abschnitts vergleichen, werden Sie feststellen, dass die ersten drei Ebenen von unten nach oben gemeinsam verwendet werden, die vierte und fünfte Ebene aber jeweils eindeutig ist.

```dockerfile
docker history doc-sample-2

IMAGE               CREATED             CREATED BY               SIZE                COMMENT
c92cc95632fb        28 seconds ago      cmd /S /C mkdir test-4   5.644 MB
2f05e6f5c523        37 seconds ago      cmd /S /C mkdir test-5   5.01 MB
68fda53ce682        3 minutes ago       cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        4 minutes ago       cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                 0 B
```

## <a name="cosmetic-optimization"></a>Kosmetische Optimierung

### <a name="instruction-case"></a>Groß-/Kleinschreibung der Anweisung

Bei Dockerfile-Anweisungen wird nicht zwischen Groß- und Kleinschreibung unterschieden, üblicherweise werden jedoch Großbuchstaben verwendet. Dies verbessert die Lesbarkeit durch die Unterscheidung zwischen Anweisungsaufruf und Anweisungsvorgang. Die folgenden beiden Beispiele vergleichen eine Dockerfile-Datei ohne Großbuchstaben mit einer Dockerfile-Datei mit Großbuchstaben.

Das folgende Beispiel zeigt eine Dockerfile-Datei ohne Großbuchstaben:

```dockerfile
# Sample Dockerfile

from mcr.microsoft.com/windows/servercore:ltsc2019
run dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
run echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
cmd [ "cmd" ]
```

Im Folgenden sehen Sie die gleiche Dockerfile-Datei, in der Großbuchstaben verwendet werden:

```dockerfile
# Sample Dockerfile

FROM mcr.microsoft.com/windows/servercore:ltsc2019
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

### <a name="line-wrapping"></a>Zeilenumbruch

Lange und komplexe Vorgänge können mit dem umgekehrten Schrägstrich (`\`) auf mehrere Zeilen verteilt werden. Die folgende Dockerfile-Datei installiert das verteilbare Paket von Visual Studio, entfernt die Dateien des Installationsprogramms und erstellt dann eine Konfigurationsdatei. Alle diese drei Vorgänge werden in einer Zeile angegeben.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -Command c:\vcredist_x86.exe /quiet ; Remove-Item c:\vcredist_x86.exe -Force ; New-Item c:\config.ini
```

Der Befehl kann durch umgekehrte Schrägstriche unterteilt werden, damit jeder Vorgang der einen `RUN`-Anweisung in einer eigenen Zeile angegeben wird.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait ; \
    Remove-Item c:\vcredist_x86.exe -Force ; \
    New-Item c:\config.ini
```

## <a name="further-reading-and-references"></a>Weitere Informationen und Referenzen

[Dockerfile unter Windows](manage-windows-dockerfile.md)

[Bewährte Methoden zum Schreiben von Dockerfile-Dateien auf Docker.com](https://docs.docker.com/engine/reference/builder/)
