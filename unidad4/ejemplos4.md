## Ejemplo 1: HELM

### Instalación de helm v3

    curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
    chmod 700 get_helm.sh
    ./get_helm.sh

### Inicializar repositorio de Chart

    helm repo add bitnami https://charts.bitnami.com/bitnami

    helm repo list
    NAME   	URL                                              
    bitnami	https://charts.bitnami.com/bitnami         

    helm repo update      

### Instalación de chart

    helm search repo 

    helm search repo nginx
    
    helm install server_web bitnami/nginx --set service.type=NodePort


Listamos las aplicaciones instaladas

    helm ls

Borramos la aplicación

    helm delete server_web

Vemos los recursos creados

    helm status server_web

Desinstalar aplicación:

    helm uninstall <name>

### Crear nuestros propios charts

    helm create mychart

* En el directorio `templates` vamos a tener nuestros manifiestos yaml.
* En estos manifiestos podemos parametrizar los valores de los distintas configuraciones. De este modo a la hora de instalar el chart podremos definir la configuracion.
* Los valores por defecto de los parámetros indicados se indican en el fichero `values.yaml`.
* En las platillas además de variables, podemos añadir control (if, loop, ...)

Podemos comprobar los maniefiestos que se han creado a partir de las plantillas:

    helm install --debug --dry-run mynginx ./mychart

Para instalar nuestro chart, con los valores por defecto:

    helm install mynginx ./mychart

Tambien podemos indicar algunos de los parámetros:

    helm install mynginx ./mychart --set replicaCount=3 --set service.type=NodePort --set ingress.hosts[0].host=mynginx.192.168.99.100.nip.io

