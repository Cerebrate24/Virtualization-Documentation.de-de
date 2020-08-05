# [Dokumentation zu Containern unter Windows](index.yml) 

# Übersicht
## [Informationen zu Windows-Containern](about/index.md)
## [-Container im Vergleich zu VMs](about/containers-vs-vm.md)
## [Systemanforderungen](deploy-containers/system-requirements.md)
## [Häufig gestellte Fragen](about/faq.md)

# Erste Schritte
## [Einrichten der Umgebung](quick-start/set-up-environment.md)
## [Ausführen Ihres ersten Containers](quick-start/run-your-first-container.md)
## [Containerisieren einer Beispiel-App](quick-start/building-sample-app.md)

# Lernprogramme
## Erstellen eines Windows-Containers
### [Schreiben einer Dockerfile-Datei](manage-docker/manage-windows-dockerfile.md)
### [Optimieren einer Dockerfile-Datei](manage-docker/optimize-windows-dockerfile.md)
## Ausführung unter Azure Kubernetes Service
### [Erstellen eines Windows-Containerclusters unter AKS](/azure/aks/windows-container-cli)
### [Aktuelle Einschränkungen](/azure/aks/windows-node-limitations)
## Ausführung unter Service Fabric
### [Bereitstellen Ihres ersten Containers](/azure/service-fabric/service-fabric-quickstart-containers)
### [Bereitstellen einer .NET-Anwendung in einem Windows-Container](/azure/service-fabric/service-fabric-host-app-in-a-container)
## Ausführung unter Azure App Service
### [Azure App Service-Schnellstart](/azure/app-service/app-service-web-get-started-windows-container)
### [Migrieren einer ASP.NET-App mit Windows-Containern und Azure App Service](/azure/app-service/app-service-web-tutorial-windows-containers-custom-fonts)
## Linux-Container unter Windows
### [Übersicht](deploy-containers/linux-containers.md)
### [Ausführen Ihres ersten Linux-Containers](quick-start/quick-start-windows-10-linux.md)
## Verwenden von Containern mit dem Windows-Insider-Programm
### [Übersicht](deploy-containers/insider-overview.md)

# Konzepte
## Grundlagen von Windows-Containern
### [Containerbasisimages](manage-containers/container-base-images.md)
### [Isolationsmodi](manage-containers/hyperv-container.md)
### [Versionskompatibilität](deploy-containers/version-compatibility.md)
### [Aktualisieren von Containern](deploy-containers/update-containers.md)
### [Ressourcensteuerung](manage-containers/resource-controls.md)
## Docker
### [Docker-Engine unter Windows](manage-docker/configure-docker-daemon.md)
### [Remoteverwaltung eines Windows Docker-Hosts](management/manage_remotehost.md)
## Containerorchestrierung
### [Übersicht](about/overview-container-orchestrators.md)
### Kubernetes unter Windows
#### [Kubernetes unter Windows](kubernetes/getting-started-kubernetes-windows.md)
#### [Erstellen eines Kubernetes-Masters](kubernetes/creating-a-linux-master.md)
#### [Auswählen einer Netzwerklösung](kubernetes/network-topologies.md)
#### [Windows-Workern beitreten](kubernetes/joining-windows-workers.md)
#### [Linux-Workern beitreten](kubernetes/joining-linux-workers.md)
#### [Bereitstellen von Kubernetes-Ressourcen](kubernetes/deploying-resources.md)
#### [Problembehandlung](kubernetes/common-problems.md)
#### [Kubernetes als Windows-Dienst](kubernetes/kube-windows-services.md)
#### [Kompilieren von Kubernetes-Binärdateien](kubernetes/compiling-kubernetes-binaries.md)
### Service Fabric
#### [Service Fabric und Container](/azure/service-fabric/service-fabric-containers-overview)
#### [Ressourcengovernance](/azure/service-fabric/service-fabric-resource-governance)
### Docker Swarm
#### [Swarm-Modus](manage-containers/swarm-mode.md)
## Arbeitsauslastungen
### Gruppenverwaltete Dienstkonten
#### [Erstellen eines gMSA](manage-containers/manage-serviceaccounts.md)
#### [Konfigurieren Ihrer App für die Verwendung eines gMSA](manage-containers/gmsa-configure-app.md)
#### [Ausführen eines Containers mit einem gMSA](manage-containers/gmsa-run-container.md)
#### [Orchestrieren von Containern mit einem gMSA](manage-containers/gmsa-orchestrate-containers.md)
#### [Problembehandlung von gMSAs](manage-containers/gmsa-troubleshooting.md)
### [Druckerdienste](deploy-containers/print-spooler.md)
## Netzwerk
### [Übersicht](container-networking/architecture.md)
### [Netzwerktopologien und Treiber](container-networking/network-drivers-topologies.md)
### [Netzwerkisolation und Sicherheit](container-networking/network-isolation-security.md)
### [Konfigurieren erweiterter Netzwerkoption](container-networking/advanced.md)
## Speicher
### [Übersicht](manage-containers/container-storage.md)
### [Permanenter Speicher](manage-containers/persistent-storage.md)
## Geräte
### [Hardwaregeräte](deploy-containers/hardware-devices-in-containers.md)
### [GPU-Beschleunigung](deploy-containers/gpu-acceleration.md)

# Verweis
## [Wartungslebenszyklen von Basisimages](deploy-containers/base-image-lifecycle.md)
## [Antivirusoptimierung](https://docs.microsoft.com/windows-hardware/drivers/ifs/anti-virus-optimization-for-windows-containers)
## [Containerplattformtools](deploy-containers/containerd.md)
## [Lizenzbedingungen für Containerbetriebssystem-Image](Images_EULA.md)

# Ressourcen
## [Bekannte Probleme](manage-containers/known-issues.md)
## [Containerbeispiele](samples.md)
## [Problembehandlung](troubleshooting.md)
## [Containerforum](https://social.msdn.microsoft.com/Forums/home?forum=windowscontainers)
## [Videos und Blogs der Community](communitylinks.md)
