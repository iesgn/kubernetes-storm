# Kubernetes for Storm

* Despliegue de aplicaciones en contenedores.

** Introducción a los contenedores
** Arquitectura de microservicios
** Tecnologías subyacentes y diferencias entre ellas: docker, cri-o, LXC, ...
** Ciclo de vida en el despliegue de aplicaciones con docker

* Introducción a kubernetes

** Características, historia, estado actual del proyecto kubernetes (k8s)
** Arquitectura básica de k8s
** Alternativas para instalación simple de k8s: minikube, kubeadm, k3s

* Despliegue de aplicaciones con k8s

** Pods
** ReplicaSet: Tolerancia y escalabilidad
** Deployment: Actualizaciones y despliegues automáticos
** Estrategias de despliegue en k8s: blue/green, canary, etc.
** Services
** DNS
** Ingress
** Ejemplos de uso y despliegues
** Configuración de aplicaciones: Variables de entorno, ConfigMaps, Secrets, ...
** Ejemplo de despliegue parametrizado
** StatefulSet
** DaemonSet
** AutoScale
** Jobs, cronjobs

* Despliegue de aplicaciones con Helm

* Almacenamiento

* Despliegue de aplicaciones de k8s en entornos en producción
