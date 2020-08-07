---
title: Beitreten zu Linux-Knoten
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: how-to
description: Bereitstellen von Kubernetes-Ressourcen auf einem Kubernetes-Cluster mit gemischtem Betriebssystem.
keywords: kubernetes, 1,14, Windows, Getting Started
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: a342a03153564b2f76af45a9b792b58ae8afc18c
ms.sourcegitcommit: 186ebcd006eeafb2b51a19787d59914332aad361
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 08/07/2020
ms.locfileid: "87985334"
---
# <a name="deploying-kubernetes-resources"></a>Bereitstellen von Kubernetes Ressourcen #
Angenommen, Sie verfügen über einen Kubernetes-Cluster, der aus mindestens einem Master-und 1-Worker besteht, können Sie Kubernetes-Ressourcen bereitstellen.
> [!TIP]
> Seien Sie neugierig, welche Kubernetes-Ressourcen heute unter Windows unterstützt werden? Weitere Informationen finden Sie unter [offiziell unterstützte Features](https://kubernetes.io/docs/setup/production-environment/windows/intro-windows-in-kubernetes/#supported-functionality-and-limitations) und [Kubernetes in der Roadmap für Windows](https://github.com/orgs/kubernetes/projects/8) .


## <a name="running-a-sample-service"></a>Ausführen eines Beispiel Dienst ##
Sie werden einen sehr einfachen [PowerShell-basierten Webdienst](https://github.com/Microsoft/SDN/blob/master/Kubernetes/WebServer.yaml) bereitstellen, um sicherzustellen, dass der Cluster erfolgreich hinzugefügt und unser Netzwerk ordnungsgemäß konfiguriert wurde.

Bevor Sie dies tun, ist es immer eine gute Idee, sicherzustellen, dass alle unsere Knoten fehlerfrei sind.
```bash
kubectl get nodes
```

Wenn alles gut aussieht, können Sie den folgenden Dienst herunterladen und ausführen:
> [!Important]
> Vergewissern Sie sich vor `kubectl apply` , dass Sie das `microsoft/windowsservercore` Image in der Beispieldatei auf ein Container Image überprüfen bzw. ändern, [das von Ihren Knoten durch](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/version-compatibility#choosing-container-os-versions)laufen werden kann.

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/l2bridge/manifests/simpleweb.yml -O win-webserver.yaml
kubectl apply -f win-webserver.yaml
watch kubectl get pods -o wide
```

Dadurch wird eine Bereitstellung und ein Dienst erstellt. Mit dem Befehl Letzte Überwachung werden die Pods unbegrenzt abgefragt, um Ihren Status zu verfolgen. Drücken `Ctrl+C` Sie einfach, um den Befehl zu beenden, wenn Sie das `watch` beobachten.

Wenn alles erfolgreich verlaufen ist:

  - siehe 2 Container pro Pod unter- `docker ps` Befehl auf dem Windows-Knoten.
  - werden 2 Pods unter einem `kubectl get pods`-Befehl vom Linus-Master angezeigt
  - `curl`auf den *Pod* -IPS an Port 80 des Linux-Masters erhält eine Webserver Antwort. Dies veranschaulicht die richtige Knoten-zu-Pod-Kommunikation über das Netzwerk.
  - der Ping *zwischen Pods* (einschließlich zwischen Hosts, wenn Sie mehrere Windows-Knoten haben) über `docker exec`. Dies veranschaulicht die ordnungsgemäße Pod-zu-Pod-Kommunikation
  - `curl`die *IP-Adresse des virtuellen Dienstanbieter* (unter `kubectl get services` ) vom Linux-Master und von einzelnen Pods. Dies zeigt die korrekte Kommunikation zwischen Diensten.
  - `curl`der *Dienst Name* mit dem [DNS-Suffix](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#services)Kubernetes, der die richtige Dienst Ermittlung demonstriert.
  - `curl`der *nodeport* des Linux-Masters oder der Computer außerhalb des Clusters. Dadurch wird die eingehende Konnektivität veranschaulicht.
  - `curl`externe IPS innerhalb des Pod Dadurch wird die ausgehende Konnektivität veranschaulicht.

> [!Note]
> Windows- *Container Hosts* sind **nicht** in der Lage, auf die Dienst-IP von den für Sie geplanten Diensten zuzugreifen. Dies ist eine [bekannte Platt Form Einschränkung](./common-problems.md#my-windows-node-cannot-access-my-services-using-the-service-ip) , die in zukünftigen Versionen von Windows Server verbessert wird. Windows- *Pods* **sind** jedoch in der Lage, auf die Dienst-IP zuzugreifen.

## <a name="next-steps"></a>Nächste Schritte ##
In diesem Abschnitt wird beschrieben, wie Sie Kubernetes-Ressourcen auf Windows-Knoten planen. Dies schließt die Anleitung ab. Wenn Probleme aufgetreten sind, lesen Sie den Abschnitt zur Problembehandlung:

> [!div class="nextstepaction"]
> [Problembehandlung](./common-problems.md)

Andernfalls sind Sie möglicherweise auch daran interessiert, Kubernetes-Komponenten als Windows-Dienste zu ausführen:
> [!div class="nextstepaction"]
> [Windows-Dienste](./kube-windows-services.md)
