---
title: Kompilieren von Kubernetes-Binärdateien
author: gkudra-msft
ms.author: gekudray
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Kompilieren und übergreifendes Kompilieren von Kubernetes-Binärdateien aus der Quelle.
keywords: kubernetes, 1,12, Linux, kompilieren
ms.openlocfilehash: 40bf7e65a8910cdab095abb269aa0a92508189cd
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909870"
---
# <a name="compiling-kubernetes-binaries"></a>Kompilieren von Kubernetes-Binärdateien #
Das Kompilieren von Kubernetes erfordert eine funktionierende Go-Umgebung. Diese Seite erläutert mehrere Möglichkeiten zum Kompilieren von Linux-Binärdateien und zum übergreifenden Kompilieren von Windows-Binärdateien.
> [!NOTE] 
> Diese Seite ist vollständig freiwillig und nur für interessierte Kubernetes Entwickler, die mit dem neuesten & größten Quellcode experimentieren möchten.

> [!tip]
> Um Benachrichtigungen zu den neuesten Entwicklungen zu erhalten, können Sie [@kubernetes-announce](https://groups.google.com/forum/#!forum/kubernetes-announce)abonnieren.

## <a name="installing-go"></a>Installieren von Go ##
Der Einfachheit halber wird Go in einem temporären, benutzerdefinierten Speicherort installiert:

```bash
cd ~
wget https://redirector.gvt1.com/edgedl/go/go1.11.1.linux-amd64.tar.gz -O go1.11.1.tar.gz
tar -vxzf go1.11.1.tar.gz
mkdir gopath
export GOROOT="$HOME/go"
export GOPATH="$HOME/gopath"
export PATH="$GOROOT/bin:$PATH"
```

> [!Note]  
> Dort werden die Umgebungsvariablen für Ihre Sitzung festgelegt. Fügen Sie die `export`e zu Ihrem `~/.profile` als permanente Einstellung hinzu.

Führen Sie `go env` aus, um sicherzustellen, dass die Pfade ordnungsgemäß festgelegt wurden. Es gibt mehrere Optionen zum Erstellen von Kubernetes-Binärdateien:

  - Erstellen Sie diese [lokal](#build-locally).
  - Generieren Sie die Binärdateien mit [Vagrant](#build-with-vagrant).
  - Nutzen Sie die [Standard-Buildskripts in Containern](https://github.com/kubernetes/kubernetes/tree/master/build#key-scripts) im Kubernetes-Projekt. Führen Sie dazu die Schritte für die [lokale Erstellung](#build-locally) bis zu den Schritten `make` aus, und verwenden Sie anschließend die verknüpften Anweisungen.

Verwenden Sie zum Kopieren der Windows-Binärdateien in den entsprechenden Knoten ein visuelles Tool wie [WinSCP](https://winscp.net/eng/download.php) oder ein Befehlszeilentool wie [pscp](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html), um diese in das Verzeichnis `C:\k` zu übertragen.


## <a name="building-locally"></a>Lokales aufbauen ##
> [!Tip]  
> Vermeiden Sie den Fehler "Zugriff verweigert", indem Sie das Linux `kubelet` zuerst und entsprechend den Anweisungen unter [`acs-engine`](https://github.com/Azure/acs-engine/blob/master/scripts/build-windows-k8s.sh#L176) erstellen:
>  
> _Aufgrund eines Fehlers im Windows-Buildsystem Kubernetes muss zunächst eine Linux-Binärdatei erstellt werden, um `_output/bin/deepcopy-gen`zu generieren. Durch das Erstellen auf Windows-e/a wird eine leere `deepcopy-gen`generiert._

Rufen Sie zunächst das Kubernetes-Repository ab:

```bash
KUBEREPO="k8s.io/kubernetes"
go get -d $KUBEREPO
# Note: the above command may spit out a message about 
#       "no Go files in...", but it can be safely ignored!
cd $GOPATH/src/$KUBEREPO
```

Wählen Sie die Verzweigung, aus der es erstellt werden soll und erstellen Sie die Linux `kubelet`-Binärdatei. Dies ist erforderlich, um die oben aufgeführten Windows-Buildfehler zu vermeiden. Hier wird `v1.12.2` verwendet. Nach dem `git checkout` können Sie ausstehende PRs, Patches oder andere Änderungen für die benutzerdefinierten Binärdateien vornehmen.

```bash
git checkout tags/v1.12.2
make clean && make WHAT=cmd/kubelet
```

Erstellen Sie abschließend die erforderlichen Windows-Clientbinärdateien (der letzte Schritt variiert je nach Ort, von dem die Windows-Binärdateien an einem späteren Zeitpunkt abgerufen werden sollen):

```bash
KUBE_BUILD_PLATFORMS=windows/amd64 make WHAT=cmd/kubectl
KUBE_BUILD_PLATFORMS=windows/amd64 make WHAT=cmd/kubelet
KUBE_BUILD_PLATFORMS=windows/amd64 make WHAT=cmd/kube-proxy
cp _output/local/bin/windows/amd64/kube*.exe ~/kube-win/
```

Die Schritte zum Erstellen von Linux-Binärdateien sind identisch. Lassen Sie den Präfix `KUBE_BUILD_PLATFORMS=windows/amd64` für die Befehle einfach aus. Das Ausgabeverzeichnis ist stattdessen `_output/.../linux/amd64`.


## <a name="build-with-vagrant"></a>Erstellen mit Vagrant ##
Der Vagrant-Setup ist [hier](https://github.com/Microsoft/SDN/tree/master/Kubernetes/linux/vagrant) verfügbar. Verwenden Sie ihn, um eine Vagrant-VM vorzubereiten, und führen Sie anschließend folgende Befehle darin aus:

```bash
DIST_DIR="${HOME}/kube/"
SRC_DIR="${HOME}/src/k8s-main/"
mkdir ${DIST_DIR}
mkdir -p "${SRC_DIR}"

git clone https://github.com/kubernetes/kubernetes.git ${SRC_DIR}

cd ${SRC_DIR}
git checkout tags/v1.12.2
KUBE_BUILD_PLATFORMS=linux/amd64   build/run.sh make WHAT=cmd/kubelet
KUBE_BUILD_PLATFORMS=windows/amd64 build/run.sh make WHAT=cmd/kubelet 
KUBE_BUILD_PLATFORMS=windows/amd64 build/run.sh make WHAT=cmd/kube-proxy 
cp _output/dockerized/bin/windows/amd64/kube*.exe ${DIST_DIR}

ls ${DIST_DIR}
```

