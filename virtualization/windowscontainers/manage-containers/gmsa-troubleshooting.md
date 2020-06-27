---
title: Problembehandlung bei gMSAs für Windows-Container
description: Behandeln von Problemen mit gruppenverwalteten Dienstkonten (Group Managed Service Accounts, gMSAs) für Windows-Container.
keywords: Docker, Container, aktives Verzeichnis, gMSA, gruppenverwaltetes Dienstkonto, gruppenverwaltete Dienstkonten, Problembehandlung, behandeln von Problemen
author: rpsqrd
ms.date: 10/03/2019
ms.topic: troubleshooting
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: e7cf5685620d3cb50c93f48e5aa6917d9044b860
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192827"
---
# <a name="troubleshoot-gmsas-for-windows-containers"></a>Problembehandlung bei gMSAs für Windows-Container

## <a name="known-issues"></a>Bekannte Probleme

### <a name="container-hostname-must-match-the-gmsa-name-for-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>Der Containerhostname muss mit dem gMSA-Namen für Windows Server 2016 und Windows 10, Versionen 1709 und 1803, übereinstimmen.

Wenn Sie Windows Server 2016, Version 1709 oder 1803, ausführen, muss der Hostname Ihres Containers mit Ihrem gMSA SAM-Kontonamen übereinstimmen.

Wenn der Hostname nicht mit dem gMSA-Namen übereinstimmt, treten bei eingehenden NTLM-Authentifizierungsanforderungen und bei der Name/SID-Übersetzung (die von vielen Bibliotheken, z. B. dem Anbieter von ASP.NET-Mitgliedschaftsrollen, verwendet wird) Fehler auf. Kerberos funktioniert weiterhin normal, auch wenn der Hostname und der gMSA-Name nicht übereinstimmen.

Diese Einschränkung wurde in Windows Server 2019 behoben, wo der Container jetzt immer seinen gMSA-Namen im Netzwerk verwendet, unabhängig vom zugewiesenen Hostnamen.

### <a name="using-a-gmsa-with-more-than-one-container-simultaneously-leads-to-intermittent-failures-on-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>Die gleichzeitige Verwendung eines gMSA mit mehr als einem Container führt zu zeitweiligen Ausfällen unter Windows Server 2016 und Windows 10, Versionen 1709 und 1803.

Da alle Container denselben Hostnamen verwenden müssen, betrifft ein zweites Problem Versionen von Windows vor Windows Server 2019 und Windows 10, Version 1809. Wenn mehreren Containern dieselbe Identität und derselbe Hostname zugewiesen werden, kann eine Racebedingung auftreten, wenn zwei Container gleichzeitig mit demselben Domänencontroller kommunizieren. Wenn ein anderer Container mit demselben Domänencontroller kommuniziert, bricht er die Kommunikation mit allen vorherigen Containern mit derselben Identität ab. Dies kann zu zeitweiligen Authentifizierungsfehlern führen und gelegentlich als Vertrauensfehler beobachtet werden, wenn Sie `nltest /sc_verify:contoso.com` innerhalb des Containers ausführen.

Wir haben das Verhalten in Windows Server 2019 geändert, um die Containeridentität vom Computernamen zu trennen, sodass mehrere Container gleichzeitig dasselbe gMSA verwenden können.

### <a name="you-cant-use-gmsas-with-hyper-v-isolated-containers-on-windows-10-versions-1703-1709-and-1803"></a>Sie können gMSAs nicht mit isolierten Hyper-V-Containern unter Windows 10, Versionen 1703, 1709 und 1803, verwenden.

Die Containerinitialisierung reagiert nicht mehr oder weist einen Fehler auf, wenn Sie versuchen, ein gMSA mit einem isolierten Hyper-V-Container unter Windows 10 und den Windows Server-Versionen 1703, 1709 und 1803 zu verwenden.

Dieser Fehler wurde in Windows Server 2019 und Windows 10, Version 1809, behoben. Sie können auch isolierte Hyper-V-Container mit gMSAs unter Windows Server 2016 und Windows 10, Version 1607, ausführen.

## <a name="general-troubleshooting-guidance"></a>Anleitung zur allgemeinen Problembehandlung

Wenn Sie bei der Ausführung eines Containers mit einem gMSA Fehler auftreten, können Ihnen die folgenden Anweisungen helfen, die Ursache zu ermitteln.

### <a name="make-sure-the-host-can-use-the-gmsa"></a>Stellen Sie sicher, dass der Host das gMSA verwenden kann.

1. Überprüfen Sie, ob der Host mit der Domäne verknüpft ist und den Domänencontroller erreichen kann.
2. Installieren Sie die AD PowerShell-Tools von RSAT, und führen Sie [Test-ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount) aus, um zu prüfen, ob der Computer über den Zugriff zum Abrufen des gMSAs verfügt. Wenn das Cmdlet **False** zurückgibt, hat der Computer keinen Zugriff auf das gMSA-Kennwort.

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

3. Wenn **Test-ADServiceAccount** den Wert **False** zurückgibt, überprüfen Sie, ob der Host zu einer Sicherheitsgruppe gehört, die auf das gMSA-Kennwort zugreifen kann.

    ```powershell
    # Get the current computer's group membership
    Get-ADComputer $env:computername | Get-ADPrincipalGroupMembership | Select-Object DistinguishedName

    # Get the groups allowed to retrieve the gMSA password
    # Change "WebApp01" for your own gMSA name
    (Get-ADServiceAccount WebApp01 -Properties PrincipalsAllowedToRetrieveManagedPassword).PrincipalsAllowedToRetrieveManagedPassword
    ```

4. Wenn Ihr Host zu einer Sicherheitsgruppe gehört, die berechtigt ist, das gMSA-Kennwort abzurufen, aber bei **Test-ADServiceAccount** weiterhin ein Fehler auftritt, müssen Sie möglicherweise Ihren Computer neu starten, um ein neues Ticket zu erhalten, das die aktuelle Gruppenmitgliedschaft widerspiegelt.

#### <a name="check-the-credential-spec-file"></a>Überprüfen der Datei für die Spezifikation der Anmeldeinformationen

1. Führen Sie **Get-CredentialSpec** des [CredentialSpec PowerShell-Moduls](https://aka.ms/credspec) aus, um alle Spezifikationen der Anmeldeinformationen auf dem Computer zu ermitteln. Die Spezifikationen der Anmeldeinformationen müssen im Verzeichnis „CredentialSpecs“ unter dem Docker-Stammverzeichnis gespeichert werden. Sie finden das Docker-Stammverzeichnis, indem Sie **docker info -f "{{.DockerRootDir}}"** ausführen.
2. Öffnen Sie die CredentialSpec-Datei, und stellen Sie sicher, dass die folgenden Felder ordnungsgemäß ausgefüllt sind:
    - **Sid**: Die SID Ihres gMSA-Kontos.
    - **MachineAccountName**: Der gMSA SAM-Kontoname (enthält nicht den vollständigen Domänennamen oder das Dollarzeichen).
    - **DnsTreeName**: Der FQDN Ihrer Active Directory-Gesamtstruktur.
    - **DnsName**: Der FQDN der Domäne, zu der das gMSA gehört.
    - **NetBiosName**: NETBIOS-Name für die Domäne, zu der das gMSA gehört.
    - **GroupManagedServiceAccounts/Name**: Der gMSA SAM-Kontoname (enthält nicht den vollständigen Domänennamen oder das Dollarzeichen).
    - **GroupManagedServiceAccounts/Scope**: Ein Eintrag für den Domänen-FQDN und ein Eintrag für NETBIOS.

    Ihre Eingabe sollte wie das folgende Beispiel einer vollständigen Spezifikation für die Anmeldeinformationen aussehen:

    ```json
    {
        "CmsPlugins": [
            "ActiveDirectory"
        ],
        "DomainJoinConfig": {
            "Sid": "S-1-5-21-702590844-1001920913-2680819671",
            "MachineAccountName": "webapp01",
            "Guid": "56d9b66c-d746-4f87-bd26-26760cfdca2e",
            "DnsTreeName": "contoso.com",
            "DnsName": "contoso.com",
            "NetBiosName": "CONTOSO"
        },
        "ActiveDirectoryConfig": {
            "GroupManagedServiceAccounts": [
                {
                    "Name": "webapp01",
                    "Scope": "contoso.com"
                },
                {
                    "Name": "webapp01",
                    "Scope": "CONTOSO"
                }
            ]
        }
    }
    ```

3. Überprüfen Sie, ob der Pfad zur Spezifikationsdatei für die Anmeldeinformationen für Ihre Orchestrierungslösung richtig ist. Wenn Sie Docker verwenden, stellen Sie sicher, dass der Befehl zum Ausführen des Containers `--security-opt="credentialspec=file://NAME.json"` enthält, wobei „NAME.json“ durch den Namen ersetzt wird, der durch **Get-CredentialSpec** ausgegeben wird. Der Name ist ein Flatfilename, relativ zum CredentialSpecs-Ordner unter dem Docker-Stammverzeichnis.

### <a name="check-the-firewall-configuration"></a>Überprüfen der Firewallkonfiguration

Wenn Sie eine strikte Firewallrichtlinie für das Container- oder Hostnetzwerk verwenden, blockiert diese möglicherweise erforderliche Verbindungen zum Active Directory-Domänencontroller oder DNS-Server.

| Protokoll und Port | Zweck |
|-------------------|---------|
| TCP und UDP 53 | Domain Name System |
| TCP und UDP 88 | Kerberos |
| TCP 139 | Anmeldedienst |
| TCP und UDP 389 | LDAP |
| TCP 636 | LDAP SSL |

Je nach Art des Datenverkehrs, den Ihr Container an einen Domänencontroller sendet, müssen Sie möglicherweise den Zugriff auf zusätzliche Ports erlauben.
Eine vollständige Liste der von Active Directory verwendeten Ports finden Sie unter [Active Directory- und Active Directory Domain Services-Portanforderungen](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd772723(v=ws.10)#communication-to-domain-controllers).

### <a name="check-the-container"></a>Überprüfen des Containers

1. Wenn Sie eine Version von Windows vor Windows Server 2019 oder Windows 10, Version 1809, ausführen, muss Ihr Containerhostname mit dem gMSA-Namen übereinstimmen. Stellen Sie sicher, dass der Parameter `--hostname` mit dem gMSA-Kurznamen übereinstimmt (keine Domänenkomponente, z. B. „webapp01“ anstelle von „webapp01.contoso.com“).

2. Überprüfen Sie die Containernetzwerkkonfiguration, um sicherzustellen, dass der Container einen Domänencontroller für die Domäne des gMSAs auflösen und auf ihn zugreifen kann. DNS-Server mit fehlerhafter Konfiguration im Container sind ein häufiger Verursacher von Identitätsproblemen.

3. Überprüfen Sie, ob der Container über eine gültige Verbindung zur Domäne verfügt, indem Sie das folgende Cmdlet im Container ausführen (unter Verwendung von `docker exec` oder einem äquivalenten Cmdlet):

    ```powershell
    nltest /sc_verify:contoso.com
    ```

    Die Überprüfung der Vertrauenswürdigkeit sollte `NERR_SUCCESS` zurückgeben, wenn das gMSA verfügbar ist und die Netzwerkverbindungen es dem Container gestatten, mit der Domäne zu kommunizieren. Wenn dies nicht der Fall ist, überprüfen Sie die Netzwerkkonfiguration des Hosts und Containers. Beide müssen mit dem Domänencontroller kommunizieren können.

4. Überprüfen Sie, ob der Container ein gültiges Kerberos-TGT (Ticket-Granting Ticket) abrufen kann:

    ```powershell
    klist get krbtgt
    ```

    Dieser Befehl sollte „Ein Ticket für krbtgt wurde erfolgreich abgerufen“ zurückgeben und den Domänencontroller auflisten, der zum Abrufen des Tickets verwendet wurde. Wenn Sie ein TGT abrufen können, aber bei `nltest` aus dem vorherigen Schritt ein Fehler auftritt, kann dies ein Hinweis darauf sein, dass das gMSA-Konto falsch konfiguriert ist. Weitere Informationen finden Sie unter [Überprüfen des gMSA-Kontos](#check-the-gmsa-account).

    Wenn Sie innerhalb des Containers kein TGT abrufen können, kann dies auf DNS- oder Netzwerkverbindungsprobleme hinweisen. Stellen Sie sicher, dass der Container einen Domänencontroller unter Verwendung des DNS-Namens der Domäne auflösen kann und dass der Domänencontroller über den Container routingfähig ist.

5. Stellen Sie sicher, dass Ihre App für die [Verwendung des gMSAs konfiguriert ist](gmsa-configure-app.md). Das Benutzerkonto innerhalb des Containers ändert sich nicht, wenn Sie ein gMSA verwenden. Vielmehr verwendet das Systemkonto das gMSA, wenn es mit anderen Netzwerkressourcen kommuniziert. Das bedeutet, dass Ihre Anwendung als Netzwerkdienst oder lokales System ausgeführt werden muss, um die gMSA-Identität zu nutzen.

    > [!TIP]
    > Wenn Sie `whoami` ausführen oder ein anderes Tool verwenden, um Ihren aktuellen Benutzerkontext im Container zu identifizieren, wird der gMSA-Name selbst nicht angezeigt. Dies liegt daran, dass Sie sich immer als lokaler Benutzer und nicht als Domänenidentität am Container anmelden. Das gMSA wird vom Computerkonto immer dann verwendet, wenn es mit Netzwerkressourcen kommuniziert, weshalb Ihre Anwendung als Netzwerkdienst oder lokales System ausgeführt werden muss.

### <a name="check-the-gmsa-account"></a>Überprüfen des gMSA-Kontos

1. Wenn Ihr Container ordnungsgemäß konfiguriert zu sein scheint, aber Benutzer oder andere Dienste nicht in der Lage sind, sich automatisch bei Ihrer Container-App zu authentifizieren, überprüfen Sie die SPNs für Ihr gMSA-Konto. Clients finden das gMSA-Konto unter dem Namen, unter dem sie Ihre Anwendung erreichen. Dies kann bedeuten, dass Sie zusätzliche `host`-SPNs für Ihr gMSA benötigen, wenn sich z. B. Clients über einen Lastenausgleich oder einen anderen DNS-Namen mit Ihrer App verbinden.

2. Stellen Sie sicher, dass das gMSA und der Containerhost zu derselben Active Directory-Domäne gehören. Der Containerhost ist nicht in der Lage, das gMSA-Kennwort abzurufen, wenn das gMSA zu einer anderen Domäne gehört.

3. Stellen Sie sicher, dass es in Ihrer Domäne nur ein Konto mit demselben Namen wie Ihr gMSA gibt. An gMSA-Objekte ist ein Dollarzeichen ($) an ihren SAM-Kontonamen angefügt, sodass es möglich ist, dass ein gMSA den Namen „myaccount$“ und ein nicht verwandtes Benutzerkonto in derselben Domäne den Namen „myaccount“ erhält. Dies kann Probleme verursachen, wenn der Domänencontroller oder die Anwendung das gMSA anhand des Namens suchen muss. Mit dem folgenden Befehl können Sie AD nach ähnlich benannten Objekten durchsuchen:

    ```powershell
    # Replace "GMSANAMEHERE" with your gMSA account name (no trailing dollar sign)
    Get-ADObject -Filter 'sAMAccountName -like "GMSANAMEHERE*"'
    ```

4. Wenn Sie die uneingeschränkte Delegierung für das gMSA-Konto aktiviert haben, stellen Sie sicher, dass für das [UserAccountControl-Attribut](https://support.microsoft.com/help/305144/how-to-use-useraccountcontrol-to-manipulate-user-account-properties) immer noch das `WORKSTATION_TRUST_ACCOUNT`-Flag aktiviert ist. Dieses Flag ist erforderlich, damit NETLOGON im Container mit dem Domänencontroller kommunizieren kann, wie es der Fall ist, wenn eine App einen Namen in eine SID auflösen muss oder umgekehrt. Sie können mit den folgenden Befehlen überprüfen, ob das Flag ordnungsgemäß konfiguriert ist:

    ```powershell
    $gMSA = Get-ADServiceAccount -Identity 'yourGmsaName' -Properties UserAccountControl
    ($gMSA.UserAccountControl -band 0x1000) -eq 0x1000
    ```

    Wenn die obigen Befehle `False` zurückgeben, verwenden Sie Folgendes, um der UserAccountControl-Eigenschaft des gMSA-Kontos das Flag `WORKSTATION_TRUST_ACCOUNT` hinzuzufügen. Dieser Befehl löscht auch die Flags `NORMAL_ACCOUNT`, `INTERDOMAIN_TRUST_ACCOUNT` und `SERVER_TRUST_ACCOUNT` aus der UserAccountControl-Eigenschaft.

    ```powershell
    Set-ADObject -Identity $gMSA -Replace @{ userAccountControl = ($gmsa.userAccountControl -band 0x7FFFC5FF) -bor 0x1000 }
    ```
