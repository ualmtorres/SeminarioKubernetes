# Introducción a Kubernetes

## Conceptos básicos

* Kubernetes es un orquestador de contenedores (p.e. Docker, Cri-O, Moby project) que elimina gran parte del trabajo relacionado con la construcción y escalabilidad de un aplicaciones que se ejecutan en contenedores.
* También proporciona almacenamiento para ejecutar aplicaciones
* Comprobación de estado y autoregeneración de aplicaciones
* Como alternativas a Kubernetes está Docker Swarm, Marathon, ...

## Instalación (1 manager y 2 nodos)

### Preparación del entorno

Hacer esto en todos los nodos:

* Desactivar la swap (cambios permanentes en `/etc/fstab`)

```
swapoff -a
```

* Resolución DNS (para pruebas basta con `/etc/hosts`)

```
192.168.1.61 manager
192.168.1.62 manager2
192.168.1.63 manager3
192.168.1.64 node1
192.168.1.65 node2
```

* Desactivar el bridge en iptables 6

```
# cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

# sysctl --system

```
* Desactivar SELinux (cambios permanentes en `/etc/selinux/config`)

```
# setenforce 0
# sestatus
```
* Instalar Docker

### Instalación de Kubernetes

* Activar Docker en el inicio

```
systemctl enable docker && systemctl start docker
```

* Crear el fichero que apunta al repo de Kubernetes

```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```
* Instalar los paquetes de Kubernetes y configurar kubelet en el arranque 

```
yum install -y kubelet kubeadm kubectl
systemctl enable kubelet && systemctl start kubelet
```

#### Nodo Master

* Inicializar el cluster (devolverá un token para usar en los nodos para unirse al cluster)

```
# kubeadm init

...

kubeadm join 192.168.1.142:6443 --token 0gx03i.bkjfviauhce6q3ns --discovery-token-ca-cert-hash sha256:bed83e5c33b66416d8d6081b7416d8f7d25777238bcfe9efd1fa88c2ddb79974
```
* Exportar la variable `KUBECONFIG`

```
# export KUBECONFIG=/etc/kubernetes/admin.conf
```

* Crear la red (descargaremos contenedores paa virtualizar la red)

```
# kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

* Comprobar estado con 

```
kubectl get pods --all-namespaces
```

NOTA: Por defecto el nodo Master no ejecuta pods, pero si por algún casual se quieren ejecutar en él ejecutar: `kubectl taint nodes --all node-role.kubernetes.io/master-`

#### Nodos

* Unirse al cluster (Es necesario haber activado Docker y Kubelet)

```
# kubeadm join 192.168.1.142:6443 --token 0gx03i.bkjfviauhce6q3ns --discovery-token-ca-cert-hash sha256:bed83e5c33b66416d8d6081b7416d8f7d25777238bcfe9efd1fa88c2ddb79974
```

* Chequeo (En manager)

```
kubectl get nodes
```

## Creación de un contenedor en Kubernetes

* Crear un Kubernetes Deployment Object, que es un fichero donde puedes describir un despliegue. El formato es YAML 
* Archivo `nginx-deployment.yaml`

```
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # tells deployment to run 2 pods matching the template
  template: # create pods using pod definition in this template
    metadata:
      # unlike pod-nginx.yaml, the name is not included in the meta data as a unique name is
      # generated from the deployment name
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

* Comandos de interés

```
kubectl apply -f nginx-deployment.yaml

kubectl describe deployment nginx-deployment

kubectl get pods -l app=nginx

kubectl describe pod nginx2-deployment-647ff4fb7f-f8q7k

kubectl delete deployment nginx-deployment
```

## Examen

1. Diferencias entre master y node

Master no puede ejecutar contenedores.
No hay diferencias.
X Master gestiona todo el cluster.
Ninguna es correcta

2. ¿Qué es un contenedor?

Una aplicación.
X Un entorno ligero en tiempo de ejecución.
Una máquina virtual.
Un espacio en memoria.

3. ¿Qué es un KDO?

Un fichero de configuración de Kubernetes.
Un componente de Kubernetes.
X Un fichero descriptivo de Pods.
Un agradecimiento en metodología Agile.

4. Selecciona las principales ventajas de Kubernetes:

No tiene ventajas.
X Orquestación en multiples hosts, optimizar recursos, escalabilidad...
Escalabilidad
Facil uso y aprendizaje.

5. ¿Qué diferencias hay entre Kubernetes y Docker?

No hay diferencias.
Kubernetes es un motor de contenedores y Docker un orquestador.
X Docker es un motor de contenedores y Kubernetes un orquestador.
Kubernetes es un motor de contenedores más ligero que Docker.

6. ¿Es kubernetes la única alternativa en el mercado?

No, hay otras OpenSource.
INCORRECTA No.
Si.
No, pero es la única OpenSource

7. ¿Dónde podemos desplegar Kubernetes?

VM's.
Baremetal
X Baremetal, VM's, Clouds, ...
Containers.

8. ¿Qué características tiene un contenedor?

Ligereza.
Autosuficiencia.
Portabilidad
X Las 3 anteriores son correctas.

9. ¿Qué alternativas hay a Docker dentro de Kubernetes?

Cri-o.
Moby
X Las tres anteriores.
RKT.

10. ¿Qué es Kubectl?

X El comando para interactuar con Kubernetes.
Un componente arquitectónico de Kubernetes.
El fichero de control de Kubernetes.
El demonio de Kubernetes.