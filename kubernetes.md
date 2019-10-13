# Kubernetes

### Resumen y comandos más importantes de Kubernetes

Un **pod** es un envoltorio de un _contenedor<br/>
Los **pods** se definen en un archivo yml<br/>
Los pods **no** se pueden ver desde afuera del cluster, o sea que no es posible acceder a un pod por su ip/puerto<br/>
**minikube ip** nos devuelve la ip del minukube para poder conectarnos a sus servicios<br/>
**kubectl get pods/kubectl get po** muestra todo los _pods_ que están corriendo<br/>
**kubectl get services/kubectl get service** muestra todo los _servicios_ que están corriendo<br/>
**kubectl get all** muestra todo lo que tenemos definido en el cluster<br/>
**kubectl apply -f name-file.yml** crea un pod en base al archivo leído<br/>
**kubectl describe pod name-of-pod** describe el pod con el nombre que especifiquemos<br/>
**kubectl describe service name-of-service** describe el servicio con el nombre que especifiquemos<br/>
**kubectl delete pod mame-of-pod** borra el pod específicado<br/>
**kubectl delete pods --all** elimina todos los pods<br/>
**kubectl delete rs name-of-resultset** elimina el _ResultSet_ que le indiquemos<br/>
**kubectl describe rs name-of-replica-set** Describe el _ReplicaSet_ que indiquemos<br/>
**kubectl exec -it pod-name sh** Permite ejecutar un _pod_ que no tiene _bash_ de forma interactiva<br/>
**kubectl logs name-of-pod** Muestra los logs del por especificado<br/>

### Comandos más avanzados

**kubectl -it exec webapp sh** comando utiliado para conectarnos con un pod de forma interactiva y poder ejecutar comandos directamente a ese pod<br/>
**kubectl get po --show-labels** muestra los pods y además agerga las etiquetas<br/>
**kubectl get po --show-labels -l release=0** muestra los pods, filtrando por la etiqueta especificada y agregando la etiqueta, en el ejemplo se realiza un filtrado por la etiqueta _release=0_<br/>

### Creación de un pod

Un pod lo empezamos por definir en un archivo yml, ejemplo:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
  labels:
    app: webapp
spec:
  containers:
    - name: webapp
      image: richardchesterwood/k8s-fleetman-webapp-angular:release0-5
```
Posteriormente debemos desplegar ese pod, para lo cual usaremos el comando **kubectl apply -f mi-pod.yml**

### Creación de un servicio

Un pod no puede ser accedido desde afuera, pero un servicio sí, por ese motivo crearemos un servicio que nos permita acceder al pod creado anteriormente, esto se logra por medio de un archivo yml, ejemplo:

```yaml
apiVersion: v1
kind: Service
metadata:
  # Unique key of the Service instance
  name: fleetman-webapp

spec:
  selector:
    app: webapp

  ports:
    - port: 80
      name: http
      nodePort: 30080

  type: NodePort
```

Posteriormente desplegamos el servicio con el mismo comando que se despliega un pod **kubectl apply -f mi-servicio.yml**<br/>

Nota: es importante señalar que el _label_ del servicio debe coincider con el _app_ del pod para que tengan comunicación.

Ahora, para acceder a la aplicación y validar que todo esté bien, necesitamos la ip del minikube dado que todo lo estamos corriendo en un minikube, entonces ejecutamos: **minikube ip**, nos devielve la ip eg. 192.168.99.100 a la cual solo agregamos el puerto que definimos en el servicio y listo: http://192.168.99.100:30080/<br/>

### Un servicio, múltiples pods

Se puede tener un solo servicio y múltiples pods, de tal forma que el servicio se conecta a un solo pod, retomando el ejemplo anterior de _un pod-un servicio_, ahora crearemos un segundo pod, agergando ademas la propiedad _release_ dentro de _labels_, obviamente el _name_ del segundo pod debe ser disitnto del primer pod. entonces tenemos las siguientes definiciones de los pods:<br/>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
  labels:
    app: webapp
    release: "0"
spec:
  containers:
    - name: webapp
      image: richardchesterwood/k8s-fleetman-webapp-angular:release0
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-release-0-5
  labels:
    app: webapp
    release: "0-5"
spec:
  containers:
    - name: webapp
      image: richardchesterwood/k8s-fleetman-webapp-angular:release0-5
```
Nótese que el _name_ es distinto, así como el _release_ y por supuesto también la propiedad _image_.<br/>

Respecto al servicio tenemos que agregar la propiedad _release_ dentro de _selector_, obteniendo lo siguiente:

```yaml
apiVersion: v1
kind: Service
metadata:
  # Unique key of the Service instance
  name: fleetman-webapp

spec:
  selector:
    app: webapp
    release: "0-5"

  ports:
    - port: 80
      name: http
      nodePort: 30080

  type: NodePort
```

Ahora solo falta desplegar el respectivo pod y desde luego el servicio.

### ReplicaSets

* Lo primero que vamos a cambiar será el _kind_, que antes era _Pod_, ahora será _ReplicaSet_, ya que no puede ser ambos.<br/> 
* El _name_ pasa a ser algo secundario ya que los nombres se generan en automático, así que lo borramos.<br/>
* En este ejemplo en particular el _release_ tampoco es necesario.<br/>
* Agregamos la sección _spec_, donde se especifica el número de réplicas y el _template_.<br/>
* El _template_ tiene la información del pod, que hemos venido usando con anterioridad, es decir _metadata_ y _spec_.<br/>
* Es muy importante agregar un _name_ para nuestro _ReplicaSet_, esto lo hacemos agregando la etiqueta _metadata_ y anidamos _name_.<br/>
* También en _apiVersion_, aparte de la versión agregamos "_apps/_".<br/>
* Dentro de _spec_ tenemos que agregar el _selector_, el cual a su vez anida _matchLabels_ y éste a su vez agrega la etiqueta _app_, necesaria para hacer "match" con el _label_ dentro del _template_.<br/>

Con todos estos cambios obtenemos el siguiente archivo:<br/>

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: webapp
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
        - name: webapp
          image: richardchesterwood/k8s-fleetman-webapp-angular:release0-5
  selector:
    matchLabels:
      app: webapp
```

Ahora solo falta aplicar los cambios con el comando kubectl apply -f **nombre-del-archivo.yml** y en caso de que queramos ver sus detalles, podemos recurrir al siguiente comando: kubectl describe rs **name-of-replica-set**

### Deployments

Es preferible trabajar con _Deployments_, los cuales son similares a las _ReplicaSet_ pero con una característica adiciional: el tiempo que están abajo los pods cuando se hace algún cambio en el selector de versión (release) es cero.<br/>

Para este nuevo tema utilizaremos el _yml_ que hemos utilizado en los _ReplicaSet_, aplicando las respectivas modificaciones, las cuales son:

* En _Kind_ debe llevar _Deployment_.<br/>
* Dentro de _spec_ opcionalemnte se puede agregar _minReadySeconds_ que es para indicar el número de segundos de retraso que tardará en desplegar la nueva versión de la imagen.<br/>

Con todo lo anterior, obtenemos el siguiente documento:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deploy
spec:
  minReadySeconds: 10
  replicas: 5
  template:
    metadata:
      labels:
        app: webapp-deployment
    spec:
      containers:
        - name: whatever
          image: richardchesterwood/k8s-fleetman-webapp-angular:release0-5
  selector:
    matchLabels:
      app: webapp-deployment
```
Y su respectivo servicio:<br/>
```yaml
apiVersion: v1
kind: Service
metadata:
  name: fleet-web-deployment

spec:
  selector:
    app: webapp-deployment
  ports:
    - port: 80
      name: http
      nodePort: 30085
  type: NodePort
```
Ahora, cuando se cambié del release 0 al release 0.5 el tiempo que tardará en hacer el cambio será de 10 segundos, si esta etiqueta se omite, el valor por defecto es cero, cabe señalar que el _ReplicaSet_ a pesar que se queda sin _pods_ no es eliminado, ya que en caso que necesitemos hacer un _rollback_ ese mismo _ReplicaSet_ nos va a funcionar, por eso no es eliminado en automático.<br/>

Nota: cuando un _deployment_ no se puede llevar a cabo por alguna razón se mantiene la última versión funcional corriendo, mientras que Kubernetes entra en un ciclo infinito tratando de llevar a cabo el _deploy_.<br/>

#### Rollouts

Esta característica permite hacer _rollouts_ entre _releases_ a voluntad, por defecto almacena los últimos 10 _releases_ pudiendo especificar a cual _release_ queremos apuntar, cabe señalar que no es recomendable usar los _Rollouts_ ya que la versión o _release_ desplegada no coincidirá con lo que tenemos en el archivo _yml_, por éste motivo es mejor solo utilizarlo en casos de emergencia. El comando para llevar a cabo esta tarea es:<br/>

**kubectl rollout status deploy deploy_name**<br/>

Ahora, si queremos revertir o deshacer el _rollout_, disponemos de un comando, que es el siguiente:<br/>

**kubectl rollout undo deploy deploy-name**<br/>

Así mismo tenemos un comando que nos permie consultar el historial de los _rollouts_ que se han llevado a cabo:<br/>

**kubectl rollout history deploy webapp**<br/>

### Namespaces

Los _Namespaces_ son espacios (recursos) que están aislados entre sí, dentro de ellos pude haber _deployments_, _services_, _pods_, etc. por defecto se usa el _defult namespace_.<br/>

Algunos puntos a tener en cuenta son:<br/>

* **kubectl get ns** muestra los _name results_ que estén corriendo<br/>
* Por defecto kubernetes crea dos _name sapces_ que usa para operaciones internas: **kube-public** y **kube-system**<br/>
* Si queremos apuntar a un _name sapce_ en particular lo hacemos con el argumento _-n_ por ejemplo: **kubectl get all -n kube-system**<br/>

### Service Discovery

El _Service Discovery_ es un mecanismo por el cual distintos servicios pueden interactuar entre sí sin conocer _a priori_ las direcciones IP. <br/>

Por ejemplo, supongamos que el servicio A necesita comunicarse con el servicio B, pero éste no conoce la IP del servicio B. Esto se resuelve por medio de un _kube-dns_ (que básicamente es un servidor DNS), de tal manera que el servico A solicita al _kube-dns_ la IP del servicio B, el _kube_dns_ entonces busca cual es la IP que corresponde a un servicio de nombre B y la devuelve como respuesta al servicio A, el servicio A ahora conoce la IP del servicio B y puede establecer comunicación con él.<br/>

Entre las ventajas que podemos destacar de este mecanismo es que si un servicio muere al ser restaurado por _Kubernetes_ su IP cambiará, pero ésto ya no supondrá un problema, ya que la IP se actualiza en el _kube-dns_.

Para ejemplificar éstos conceptos vamos a crear un servicio que implemente _MySQL_ y nos conectaremos a él por medio de otro servicio sin conocer la IP. Tenemos entonces el archivo del pod:<br/>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mysql
  labels:
    app: mysql
spec:
  containers:
    - name: mysql
      image: mysql:5
      env:
        - name: MYSQL_ROOT_PASSWORD
          value: password
        - name: MYSQL_DATABASE
          value: fleetman
```
Con su respectivo servicio:<br/>

```yaml
kind: Service
apiVersion: v1
metadata:
  name: database
spec:
  selector:
    app: mysql
  ports:
    - port: 3306
  type: ClusterIP
```

Por otro lado utilizaremos uno de los _pods_ que hemos venido usando hasta el momento o cualquiera que corra sobre la imagen _alpine_:

```shell script
exec -it pod-name sh
```
A continuación y después de haber entrado en el _pod_ procedemos a instalar _MySQL_:<br/>

```shell script
# apk update
# apk add mysql-client
```

Acto seguido procedemos a conectarnos con el servicio de _MySQL_, el cual según la etiqueta _metadata.name_ se denomina _*database*_:<br/>

```shell script
# mysql -h database -uroot -ppassword fleetman
```

Donde:<br/>
* -h indica la url del servidor al que nos deseamos conectar
* database es el nombre del servicio, del cual desconocemos su IP, pero el _Server Discovery_ se encarcará de resolver
* -u indica el nombre de usuario que se ha de concatenar al parámetro, en este caso el usuario es _root_
* -p indica el valor de la contraseña que se ha de adjuntar al parámetro, en este caso la contraseña es _password_
* Y por último tenemos el nombre de la base de datos, en este caso la base a la que nos conectamos se llama _fleetman_

Si todo salió bien, nos conectaremos al servicio denombre _database_, que a su vez tiene un _pod_ con _MySQL_ y en la base de datos _fleetman_

```shell script
MySQL [fleetman]> 
```
