---
title: Bekannte Probleme
description: Bekannte Probleme bei Windows Server-Containern
keywords: Metadaten, Container, Version
author: weijuans
ms. author: weijuans
manager: taylob
ms.date: 05/26/2020
ms.openlocfilehash: f53ff0c8c07e86b25358a3acba09622fdd3a6bbb
ms.sourcegitcommit: 82f45088d8b39e2c1f3f2a33bf359d18a88f975a
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 06/18/2020
ms.locfileid: "84978332"
---
# <a name="known-issues"></a>Bekannte Probleme

## <a name="know-issues-of-windows-server-version-2004"></a>Bekannte Probleme bei Windows Server, Version 2004

### <a name="1-performance-issue-on-server-core-container"></a>1. Leistungsprobleme bei Server Core-Containern
In Vorbereitung auf die allgemein verfügbare Windows Server-Version 2004 erkannten wir mit dem .NET-Team ein potenzielles Leistungsproblem im aktuellen Image des Server Core-Containers in der Version vom 27. Mai 2020, verglichen mit der verbesserten Leistung, über die wir im Blog im Dezember 2019 [hier](https://techcommunity.microsoft.com/t5/containers/making-windows-server-core-containers-40-smaller/ba-p/1058874) und im .NET Team-Blog [hier](https://devblogs.microsoft.com/dotnet/we-made-windows-server-core-container-images-40-smaller/) berichteten. Die Leistungsanalyse erfolgte damals auf einem Containerimage von Windows Server Core, Version 2004 Insider Preview. 

Dies sind die damals beobachteten Symptome:

Wenn Sie das Server Core-Containerimage verwenden, um ein eigenes Image zu erstellen und es auf eine Container-Remoteregistrierung wie etwa Azure Container Registry hochzuladen, dieses Image dann bei der Registrierung abrufen und es ausführen, werden Sie eine geringere Leistung des Containers feststellen. Wenn Sie das Image jedoch lokal erstellen und ausführen, tritt dieser Leistungsunterschied nicht auf.

Dies ist ein grundsätzliches Problem, und wir arbeiten an seiner Behebung. Sie können das Problem über die folgenden Links nachverfolgen: [microsoft/hcsshim#830](https://github.com/microsoft/hcsshim/issues/830);

[moby/moby#41066](https://github.com/moby/moby/issues/41066);

[containerd/containerd#4301](https://github.com/containerd/containerd/issues/4301).

Wir arbeiten außerdem aktiv mit unserem Partner Mirantis daran, diesen Fix in Docker EE bereitzustellen.

## <a name="know-issues-of-windows-server-version-1909"></a>Bekannte Probleme bei Windows Server, Version 1909

## <a name="know-issues-of-windows-server-version-1903"></a>Bekannte Probleme bei Windows Server, Version 1903

## <a name="know-issues-of-windows-server-2019windows-server-version-1809"></a>Bekannte Probleme bei Windows Server 2019/Windows Server, Version 1809

## <a name="know-issues-of-windows-server-2016"></a>Bekannte Probleme bei Windows Server 2016
