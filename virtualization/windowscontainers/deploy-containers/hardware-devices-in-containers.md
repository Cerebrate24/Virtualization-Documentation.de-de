---
title: Geräte in Containern unter Windows
description: Verfügbare Geräteunterstützung für Container unter Windows
keywords: Docker, Container, Geräte, Hardware
author: cwilhit
ms.openlocfilehash: 1ad63c158a42f116882c949b242274dde8d893fc
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910600"
---
# <a name="devices-in-containers-on-windows"></a>Geräte in Containern unter Windows

Standardmäßig erhalten Windows-Container minimalen Zugriff auf Hostgeräte – genau wie Linux-Container. Es gibt bestimmte Workloads, bei denen es vorteilhaft (oder sogar erforderlich) ist, auf Hosthardwaregeräte zuzugreifen und mit ihnen zu kommunizieren. In diesem Leitfaden erfahren Sie, welche Geräte in Containern unterstützt werden und wie Sie die ersten Schritte unternehmen können.

## <a name="requirements"></a>Anforderungen

Damit dieses Feature funktioniert, muss Ihre Umgebung die folgenden Anforderungen erfüllen:
- Auf dem Containerhost muss Windows Server 2019 oder Windows 10, Version 1809 oder höher, ausgeführt werden.
- Ihre Basisimageversion für den Container muss 1809 oder höher sein.
- Ihre Container müssen Windows-Container sein, die im prozessisolierten Modus ausgeführt werden.
- Auf dem Containerhost muss das Docker-Modul 19.03 oder höher ausgeführt werden.

## <a name="run-a-container-with-a-device"></a>Ausführen eines Containers mit einem Gerät

Verwenden Sie den folgenden Befehl, um einen Container mit einem Gerät zu starten:

```shell
docker run --isolation=process --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

Sie müssen die `{interface class guid}` durch eine entsprechende [Geräteschnittstellenklasse-GUID](https://docs.microsoft.com/windows-hardware/drivers/install/overview-of-device-interface-classes) ersetzen, die im folgenden Abschnitt zu finden ist.

Verwenden Sie zum Starten eines Containers mit mehreren Geräten den folgenden Befehl, und fügen Sie mehrere `--device`-Argumente aneinander:

```shell
docker run --isolation=process --device="class/{interface class GUID}" --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

In Windows deklarieren alle Geräte eine Liste von Schnittstellenklassen, die sie implementieren. Indem dieser Befehl an Docker übergeben wird, stellt dieser sicher, dass alle Geräte, die die angeforderte Klasse implementieren, in den Container integriert werden.

Dies bedeutet, dass Sie **nicht** das Gerät vom Host entfernt zuweisen. Stattdessen nutzt der Host es gemeinsam mit dem Container. Ebenso werden, da Sie eine Klassen-GUID angeben, _alle_ Geräte, die diese GUID implementieren, mit dem Container gemeinsam genutzt.

## <a name="what-devices-are-supported"></a>Welche Geräte werden unterstützt?

Die folgenden Geräte (und ihre Geräteschnittstellenklassen-GUIDs) werden heute unterstützt:
  
<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:75%" cellpadding="5" cellspacing="5">
<thead>
<tr valign="top">
<th><center>Gerätetyp</center></th>
<th><center>Schnittstellenklassen-GUID</center></th>
</tr>
</thead>
<tbody>
<tr valign="top">
<td><center>GPIO</center></td>
<td><center>916EF1CB-8426-468D-A6F7-9AE8076881B3</center></td>
</tr>
<tr valign="top">
<td><center>I2C-Bus</center></td>
<td><center>A11EE3C6-8421-4202-A3E7-B91FF90188E4</center></td>
</tr>
<tr valign="top">
<td><center>COM-Anschluss</center></td>
<td><center>86E0D1E0-8089-11D0-9CE4-08003E301F73</center></td>
</tr>
<tr valign="top">
<td><center>SPI-Bus</center></td>
<td><center>DCDE6AF9-6610-4285-828F-CAAF78C424CC</center></td>
</tr>
<tr valign="top">
<td><center>DirectX-GPU-Beschleunigung</center></td>
<td><center>Weitere Informationen finden Sie in den Dokumenten zur <a href="https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/gpu-acceleration">GPU-Beschleunigung</a></center></td>
</tr>
</tbody>
</table>

> [!IMPORTANT]
> Die Geräteunterstützung ist vom Treiber abhängig. Der Versuch, Klassen-GUIDs zu übergeben, die nicht in der obigen Tabelle definiert sind, kann zu undefiniertem Verhalten führen.

## <a name="hyper-v-isolated-windows-container-support"></a>Unterstützung für Windows-Container mit Hyper-V-Isolation

Gerätezuweisung und Gerätefreigabe für Workloads in Windows-Containern mit Hyper-V-Isolation wird derzeit nicht unterstützt.

## <a name="hyper-v-isolated-linux-container-support"></a>Unterstützung von Linux-Containern mit Hyper-V-Isolation

Gerätezuweisung und Gerätefreigabe für Workloads in Linux-Containern mit Hyper-V-Isolation wird derzeit nicht unterstützt.
