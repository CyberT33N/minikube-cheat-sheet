# Minikube cheatsheet












# ufw
```shell
sudo ufw allow out on <yourAdapter> from fe80::/64 to any port 22
```







<br><br>
<br><br>
_______________________________
_______________________________
<br><br>
<br><br>

## Guide
- https://minikube.sigs.k8s.io/docs/start/?arch=%2Flinux%2Fx86-64%2Fstable%2Fbinary+download







<br><br>
<br><br>
_______________________________
_______________________________
<br><br>
<br><br>

## Installation
- https://kubernetes.io/de/docs/tasks/tools/install-minikube/#linux
```yaml
# Sie können Minikube unter Linux installieren, indem Sie eine statische Binärdatei herunterladen:
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
  && chmod +x minikube
  
# So fügen Sie die Minikube-Programmdatei auf einfache Weise Ihrem Pfad hinzu:
sudo cp minikube /usr/local/bin && rm minikube
```

<br><br>
<br><br>

## Deinstallation
```shell
minikube delete
```


















<br><br>
<br><br>
_______________________________
_______________________________
<br><br>
<br><br>

## Start
- The context for minikube will be created automatically. If you accedentily deleted it you can just restart minikube to create the minikube context again
- If you did not already have the docker group then create it and then **log out and log in again**
```shell
sudo usermod -aG docker $USER
newgrp docker
```


```shell
minikube start \
   --cpus=8 \
   --memory=18G \
   --driver=docker \
   --insecure-registry='xxxxxxxxxxxx' \
   --addons dashboard \
   --addons metrics-server \
   --addons storage-provisioner

MINIKUBE_IP=$(minikube ip)

echo "Minikube IP: $MINIKUBE_IP"

FROM_IP="${MINIKUBE_IP%.*}.$((${MINIKUBE_IP##*.}+1))"
TO_IP="${MINIKUBE_IP%.*}.$((${MINIKUBE_IP##*.}+10))"

# make sure we are in the correct context
kubectl config use minikube

# We have to congiure the loadbalancer IPs manually because of https://github.com/kubernetes/minikube/issues/8283
# We configure the IPs for metallb loadbalancer in minikubes config.json and then start metallb addon. It will read the config and use the IPs for the metallb secret `config`.
# This way we can avoid interactive command `minikube addons configure metallb`, and the ips are persistent.
cat ~/.minikube/profiles/minikube/config.json | jq ".KubernetesConfig.LoadBalancerStartIP=\"$FROM_IP\"" | jq ".KubernetesConfig.LoadBalancerEndIP=\"$TO_IP\"" > ~/.minikube/profiles/minikube/config.json.tmp && mv ~/.minikube/profiles/minikube/config.json.tmp ~/.minikube/profiles/minikube/config.json 
minikube addons enable metallb

_files=$(find  $_directory  | grep 'values')

# make sure Treafik loadbalancer IP is correct in all configs
for file in $_files
do
    while read -r _line; do
    if [ "$_line" != "" ]; then
      # replace loadbalancer ip's with actual ip's from minikube (traefik should always use the first loadbalancer ip)
      sed -E -i "s/([.]*:\s)'([a-z-]*){0,1}([0-9]{1,3}\.){3}[0-9]{1,3}.nip.io'/\1'\2${FROM_IP}.nip.io'/g" $file
    fi
  done <<< $(cat $file | grep 'loadbalancer-IP')  
done

echo "Minikube Loadbalancer can be accessed with ${FROM_IP}.nip.io"

```
- If you are hanging while the start process at the message container creating then try `minikube logs` to check whats going on. When you have firewall rules then you may have a problem here with docker and your firewall
    - Exiting due to DRV_CP_ENDPOINT: Unable to get control-plane node minikube endpoint: failed to lookup ip for ""
      - If you deny all outgoing traffic via ufw make sure to allow outgoing traffic on your desired network adapter to ipv6 range at port 22 that docker has ssh acces
      ```
      sudo ufw allow out on nordlynx from fe80::/64 to any port 22
      ```

















<br><br>
<br><br>
_______________________________
_______________________________
<br><br>
<br><br>

## Drivers
- https://minikube.sigs.k8s.io/docs/drivers/
```shell
minikube start --driver=docker
```

### Docker (container-based)
- **Schnelle Bereitstellung:** Container starten in Sekunden.
- **Ressourcenschonend:** Weniger Overhead als VMs.
- **Leichtgewichtig:** Geringer Speicher- und CPU-Verbrauch.
- **Portabilität:** Docker-Container sind leicht auf verschiedene Systeme übertragbar.

### KVM2 (VM-based)
- **Isolation:** Bietet stärkere Isolation als Container.
- **Kompatibilität:** Gut geeignet für komplexere Workloads oder wenn Hardware-Virtualisierung benötigt wird.
- **Stabilität:** Reif und gut unterstützt in der Linux-Welt.

### Vergleich Docker vs. KVM2
- **Leistung:** Docker ist leichter und schneller, was es ideal für Entwicklungs- und Testumgebungen macht. KVM2 ist robuster und bietet bessere Isolation.
- **Einfachheit:** Docker erfordert weniger Konfiguration und ist einfacher zu verwalten. KVM2 kann komplexer sein, bietet aber mehr Kontrolle über die VM-Umgebung.
- **Anwendungsfall:** Docker eignet sich besser für Anwendungen, die schnelle Iteration und geringere Ressourcen benötigen. KVM2 ist besser für Anwendungen, die stärkere Isolation und Virtualisierung benötigen.

Für die meisten Entwicklungs- und Testaufgaben wäre Docker die bevorzugte Wahl aufgrund seiner Geschwindigkeit und Einfachheit. KVM2 wäre vorzuziehen, wenn eine stärkere Isolation und Kompatibilität mit verschiedenen Workloads erforderlich ist.













<br><br>
<br><br>
_______________________________
_______________________________
<br><br>
<br><br>

## Addons
```shell
minikube start \
   --addons dashboard \
   --addons metrics-server \
   --addons storage-provisioner
```

```
╰─> minikube addons list
# Minikube Add-ons

| Add-on Name                   | Maintainer                     | Beschreibung |
|-------------------------------|--------------------------------|--------------|
| **ambassador**                | 3rd party (Ambassador)         | API-Gateway und Ingress-Controller für Kubernetes. |
| **auto-pause**                | minikube                       | Pausiert automatisch den Minikube-Cluster, wenn er nicht aktiv verwendet wird, um Ressourcen zu sparen. |
| **cloud-spanner**             | Google                         | Integration mit Google Cloud Spanner für verteilte relationale Datenbanken. |
| **csi-hostpath-driver**       | Kubernetes                     | CSI-Treiber zur Verwendung des Hostpfads für Speicher in Kubernetes. |
| **dashboard**                 | Kubernetes                     | Grafische Benutzeroberfläche zur Verwaltung und Überwachung des Kubernetes-Clusters. |
| **default-storageclass**      | Kubernetes                     | Stellt die Standard-StorageClass für den Cluster bereit. |
| **efk**                       | 3rd party (Elastic)            | Elasticsearch, Fluentd und Kibana für Logging und Monitoring. |
| **freshpod**                  | Google                         | Automatische Neustart von Pods, um sicherzustellen, dass sie auf dem neuesten Stand sind. |
| **gcp-auth**                  | Google                         | Authentifizierungs-Plugin für Google Cloud Platform-Dienste. |
| **gvisor**                    | minikube                       | Laufzeit-Umgebung zur Verbesserung der Sicherheit durch Sandbox-Container. |
| **headlamp**                  | 3rd party (kinvolk.io)         | Moderne Benutzeroberfläche zur Verwaltung von Kubernetes-Clustern. |
| **helm-tiller**               | 3rd party (Helm)               | Server-Komponente von Helm zur Verwaltung von Kubernetes-Paketen. |
| **inaccel**                   | 3rd party (InAccel)            | Unterstützung für FPGA-Beschleunigung in Kubernetes. |
| **ingress**                   | Kubernetes                     | Ermöglicht den Zugriff auf Anwendungen von außerhalb des Kubernetes-Clusters. |
| **ingress-dns**               | minikube                       | Stellt DNS-Unterstützung für Ingress-Ressourcen bereit. |
| **inspektor-gadget**          | 3rd party (inspektor-gadget.io)| Werkzeug zur Überwachung und Fehlersuche in Kubernetes-Clustern. |
| **istio**                     | 3rd party (Istio)              | Service-Mesh für Traffic-Management, Sicherheit und Überwachung. |
| **istio-provisioner**         | 3rd party (Istio)              | Unterstützt die Bereitstellung von Istio in Kubernetes. |
| **kong**                      | 3rd party (Kong HQ)            | API-Gateway und Ingress-Controller für Kubernetes. |
| **kubeflow**                  | 3rd party                      | Plattform zur Vereinfachung des Maschinenlernens auf Kubernetes. |
| **kubevirt**                  | 3rd party (KubeVirt)           | Ermöglicht das Ausführen und Verwalten von virtuellen Maschinen auf Kubernetes. |
| **logviewer**                 | 3rd party (unknown)            | Ermöglicht das Ansehen und Durchsuchen von Pod-Logs über eine Weboberfläche. |
| **metallb**                   | 3rd party (MetalLB)            | Load-Balancer für Kubernetes-Cluster, die keine Cloud-Provider-Integration haben. |
| **metrics-server**            | Kubernetes                     | Aggregiert und bietet Zugriff auf Ressourcenmetriken von Nodes und Pods. |
| **nvidia-device-plugin**      | 3rd party (NVIDIA)             | Unterstützung für NVIDIA GPUs in Kubernetes. |
| **nvidia-driver-installer**   | 3rd party (Nvidia)             | Installiert NVIDIA-Treiber auf Nodes. |
| **nvidia-gpu-device-plugin**  | 3rd party (Nvidia)             | Erweiterte Unterstützung für NVIDIA GPUs in Kubernetes. |
| **olm**                       | 3rd party (Operator Framework) | Operator Lifecycle Manager zur Verwaltung von Kubernetes-Operatoren. |
| **pod-security-policy**       | 3rd party (unknown)            | Erzwingt Sicherheitsrichtlinien für Pods im Cluster. |
| **portainer**                 | 3rd party (Portainer.io)       | Benutzerfreundliche Management-Oberfläche für Docker und Kubernetes. |
| **registry**                  | minikube                       | Lokales Container-Image-Registry. |
| **registry-aliases**          | 3rd party (unknown)            | Verwaltet Aliase für Container-Registries. |
| **registry-creds**            | 3rd party (UPMC Enterprises)   | Verwalten von Anmeldeinformationen für Container-Registries. |
| **storage-provisioner**       | minikube                       | Automatisiert die Bereitstellung von Persistent Volumes. |
| **storage-provisioner-gluster**| 3rd party (Gluster)            | Unterstützung für GlusterFS als Speicher-Backend. |
| **storage-provisioner-rancher**| 3rd party (Rancher)            | Unterstützung für Rancher-Storage-Backend. |
| **volumesnapshots**           | Kubernetes                     | Unterstützung für Volume-Snapshots in Kubernetes. |
| **yakd**                      | 3rd party (marcnuri.com)       | Tool zur Bereitstellung von Kubernetes-Anwendungen. |
```


### Dashboard
- **Funktion:** Bietet eine grafische Benutzeroberfläche für die Verwaltung und Überwachung des Kubernetes-Clusters.
- **Nutzen:** Erleichtert die Visualisierung von Ressourcen, das Erstellen und Verwalten von Deployments, Services und anderen Kubernetes-Objekten.
- **Zugriff:** Nach der Aktivierung kann das Dashboard über einen Webbrowser aufgerufen werden.

### Metrics-server
- **Funktion:** Aggregiert und bietet Zugriff auf Ressourcenmetriken (CPU, Speicher) von Kubernetes Nodes und Pods.
- **Nutzen:** Ermöglicht die Überwachung und Skalierung von Anwendungen basierend auf Ressourcenverbrauchsdaten.
- **Verwendung:** Notwendig für die horizontale Pod-Autoskalierung (HPA) und zur Anzeige von Metriken im Kubernetes Dashboard.

### Storage-provisioner
- **Funktion:** Automatisiert die Bereitstellung von Persistent Volumes (PV) in Kubernetes.
- **Nutzen:** Erleichtert das dynamische Erstellen und Verwalten von Speicherressourcen für Pods, ohne manuelle PV-Konfiguration.
- **Verwendung:** Praktisch für Anwendungen, die persistenten Speicher benötigen, wie Datenbanken oder Stateful Applications.






















<br><br>
<br><br>
_______________________________
_______________________________
<br><br>
<br><br>

# Deployment
```shell
kubectl create deployment hello-minikube --image=kicbase/echo-server:1.0
kubectl expose deployment hello-minikube --type=NodePort --port=8080

# Check k9s or
kubectl get services hello-minikube

# The easiest way to access this service is to let minikube launch a web browser for you:
minikube service hello-minikube

# Alternatively, use kubectl to forward the port:
kubectl port-forward service/hello-minikube 7080:8080
```


<br><br>
<br><br>


## Loadbalancer Deployment
```shell
# To access a LoadBalancer deployment, use the “minikube tunnel” command. Here is an example deployment:
kubectl create deployment balanced --image=kicbase/echo-server:1.0
kubectl expose deployment balanced --type=LoadBalancer --port=8080

# In another window, start the tunnel to create a routable IP for the ‘balanced’ deployment:
minikube tunnel

# To find the routable IP, run this command and examine the EXTERNAL-IP column:
kubectl get services balanced

# Your deployment is now available at <EXTERNAL-IP>:8080
```






<br><br>
<br><br>


## Ingress
- `minikube addons enable ingress`

- The following example creates simple echo-server services and an Ingress object to route to these services.
```shell
kind: Pod
apiVersion: v1
metadata:
  name: foo-app
  labels:
    app: foo
spec:
  containers:
    - name: foo-app
      image: 'kicbase/echo-server:1.0'
---
kind: Service
apiVersion: v1
metadata:
  name: foo-service
spec:
  selector:
    app: foo
  ports:
    - port: 8080
---
kind: Pod
apiVersion: v1
metadata:
  name: bar-app
  labels:
    app: bar
spec:
  containers:
    - name: bar-app
      image: 'kicbase/echo-server:1.0'
---
kind: Service
apiVersion: v1
metadata:
  name: bar-service
spec:
  selector:
    app: bar
  ports:
    - port: 8080
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  rules:
    - http:
        paths:
          - pathType: Prefix
            path: /foo
            backend:
              service:
                name: foo-service
                port:
                  number: 8080
          - pathType: Prefix
            path: /bar
            backend:
              service:
                name: bar-service
                port:
                  number: 8080
---

```

Apply the contents
```shell
kubectl apply -f https://storage.googleapis.com/minikube-site-examples/ingress-example.yaml
```

Wait for ingress address
```shell
kubectl get ingress
NAME              CLASS   HOSTS   ADDRESS          PORTS   AGE
example-ingress   nginx   *       <your_ip_here>   80      5m45s
```

Note for Docker Desktop Users:
To get ingress to work you’ll need to open a new terminal window and run minikube tunnel and in the following step use 127.0.0.1 in place of <ip_from_above>.

Now verify that the ingress works
```shell
$ curl <ip_from_above>/foo
Request served by foo-app
...

$ curl <ip_from_above>/bar
Request served by bar-app
...
```
