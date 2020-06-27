---
title: Informationen zu Docker
description: Erfahren Sie mehr über Docker.
keywords: Docker, Container
author: Heidilohr
ms.author: helohr
ms.date: 05/22/2019
ms.topic: overview
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: a2a1f89d7a62479dfae073c7f28fa4a86c693597
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192877"
---
# <a name="about-docker"></a>Informationen zu Docker

Informationen zu Containern erwähnen zwangsläufig auch Docker. Die Docker-Engine ist ein Containerverwaltungstool, das Containerimages verpackt und übermittelt. Die sich ergebenden Images können überall als Container ausgeführt werden: lokal, in der Cloud oder auf einem PC.

![](media/docker.png)

Ein Windows Server-Container kann wie jeder andere Container mit [Docker](https://www.docker.com) verwaltet werden.

Das Konzept von Isolierung von Namespaces und Ressourcenkontrolle durch einen Container ist nicht wirklich neu und geht zurück auf BSD Jails, Solaris Zones und den grundlegenden UNIX-Mechanismus „change root“ (chroot). Docker stellt eine solide Grundlage für die Entwicklung durch ein allgemeines Toolset, ein Verpackungsmodell und einen Bereitstellungsmechanismus zur Verfügung, die die Containerisierung und die Verteilung von Anwendungen vereinfachen. Diese Anwendungen können dann auf einem beliebigen Linux-Host und in Windows ausgeführt werden.

Ein weit verbreitetes Verpackungsmodell und eine Bereitstellungstechnologie vereinfachen die Verwaltung durch Verfügbarkeit derselben Verwaltungsbefehle auf beliebigen Hosts, sodass der Weg für reibungslose DevOps-Vorgänge geebnet wird. Sie können auch ein Docker-Image erstellen, das in Sekundenschnelle in jeder Umgebung identisch bereitgestellt werden kann, und zwar unabhängig davon, ob es sich um den Desktop eines Entwicklers, einen Testcomputer oder eine Gruppe von Produktionscomputern handelt. Mittlerweile gibt es eine riesiges und weiter wachsendes Ökosystem von Anwendungen, die in Docker-Containern gepackt sind, und zwar in DockerHub, der von Docker verwalteten öffentlichen containerisierten Anwendungsregistrierung.

Befassen wir uns nun mit diesem Ökosystem und damit, wie Sie auf Docker-Konzepten aufbauen können, um einen Entwicklungs- und Bereitstellungsworkflow entsprechend Ihren Anforderungen einzurichten.

## <a name="get-started-with-docker"></a>Erste Schritte mit Docker

Informationen zum Erstellen von Containern mit Docker finden Sie unter [Docker-Engine unter Windows](../manage-docker/configure-docker-daemon.md). Ausführlichere Informationen zur Verwendung von Docker finden Sie auch auf der [Docker-Website](https://www.docker.com).