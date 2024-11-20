
# OpenShift with Developer Sandbox Red Hat Lab - CKA Course

## Descripción
Este repositorio contiene un laboratorio diseñado para aprender a trabajar con OpenShift utilizando la **Developer Sandbox gratuita de Red Hat**. Exploraremos conceptos avanzados como despliegues de aplicaciones, resolución de problemas en Pods, y estrategias de despliegue como **Blue-Green** y **Canary**.

## Prerrequisitos
- **Registro en la Sandbox de Red Hat OpenShift**:
   - Regístrate en la [Sandbox de OpenShift](https://www.redhat.com/en/openshift-sandbox).
   - Accede a tu cuenta personal y asegúrate de tener un entorno funcional.
- **Herramientas necesarias**:
   - CLI de OpenShift (`oc`), ya preinstalada en la Sandbox.
   - Navegador web para interactuar con la consola gráfica de OpenShift.
- **Conocimientos previos**:
   - Kubernetes (pods, servicios, manifiestos YAML).
   - Docker y estrategias de contenedores.

## Objetivos
- Aprender a utilizar la Sandbox de OpenShift proporcionada por Red Hat.
- Realizar configuraciones avanzadas, como rutas, servicios e ingress.
- Implementar estrategias de despliegue Blue-Green y Canary.
- Utilizar herramientas de resolución de problemas en Pods.
- Desplegar aplicaciones desde repositorios Git y manifiestos YAML.

## Paso 1: Acceso a la Sandbox
- Dirígete a la [consola de OpenShift](https://console-openshift-console.apps.sandbox-m3.1530.p1.openshiftapps.com/).
- Usa las credenciales de tu cuenta Red Hat para iniciar sesión.
- Una vez dentro, utiliza la consola gráfica o el terminal en la esquina superior derecha para ejecutar los comandos.

## Paso 2: Crear un nuevo proyecto (namespace)
- Configurar tu entorno de trabajo con el siguente comando:
  ```bash
  oc project <nombre-del-proyecto>
  ```

### Lab 1: Despliegue de Aplicaciones**

Desplegar una aplicación desde un repositorio Git local
```bash
# Build of an application from local repo. Fail.
git clone https://github.com/openshift/source-to-image.git
cd examples/nginx-centos7
```
```bash
oc new-build --name s2i-nginx --binary --strategy docker  # New build crea una risorsa di tipo build-config, input=repository output=docker image che push to a docker registry. 
```

```bash
# Puoi caricarlo su uno esterno o quello locale di openshift  
oc start-build s2i-nginx --from-dir=.
```

## Lab 2: Install App On Openshift. Import image and install it. Fail.
```bash
oc import-image rhel9/python-39:9.5-1730569177 --from=registry.redhat.io/rhel9/python-39:9.5-1730569177 --confirm
oc new-app --image-stream="marcoglorioso1594-dev/python-39:9.5-1730569177"
```

## Lab 3: Troubleshooting crashloop
In kubernetes c'e la limitazione che non possiamo entrae nel pod quando continua a fallire e reavviarsi sempre.
Utilizar `oc debug` para solucionar problemas en pods con errores:
```bash
oc debug deployment/<nombre-del-despliegue>
```

## Lab 4: Despliegue Blue-Green
CRD OS Template = un manifest kubernetes pero con variabili che si possono sostituire. Molto simile a un helm chart

```bash
oc new-app --template=openshift/nginx-example --name=my-nginx-example-v1 --param=NAME=my-nginx-example-v1
```
Prueba la aplicación desde el navegador senza https:
- [http://my-nginx-example-v1-marcoglorioso1594-dev.apps.sandbox-m3.1530.p1.openshiftapps.com/](http://my-nginx-example-v1-marcoglorioso1594-dev.apps.sandbox-m3.1530.p1.openshiftapps.com/)

Prueba applicacion con unas llamadas curl: 
```bash
curl -sS -D - $(oc get route my-nginx-example-v1 -o jsonpath='{.spec.host}') -o /dev/null | grep server
```

Crea un route -> service -> ingress(loadbalancer) per andare all'esterno

Deploy second app with nginx version:

```bash
oc new-app --template=openshift/nginx-example --name=my-nginx-example-v2 --param=NAME=my-nginx-example-v2 --param=NGINX_VERSION=1.22-ubi8 
```

Prueba applicacion con unas llamadas curl: 
```bash
curl -sS -D - $(oc get route my-nginx-example-v2 -o jsonpath='{.spec.host}') -o /dev/null | grep server
```
Implementar despliegues Blue-Green y Canary usando rutas y pesos en servicios:
```bash
oc patch route/<nombre-de-la-ruta> -p '{"spec": {"to": {"name": "nuevo-servicio"}}}'
```

### Verificacion
Return version 2 calling service
```bash
curl -sS -D - $(oc get route my-nginx-example-v1 -o jsonpath='{.spec.host}') -o /dev/null | grep server 
```

## Lab 5: Despliegue Canary
```bash
oc edit route my-nginx-example-v1
```
```yaml
spec:
  host: my-nginx-example-v1-marcoglorioso1594-dev.apps.sandbox-m3.1530.p1.openshiftapps.com
  to:
    kind: Service
    name: my-nginx-example-v1
    weight: 50
  alternateBackends:
  - kind: Service
    name: my-nginx-example-v2
    weight: 50
  wildcardPolicy: None
```

To see the weight in the route

```bash
oc get route my-nginx-example-v1
```

### Verificacion
```bash
for i in $(seq 1 6); do curl -sS -D - $(oc get route my-nginx-example-v1 -o jsonpath='{.spec.host}') -o /dev/null | grep server; done
```

## Lab 5: Despliegue de una Aplicación NGINX
- Desplegar una aplicación NGINX desde repositorio GitHub.
- Crear rutas e ingress para exponer los servicios.

Queremos deplegar una simpe applicacion Nginx en nuestro cluster Openshift.

Repositorio Git
https://github.com/arol-dev/nodejs-app-aroldev/blob/main/index.js

En la SandBox Openshit en la parte alta a izquierda click en la entry "+Add", y despues la opcion Git repository. 

Proporociona los dettales para hacer un import con el link proporcionado anteriormente.

### Verificacion


## Lab 6: Despliegue de la Aplicación Bookinfo
- Desplegar la aplicación Bookinfo desde un manifiesto YAML.
- Crear rutas e ingress para exponer los servicios.

# Deploy Yaml file Booking info

Proporcionado el siguente Yaml Manifest: 
https://github.com/istio/istio/blob/master/samples/bookinfo/platform/kube/bookinfo.yaml

Copy Paste all code from the yaml file and deploy to Import Yaml section (+ Add seccion)

Create una route (Punto d'ingresso) por el path "/"

Create ingress como sigue adelante: 

```yaml
kind: Ingress
apiVersion: networking.k8s.io/v1
metadata:
  name: example
  namespace: marcoglorioso1594-dev
  uid: 051a51b9-6106-4e2c-8dcf-f270dfe6e2ea
  resourceVersion: '3114263227'
  generation: 1
  creationTimestamp: '2024-11-14T22:07:15Z'
  managedFields:
    - manager: Mozilla
      operation: Update
      apiVersion: networking.k8s.io/v1
      time: '2024-11-14T22:07:15Z'
      fieldsType: FieldsV1
      fieldsV1:
        'f:spec':
          'f:rules': {}
    - manager: route-controller-manager
      operation: Update
      apiVersion: networking.k8s.io/v1
      time: '2024-11-14T22:07:15Z'
      fieldsType: FieldsV1
      fieldsV1:
        'f:status':
          'f:loadBalancer':
            'f:ingress': {}
      subresource: status
spec:
  rules:
    - host: productpage-marcoglorioso1594-dev.apps.sandbox-m3.1530.p1.openshiftapps.com
      http:
        paths:
          - path: /productpage
            pathType: Prefix
            backend:
              service:
                name: productpage
                port:
                  number: 9080
          - path: /login
            pathType: Prefix
            backend:
              service:
                name: productpage
                port:
                  number: 9080
          - path: /logout
            pathType: Prefix
            backend:
              service:
                name: productpage
                port:
                  number: 9080
          - path: /static
            pathType: Prefix
            backend:
              service:
                name: productpage
                port:
                  number: 9080
status:
  loadBalancer:
    ingress:
      - hostname: router-default.apps.sandbox-m3.1530.p1.openshiftapps.com
```

### Verificacion