---
title: Ausführen eines Containers mit einem gMSA
description: Ausführen eines Windows-Containers mit einem gruppenverwalteten Dienstkonto (gMSA)
keywords: Docker, Container, aktives Verzeichnis, gMSA, gruppenverwaltetes Dienstkonto, gruppenverwaltete Dienstkonten
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: b997cf79cdf7f1782b6299198859714563c45f8c
ms.sourcegitcommit: ac923217ee2f74f08df2b71c2a4c57b694f0d7c3
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 03/06/2020
ms.locfileid: "78853934"
---
# <a name="run-a-container-with-a-gmsa"></a>Ausführen eines Containers mit einem gMSA

Stellen Sie zum Ausführen eines Containers mit einem gruppenverwalteten Dienstkonto (gMSA) die Spezifikationsdatei für die Anmeldeinformationen dem Parameter `--security-opt` von [docker run](https://docs.docker.com/engine/reference/run) zur Verfügung:

```powershell
# For Windows Server 2016, change the image name to mcr.microsoft.com/windows/servercore:ltsc2016
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

>[!IMPORTANT]
>Unter Windows Server 2016, Version 1709 und 1803, muss der Hostname des Containers mit dem Namen des gMSA-Kurznamens identisch sein.

Im vorherigen Beispiel lautet der gMSA SAM-Kontoname „webapp01“, sodass der Containerhostname ebenfalls „webapp01“ genannt wird.

Unter Windows Server 2019 und höher ist das Feld „Hostname“ nicht erforderlich, aber der Container identifiziert sich immer noch über den gMSA-Namen anstelle des Hostnamens, auch wenn Sie explizit einen anderen angeben.

Führen Sie das folgende Cmdlet im Container aus, um zu überprüfen, ob das gMSA ordnungsgemäß funktioniert:

```powershell
# Replace contoso.com with your own domain
PS C:\> nltest /sc_verify:contoso.com

Flags: b0 HAS_IP  HAS_TIMESERV
Trusted DC Name \\dc01.contoso.com
Trusted DC Connection Status Status = 0 0x0 NERR_Success
Trust Verification Status = 0 0x0 NERR_Success
The command completed successfully
```

Wenn der vertrauenswürdige DC-Verbindungsstatus und der Überprüfungsstatus der Vertrauensstellung nicht `NERR_Success` sind, befolgen Sie die [Anweisungen zur Problembehandlung](gmsa-troubleshooting.md#check-the-container), um das Problem zu beheben.

Sie können die gMSA-Identität aus dem Container heraus überprüfen, indem Sie den folgenden Befehl ausführen und den Clientnamen überprüfen:

```powershell
PS C:\> klist get webapp01

Current LogonId is 0:0xaa79ef8
A ticket to krbtgt has been retrieved successfully.

Cached Tickets: (2)

#0>     Client: webapp01$ @ CONTOSO.COM
        Server: krbtgt/webapp01 @ CONTOSO.COM
        KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
        Ticket Flags 0x40a10000 -> forwardable renewable pre_authent name_canonicalize
        Start Time: 3/21/2019 4:17:53 (local)
        End Time:   3/21/2019 14:17:53 (local)
        Renew Time: 3/28/2019 4:17:42 (local)
        Session Key Type: AES-256-CTS-HMAC-SHA1-96
        Cache Flags: 0
        Kdc Called: dc01.contoso.com

[...]
```

Zum Öffnen von PowerShell oder einer anderen Konsolenanwendung als gMSA-Konto können Sie den Container auffordern, unter dem Netzwerkdienstkonto statt unter dem normalen ContainerAdministrator-Konto (oder ContainerUser für NanoServer) ausgeführt zu werden:

```powershell
# NOTE: you can only run as Network Service or SYSTEM on Windows Server 1709 and later
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 --user "NT AUTHORITY\NETWORK SERVICE" -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

Wenn Sie als Netzwerkdienst ausführen, können Sie die Netzwerkauthentifizierung als gMSA testen, indem Sie versuchen, eine Verbindung zu SYSVOL auf einem Domänencontroller herzustellen:

```powershell
# This command should succeed if you're successfully running as the gMSA
PS C:\> dir \\contoso.com\SYSVOL


    Directory: \\contoso.com\sysvol


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d----l        2/27/2019   8:09 PM                contoso.com
```

## <a name="next-steps"></a>Nächste Schritte

Zusätzlich zum Ausführen von Containern können Sie gMSAs auch zu Folgendem verwenden:

- [Konfigurieren von Apps](gmsa-configure-app.md)
- [Orchestrieren von Containern](gmsa-orchestrate-containers.md)

Wenn während des Setups Probleme auftreten, überprüfen Sie unseren [Leitfaden zur Problembehandlung](gmsa-troubleshooting.md) auf mögliche Lösungen.
