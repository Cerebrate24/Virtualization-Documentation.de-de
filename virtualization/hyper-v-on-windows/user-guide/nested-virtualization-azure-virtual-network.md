---
title: Konfigurieren geschachtelter VMs für die direkte Kommunikation mit Ressourcen in einem Azure Virtual Network
description: Geschachtelte Virtualisierung
keywords: Windows 10, Hyper-V, Azure
author: mrajess
ms.date: 12/10/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 1ecb85a6-d938-4c30-a29b-d18bd007ba08
ms.openlocfilehash: b287ea091ad283ddf57727f315c7086865375ce7
ms.sourcegitcommit: e9b3c9dcf7b5c9b9222edbc344764fb038529739
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 06/03/2020
ms.locfileid: "84334100"
---
# <a name="configure-nested-vms-to-communicate-with-resources-in-an-azure-virtual-network"></a>Konfigurieren geschachtelter VMs für die Kommunikation mit Ressourcen in einem Azure Virtual Network

Die ursprüngliche Anleitung zum Bereitstellen und Konfigurieren von geschachtelten virtuellen Computern in Azure erfordert, dass Sie über einen NAT-Switch auf diese VMs zugreifen. Daraus ergeben sich verschiedene Einschränkungen:

1. Geschachtelte VMs können nicht auf lokale Ressourcen oder innerhalb eines Azure Virtual Network zugreifen.
2. Lokale Ressourcen oder Ressourcen in Azure können nur über eine NAT auf die geschachtelten VMs zugreifen. Dies bedeutet, dass mehrere Gäste denselben Port nicht gemeinsam verwenden können.

In diesem Dokument wird eine Bereitstellung untersucht, bei der wir RRAS, benutzerdefinierte Routen sowie ein Subnetz für ausgehende NAT, um Gastinternetzugriff zu ermöglichen, und einen „schwebenden“ Adressraum verwenden, damit sich geschachtelte VMs wie jeder andere virtuelle Computer verhalten und kommunizieren können, der direkt in einem VNET in Azure bereitgestellt wird.

Bevor Sie mit diesem Leitfaden beginnen, sollten folgendermaßen vorgehen:

1. Lesen Sie die [hier bereitgestellten Anleitungen](https://docs.microsoft.com/azure/virtual-machines/windows/nested-virtualization) zur geschachtelter Virtualisierung.
2. Lesen Sie den gesamten Artikel vor der Implementierung.

## <a name="high-level-overview-of-what-were-doing-and-why"></a>Allgemeiner Überblick über unsere Vorgehensweise und die Gründe dafür
* Wir erstellen einen virtuellen Computer mit zwei NICs, der schachtelungsfähig ist. 
* Eine NIC wird verwendet, um unseren geschachtelten VMs den Internetzugang über NAT zu ermöglichen, und die andere NIC wird verwendet, um den Verkehr von unserem internen Switch zu Ressourcen außerhalb des Hypervisors zu leiten. Jede NIC muss sich in einer anderen Routingdomäne (also in einem anderen Subnetz) befinden.
* Das bedeutet, dass wir ein virtuelles Netzwerk mit mindestens drei Subnetzen benötigen. Eines für NAT, eines für LAN-Routing und eines, das nicht verwendet wird, sondern für unsere geschachtelten VMs „reserviert“ ist. Die Namen, die wir für die Subnetze in diesem Dokument verwenden, sind „NAT“, „Hyper-V-LAN“ und „Ghosted“.
* Die Größe dieser Subnetze liegt in Ihrem Ermessen, aber es gibt einige Überlegungen dazu. Die Größe der „Ghosted“-Subnetze bestimmt, wie viele IP-Adressen für Ihre virtuellen Computer vorhanden sind. Außerdem bestimmt die Größe der „NAT“- und „Hyper-V-LAN“-Subnetze, wie viele IP-Adressen für Hypervisoren vorhanden sind. Daher könnten Sie hier technisch gesehen sehr kleine Subnetze erstellen, wenn Sie nur einen oder zwei Hypervisoren planen.
* Hintergrund: Geschachtelte VMs empfangen DHCP selbst dann NICHT aus dem VNET, mit dem der Host verbunden ist, wenn Sie einen internen oder externen Switch konfigurieren. 
  * Dies bedeutet, dass der Hyper-V-Host DHCP bereitstellen muss.
* Der Hyper-V-Host kennt die derzeit zugewiesenen Leases im VNET nicht. Um eine Situation zu vermeiden, in der der Host eine bereits bestehende IP-Adresse zuweist, müssen wir einen Block von IP-Adressen für die Verwendung nur durch den Hyper-V-Host zuordnen. Auf diese Weise können wir ein Szenario mit doppelten IP-Adressen vermeiden.
  * Der von ausgewählte Block von IP-Adressen entspricht einem Subnetz innerhalb desselben VNETs, in dem sich Ihr Hyper-V befindet.
  * Der Grund, warum wir möchten, dass dies einem vorhandenen Subnetz entspricht, ist die Rückabwicklung von BGP-Ankündigungen über eine ExpressRoute. Wenn wir soeben einen IP-Adressbereich für den Hyper-V-Host erstellt haben, müssten wir eine Reihe statischer Routen erstellen, um den lokalen Clients die Kommunikation mit den geschachtelten VMs zu ermöglichen. Dies bedeutet, dass dies keine feste Anforderung ist, da Sie einen IP-Adressbereich für die geschachtelten VMs erstellen und dann alle Routen erstellen können, die zum Weiterleiten von Clients an den Hyper-V-Host für diesen Bereich benötigt werden.
* Wir erstellen einen internen Switch in Hyper-V und weisen dann der neu erstellten Schnittstelle eine IP-Adresse in einem Bereich zu, der für DHCP reserviert wurde. Diese IP-Adresse wird zum Standardgateway für unsere geschachtelten VMs und zum Weiterleiten zwischen dem internen Switch und der NIC des Hosts verwendet, der mit dem VNET verbunden ist.
* Wir installieren die Rolle „Routing und RAS“ auf dem Host, wodurch der Host zu einem Router wird.  Dies ist erforderlich, um die Kommunikation zwischen Ressourcen zuzulassen, die für den Host und die geschachtelten VMs extern sind.
* Wir informieren die anderen Ressourcen, wie sie auf diese geschachtelten VMs zugreifen können. Dies erfordert, dass eine benutzerdefinierte Routingtabelle erstellt wird, die eine statische Route für den IP-Adressbereich enthält, in dem sich die geschachtelten VMs befinden. Diese statische Route verweist auf die IP-Adresse für den Hyper-V-Host.
* Anschließend platzieren Sie diese UDR im Gatewaysubnetz, damit aus der lokalen Umgebung stammende Clients wissen, wie sie die geschachtelten VMs erreichen.
* Außerdem platzieren Sie diese UDR in jedem anderen Subnetz innerhalb von Azure, das eine Verbindung mit den geschachtelten VMs erfordert.
* Für mehrere Hyper-V-Hosts würden Sie zusätzliche „schwebende“ Subnetze erstellen und der UDR eine zusätzliche statische Route hinzufügen.
* Wenn Sie einen Hyper-V-Host außer Betrieb nehmen, löschen Sie unser „schwebendes“ Subnetz und entfernen diese statische Route aus der UDR. Wenn es sich um den letzten Hyper-V-Host handelt, entfernen Sie die UDR vollständig.

## <a name="creating-the-host"></a>Erstellen des Hosts

Ich werde alle Konfigurationswerte, die auf persönlichen Einstellungen basieren (z. B. VM-Name, Ressourcengruppe usw.), vermeiden.

1. Navigieren Sie zu portal.Azure.com.
2. Klicken Sie oben links auf „Ressource erstellen“.
3. Wählen Sie in der Spalte „Beliebt“ die Option „Windows Server 2016.VM“ aus.
4. Wählen Sie auf der Registerkarte „Grundlagen“ unbedingt eine VM-Größe aus, die für geschachtelte Virtualisierung geeignet ist.
5. Navigieren Sie zur Registerkarte „Netzwerk“.
6. Erstellen Sie ein neues virtuelles Netzwerk mit der folgenden Konfiguration.
    * VNET-Adressraum: 10.0.0.0/22
    * Subnetz 1
        * Name: NAT
        * Adressraum: 10.0.0.0/24
    * Subnetz 2
        * Name: Hyper-V-LAN
        * Adressraum: 10.0.1.0/24
    * Subnetz 3
        * Name: Ghosted
        * Adressraum: 10.0.2.0/24
    * Subnetz 4
        * Name: Azure-VMs
        * Adressraum: 10.0.3.0/24
7. Stellen Sie sicher, dass Sie das NAT-Subnetz für die VM ausgewählt haben.
8. Navigieren Sie zu „Überprüfen und erstellen“, und wählen Sie „Erstellen“ aus.

## <a name="create-the-second-network-interface"></a>Erstellen der zweiten Netzwerkschnittstelle
1. Nachdem die Bereitstellung der VM abgeschlossen ist, navigieren Sie im Azure-Portal zu dieser VM.
2. Beenden Sie die VM.
3. Nach dem Beenden navigieren Sie unter „Einstellungen“ zu „Netzwerk“.
4. „Netzwerkschnittstelle anfügen“
5. „Netzwerkschnittstelle erstellen“
6. Geben Sie ihr einen Namen ein (dabei spielt es keine Rolle, wie Sie sie benennen, aber merken Sie sich den Namen).
7. Wählen Sie „Hyper-V-LAN“ als Subnetz aus.
8. Wählen Sie unbedingt dieselbe Ressourcengruppe aus, in der sich Ihr Host befindet.
9. „Erstellen“
10. Dadurch gelangen Sie zurück zum vorherigen Bildschirm. Stellen Sie sicher, dass die neu erstellte Netzwerkschnittstelle ausgewählt ist, und wählen Sie dann „OK“ aus.
11. Wechseln Sie zurück zum Bereich „Übersicht“, und starten Sie die VM erneut, sobald die vorherige Aktion abgeschlossen wurde.
12. Navigieren Sie zur zweiten NIC, die wir soeben erstellt haben. Sie finden sie in der zuvor ausgewählten Ressourcengruppe.
13. Navigieren Sie zu „IP-Konfigurationen“, und schalten Sie „IP-Weiterleitung“ in „Aktiviert“ um. Speichern Sie die Änderung.

## <a name="setting-up-hyper-v"></a>Einrichten von Hyper-V
1. Stellen Sie eine Remoteverbindung mit Ihrem Host her.
2. Öffnen Sie eine PowerShell-Eingabeaufforderung mit erhöhten Rechten.
3. Führen Sie den folgenden Befehl aus: `Install-WindowsFeature -Name Hyper-V -IncludeManagementTools -Restart`.
4. Dadurch wird der Host neu gestartet.
5. Stellen Sie erneut eine Verbindung mit dem Host her, um mit dem Rest des Setups fortzufahren.

## <a name="creating-our-virtual-switch"></a>Erstellen des virtuellen Switch

1. Öffnen Sie PowerShell im Verwaltungsmodus.
2. Erstellen Sie einen internen Switch: `New-VMSwitch -Name "NestedSwitch" -SwitchType Internal`.
3. Weisen Sie der neu erstellten Schnittstelle eine IP-Adresse zu: `New-NetIPAddress –IPAddress 10.0.2.1 -PrefixLength 24 -InterfaceAlias "vEthernet (NestedSwitch)"`.

## <a name="install-and-configure-dhcp"></a>Installieren und Konfigurieren von DHCP

*Viele Benutzer übersehen diese Komponente, wenn Sie zum ersten Mal versuchen, geschachtelte Virtualisierung zu verwenden. Anders als in lokalen Umgebungen, in denen Ihre Gast-VMs DHCP von dem Netzwerk empfangen, in dem sich der Host befindet, muss für geschachtelte VMs in Azure DHCP über den Host bereitgestellt werden, auf dem sie ausgeführt werden. Alternativ müssen Sie jeder geschachtelten VM statisch eine IP-Adresse zuweisen (nicht skalierbar).*

1. Installieren Sie die DHCP-Rolle: `Install-WindowsFeature DHCP -IncludeManagementTools`.
2. Erstellen Sie den DHCP-Bereich: `Add-DhcpServerV4Scope -Name "Nested VMs" -StartRange 10.0.2.2 -EndRange 10.0.2.254 -SubnetMask 255.255.255.0`.
3. Konfigurieren Sie die DNS- und Standardgatewayoptionen für den Bereich: `Set-DhcpServerV4OptionValue -DnsServer 168.63.129.16 -Router 10.0.2.1`.
    * Stellen Sie sicher, dass Sie einen gültigen DNS-Server eingeben, wenn die Namensauflösung funktionieren soll. In diesem Fall verwende ich [rekursiven DNS von Azure](https://docs.microsoft.com/azure/virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances).

## <a name="installing-remote-access"></a>Installieren von Remotezugriff

1. Öffnen Sie Server-Manager, und wählen Sie dann „Rollen und Features hinzufügen“ aus.
2. Klicken Sie auf „Weiter“, bis Sie zum Abschnitt „Serverrollen“ gelangen.
3. Aktivieren Sie „Remotezugriff“, und klicken Sie auf „Weiter“, bis Sie zu „Rollendienste“ gelangen.
4. Aktivieren Sie „Routing“, wählen Sie „Features hinzufügen“ aus, und klicken Sie dann auf „Weiter“ und auf „Installieren“. Beenden Sie den Assistenten, und warten Sie, bis die Installation abgeschlossen wurde.

## <a name="configuring-remote-access"></a>Konfigurieren von Remotezugriff

1. Öffnen Sie Server-Manager, und wählen Sie „Extras“ und dann „Routing und RAS“ aus.
2. Auf der linken Seite des Verwaltungsbereichs „Routing-und RAS“ wird ein Symbol neben dem Namen Ihres Servers angezeigt. Klicken Sie mit der rechten Maustaste darauf, und wählen Sie „Routing und RAS konfigurieren und aktivieren“ aus.
3. Klicken Sie im Assistenten auf „Weiter“, aktivieren Sie „Benutzerdefinierte Konfiguration“, und wählen Sie dann „Weiter“ aus.
4. Aktivieren Sie „NAT“ und „LAN-Routing“, und wählen Sie dann „Weiter“ und „Fertig stellen“ aus. Wenn Sie aufgefordert werden, den Dienst zu starten, starten Sie ihn.
5. Navigieren Sie nun zum Knoten „IPv4“, und erweitern Sie ihn so, dass der Knoten „NAT“ verfügbar wird.
6. Klicken Sie mit der rechten Maustaste auf „NAT“, und wählen Sie „Neue Schnittstelle“ und „Ethernet“ aus. Dies sollte Ihre erste NIC mit der IP-Adresse „10.0.0.4“ sein. Wählen Sie „Öffentliche Schnittstelle“ aus, stellen Sie die Internetverbindung her, und aktivieren Sie NAT für diese Schnittstelle. 
7. Nun müssen wir einige statische Routen erstellen, um LAN-Datenverkehr aus der zweiten NIC zu entfernen. Zu diesem Zweck navigieren Sie zum Knoten „Statische Routen“ unter „IPv4“.
8. Anschließend erstellen wir die folgenden Routen.
    * Route 1
        * Schnittstelle: Ethernet
        * Ziel: 10.0.0.0
        * Netzwerkmaske: 255.255.255.0
        * Gateway: 10.0.0.1
        * Metrik: 256
        * Hinweis: Wir legen dies hier fest, damit die primäre NIC auf an sie gerichteten Datenverkehr reagieren und aus der eigenen Schnittstelle entfernen kann. Wenn dies hier nicht festgelegt würde, würde die folgende Route dazu führen, dass für NIC 1 gedachter Datenverkehr aus NIC 2 gesendet wird. Dadurch würde eine asymmetrische Route erstellt. 10.0.0.1 ist die IP-Adresse, die Azure dem NAT-Subnetz zuweist. In Azure verwendet die erste verfügbare IP-Adresse in einem Bereich als Standardgateway. Wenn Sie also 192.168.0.0/24 für das NAT-Subnetz verwendet haben, wäre das Gateway 192.168.0.1. Beim Routing gewinnt die spezifischere Route, was bedeutet, dass diese Route Vorrang vor der nachfolgenden Route besitzt.

    * Route 2
        * Schnittstelle: Ethernet 2
        * Ziel: 10.0.0.0
        * Netzwerkmaske: 255.255.252.0
        * Gateway: 10.0.1.1
        * Metrik: 256
        * Hinweis: Dies ist eine Abfangroute für den gesamten Datenverkehr für unser Azure-VNET. Dadurch wird Datenverkehr aus der zweiten NIC entfernt. Sie müssen zusätzliche Routen für andere Bereiche hinzufügen, auf die die geschachtelten VMs zugreifen sollen. Wenn Sie also das lokale Netzwerk 172.16.0.0/22 verwenden, benötigen Sie eine andere Route zum Senden dieses Datenverkehrs aus der zweiten NIC des Hypervisors.

## <a name="creating-a-route-table-within-azure"></a>Erstellen einer Routingtabelle in Azure

Ausführliche Informationen zum Erstellen und Verwalten von Routen in Azure finden Sie in [diesem Artikel](https://docs.microsoft.com/azure/virtual-network/tutorial-create-route-table-portal).

1. Navigieren Sie zu https://portal.azure.com.
2. Wählen Sie in der oberen linken Ecke „Ressource erstellen“ aus.
3. Geben Sie im Suchfeld „Routingtabelle“ ein, und drücken Sie die EINGABETASTE.
4. Das oberste Ergebnis ist die Routingtabelle. Wählen Sie diese aus, und wählen Sie dann „Erstellen“ aus.
5. Benennen Sie die Routingtabelle. In meinem Fall habe ich den Namen „Routes-for-nested-VMs“ verwendet.
6. Stellen Sie sicher, dass Sie das gleiche Abonnement auswählen, in dem sich auch Ihre Hyper-V-Hosts befinden.
7. Erstellen Sie entweder eine neue Ressourcengruppe, oder wählen Sie eine vorhandene aus, und stellen Sie sicher, dass die Region, in der Sie die Routingtabelle erstellen, dieselbe Region ist, in der sich der Hyper-V-Host befindet.
8. Wählen Sie „Erstellen“ aus.

## <a name="configuring-the-route-table"></a>Konfigurieren der Routingtabelle

1. Navigieren Sie zu der Routingtabelle, die wir soeben erstellt haben. Sie können dazu in der Suchleiste oben im Portal nach dem Namen der Routingtabelle suchen.
2. Nachdem Sie die Routingtabelle ausgewählt haben, navigieren Sie auf dem Blatt zu „Routen“.
3. Wählen Sie „Hinzufügen“ aus.
4. Benennen Sie Ihre Route. Ich habe mich für „Nested-VMs“ entschieden.
5. Geben Sie als Adresspräfix den IP-Adressbereich für das „schwebende“ Subnetz ein. In diesem Fall wäre dies 10.0.2.0/24.
6. Wählen Sie als „Typ des nächsten Hops“ die Option „Virtuelles Gerät“ aus, und geben Sie dann die IP-Adresse für die zweite NIC der Hyper-V-Hosts (10.0.1.4) ein, und wählen Sie anschließend „OK“ aus.
7. Wählen Sie nun auf dem Blatt „Subnetze“ aus. Diese Option befindet sich direkt unterhalb von „Routen“.
8. Wählen Sie „Zuordnen“ aus, und wählen Sie dann das VNET „Nested-Fun“ aus. Wählen Sie anschließend das Subnetz „Azure-VMs“ aus, und klicken Sie auf „OK“.
9. Führen Sie den gleichen Vorgang für das Subnetz aus, in dem sich der Hyper-V-Host befindet, sowie für alle anderen Subnetze, die auf die geschachtelten VMs zugreifen müssen. Wenn verbunden 

## <a name="end-state-configuration-reference"></a>Referenz zur Endzustandskonfiguration

Die Umgebung in diesem Leitfaden weist die folgenden Konfigurationen auf. Dieser Abschnitt soll als Referenz verwendet werden.

1. Azure Virtual Network-Informationen.
    * VNET-Konfiguration auf hoher Ebene.
        * Name: Nested-Fun
        * Adressraum: 10.0.0.0/22
        * Hinweis: Diese Umgebung besteht aus vier Subnetzen. Außerdem sind diese Bereiche nicht in Stein gemeißelt. Sie können Ihre Umgebung nach Ihren Wünschen gestalten. 

    * Konfiguration des ersten Subnetzes auf hoher Ebene.
        * Name: NAT
        * Adressraum: 10.0.0.0/24
        * Hinweis: Hier befindet sich die primäre NIC des Hyper-V-Hosts. Wird verwendet, um die ausgehende NAT für die geschachtelten VMs zu verarbeiten. Dies ist das Gateway zum Internet für Ihre geschachtelten VMs.

    * Konfiguration des zweiten Subnetzes auf hoher Ebene.
        * Name: Hyper-V-LAN
        * Adressraum: 10.0.1.0/24
        * Hinweis:  Der Hyper-V-Host verfügt über eine zweite NIC, die zum Verarbeiten des Routings zwischen den geschachtelten VMs und Nicht-Internetressourcen außerhalb des Hyper-V-Hosts verwendet wird.

    * Konfiguration des dritten Subnetzes auf hoher Ebene.
        * Name: Ghosted
        * Adressraum: 10.0.2.0/24
        * Hinweis:  Dabei handelt es sich um ein „schwebendes“ Subnetz. Der Adressraum wird von unseren geschachtelten VMs genutzt und ist vorhanden, um Routenankündigungen zurück an die lokale Umgebung zu verarbeiten. In diesem Subnetz werden tatsächlich keine VMs bereitgestellt.

    * Konfiguration des vierten Subnetzes auf hoher Ebene.
        * Name: Azure-VMs
        * Adressraum: 10.0.3.0/24
        * Hinweis: Ein Subnetz mit Azure-VMs.

1. Der Hyper-V-Host verfügt über die folgenden NIC-Konfigurationen.
    * Primäre NIC 
        * IP-Adresse: 10.0.0.4
        * Subnetzmaske: 255.255.255.0
        * Standardgateway: 10.0.0.1
        * DNS: Für DHCP konfiguriert
        * IP-Weiterleitung aktiviert: Nein

    * Sekundäre NIC
        * IP-Adresse: 10.0.1.4
        * Subnetzmaske: 255.255.255.0
        * Standardgateway: Leer
        * DNS: Für DHCP konfiguriert
        * IP-Weiterleitung aktiviert: Ja

    * Von Hyper-V erstellte NIC für internen virtuellen Switch
        * IP-Adresse: 10.0.2.1
        * Subnetzmaske: 255.255.255.0
        * Standardgateway: Leer

3. Unsere Routingtabelle verfügt über eine einzelne Regel.
    * Regel 1
        * Name: Geschachtelte VMs
        * Ziel: 10.0.2.0/24
        * Nächster Hop: Virtuelles Gerät: 10.0.1.4

## <a name="conclusion"></a>Schlussbemerkung

Sie sollten jetzt in der Lage sein, einen virtuellen Computer (sogar eine 32-Bit-VM!) auf Ihrem Hyper-V-Host bereitzustellen und den Zugriff darauf aus einer lokalen Umgebung und aus Azure zu ermöglichen.
