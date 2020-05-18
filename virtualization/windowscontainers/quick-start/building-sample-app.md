---
title: Containerisieren einer .NET Core-App
description: Erfahren Sie, wie Sie eine .NET Core-Beispiel-App mit Containern erstellen.
keywords: Docker, Container
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: c9e175a7ced0f328e342f3cdd4f99adc717d5700
ms.sourcegitcommit: 62f4bcca4e07f2a34a927e5c4d786e505821d559
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 05/05/2020
ms.locfileid: "82784399"
---
# <a name="containerize-a-net-core-app"></a>Containerisieren einer .NET Core-App

In diesem Thema wird beschrieben, wie Sie eine vorhandene .NET-Beispiel-App für die Bereitstellung als Windows-Container verpacken, nachdem Sie Ihre Umgebung wie unter [Erste Schritte: Vorbreiten von Windows für Container](set-up-environment.md) beschrieben eingerichtet und Ihren ersten Container wie unter [Ausführen Ihres ersten Windows-Containers](run-your-first-container.md) beschrieben ausgeführt haben.

Außerdem muss das Git-Quellcodeverwaltungssystem auf dem Computer installiert sein. Besuchen Sie [Git](https://git-scm.com/download), um es zu installieren.

## <a name="clone-the-sample-code-from-github"></a>Klonen des Beispielcodes von GitHub

Der gesamte Containerbeispielquellcode wird in einem Ordner mit dem Namen `windows-container-samples` im Git-Repository [Virtualization-Documentation](https://github.com/MicrosoftDocs/Virtualization-Documentation) gespeichert.

1. Öffnen Sie eine PowerShell-Sitzung, und wechseln Sie in den Ordner, in dem Sie dieses Repository speichern möchten. (Andere Eingabeaufforderungs-Fenstertypen funktionieren ebenfalls, aber in den Beispielbefehlen wird PowerShell verwendet.)
2. Klonen Sie das Repository in Ihr aktuelles Arbeitsverzeichnis.

   ```PowerShell
   git clone https://github.com/MicrosoftDocs/Virtualization-Documentation.git
   ```

3. Navigieren Sie zu dem unter `Virtualization-Documentation\windows-container-samples\asp-net-getting-started` gefundenen Beispielverzeichnis, und erstellen Sie mithilfe der folgenden Befehle eine Dockerfile-Datei.

   Eine [Dockerfile](https://docs.docker.com/engine/reference/builder/)-Datei ähnelt einer makefile-Datei: Es handelt sich u eine Liste mit Anweisungen, die die Container-Engine informieren, wie das Containerimage erstellt werden soll.

   ```Powershell
   # Navigate into the sample directory
   Set-Location -Path Virtualization-Documentation\windows-container-samples\asp-net-getting-started

   # Create the Dockerfile for our project
   New-Item -Name Dockerfile -ItemType file
   ```

## <a name="write-the-dockerfile"></a>Schreiben der Dockerfile-Datei

Öffnen Sie die soeben erstellte Dockerfile-Datei mit einem beliebigen Text-Editor, und fügen Sie dann den folgenden Inhalt hinzu:

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/sdk:2.1 AS build-env
WORKDIR /app

COPY *.csproj ./
RUN dotnet restore

COPY . ./
RUN dotnet publish -c Release -o out

FROM mcr.microsoft.com/dotnet/core/aspnet:2.1
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "asp-net-getting-started.dll"]
```

Schauen wir sie uns Zeile für Zeile an und erläutern, was die einzelnen Anweisungen bewirken.

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/sdk:2.1 AS build-env
WORKDIR /app
```

Die erste Gruppe von Zeilen deklariert, auf Grundlage von welchem Basisimage unser Container erstellt wird. Wenn das lokale System nicht bereits über dieses Image verfügt, versucht Docker automatisch, es abzurufen. `mcr.microsoft.com/dotnet/core/sdk:2.1` wird mit dem .NET Core 2.1 SDK installiert. Also müssen ASP .NET Core-Projekte mit der Zielversion 2.1 erstellt werden. Mit der nächsten Anweisung wird das Arbeitsverzeichnis in unserem Container in `/app` geändert, sodass alle Befehle, die auf diese Anweisung folgen, in diesem Kontext ausgeführt werden.

```Dockerfile
COPY *.csproj ./
RUN dotnet restore
```

Anschließend kopieren diese Anweisungen die CSPROJ-Dateien in `build-env` des Verzeichnisses `/app` des Containers. Nach dem Kopieren dieser Datei wird diese von .NET gelesen. Anschließend werden alle Abhängigkeiten und Tools abgerufen, die von unserem Projekt benötigt werden.

```Dockerfile
COPY . ./
RUN dotnet publish -c Release -o out
```

Nachdem .NET alle Abhängigkeiten in den `build-env`-Container gepullt hat, kopiert die nächste Anweisung alle Projektquelldateien in den Container. Wir weisen .NET anschließend an, unsere Anwendung mit einer Releasekonfiguration zu veröffentlichen, und geben den Ausgabepfad an.

Die Kompilierung sollte erfolgreich sein. Nun müssen wir das endgültige Image erstellen. 

> [!TIP]
> In dieser Schnellstartanleitung wird ein .NET Core-Projekt aus der Quelle erstellt. Beim Entwickeln von Containerimages empfiehlt es sich, _nur_ die Produktionsnutzlast und ihre Abhängigkeiten in das Containerimage einzubeziehen. Wir möchten nicht, dass das .NET Core SDK in unser endgültiges Image integriert ist, da wir nur die .NET Core-Laufzeit benötigen. Daher wird die Dockerfile-Datei so geschrieben, dass sie einen temporären Container namens `build-env` verwendet, der mit dem SDK verpackt ist, um die App zu erstellen.

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/aspnet:2.1
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "asp-net-getting-started.dll"]
```

Da die Anwendung ASP.NET ist, geben wir ein Image mit dieser Laufzeit an. Anschließend kopieren wir alle Dateien aus dem Ausgabeverzeichnis unseres temporären Containers in unseren finalen Container. Wir konfigurieren den Container so, dass er beim Starten des Containers mit unserer neuen App als Einstiegspunkt ausgeführt wird.

Wir haben die Dockerfile-Datei für die Ausführung eines _mehrstufigen Buildvorgangs_ geschrieben. Wenn die Dockerfile-Datei ausgeführt wird, verwendet sie den temporären Container `build-env` mit dem .NET Core 2.1 SDK, um die Beispiel-App zu erstellen, und kopiert dann die ausgegebenen Binärdateien in einen anderen Container, der nur die .NET Core 2.1-Laufzeit enthält, sodass wir die Größe des endgültigen Containers minimiert haben.

## <a name="build-and-run-the-app"></a>Erstellen und Ausführen der App

Nachdem die Dockerfile-Datei erstellt wurde, können wir Docker auf unsere Dockerfile-Datei verweisen und Docker anweisen, unser Image zu erstellen und dann auszuführen:

1. Navigieren Sie in einem Eingabeaufforderungsfenster zu dem Verzeichnis, in dem sich die Dockerfile-Datei befindet, und führen Sie dann den Befehl [docker build](https://docs.docker.com/engine/reference/commandline/build/) aus, um den Container aus der Dockerfile-Datei zu erstellen.

   ```Powershell
   docker build -t my-asp-app .
   ```

2. Um den neu erstellten Container auszuführen, führen Sie den Befehl [docker run](https://docs.docker.com/engine/reference/commandline/run/) aus.

   ```Powershell
   docker run -d -p 5000:80 --name myapp my-asp-app
   ```

   Sehen wir uns diesen Befehl genauer an:

   * `-d` weist Docker an, den Container „getrennt“ auszuführen. Dies bedeutet, dass keine Konsole mit der Konsole im Container verknüpft ist. Der Container wird im Hintergrund ausgeführt. 
   * `-p 5000:80` weist Docker an, Port 5000 auf dem Host Port 80 im Container zuzuordnen. Jeder Container erhält seine eigene IP-Adresse. ASP.NET lauscht standardmäßig an Port 80. Mithilfe von Portzuordnung können wir die IP-Adresse des Hosts am zugeordneten Port aufrufen, und Docker leitet den gesamten Datenverkehr an den Zielport innerhalb des Containers weiter.
   * `--name myapp` weist Docker an, diesem Container einen geeigneten Namen für die Abfrage zu geben (anstatt die von Docker zur Laufzeit zugewiesene Container-ID nachzuschlagen).
   * `my-asp-app` ist das Image, das Docker ausführen soll. Dies ist das Containerimage, das als Höhepunkt des `docker build`-Prozesses generiert wird.

3. Öffnen Sie einen Webbrowser, und navigieren Sie zu `http://localhost:5000`, um Ihre containerisierte Anwendung zu sehen, wie in diesem Screenshot gezeigt:

   >![ASP.NET Core-Webseite, die unter localhost in einem Container ausgeführt wird](media/SampleAppScreenshot.png)

## <a name="next-steps"></a>Nächste Schritte

1. Der nächste Schritt besteht darin, Ihre containerisierte ASP.NET-Web-App mit Azure Container Registry in einer privaten Registrierung zu veröffentlichen. Auf diese Weise können Sie sie in Ihrer Organisation bereitstellen.

   > [!div class="nextstepaction"]
   > [Erstellen einer privaten Containerregistrierung](https://docs.microsoft.com/azure/container-registry/container-registry-get-started-powershell)

   Wenn Sie zu dem Abschnitt gelangen, in dem Sie Ihr [Containerimage in die Registrierung pushen](https://docs.microsoft.com/azure/container-registry/container-registry-get-started-powershell#push-image-to-registry), geben Sie den Namen der ASP.NET-App, die Sie soeben gepackt haben (`my-asp-app`), zusammen mit Ihrer Containerregistrierung (Beispiel: `contoso-container-registry`) an:

   ```PowerShell
   docker tag my-asp-app contoso-container-registry.azurecr.io/my-asp-app:v1
   ```

   Weitere App-Beispiele und die zugehörigen Dockerfile-Dateien finden Sie unter [weitere Containerbeispiele](../samples.md).

2. Nachdem Sie Ihre App in der Containerregistrierung veröffentlicht haben, besteht der nächste Schritt in der Bereitstellung der App in einem Kubernetes-Cluster, den Sie mit Azure Kubernetes Service erstellen.

   > [!div class="nextstepaction"]
   > [Erstellen eines Kubernetes-Clusters](https://docs.microsoft.com/azure/aks/windows-container-cli)
