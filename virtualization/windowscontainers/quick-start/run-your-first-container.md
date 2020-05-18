---
title: Windows- und Linux-Container unter Windows 10
description: Containerbereitstellung – Schnellstart
keywords: Docker, Container, LCOW
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 859ef38380e536a79a3dcd63397af8fd841253ee
ms.sourcegitcommit: 62f4bcca4e07f2a34a927e5c4d786e505821d559
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 05/05/2020
ms.locfileid: "82784409"
---
# <a name="get-started-run-your-first-windows-container"></a>Erste Schritte: Ausführen Ihres ersten Windows-Containers

In diesem Thema wird beschrieben, wie Sie Ihren ersten Windows-Container ausführen, nachdem Sie Ihre Umgebung wie unter [Erste Schritte: Vorbereiten von Windows für Container](./set-up-environment.md) beschrieben eingerichtet haben. Zum Ausführen eines Containers installieren Sie zunächst ein Basisimage, das eine grundlegende Ebene von Betriebssystemdiensten für Ihren Container bereitstellt. Anschließend erstellen Sie ein Containerimage, das auf dem Basisimage aufbaut, und führen es aus. Weitere Informationen finden Sie im Folgenden.

## <a name="install-a-container-base-image"></a>Installieren eines Containerbasisimages

Alle Container werden aus Containerimages erstellt. Microsoft bietet verschiedene Ausgangsimages, die als Basisimages bezeichnet werden, unter denen Sie eine Auswahl treffen können (weitere Informationen finden Sie unter [Containerbasisimages](../manage-containers/container-base-images.md)). Durch diese Vorgehensweise wird das Lightweight Nano Server-Basisimage gepullt (heruntergeladen und installiert).

1. Öffnen Sie ein Eingabeaufforderungsfenster (z. B. die integrierte Eingabeaufforderung, PowerShell oder [Windows-Terminal](https://www.microsoft.com/p/windows-terminal-preview/9n0dx20hk701?activetab=pivot:overviewtab)), und führen Sie dann den folgenden Befehl aus, um das Basisimage herunterzuladen und zu installieren:

   ```console
   docker pull mcr.microsoft.com/windows/nanoserver:1903
   ```

   > [!TIP]
   > Wenn eine Fehlermeldung angezeigt wird, die `no matching manifest for unknown in the manifest list entries`lautet, stellen Sie sicher, dass Docker nicht für das Ausführen von Linux-Containern konfiguriert ist.

2. Nachdem das Herunterladen des Images abgeschlossen ist (lesen Sie die [EULA](../images-eula.md), während Sie warten), überprüfen Sie sein Vorhandensein im System, indem Sie das lokale Docker-Imagerepository abfragen. Wenn Sie den Befehl `docker images` ausführen, wird eine Liste der installierten Images zurückgegeben.

   Im folgenden finden Sie ein Beispiel für die Ausgabe, die das Nano Server-Image zeigt.

   ```console
   REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
   microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
   ```

## <a name="run-a-windows-container"></a>Ausführen eines Windows-Containers

In diesem einfachen Beispiel wird ein „Hello World“-Containerimage erstellt und bereitgestellt. Um optimale Ergebnisse zu erzielen, führen Sie diese Befehle in einem Eingabeaufforderungsfenster mit erhöhten Rechten aus (verwenden Sie jedoch nicht die Windows PowerShell ISE – sie funktioniert nicht für interaktive Sitzungen mit Containern, da die Container scheinbar nicht mehr reagieren).

1. Starten Sie einen Container mit einer interaktiven Sitzung aus dem `nanoserver`-Image, indem Sie den folgenden Befehl in das Eingabeaufforderungsfenster eingeben:

   ```console
   docker run -it mcr.microsoft.com/windows/nanoserver:1903 cmd.exe
   ```
2. Nachdem der Container gestartet wurde, ändert das Eingabeaufforderungsfenster den Kontext in den Container. Im Container erstellen wir eine einfache Textdatei mit dem Namen „Hello World“ und beenden den Container dann, indem wir die folgenden Befehle eingeben:

   ```cmd
   echo "Hello World!" > Hello.txt
   exit
   ```   

3. Rufen Sie die Container-ID für den Container ab, den Sie soeben beendet haben, indem Sie den Befehl [docker ps](https://docs.docker.com/engine/reference/commandline/ps/) ausführen:

   ```console
   docker ps -a
   ```

4. Erstellen Sie ein neues „HelloWorld“-Image, das die Änderungen im ersten Container enthält, den Sie ausgeführt haben. Führen Sie dazu den Befehl [docker commit](https://docs.docker.com/engine/reference/commandline/commit/) aus, und ersetzen Sie `<containerid>` durch die ID Ihres Containers:

   ```console
   docker commit <containerid> helloworld
   ```

   Nach Beendigung des Vorgangs verfügen Sie über ein benutzerdefiniertes Image, das ein „Hello World“-Skript enthält. Sie können es mit dem Befehl [docker images](https://docs.docker.com/engine/reference/commandline/images/) anzeigen.

   ```console
   docker images
   ```

   Dies ist ein Beispiel für die Ausgabe:

   ```console
   REPOSITORY                             TAG                 IMAGE ID            CREATED             SIZE
   helloworld                             latest              a1064f2ec798        10 seconds ago      258MB
   mcr.microsoft.com/windows/nanoserver   1903                2b9c381d0911        3 weeks ago         256MB
   ```

5. Führen Sie schließlich den neuen Container mit dem Befehl [docker run](https://docs.docker.com/engine/reference/commandline/run/) mit dem Parameter `--rm` aus, der den Container automatisch entfernt, sobald die Befehlszeile („cmd.exe“) beendet wird.

   ```console
   docker run --rm helloworld cmd.exe /s /c type Hello.txt
   ```
   Das Ergebnis ist, dass Docker einen Container aus dem „HelloWorld“-Image erstellt, eine Instanz von „cmd.exe“ im Container startet, die cmd.exe unsere Datei dann liest und den Dateiinhalt an die Shell ausgibt. Im letzten Schritt beendet Docker den Container und entfernt ihn.

## <a name="next-steps"></a>Nächste Schritte

> [!div class="nextstepaction"]
> [Informationen zum Containerisieren einer Beispiel-App](./building-sample-app.md)
