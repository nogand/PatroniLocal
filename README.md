# Configuración de Patroni con etcd local para una sola VM con Ubuntu.
#### Probado con Ubuntu 18 y Ubuntu 20, ediciones de servidor y de escritorio

En las instrucciones siguientes, sustituir los elementos entre < > por los valores apropiados para el servidor en el que se está trabajando. Partir de una instalación de Ubuntu 20, incluyendo PostgreSQL.

#### Como el usuario root:
1. Detener el _cluster_ de BDD que se levanta por omisión.
```bash
systemctl stop postgresql@<version>-<cluster>
```
2. Eliminar el _cluster_, puesto que se va a usar Patroni para controlar las instancias locales.
```bash
pg_dropcluster <version> main
```
3. Recargar la configuración de los servicios supervisados por systemd
```bash
systemctl daemon-reload
```
4. Instalar el administrador de configuraciones etcd, el machote de orquestación Patroni y el administrador de sesiones screen.
```bash
apt install etcd patroni screen
```
5. Validar que etcd haya levandado correctamente como un _cluster_ de un solo miembro.
```bash
etcdctl member list
etcdctl cluster-health
```
6. [Opcional] Levantar el administrador de sesiones screen, si se quiere evitar abrir múltiples sesiones de terminal hacia el servidor, o se requiere dejar en ejecución el servicio aunque se desconecten las sesiones.
```bash
screen
```

#### Como el usuario postgres:
1. Descargar los dos machotes de patroni.
```bash
wget
```
2. Abrir cuatro ventanas o sesiones, sea con screen o con un cliente de terminal.
3. En la primera sesión, levantar Patroni con el primer machote, redirigiendo los mensajes y errores a sendos archivos en el directorio local.
```bash
patroni patroni_1.yml > patroni_1.log 2> patroni_1.err
```
4. En la segunda sesión, levantar Patroni con el segundo machote, redirigiendo los mensajes y errores a sendos archivos en el directorio local.
```bash
patroni patroni_2.yml > patroni_2.log 2> patroni_2.err
```
5. En la tercera sesión, dejar desplegándose los mensajes y errores de ambos servicios.
```bash
tail -f patroni_?.{log,err}
```
6. En una cuarta ventana, utilizar patronictl para confirmar el estado del _cluster_.
```bash
patronictl -d etcd://localhost:2379 list cluster_1
```

#### Ejercicios sugeridos:
Una vez realizada la configuración según los pasos anteriores, se tiene un _cluster_ de PostgreSQL de dos nodos. Lo siguiente sirve como práctica del funcionamiento y uso.
* Crear un usuario y una base de datos en el nodo líder. Crear tablas y cargarle datos.
* Leer datos del nodo réplica.
* Insertar datos en el nodo réplica. (Debe fallar)
* Bajar el nodo líder, sea matando procesos de PostgreSQL, cancelando la ejecución del primer Patroni o terminando la primera sesión.
* Confirmar estado del clúster. (Usando `patronictl`).
* Insertar datos en el nuevo nodo líder.
* Subir el primer Patroni de nuevo.
* Confirmar estado y funcionamiento del _cluster_.
* Utilizar `etcdctl ls` y `etcdctl get` para entender el funcionamiento subyacente.
* Usar `patronictl` para realizar un cambio inmediato de maestro y confirmar que sirve.
* Usar `patronictl` para realizar un cambio calendarizado de maestro y confirmar que sirve.
