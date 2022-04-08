# Documentación de Docker, Kubernetes y OpenShift

## 1. Introducción

### 1.1. Repositorios de imágenes:

Imágenes docker:
* https://hub.docker.com/

Imágenes docker preparadas para OpenShift:
* https://quay.io/
* https://catalog.redhat.com/software/containers/search
* https://www.softwarecollections.org/en/

Especificación de Open Container Intitiative:
* https://opencontainers.org

### 1.2. Resumen organización empresas:

* Un entorno (TST, PRE o PRO) contiene n datacenters.
* Un datacenter contiene n clusters (normalmente hay un único cluster por datacenter).
* Una aplicación contiene n componentes.
* Una aplicación expone n servicios.

### 1.3. Resumen Kubernetes / OpenShift:

* Acerca de clusters y nodos:
  * Un cluster contiene n nodos.
  * Un nodo contiene n pods (configurados con los deployments).
  * Un pod contiene n contenedores Docker (aunque casi siempre contiene sólo un contenedor Docker).
  * Un contenedor Docker es una imagen docker en ejecución.

* Acerca de projects:
  * Los distintos componentes de OpenShift están agrupados en projects.
  * Un project de OpenShift debe tener asociado "un y sólo un" namespace de Kubernetes.

* Acerca de deployments:
  * Un deployment es una configuración para levantar réplicas de pods en el cluster.
  * Un deployment contiene n ReplicaSets, que a su vez contienen m réplicas de pods.
  * Un DeploymentConfig es un componente de OpenShift equivalente a los deployment de Kubernetes, pero con más características (como, por ejemplo, versionado).
  * Un DeploymentConfig contiene n ReplicationControllers, que a su vez contienen m réplicas de pods.
  * No tiene sentido usar Deployments o DeploymentConfigs con más de una réplica para BBDD, salvo que se use un producto como MySQL Galera.

* Acerca de services y routes:
  * Un service ofrece una IP fija, nombre fijo y puerto fijo al que conectarse.
  * Un service tiene asociado un deployment y hace de intermediario con los pods replicados de un deployment.
  * Una route es la alternativa de OpenShift a los ingress de Kubernetes.
  * Una route es un mapeo de una tupla [URL + puerto] a un service.
  * La creación de las routes es opcional. OpenShift fuerza a que (cuando sea necesario crearlas) siempre haya que crearlas manualmente, ya que sólo deberían crearse routes para los frontales. Por ejemplo, las BBDD no deberían tener routes.

* Acerca de la construcción de imágenes:
  * Kubernetes no puede construir imágenes Docker. Para construir imágenes Docker se puede usar Docker o OpenShift.
  * Al partir de una imagen Docker, se generan nuevos componentes como ImageStreams, BuildConfigs y Builds.
  * Un ImageStream puede contener n ImageStreamTags.

* Acerca de otros elementos:
  * Un ConfigMap es como un fichero de propiedades.
  * Un Secret es como un ConfigMap, pero con los datos codificados en Base64 (ojo, no están encriptados).

### 1.4. Componentes obligatorios al crear una aplicación

En la siguiente imagen se muestran los componentes obligatorios al crear una aplicación. Como se comentó anteriormente, la creación de la route es opcional y manual.

<img src="https://github.com/Aliuken/Documentacion-docker-kubernetes-y-openshift/blob/main/Componentes_a_crear.png" width="800">

#### 1.4.1. Pods

* **Descripción general**

  Repaso:
  * Un pod contiene n contenedores Docker (aunque casi siempre contiene sólo un contenedor Docker).
  * Un contenedor Docker es una imagen docker en ejecución.

  NOTA: Los pods deben tener un puerto por encima del 1024.

  <img src="https://github.com/Aliuken/Documentacion-docker-kubernetes-y-openshift/blob/main/Pods.png" width="800">

* **Ciclo de vida**

  Los componentes de OpenShift (buildconfig, build, deployment, replicaset, pod, service, route, imagestream, imagestreamtag, configmap, secret...) pueden ser creados de **forma imperativa** (informando sus campos en la propia instrucción de creación) o de **forma declarativa** (mediante el uso de YAML).

  Al crear un componente de forma declarativa se indica su estado deseado y OpenShift (al igual que Kubernetes) hará lo posible en todo momento para alcanzar dicho estado.

  <img src="https://github.com/Aliuken/Documentacion-docker-kubernetes-y-openshift/blob/main/Ciclo_vida_pod.png" width="800">

* **Problemas de los pods**

  <img src="https://github.com/Aliuken/Documentacion-docker-kubernetes-y-openshift/blob/main/Problema_pod.png" width="350">

#### 1.4.2. Deployments y DeploymentConfigs

* **Solución a los problemas de los pods (con deployments)**

  Repaso:
  * Un deployment es una configuración para levantar réplicas de pods en el cluster.
  * Un deployment contiene n ReplicaSets, que a su vez contienen m réplicas de pods.
  * Un DeploymentConfig es un componente de OpenShift equivalente a los deployment de Kubernetes, pero con más características (como, por ejemplo, versionado).
  * Un DeploymentConfig contiene n ReplicationControllers, que a su vez contienen m réplicas de pods.
  * No tiene sentido usar Deployments o DeploymentConfigs con más de una réplica para BBDD, salvo que se use un producto como MySQL Galera.

  <img src="https://github.com/Aliuken/Documentacion-docker-kubernetes-y-openshift/blob/main/Soluci%C3%B3n_pod.png" width="500">

* **Problemas de los deployments**

  <img src="https://github.com/Aliuken/Documentacion-docker-kubernetes-y-openshift/blob/main/Problema_deployment.png" width="600">

#### 1.4.3. Services

* **Solución a los problemas de los deployments (con services)**

  Repaso:
  * Un service ofrece una IP fija, nombre fijo y puerto fijo al que conectarse.
  * Un service tiene asociado un deployment y hace de intermediario con los pods replicados de un deployment.

  <img src="https://github.com/Aliuken/Documentacion-docker-kubernetes-y-openshift/blob/main/Soluci%C3%B3n_deployment.png" width="600">

* **Problemas de los services**

  En los services no se suele poner el puerto 80 porque de ese modo no podría haber más de un service en el puerto 80. Por tanto, un service te ata a un puerto que no es el 80 (en el ejemplo, el 30205).

  <img src="https://github.com/Aliuken/Documentacion-docker-kubernetes-y-openshift/blob/main/Problema_servicio.png" width="800">

#### 1.4.4. Routes

* **Solución a los problemas de los services (con routes)**

  Repaso:
  * Una route es la alternativa de OpenShift a los ingress de Kubernetes.
  * Una route es un mapeo de una tupla [URL + puerto] a un service.
  * La creación de las routes es opcional. OpenShift fuerza a que (cuando sea necesario crearlas) siempre haya que crearlas manualmente, ya que sólo deberían crearse routes para los frontales. Por ejemplo, las BBDD no deberían tener routes.

  Los problemas de los services se solucionan creando n routes para los n services.
  
  El flujo final es el siguiente:
  1. El cliente llama al DNS con un dominio concreto para obtener la IP del servidor web.
  2. El cliente llama a la IP del servidor web (usando el puerto 80) pasándole la URL solicitada.
  3. El servidor web llama a la ruta asociada a la URL solicitada y puerto 80.
  4. La ruta llama a su servicio asociado (por ejemplo Servicio_tomcat).
  5. El servicio llama a alguno de los pods del deployment (en el ejemplo el deployment sería Tomcat) usando el puerto interno (8080).

  <img src="https://github.com/Aliuken/Documentacion-docker-kubernetes-y-openshift/blob/main/Soluci%C3%B3n_servicio.png" width="800">

### 1.5. Componentes opcionales al crear una aplicación

Repaso:
* Kubernetes no puede construir imágenes Docker. Para construir imágenes Docker se puede usar Docker o OpenShift.
* Al partir de una imagen Docker, se generan nuevos componentes como ImageStreams, BuildConfigs y Builds.
* Un ImageStream puede contener n ImageStreamTags.

<img src="https://github.com/Aliuken/Documentacion-docker-kubernetes-y-openshift/blob/main/Image_streams.png" width="700">

Un ejemplo de flujo completo de generación de todos los componentes de una aplicación de golpe (a partir de una imagen Docker) es el siguiente:

<img src="https://github.com/Aliuken/Documentacion-docker-kubernetes-y-openshift/blob/main/Flujo_creación_app_de_imagen.png" width="800">

Como se observa, al partir de una imagen docker, se generan a mayores los siguientes componentes:
* BuildConfig
* Build
* Pod del Build
* ImageStream
* ImageStreamTag
* Pod del Deployment o del DeploymentConfig (depende de qué se genere, en el diagrama de ejemplo se generó un DeploymentConfig)

#### 1.5.1. BuildConfig y Build

* **Descripción general**

  <img src="https://github.com/Aliuken/Documentacion-docker-kubernetes-y-openshift/blob/main/Estrategias_build.png" width="650">

  <img src="https://github.com/Aliuken/Documentacion-docker-kubernetes-y-openshift/blob/main/Ejemplos_build_config.png" width="750">

### 1.6. ConfigMaps y Secrets

Repaso:
* Un ConfigMap es como un fichero de propiedades.
* Un Secret es como un ConfigMap, pero con los datos codificados en Base64 (ojo, no están encriptados).

## 2. Flujos de creación de aplicaciones

### 2.1. Crear project

Repaso:
* Un project de OpenShift debe tener asociado "un y sólo un" namespace de Kubernetes.

oc new-project p1
```
Now using project "p1" on server "https://api.crc.testing:6443".

You can add applications to this project with the 'new-app' command. For example, try:

    oc new-app rails-postgresql-example

to build a new example application in Ruby. Or use kubectl to deploy a simple Kubernetes application:

    kubectl create deployment hello-node --image=k8s.gcr.io/serve_hostname
```

oc project
```
Using project "p1" on server "https://api.crc.testing:6443".
```

### 2.2. Crear todo lo necesario paso a paso

#### 2.2.1. Procedimiento

oc create deployment nginx1 --image quay.io/bitnami/nginx
```
deployment.apps/nginx1 created
```

oc scale deployment nginx1 --replicas 3
```
deployment.apps/nginx1 scaled
```

oc expose deploy nginx1 --name nginx1-svc --type NodePort --port 30205
```
service/nginx1-svc exposed
```

oc expose service nginx1-svc --name nginx1-route
```
route.route.openshift.io/nginx1-route exposed
```

#### 2.2.2. Comprobar elementos creados

oc get deployment
```
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
nginx1   3/3     3            3           2m3s
```

oc get replicaset
```
NAME                DESIRED   CURRENT   READY   AGE
nginx1-5db5f9f979   3         3         3       2m24s
```

oc get pod
```
NAME                      READY   STATUS    RESTARTS   AGE
nginx1-5db5f9f979-82sn2   1/1     Running   0          92s
nginx1-5db5f9f979-hkr8d   1/1     Running   0          92s
nginx1-5db5f9f979-rc5vr   1/1     Running   0          2m49s
```

oc get pod -o wide | grep master **(consultar pods del nodo master)**
```
nginx1-5db5f9f979-82sn2   1/1     Running   0          2m1s    10.217.0.224   crc-dzk9v-master-0   <none>           <none>
nginx1-5db5f9f979-hkr8d   1/1     Running   0          2m1s    10.217.0.225   crc-dzk9v-master-0   <none>           <none>
nginx1-5db5f9f979-rc5vr   1/1     Running   0          3m18s   10.217.0.223   crc-dzk9v-master-0   <none>           <none>
```

oc get service
```
NAME         TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)           AGE
nginx1-svc   NodePort   10.217.5.68   <none>        30205:31114/TCP   44s
```

oc get route
```
NAME           HOST/PORT                          PATH   SERVICES     PORT    TERMINATION   WILDCARD
nginx1-route   nginx1-route-p1.apps-crc.testing          nginx1-svc   30205                 None
```

### 2.3. Crear todo lo necesario de golpe (a partir de una imagen Docker)

#### 2.3.1. Procedimiento a partir de una imagen en DockerHub

oc new-app --name blog1 apasoft/blog
```
--> Found container image 5ada8e2 (2 years old) from Docker Hub for "apasoft/blog"

    Python 3.5
    ----------
    Python 3.5 available as container is a base platform for building and running various Python 3.5 applications and frameworks. Python is an easy to learn, powerful programming language. It has efficient high-level data structures and a simple but effective approach to object-oriented programming. Python's elegant syntax and dynamic typing, together with its interpreted nature, make it an ideal language for scripting and rapid application development in many areas on most platforms.

    Tags: builder, python, python35, python-35, rh-python35

    * An image stream tag will be created as "blog1:latest" that will track this image

--> Creating resources ...
    imagestream.image.openshift.io "blog1" created
    deployment.apps "blog1" created
    service "blog1" created
--> Success
    Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
     'oc expose service/blog1'
    Run 'oc status' to view your app.
```

oc expose service/blog1
```
route.route.openshift.io/blog1 exposed
```

#### 2.3.2. Procedimiento a partir de código en GitHub con Dockerfile

oc new-app --name blog2 https://github.com/almase/blog-1 --strategy=docker
```
--> Found container image 602660f (18 months old) from Docker Hub for "centos/python-36-centos7:latest"

    Python 3.6
    ----------
    Python 3.6 available as container is a base platform for building and running various Python 3.6 applications and frameworks. Python is an easy to learn, powerful programming language. It has efficient high-level data structures and a simple but effective approach to object-oriented programming. Python's elegant syntax and dynamic typing, together with its interpreted nature, make it an ideal language for scripting and rapid application development in many areas on most platforms.

    Tags: builder, python, python36, python-36, rh-python36

    * An image stream tag will be created as "python-36-centos7:latest" that will track the source image
    * A Docker build using source code from https://github.com/almase/blog-1 will be created
      * The resulting image will be pushed to image stream tag "blog2:latest"
      * Every time "python-36-centos7:latest" changes a new build will be triggered

--> Creating resources ...
    imagestream.image.openshift.io "python-36-centos7" created
    imagestream.image.openshift.io "blog2" created
    buildconfig.build.openshift.io "blog2" created
    deployment.apps "blog2" created
    service "blog2" created
--> Success
    Build scheduled, use 'oc logs -f buildconfig/blog2' to track its progress.
    Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
     'oc expose service/blog2'
    Run 'oc status' to view your app.
```

oc expose service/blog2
```
route.route.openshift.io/blog2 exposed
```

#### 2.3.3. Comprobar elementos creados

oc get buildconfig
```
NAME    TYPE     FROM   LATEST
blog2   Docker   Git    1
```

oc get build
```
NAME      TYPE     FROM          STATUS     STARTED          DURATION
blog2-1   Docker   Git@bc87e85   Complete   11 minutes ago   3m2s
```

oc get deployment
```
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
blog1   1/1     1            1           13m
blog2   1/1     1            1           12m
```

oc get replicaset
```
NAME               DESIRED   CURRENT   READY   AGE
blog1-78745898c4   1         1         1       14m
blog1-7f989fd9f7   0         0         0       14m
blog2-57f544b445   0         0         0       12m
blog2-5dbf79c66d   1         1         1       9m40s
```

oc get pod
```
NAME                     READY   STATUS      RESTARTS   AGE
blog1-78745898c4-8p7t9   1/1     Running     0          14m
blog2-1-build            0/1     Completed   0          13m
blog2-5dbf79c66d-b8tgn   1/1     Running     0          10m
```

oc get service
```
NAME    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
blog1   ClusterIP   10.217.5.208   <none>        8080/TCP   15m
blog2   ClusterIP   10.217.4.189   <none>        8080/TCP   13m
```
  
oc get route
```
NAME    HOST/PORT                   PATH   SERVICES   PORT       TERMINATION   WILDCARD
blog1   blog1-p1.apps-crc.testing          blog1      8080-tcp                 None
blog2   blog2-p1.apps-crc.testing          blog2      8080-tcp                 None
```

oc get imagestream
```
NAME                IMAGE REPOSITORY                                                               TAGS     UPDATED
blog1               default-route-openshift-image-registry.apps-crc.testing/p1/blog1               latest   9 minutes ago
blog2               default-route-openshift-image-registry.apps-crc.testing/p1/blog2               latest   4 minutes ago
python-36-centos7   default-route-openshift-image-registry.apps-crc.testing/p1/python-36-centos7   latest   7 minutes ago
```

oc get imagestreamtag
```
NAME                       IMAGE REFERENCE                                                                                                                     UPDATED
blog1:latest               apasoft/blog@sha256:9c8d3cedcd722ae6a9508ca532102501dfb796600c53a61d5178cdfaf22e02ed                                                About an hour ago
blog2:latest               image-registry.openshift-image-registry.svc:5000/p1/blog2@sha256:7dff1f108e0bd79fb3c2ca3ee94ac59496df7652bf14d37a622d9d6936d630fd   About an hour ago
python-36-centos7:latest   centos/python-36-centos7@sha256:ac50754646f0d37616515fb30467d8743fb12954260ec36c9ecb5a94499447e0                                    About an hour ago
```

oc rsh blog2-5dbf79c66d-b8tgn
* **(app-root) sh-4.2$** pwd
  ```
  /opt/app-root/src
  ```

* **(app-root) sh-4.2$** ls -lart
  ```
  total 84
  drwxrwxr-x. 1 default    root    19 Sep  9  2020 .pki
  -rw-rw----. 1 default    root  1024 Sep 17  2020 .rnd
  drwxrwxr-x. 2 default    root   148 Apr  7 15:04 templates
  -rw-rw-r--. 1 default    root   203 Apr  7 15:04 requirements.txt
  -rw-rw-r--. 1 default    root  7861 Apr  7 15:04 README.md
  -rw-rw-r--. 1 default    root   832 Apr  7 15:04 posts.json
  -rwxrwxr-x. 1 default    root   806 Apr  7 15:04 manage.py
  drwxrwxr-x. 2 default    root    25 Apr  7 15:04 htdocs
  -rw-rw-r--. 1 default    root   230 Apr  7 15:04 cronjobs.py
  drwxrwxr-x. 2 default    root    25 Apr  7 15:04 configs
  -rwxrwxr-x. 1 default    root  1454 Apr  7 15:04 app.sh
  -rw-rw-r--. 1 default    root  1126 Apr  7 15:04 Dockerfile
  drwxrwxr-x. 4 default    root    57 Apr  7 15:05 .s2i
  drwxrwxr-x. 1 default    root    28 Apr  7 15:05 ..
  drwxrwxr-x. 1 default    root    25 Apr  7 15:06 katacoda
  drwxrwxr-x. 1 default    root    43 Apr  7 15:06 blog
  drwxrwxr-x. 4 default    root    30 Apr  7 15:06 static
  drwxrwxr-x. 3 default    root    20 Apr  7 15:06 media
  -rw-r--r--. 1 1000680000 root 44032 Apr  7 15:07 db.sqlite3
  -rw-------. 1 1000680000 root    19 Apr  7 15:09 .bash_history
  drwxrwxr-x. 1 default    root    73 Apr  7 15:09 .
  ```

* **(app-root) sh-4.2$** env
  ```
  BLOG1_SERVICE_PORT=8080
  BLOG2_SERVICE_PORT=8080
  ...
  ```
* **(app-root) sh-4.2$** echo $BLOG2_SERVICE_PORT
  ```
  8080
  ```

* **(app-root) sh-4.2$** exit
  ```
  exit
  ```

### 2.4. Crear ConfigMaps y Secrets

#### 2.4.1. Crear ConfigMaps

nano propiedades.txt
```
V1=hola
V2=adios
```

oc create configmap cm1 --from-env-file=propiedades.txt
```
configmap/cm1 created
```

oc get configmap
```
NAME                       DATA   AGE
blog2-1-ca                 2      9m52s
blog2-1-global-ca          1      9m52s
blog2-1-sys-config         0      9m52s
cm1                        2      30s
kube-root-ca.crt           1      16m
openshift-service-ca.crt   1      16m
```

oc get configmap cm1
```
NAME   DATA   AGE
cm1    2      56s
```

#### 2.4.2. Crear Secrets

nano secretos.txt
```
V3=¿qué tal?
V4=¿cómo estás?
```

oc create secret generic secreto1 --from-env-file=secretos.txt
```
secret/secreto1 created
```

oc get secret
```
NAME                       TYPE                                  DATA   AGE
builder-dockercfg-hjq7n    kubernetes.io/dockercfg               1      38m
builder-token-dhjzm        kubernetes.io/service-account-token   4      38m
builder-token-pbjht        kubernetes.io/service-account-token   4      38m
default-dockercfg-bvzxw    kubernetes.io/dockercfg               1      38m
default-token-652xx        kubernetes.io/service-account-token   4      38m
default-token-79jx4        kubernetes.io/service-account-token   4      38m
deployer-dockercfg-nr5h9   kubernetes.io/dockercfg               1      38m
deployer-token-5hmzq       kubernetes.io/service-account-token   4      38m
deployer-token-j86nz       kubernetes.io/service-account-token   4      38m
secreto1                   Opaque                                2      8s
```

oc get secret secreto1
```
NAME       TYPE     DATA   AGE
secreto1   Opaque   2      15s
```
