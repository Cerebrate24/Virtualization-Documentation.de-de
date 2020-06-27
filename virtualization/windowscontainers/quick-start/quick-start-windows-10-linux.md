---
title: Windows- und Linux-Container unter Windows 10
description: Containerbereitstellung – Schnellstart
keywords: Docker, Container, LCOW
author: taylorb-microsoft
ms.date: 08/16/2019
ms.topic: tutorial
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: df02dada3e5cf759f003999b38270dd1bf3131fe
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192187"
---
# <a name="linux-containers-on-windows-10"></a>Linux-Container unter Windows 10

Diese Übung führt Sie schrittweise durch das Erstellen und Ausführen von Linux-Containern unter Windows 10.

In diesem Schnellstart lernen Sie Folgendes:

1. Installieren von Docker Desktop
2. Ausführen eines einfachen Linux-Containers

Dieser Schnellstart bezieht sich speziell auf Windows 10. Weitere Schnellstartdokumentation finden Sie links auf dieser Seite im Inhaltsverzeichnis.

## <a name="prerequisites"></a>Voraussetzungen

Stellen Sie sicher, dass Sie die folgenden Anforderungen erfüllen:
- Ein physisches Computersystem mit Windows 10 Professional, Windows 10 Enterprise oder Windows Server 2019, Version 1809 oder höher.
- Stellen Sie sicher, dass [Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) aktiviert ist.

## <a name="install-docker-desktop"></a>Installieren von Docker Desktop

Laden Sie [Docker Desktop](https://store.docker.com/editions/community/docker-ce-desktop-windows) herunter, und führen Sie das Installationsprogramm aus. (Sie müssen sich anmelden. Erstellen Sie ein Konto, sofern noch nicht vorhanden.) [Ausführliche Informationen zur Installation](https://docs.docker.com/docker-for-windows/install) finden Sie in der Dokumentation zu Docker.

## <a name="run-your-first-linux-container"></a>Ausführen Ihres ersten Linux-Containers

Beim Ausführen von Linux-Containern müssen Sie darauf achten, dass Docker den richtigen Daemon anzielt. Diese Einstellung können Sie beim Klicken auf das Docker-Walsymbol umschalten, indem Sie `Switch to Linux Containers` im Aktionsmenü auswählen. Wenn `Switch to Windows Containers` angezeigt wird, ist der Linux-Daemon bereits als Ziel festgelegt.

![Docker-Taskleistenmenü mit dem Befehl „Auf Windows-Container umstellen“.](./media/switchDaemon.png)

Nachdem Sie sich vergewissert haben, dass Sie den richtigen Daemon anzielen, führen Sie den Container mit dem folgenden Befehl aus:

```console
docker run --rm busybox echo hello_world
```

Der Container sollte ausgeführt werden, „hello_world“ ausgeben und dann beendet werden.

Wenn Sie `docker images` abfragen, sollte das Linux-Container Image angezeigt werden, das Sie soeben abgerufen haben:

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
busybox                latest              59788edf1f3e        4 weeks ago         3.41MB
```

## <a name="next-steps"></a>Nächste Schritte

> [!div class="nextstepaction"]
> [Erfahren Sie, wie eine Beispiel-App erstellt wird](./building-sample-app.md)
