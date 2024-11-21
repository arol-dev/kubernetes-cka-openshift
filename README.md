# OpenShift con Developer Sandbox Red Hat - Curso CKA

## Descripción
Este repositorio contiene un laboratorio diseñado para aprender a trabajar con OpenShift utilizando la **Developer Sandbox gratuita de Red Hat**. Exploraremos conceptos avanzados como despliegues de aplicaciones, resolución de problemas en Pods, y estrategias de despliegue como **Blue-Green** y **Canary**.

## Prerrequisitos
- **Registro en la Sandbox de Red Hat OpenShift**:
   - Regístrate en la [Sandbox de OpenShift](https://sso.redhat.com/auth/realms/redhat-external/protocol/openid-connect/auth?client_id=cloud-services&redirect_uri=https%3A%2F%2Fconsole.redhat.com%2Fopenshift%2Foverview&response_type=code&scope=openid&nonce=acf49cfd-a545-433c-a231-6ead21cda073&state=91ea2b9791534a93b455cb4c1db5329e&response_mode=fragment).
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
- Dirígete a la [Overview console de Red Hat](https://console.redhat.com/openshift/overview)
- Usa las credenciales de tu cuenta Red Hat para iniciar sesión.
- Una vez dentro, dirigete a *Developer Sandbox*

![Developer Sandbox](assets/images/developer-sandbox.PNG)
![Developer Sandbox Openshift](assets/images/developer-sandbox_2.PNG)

- Una vez dentro de la Developer Sandbox, podras utilizar las funcionalidades gráfica para crear los recursos o el terminal en la esquina superior derecha para ejecutar los comandos.

![Developer Sandbox console](assets/images/developer_sandbox_console.PNG)

## Paso 2: Crear un nuevo proyecto (namespace)
- Configura tu entorno de trabajo con el siguiente comando:
  ```bash
  oc project <nombre-del-proyecto>
  ```
  > Este comando establece el namespace o proyecto en el cual trabajarás.

## Lab 1: Despliegue de Aplicaciones

Desplegar una aplicación desde un repositorio Git local:

1. Clona el repositorio de ejemplo:
  ```bash
  git clone https://github.com/openshift/source-to-image.git
  cd source-to-image/examples/nginx-centos7
  ```
  > Aquí clonamos el repositorio localmente y navegamos hasta la carpeta con la aplicación.

2. Crear un nuevo BuildConfig basado en Docker:
  ```bash
  oc new-build --name s2i-nginx --binary --strategy docker
  ```
  > `oc new-build` crea un objeto `BuildConfig` para construir una imagen Docker a partir de la fuente local especificada. Este es un objecto clave para automatizar y gestionar la integración continua en OpenShift.

   El objeto **BuildConfig** en OpenShift es una definición que describe cómo construir una aplicación en la plataforma. Específicamente, define:

   1. **Origen del código fuente**: De dónde obtener el código (repositorios Git, etc.).
   2. **Estrategia de construcción**: Cómo construir la aplicación (Docker, S2I - Source-to-Image, etc.).
   3. **Desencadenantes de construcción**: Eventos que inician una nueva construcción (cambios en el código fuente, actualizaciones de imágenes base, activaciones manuales).
   4. **Configuración del entorno**: Variables, recursos y opciones necesarias para el proceso de construcción.

   Puedes ver el objeto **BuildConfig** recién creado en el menú de la derecha, en la sección **Builds**. Allí encontrarás un objeto llamado **s2i-nginx**. En la sección **YAML**, podrás visualizar un manifiesto YAML del tipo **BuildConfig**, correspondiente a la API `build.openshift.io/v1`.

3. Inicia el proceso de construcción:
  ```bash
  oc start-build s2i-nginx --from-dir=.
  ```
  > `oc start-build` inicia la construcción con el contenido local. Este comando tomará los archivos presentes en el directorio actual para construir la imagen.

## Lab 2: Importar e Instalar Aplicaciones

1. Importar la imagen de Python desde el registro de Red Hat:
  ```bash
  oc import-image rhel9/python-39:9.5-1730569177 --from=registry.redhat.io/rhel9/python-39:9.5-1730569177 --confirm
  ```
  > Este comando importa una imagen Docker del registro de Red Hat al registro local de OpenShift.

2. Crear una nueva aplicación desde la imagen:
  ```bash
  oc new-app --image-stream="marcoglorioso1594-dev/python-39:9.5-1730569177"
  ```
  > `oc new-app` crea una nueva aplicación en OpenShift, utilizando el flujo de imagen importado.

## Lab 3: Resolución de Problemas en Pods (CrashLoopBackOff)

Kubernetes tiene la limitación de que no se puede acceder a un pod cuando está en estado `CrashLoopBackOff`. Para solucionar esto, se puede utilizar `oc debug`:
```bash
oc debug deployment/<nombre-del-despliegue>
```
> Este comando permite ejecutar una sesión de depuración, creando una copia del contenedor del despliegue, pero con acceso interactivo.

## Lab 4: Despliegue Blue-Green

1. Crear una instancia de la aplicación con una plantilla:
  ```bash
  oc new-app --template=openshift/nginx-example --name=my-nginx-example-v1 --param=NAME=my-nginx-example-v1
  ```
  > `oc new-app` genera un despliegue a partir de la plantilla `nginx-example`, con el nombre especificado.

2. Probar la aplicación en el navegador:
   - [http://my-nginx-example-v1-marcoglorioso1594-dev.apps.sandbox-m3.1530.p1.openshiftapps.com/](http://my-nginx-example-v1-marcoglorioso1594-dev.apps.sandbox-m3.1530.p1.openshiftapps.com/)
   > La URL de la aplicación se genera automáticamente mediante una ruta definida en OpenShift.

3. Probar con una llamada `curl`:
  ```bash
  curl -sS -D - $(oc get route my-nginx-example-v1 -o jsonpath='{.spec.host}') -o /dev/null | grep server
  ```
  > Utiliza `curl` para hacer una solicitud HTTP a la aplicación y muestra información del servidor.

4. Desplegar una segunda versión de la aplicación (Blue-Green):
  ```bash
  oc new-app --template=openshift/nginx-example --name=my-nginx-example-v2 --param=NAME=my-nginx-example-v2 --param=NGINX_VERSION=1.22-ubi8
  ```
  > Despliega una nueva versión con una versión diferente de NGINX.

5. Implementar el cambio de rutas:
  ```bash
  oc patch route/<nombre-de-la-ruta> -p '{"spec": {"to": {"name": "nuevo-servicio"}}}'
  ```
  > `oc patch` permite actualizar las rutas para redirigir el tráfico al nuevo servicio.

6. Verificación:
  ```bash
  curl -sS -D - $(oc get route my-nginx-example-v1 -o jsonpath='{.spec.host}') -o /dev/null | grep server
  ```
  > Utiliza `curl` para confirmar que la nueva versión del servidor está activa.

## Lab 5: Despliegue Canary

1. Editar la ruta para definir el balanceo de tráfico:
  ```bash
  oc edit route my-nginx-example-v1
  ```
  > Edita la ruta para establecer los pesos de balanceo entre las dos versiones.

2. Definir configuración de balanceo en el YAML:
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
  > Define un peso igual entre ambas versiones para un despliegue Canary.

3. Verificación:
  ```bash
  for i in $(seq 1 6); do curl -sS -D - $(oc get route my-nginx-example-v1 -o jsonpath='{.spec.host}') -o /dev/null | grep server; done
  ```
  > Ejecuta varias solicitudes para confirmar que ambas versiones reciben el tráfico esperado.

## Lab 6: Despliegue de la Aplicación Bookinfo
- Desplegar la aplicación Bookinfo desde un manifiesto YAML.
- Crear rutas e ingress para exponer los servicios.

### Paso 1: Desplegar la aplicación con el manifiesto YAML

Utiliza el manifiesto de Bookinfo:
- [YAML Bookinfo](https://github.com/istio/istio/blob/master/samples/bookinfo/platform/kube/bookinfo.yaml)

Copia el contenido del YAML y realiza el despliegue desde la sección `+ Add` en OpenShift.

### Paso 2: Crear una ruta e ingress

Define el siguiente manifiesto para el `Ingress`:
```yaml
kind: Ingress
apiVersion: networking.k8s.io/v1
metadata:
  name: example
  namespace: marcoglorioso1594-dev
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

### Verificación
- Accede a la URL proporcionada para la aplicación y confirma que los diferentes servicios están operativos.

## Lab 7: Autoescalado Horizontal (Horizontal Pod Autoscaler - HPA)

1. Crear un despliegue para una aplicación simple:
  ```bash
  oc new-app --name=my-httpd --docker-image=httpd:latest
  ```
  > `oc new-app` crea un despliegue de Apache HTTP Server a partir de una imagen Docker pública.

2. Crear un Autoescalador Horizontal para la aplicación:
  ```bash
  oc autoscale deployment/my-httpd --min=1 --max=10 --cpu-percent=50
  ```
  > `oc autoscale` configura el escalado automático del despliegue `my-httpd` para mantener entre 1 y 10 réplicas, dependiendo del uso de CPU.

3. Generar carga para probar el escalado:
  ```bash
  for i in {1..100}; do curl -s http://$(oc get route my-httpd -o jsonpath='{.spec.host}'); done
  ```
  > Este comando genera múltiples solicitudes HTTP para aumentar la carga de la aplicación y activar el escalado.

4. Verificación:
  ```bash
  oc get hpa
  ```
  > Verifica el estado del Autoescalador Horizontal y las métricas de uso de la CPU.

## Lab 8: Implementación de ConfigMap y Secret

1. Crear un ConfigMap para una aplicación:
  ```bash
  oc create configmap my-config --from-literal=APP_COLOR=blue
  ```
  > `oc create configmap` crea un ConfigMap con una clave-valor llamada `APP_COLOR`.

2. Crear un Secret para la aplicación:
  ```bash
  oc create secret generic my-secret --from-literal=DB_PASSWORD=mysecretpassword
  ```
  > `oc create secret` crea un Secret que almacena una contraseña de base de datos.

3. Actualizar el despliegue para usar el ConfigMap y el Secret:
  ```bash
  oc set env deployment/my-httpd --from=configmap/my-config
  oc set env deployment/my-httpd --from=secret/my-secret
  ```
  > `oc set env` agrega variables de entorno al despliegue desde el ConfigMap y el Secret.

4. Verificación:
  ```bash
  oc describe deployment/my-httpd
  ```
  > Verifica que el despliegue `my-httpd` tenga las variables de entorno configuradas desde el ConfigMap y el Secret.