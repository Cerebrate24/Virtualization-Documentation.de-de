# <a name="hyper-v-backup-approaches"></a>Hyper-V-Sicherungsansätze
Mit Hyper-V können Sie virtuelle Computer vom Hostbetriebssystem aus sichern, ohne dass im virtuellen Computer benutzerdefinierte Sicherungssoftware ausgeführt werden muss.  Es gibt verschiedene Ansätze, die Entwickler abhängig von Ihren Anforderungen verwenden können.
## <a name="hyper-v-vss-writer"></a>Hyper-V-VSS Writer
Hyper-V implementiert einen VSS Writer für alle Versionen von Windows Server, unter denen Hyper-V unterstützt wird.  Dieser VSS Writer ermöglicht Entwicklern die Verwendung der vorhandenen VSS-Infrastruktur zum Sichern virtueller Computer.  Er ist jedoch für kleine Sicherungsvorgänge konzipiert, bei denen alle virtuellen Computer auf einem Server gleichzeitig gesichert werden.

Um diese Architektur besser zu verstehen, sehen Sie sich die folgende Präsentation an: https://channel9.msdn.com/Events/TechEd/NorthAmerica/2010/VIR322
## <a name="hyper-v-wmi-based-backup"></a>Hyper-V-WMI-basierte Sicherung
Ab Windows Server 2016 unterstützt Hyper-V Sicherung über die Hyper-V-WMI-API.  Bei diesem Ansatz wird weiterhin VSS innerhalb des virtuellen Computers für Sicherungszwecke verwendet, aber VSS wird nicht mehr im Hostbetriebssystem verwendet.  Stattdessen wird eine Kombination aus Referenzpunkten und RCT (Resilient Change Tracking) verwendet, damit Entwickler auf effiziente Weise auf die Informationen zu gesicherten virtuellen Computern zugreifen können.  Dieser Ansatz ist besser skalierbar als die Verwendung von VSS im Host, allerdings ist er nur unter Windows Server 2016 oder höher verfügbar.

Um diese Architektur besser zu verstehen, sehen Sie sich die folgende Präsentation an: https://channel9.msdn.com/Events/TechEd/Europe/2014/CDP-B318 

Ein Beispiel für die Verwendung dieser APIs finden Sie hier: https://www.powershellgallery.com/packages/xHyper-VBackup
## <a name="methods-for-reading-backups-from-wmi-based-backup"></a>Methoden zum Lesen von Sicherungen aus WMI-basierten Sicherungen
Beim Erstellen von Sicherungen virtueller Computer mit Hyper-V-WMI gibt es drei Methoden zum Lesen der tatsächlichen Daten aus der Sicherung.  Jede dieser Methoden hat eindeutige Vor- und Nachteile.
### <a name="wmi-export"></a>WMI-Export
Entwickler können die Sicherungsdaten über die Hyper-V-WMI-Schnittstellen exportieren (wie im Beispiel oben gezeigt).  Die Änderungen werden von Hyper-V in eine virtuelle Festplatte kompiliert, und diese Datei wird an den angeforderten Speicherort kopiert.  Diese Methode ist einfach zu verwenden, funktioniert für alle Szenarien und ist remotefähig.  Die generierte virtuelle Festplatte erstellt jedoch häufig eine große Menge an Daten, die über das Netzwerk übertragen werden.
### <a name="win32-apis"></a>Win32-APIs
Entwickler können die SetVirtualDiskInformation-, GetVirtualDiskInformation- und QueryChangesVirtualDisk-APIs für den Win32-API-Satz der virtuellen Festplatte verwenden, wie hier dokumentiert: https://docs.microsoft.com/windows/desktop/api/_vhd/ Beachten Sie, dass zur Verwendung dieser APIs noch Hyper-V-WMI verwendet werden muss, um Referenzpunkte auf zugehörigen virtuellen Computern zu erstellen.  Diese Win32-APIs ermöglichen dann effizienten Zugriff auf die Daten des gesicherten virtuellen Computers.  Die Win32-APIs weisen einige Einschränkungen auf:
* Auf sie kann nur lokal zugegriffen werden.
* Sie unterstützen das Lesen von Daten aus freigegebenen virtuellen Festplattendateien nicht.
* Sie geben Datenadressen zurück, die relativ zur internen Struktur der virtuellen Festplatte sind.

### <a name="remote-shared-virtual-disk-protocol"></a>RSVD-Protokoll (Remote Shared Virtual Disk)
Wenn ein Entwickler schließlich effizient auf die Sicherungsdateninformationen aus einer freigegebenen virtuellen Festplattendatei zugreifen muss, muss er das RSVD-Protokoll (Remote Shared Virtual Disk) verwenden.  Dokumentation zu diesem Protokoll finden Sie [hier](https://docs.microsoft.com/openspecs/windows_protocols/ms-rsvd/c865c326-47d6-4a91-a62d-0e8f26007d15).
