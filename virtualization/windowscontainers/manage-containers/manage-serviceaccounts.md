---
title: Erstellen von gMSAs für Windows-Container
description: Erstellen von gruppenverwalteten Dienstkonten (Group Managed Service Accounts, gMSAs) für Windows-Container.
keywords: Docker, Container, aktives Verzeichnis, gMSA, gruppenverwaltetes Dienstkonto, gruppenverwaltete Dienstkonten
author: rpsqrd
ms.date: 01/03/2019
ms.topic: how-to
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: ea0695e94bd0b4898f6f99b5e797a23c60ef5d1f
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192787"
---
# <a name="create-gmsas-for-windows-containers"></a>Erstellen von gMSAs für Windows-Container

Windows-basierte Netzwerke verwenden häufig Active Directory (AD), um die Authentifizierung und Autorisierung zwischen Benutzern, Computern und anderen Netzwerkressourcen zu erleichtern. Entwickler von Unternehmensanwendungen entwerfen ihre Apps häufig so, dass sie in AD integriert sind und auf Servern ausgeführt werden, die mit Domänen verbunden sind, um die Vorteile der integrierten Windows-Authentifizierung zu nutzen, die es Benutzern und anderen Diensten erleichtert, sich automatisch und transparent mit ihrer Identität bei der Anwendung anzumelden.

Obwohl Windows-Container nicht mit Domänen verbunden werden können, können sie dennoch Active Directory-Domänenidentitäten verwenden, um verschiedene Authentifizierungsszenarien zu unterstützen.

Dazu können Sie einen Windows-Container so konfigurieren, dass er mit einem [gMSA](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview) (gruppenverwalteten Dienstkonto) ausgeführt wird, bei dem es sich um eine besondere Art von Dienstkonto handelt, das mit Windows Server 2012 eingeführt wurde und es mehreren Computern ermöglichen soll, eine Identität gemeinsam zu nutzen, ohne dass sie das Kennwort kennen müssen.

Wenn Sie einen Container mit einem gMSA ausführen, ruft der Containerhost das gMSA-Kennwort von einem Active Directory-Domänencontroller ab und gibt es an die Containerinstanz weiter. Der Container verwendet die gMSA-Anmeldeinformationen immer dann, wenn sein Computerkonto (SYSTEM) auf Netzwerkressourcen zugreifen muss.

In diesem Artikel werden die ersten Schritte bei der Verwendung von gruppenverwalteten Active Directory-Konten mit Windows-Containern erläutert.

## <a name="prerequisites"></a>Voraussetzungen

Sie benötigen Folgendes, um einen Windows-Container mit einem gruppenverwalteten Dienstkonto auszuführen:

- Eine Active Directory-Domäne mit mindestens einem Domänencontroller mit Windows Server 2012 oder höher. Es gibt keine Anforderungen auf Gesamtstruktur- oder Domänenfunktionsebene für die Verwendung von gMSAs, aber die gMSA-Kennwörter können nur von Domänencontrollern mit Windows Server 2012 oder höher verteilt werden. Weitere Informationen finden Sie unter [Active Directory-Anforderungen für gMSAs](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/getting-started-with-group-managed-service-accounts#BKMK_gMSA_Req).
- Berechtigung zum Erstellen eines gMSA-Kontos. Um ein gMSA-Konto zu erstellen, müssen Sie Domänenadministrator sein oder ein Konto verwenden, dem die Berechtigung *msDS-GroupManagedServiceAccount-Objekte erstellen* delegiert wurde.
- Zugriff auf das Internet, um das CredentialSpec PowerShell-Modul herunterzuladen. Wenn Sie in einer nicht verbundenen Umgebung arbeiten, können Sie [das Modul auf einem Computer mit Internetzugang speichern](https://docs.microsoft.com/powershell/module/powershellget/save-module?view=powershell-5.1) und es auf Ihren Entwicklungscomputer oder Containerhost kopieren.

## <a name="one-time-preparation-of-active-directory"></a>Einmalige Vorbereitung von Active Directory

Wenn Sie nicht bereits einen gMSA in Ihrer Domäne erstellt haben, müssen Sie den Stammschlüssel des Schlüsselverteilungsdiensts (KDS) generieren. Der KDS ist für die Erstellung, Rotation und Freigabe des gMSA-Kennworts für autorisierte Hosts verantwortlich. Wenn ein Containerhost das gMSA zum Betrieb eines Containers verwenden muss, kontaktiert er den KDS, um das aktuelle Kennwort abzurufen.

Um zu überprüfen, ob der KDS-Stammschlüssel bereits erstellt wurde, führen Sie das folgende PowerShell-Cmdlet als Domänenadministrator auf einem Domänencontroller oder auf einem Domänenmitglied aus, auf dem die AD PowerShell-Tools installiert sind:

```powershell
Get-KdsRootKey
```

Wenn der Befehl eine Schlüssel-ID zurückgibt, sind Sie bereit und können zum Abschnitt [Erstellen eines gruppenverwalteten Dienstkontos](#create-a-group-managed-service-account) übergehen. Andernfalls fahren Sie mit der Erstellung des KDS-Stammschlüssels fort.

Führen Sie in einer Produktionsumgebung oder Testumgebung mit mehreren Domänencontrollern das folgende Cmdlet in PowerShell als Domänenadministrator aus, um den KDS-Stammschlüssel zu erstellen.

```powershell
# For production environments
Add-KdsRootKey -EffectiveImmediately
```

Obwohl der Befehl impliziert, dass der Schlüssel sofort wirksam wird, müssen Sie 10 Stunden warten, bevor der KDS-Stammschlüssel repliziert wird und zur Verwendung auf allen Domänencontrollern verfügbar ist.

Wenn Sie in Ihrer Domäne nur über einen Domänencontroller verfügen, können Sie den Prozess beschleunigen, indem Sie den Schlüssel so einstellen, dass er vor 10 Stunden wirksam wird.

>[!IMPORTANT]
>Verwenden Sie dieses Verfahren nicht in einer Produktionsumgebung.

```powershell
# For single-DC test environments ONLY
Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
```

## <a name="create-a-group-managed-service-account"></a>Erstellen eines gruppenverwalteten Dienstkontos

Jeder Container, der die integrierte Windows-Authentifizierung verwendet, benötigt mindestens ein gMSA. Das primäre gMSA wird immer dann verwendet, wenn Anwendungen, die als System- oder Netzwerkdienst ausgeführt werden, auf Ressourcen im Netzwerk zugreifen. Der Name des gMSAs wird im Netzwerk zum Namen des Containers, unabhängig vom Hostnamen, der dem Container zugewiesen ist. Container können auch mit zusätzlichen gMSAs konfiguriert werden, falls Sie einen Dienst oder eine Anwendung im Container als eine andere Identität als das Containercomputerkonto ausführen möchten.

Wenn Sie ein gMSA erstellen, erstellen Sie auch eine gemeinsame Identität, die gleichzeitig auf vielen verschiedenen Computern verwendet werden kann. Der Zugriff auf das gMSA-Kennwort ist durch eine Active Directory-Zugriffssteuerungsliste geschützt. Wir empfehlen, für jedes gMSA-Konto eine Sicherheitsgruppe zu erstellen und die entsprechenden Containerhosts zur Sicherheitsgruppe hinzuzufügen, um den Zugriff auf das Kennwort zu beschränken.

Da Container nicht automatisch Dienstprinzipalnamen (SPN) registrieren, müssen Sie schließlich mindestens einen Host-SPN für Ihr gMSA-Konto manuell erstellen.

Normalerweise wird der Host- oder HTTP-SPN unter demselben Namen wie das gMSA-Konto registriert, aber Sie müssen möglicherweise einen anderen Dienstnamen verwenden, wenn Clients auf die Container-App hinter einem Lastenausgleich oder einem DNS-Namen zugreifen, der sich vom gMSA-Namen unterscheidet.

Wenn das gMSA-Konto z. B. „WebApp01“ heißt, Ihre Benutzer jedoch auf den Standort unter `mysite.contoso.com` zugreifen, sollten Sie ein `http/mysite.contoso.com`-SPN für das gMSA-Konto registrieren.

Einige Anwendungen erfordern möglicherweise zusätzliche SPNs für ihre eindeutigen Protokolle. Beispielsweise ist für SQL Server der `MSSQLSvc/hostname`-SPN erforderlich.

Die folgende Tabelle listet die erforderlichen Attribute für die Erstellung eines gMSAs auf.

|gMSA-Eigenschaft | Erforderlicher Wert | Beispiel |
|--------------|----------------|--------|
|Name | Ein beliebiger gültiger Kontoname. | `WebApp01` |
|DnsHostName | Der an den Kontonamen angefügte Domänenname. | `WebApp01.contoso.com` |
|ServicePrincipalNames | Legen Sie mindestens den Host-SPN fest. Fügen Sie bei Bedarf weitere Protokolle hinzu. | `'host/WebApp01', 'host/WebApp01.contoso.com'` |
|PrincipalsAllowedToRetrieveManagedPassword | Die Sicherheitsgruppe, die Ihre Containerhosts enthält. | `WebApp01Hosts` |

Nachdem Sie sich für den Namen für Ihre gMSA entschieden haben, führen Sie die folgenden Cmdlets in PowerShell aus, um die Sicherheitsgruppe und das gMSA zu erstellen.

> [!TIP]
> Sie müssen ein Konto verwenden, das zur Sicherheitsgruppe **Domänen-Admins** gehört oder dem die Berechtigung **msDS-GroupManagedServiceAccount-Objekte erstellen** zugewiesen wurde, um die folgenden Befehle auszuführen.
> Das Cmdlet [New-ADServiceAccount](https://docs.microsoft.com/powershell/module/addsadministration/new-adserviceaccount?view=win10-ps) ist Teil der AD PowerShell-Tools von [Remoteserver-Verwaltungstools](https://aka.ms/rsat).

```powershell
# Replace 'WebApp01' and 'contoso.com' with your own gMSA and domain names, respectively

# To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
# To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
# To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

# Create the security group
New-ADGroup -Name "WebApp01 Authorized Hosts" -SamAccountName "WebApp01Hosts" -GroupScope DomainLocal

# Create the gMSA
New-ADServiceAccount -Name "WebApp01" -DnsHostName "WebApp01.contoso.com" -ServicePrincipalNames "host/WebApp01", "host/WebApp01.contoso.com" -PrincipalsAllowedToRetrieveManagedPassword "WebApp01Hosts"

# Add your container hosts to the security group
Add-ADGroupMember -Identity "WebApp01Hosts" -Members "ContainerHost01$", "ContainerHost02$", "ContainerHost03$"
```

Es wird empfohlen, separate gMSA-Konten für Ihre Entwicklungs-, Test- und Produktionsumgebung zu erstellen.

## <a name="prepare-your-container-host"></a>Vorbereiten des Containerhosts

Jeder Containerhost, auf dem ein Windows-Container mit einem gMSA ausgeführt werden soll, muss mit der Domäne verknüpft sein und Zugriff auf das gMSA-Kennwort haben.

1. Verbinden Sie Ihren Computer mit Ihrer Active Directory-Domäne.
2. Stellen Sie sicher, dass Ihr Host zu der Sicherheitsgruppe gehört, die den Zugriff auf das gMSA-Kennwort kontrolliert.
3. Starten Sie den Computer neu, damit er seine neue Gruppenmitgliedschaft erhält.
4. Richten Sie [Docker Desktop für Windows 10](https://docs.docker.com/docker-for-windows/install/) oder [Docker für Windows Server](https://docs.docker.com/install/windows/docker-ee/) ein.
5. (Empfohlen) Überprüfen Sie, ob der Host das gMSA-Konto verwenden kann, indem Sie [Test-ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount) ausführen. Wenn der Befehl **False** zurückgibt, befolgen Sie die [Anweisungen zur Problembehandlung](gmsa-troubleshooting.md#make-sure-the-host-can-use-the-gmsa).

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

## <a name="create-a-credential-spec"></a>Erstellen einer Spezifikation der Anmeldeinformationen

Eine Spezifikationsdatei für die Anmeldeinformationen ist ein JSON-Dokument, das Metadaten über die gMSA-Konten enthält, die ein Container verwenden soll. Indem Sie die Identitätskonfiguration vom Containerimage getrennt halten, können Sie ändern, welches gMSA der Container verwendet, indem Sie einfach die Spezifikationsdatei der Anmeldeinformationen austauschen, ohne dass Codeänderungen erforderlich sind.

Die Spezifikationsdatei für die Anmeldeinformationen wird mit dem [CredentialSpec PowerShell-Modul](https://aka.ms/credspec) auf einem Containerhost erstellt, der mit der Domäne verbunden ist.
Nachdem Sie die Datei erstellt haben, können Sie sie auf andere Containerhosts oder Ihren Containerorchestrator kopieren.
Die Spezifikationsdatei für die Anmeldeinformationen enthält keine Geheimnisse, z. B. das gMSA-Kennwort, da der Containerhost das gMSA im Auftrag des Containers abruft.

Docker erwartet, dass die Spezifikationsdatei für die Anmeldeinformationen unter dem Verzeichnis **CredentialSpecs** im Docker-Datenverzeichnis zu finden ist. In einer Standardinstallation finden Sie diesen Ordner unter `C:\ProgramData\Docker\CredentialSpecs`.

So erstellen Sie eine Spezifikationsdatei für die Anmeldeinformationen auf Ihrem Containerhost

1. Installieren der RSAT AD PowerShell-Tools
    - Führen Sie für Windows Server **Install-WindowsFeature RSAT-AD-PowerShell** aus.
    - Für Windows 10, Version 1809 oder höher, führen Sie **WindowsCapability -Online - Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~~~0.0.1.0'** aus.
    - Informationen zu älteren Versionen von Windows 10 finden Sie unter <https://aka.ms/rsat>.
2. Führen Sie das folgende Cmdlet aus, um die neueste Version des [CredentialSpec PowerShell-Moduls](https://aka.ms/credspec) zu installieren:

    ```powershell
    Install-Module CredentialSpec
    ```

    Wenn Sie auf Ihrem Containerhost keinen Internetzugriff haben, führen Sie `Save-Module CredentialSpec` auf einem mit dem Internet verbundenen Computer aus, und kopieren Sie den Modulordner nach `C:\Program Files\WindowsPowerShell\Modules` oder an einen anderen Speicherort in `$env:PSModulePath` auf dem Containerhost.

3. Führen Sie das folgende Cmdlet aus, um die neue Spezifikationsdatei für die Anmeldeinformationen zu erstellen:

    ```powershell
    New-CredentialSpec -AccountName WebApp01
    ```

    Standardmäßig erstellt das Cmdlet eine Spezifikationsdatei für die Anmeldeinformationen unter Verwendung des angegebenen gMSA-Namens als Computerkonto für den Container. Die Datei wird im Verzeichnis „Docker CredentialSpecs“ unter Verwendung der gMSA-Domäne und des Kontonamens für den Dateinamen gespeichert.

    Wenn Sie die Datei in einem anderen Verzeichnis speichern möchten, verwenden Sie den Parameter `-Path`:

    ```powershell
    New-CredentialSpec -AccountName WebApp01 -Path "C:\MyFolder\WebApp01_CredSpec.json"
    ```

    Sie können auch eine Spezifikation der Anmeldeinformationen erstellen, die zusätzliche gMSA-Konten enthält, wenn Sie einen Dienst oder Prozess als sekundäres gMSA im Container ausführen. Verwenden Sie dazu den Parameter `-AdditionalAccounts`:

    ```powershell
    New-CredentialSpec -AccountName WebApp01 -AdditionalAccounts LogAgentSvc, OtherSvc
    ```

    Eine vollständige Liste der unterstützten Parameter erhalten Sie, wenn Sie `Get-Help New-CredentialSpec -Full` ausführen.

4. Mit dem folgenden Cmdlet können Sie eine Liste aller Spezifikationen der Anmeldeinformationen und deren vollständigen Pfad anzeigen:

    ```powershell
    Get-CredentialSpec
    ```

## <a name="next-steps"></a>Nächste Schritte

Nachdem Sie Ihr gMSA-Konto eingerichtet haben, können Sie es für Folgendes verwenden:

- [Konfigurieren von Apps](gmsa-configure-app.md)
- [Ausführen von Containern](gmsa-run-container.md)
- [Orchestrieren von Containern](gmsa-orchestrate-containers.md)

Wenn während des Setups Probleme auftreten, überprüfen Sie unseren [Leitfaden zur Problembehandlung](gmsa-troubleshooting.md) auf mögliche Lösungen.

## <a name="additional-resources"></a>Zusätzliche Ressourcen

- Weitere Informationen zu gMSAs finden Sie unter [Gruppenverwaltete Dienstkonten: Übersicht](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview).
- Eine Videodemonstration finden Sie in unserer [aufgezeichneten Demo](https://youtu.be/cZHPz80I-3s?t=2672) von Ignite 2016.
