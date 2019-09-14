# Práctica Módulo 03 - Migración a la Nube

Proyecto académico con el objetivo de utilizar la infraestructura de Google Cloud Platform ([enunciado](./Enunciado-Practica_Migracion_a_la_Nube.pdf)).

Se ha comprobado la ejecución correcta de los ejercicios en un sistema operativo Windows 10, con las siguientes aplicaciones previamente instaladas:

* [Google Cloud SDK](https://cloud.google.com/sdk/install) 254.0.0.
  * bq 2.0.45.
  * core 2019.07.15.
  * gsutil 4.40.

## Parte 1

Crear un proyecto en GCP (se ha utilizado como nombre para el proyecto "*practica-kc-03-migranube*"):

```bash
gcloud projects create practica-kc-03-migranube
```

![Crear proyecto](./parte1/img/crear_proyecto.png)

Dar acceso completo al proyecto al usuario "​javioreto@gmail.com" con rol "*editor*":

```bash
gcloud projects add-iam-policy-binding practica-kc-03-migranube --member user:javioreto@gmail.com --role roles/editor
```

![Dar acceso usuario](./parte1/img/dar_acceso_usuario.png)

Crear presupuesto de 100€ con alertas al llegar al 50%, 90% y 100% de consumo:

![Crear presupuesto paso 1](./parte1/img/crear-ppto-alertas-01.png)
![Crear presupuesto paso 2](./parte1/img/crear-ppto-alertas-02.png)
![Crear presupuesto paso 3](./parte1/img/crear-ppto-alertas-03.png)

## Parte 2

Crear base de datos MySQL con CloudSQL (se ha utilizado para el nombre de la instancia "*mysql-migranube*" con contraseña de root "*passwordmysqlmigranube01*"):

![Crear base datos mysql paso 1](./parte2/img/crear_instancia_mysql_cloudsql_01.png)
![Crear base datos mysql paso 2](./parte2/img/crear_instancia_mysql_cloudsql_02.png)
![Crear base datos mysql paso 3](./parte2/img/crear_instancia_mysql_cloudsql_03.png)

Elegir la máquina, para desarrollo y pruebas, es "*db-g1-small*" (núcleo compartido con 1 CPU, 1,7 GB de RAM y almacenamiento HDD de 10 GB):

![Crear base datos mysql paso 4](./parte2/img/crear_instancia_mysql_cloudsql_04.png)

Habilitar las copias de seguridad al medio día:

![Crear base datos mysql paso 5](./parte2/img/crear_instancia_mysql_cloudsql_05.png)

Conectar con la base de datos utilizando CloudShell:

```bash
gcloud sql connect mysql-migranube --user=root
```

![Conectar instancia mysql](./parte2/img/conectar_instancia_mysql_cloudsql.png)

Crear bases de datos "*google*" y "*cloud*":

```SQL
CREATE DATABASE google;
CREATE DATABASE cloud;
```

![Crear bases de datos](./parte2/img/crear_bases_datos_instancia_mysql_cloudsql.png)

Crear usuario de MySQL "*alumno*" con contraseña "*googlecloud*", y darle todos los privilegios en las bases de datos "*google*" y "*cloud*":

```SQL
CREATE USER 'alumno'@'%' IDENTIFIED BY 'googlecloud';
GRANT ALL PRIVILEGES ON google.* TO 'alumno'@'%';
GRANT ALL PRIVILEGES ON cloud.* TO 'alumno'@'%';
FLUSH PRIVILEGES;
exit
```

![Crear usuario](./parte2/img/crear_usuario_datos_instancia_mysql_cloudsql.png)

Crear segmento de almacenamiento:

```bash
gsutil mb -l europe-west1 gs://migranube/
```

![Crear segmento](./parte2/img/crear_segmento.png)

Exportar las bases de datos "*google*" y "*cloud*" a un fichero del segmento "*gs://migranube/*":

![Exportar bases de datos 1](./parte2/img/exportar_bases_datos_01.png)

Establecer la ubicación del fichero a exportar en la carpeta "*exportacion-bd*" dentro del segmento:

![Exportar bases de datos 2](./parte2/img/exportar_bases_datos_02.png)

Las bases de datos "*google*" y "*cloud*" se exportan a un fichero SQL ([exportacion.sql](./parte2/exportacion.sql)):

![Exportar bases de datos 3](./parte2/img/exportar_bases_datos_03.png)

Importar las bases de datos exportadas "*google*" y "*cloud*" con el fichero exportado en el segmento "*gs://migranube/exportacion-bd/*":

![Importar bases de datos 1](./parte2/img/importar_bases_datos_01.png)

Comprobar que la importación ha resultado exitosa revisando los logs:

![Importar bases de datos 2](./parte2/img/importar_bases_datos_02.png)

Desescalar la máquina de la instancia, para desarrollo y pruebas, a "*db-f1-micro*" (núcleo compartido con 1 CPU y 614,4 MB de RAM):

![Desescalar base datos mysql](./parte2/img/desescalar_instancia_mysql_cloudsql.png)

## Parte 3

Crear una máquina virtual (se ha utilizado para el nombre de la instancia "*vm-apache-migranube*", núcleo compartido con 1 CPU, 614 MB de RAM y almacenamiento HDD de 10 GB):

![Crear instancia de máquina virtual 1](./parte3/img/crear_instancia_maquina_virtual_01.png)

![Crear instancia de máquina virtual 2](./parte3/img/crear_instancia_maquina_virtual_02.png)

Esperar a que la máquina virtual esté disponible:

![Crear instancia de máquina virtual 3](./parte3/img/crear_instancia_maquina_virtual_03.png)

Conectar con la máquina virtual por SSH:

![Conectar SSH 1](./parte3/img/conectar_ssh_01.png)

![Conectar SSH 2](./parte3/img/conectar_ssh_02.png)

Actualizar los paquetes disponibles del sistema operativo:

```bash
sudo apt-get update
```

![Actualizar paquetes](./parte3/img/actualizar_paquetes.png)

Instalar el servidor web Apache:

```bash
sudo apt-get install apache2 -y
```

![Instalar Apache](./parte3/img/instalar_apache.png)

Visitar la página expuesta en la máquina virtual para comprobar que el servidor web está levantado ("*http://ip_externa/*", no es "*https*"):

![Visitar página web 1](./parte3/img/visitar_pagina_web_01.png)

![Visitar página web 2](./parte3/img/visitar_pagina_web_02.png)

Terminar la sesión SSH:

```bash
exit
```

Detener la máquina virtual:

![Detener máquina virtual 1](./parte3/img/detener_maquina_virtual_01.png)

![Detener máquina virtual 2](./parte3/img/detener_maquina_virtual_02.png)

Crear una imagen personalizada basada en el disco de la máquina virtual "*vm-apache-migranube*", (se ha utilizado para el nombre de la imagen "*img-vm-apache-migranube*"):

![Crear imagen personalizada 1](./parte3/img/crear_imagen_personalizada_01.png)

![Crear imagen personalizada 2](./parte3/img/crear_imagen_personalizada_02.png)

![Crear imagen personalizada 3](./parte3/img/crear_imagen_personalizada_03.png)

Esperar a que se haya creado la imagen:

![Crear imagen personalizada 4](./parte3/img/crear_imagen_personalizada_04.png)

Crear la plantilla de instancias de máquinas virtuales basadas en la imagen personalizada "*img-vm-apache-migranube*", (se ha utilizado para el nombre de la plantilla "*tmplt-vm-apache-migranube*", núcleo compartido con 1 CPU, 614 MB de RAM y almacenamiento HDD de 10 GB):

![Crear plantilla de instancias 1](./parte3/img/crear_plantilla_instancias_01.png)

![Crear plantilla de instancias 2](./parte3/img/crear_plantilla_instancias_02.png)

![Crear plantilla de instancias 3](./parte3/img/crear_plantilla_instancias_03.png)

Esperar a que la plantilla de instancias de máquinas virtuales se haya creado:

![Crear plantilla de instancias 4](./parte3/img/crear_plantilla_instancias_04.png)

Crear un grupo de instancias de máquinas virtuales basadas en la plantilla "*tmplt-vm-apache-migranube*", (se ha utilizado para el nombre del grupo de instancias "*grp-autoscaling-vm-apache-migranube*":

![Crear grupo de instancias 1](./parte3/img/crear_grupo_instancias_01.png)

![Crear grupo de instancias 2](./parte3/img/crear_grupo_instancias_02.png)

Esperar a que el grupo de instancias se haya creado:

![Crear grupo de instancias 3](./parte3/img/crear_grupo_instancias_03.png)

Ver el grupo de instancias creado:

![Ver grupo de instancias](./parte3/img/ver_grupo_instancias_01.png)

Visitar la página expuesta con el grupo de instancias para comprobar que el servidor web está levantado ("*http://ip_externa/*", no es "*https*"):

![Visitar página web grupo de instancias 1](./parte3/img/visitar_pagina_web_grupo_instancias_01.png)

![Visitar página web grupo de instancias 2](./parte3/img/visitar_pagina_web_grupo_instancias_02.png)

Supervisar el grupo de instancias para ver el autoescalado en reposo:

![Supervisar grupo de instancias 1](./parte3/img/supervisar_grupo_instancias_01.png)

Crear una máquina virtual de test para atacar el grupo de instancias (se ha utilizado para el nombre de la instancia "*test-autoscaling-vm-apache-migranube*", núcleo compartido con 1 CPU, 614 MB de RAM y almacenamiento HDD de 10 GB):

![Crear instancia de máquina virtual test 1](./parte3/img/crear_instancia_maquina_virtual_test_01.png)

![Crear instancia de máquina virtual test 2](./parte3/img/crear_instancia_maquina_virtual_test_02.png)

Esperar a que la máquina virtual esté disponible:

![Crear instancia de máquina virtual test 3](./parte3/img/crear_instancia_maquina_virtual_test_03.png)

Conectar con la máquina virtual por SSH:

![Conectar SSH test 1](./parte3/img/conectar_ssh_test_01.png)

![Conectar SSH test 2](./parte3/img/conectar_ssh_test_02.png)

Actualizar los paquetes disponibles del sistema operativo:

```bash
sudo apt-get update
```

![Actualizar paquetes test](./parte3/img/actualizar_paquetes_test.png)

Instalar la herramienta de pruebas de carga "*Siege*":

```bash
sudo apt-get install siege -y
```

![Instalar Siege test](./parte3/img/instalar_siege_test.png)

Ejecutar la prueba de carga en la máquina "*test-autoscaling-vm-apache-migranube*" contra el grupo de instancias de máquinas virtuales:

```bash
siege -c150 http://35.238.214.225/
```

![Ejecutar Siege test](./parte3/img/ejecutar_siege_test.png)

Supervisar el grupo de instancias para ver el autoescalado durante la prueba de carga:

![Supervisar grupo de instancias 2](./parte3/img/supervisar_grupo_instancias_02.png)

Detener la prueba de carga en la máquina "*test-autoscaling-vm-apache-migranube*" pulsando "*Ctrl+C*":

![Detener Siege test](./parte3/img/detener_siege_test.png)

Terminar la sesión SSH:

```bash
exit
```

Detener la máquina virtual:

![Detener máquina virtual test 1](./parte3/img/detener_maquina_virtual_test_01.png)

![Detener máquina virtual test 2](./parte3/img/detener_maquina_virtual_test_02.png)

Supervisar el grupo de instancias para ver el autoescalado después de la prueba de carga (el período de estabilización fue de 10 minutos):

![Supervisar grupo de instancias 3](./parte3/img/supervisar_grupo_instancias_03.png)

## Parte 4

Activar "*Google Cloud Shell*":

![Activar Cloud Shell 1](./parte4/img/activar_cloud_shell_01.png)

![Activar Cloud Shell 2](./parte4/img/activar_cloud_shell_02.png)

Clonar el repositorio de la aplicación:

```bash
git clone https://github.com/GoogleCloudPlatform/python-docs-samples.git
```

![Clonar repositorio](./parte4/img/clonar_repositorio.png)

Ir al directorio de la aplicación:

```bash
cd ./python-docs-samples/appengine/standard/cloudsql/
```

![Directorio aplicación](./parte4/img/ir_directorio_aplicacion.png)

Modificar el archivo [app.yaml](./parte4/app.yaml), estableciendo los valores establecidos en la parte 2:

* CLOUDSQL_CONNECTION_NAME: practica-kc-03-migranube:europe-west1:mysql-migranube
* CLOUDSQL_PASSWORD: passwordmysqlmigranube01

```bash
nano ./app.yaml
```

![Modificar app.yaml 1](./parte4/img/modificar_app_yaml_01.png)

Desplegar la aplicación:

```bash
gcloud app deploy
```

![Desplegar aplicación 1](./parte4/img/desplegar_aplicacion_01.png)

Obtener la dirección de la aplicación desplegada, y visitar la dirección obtenida:

```bash
gcloud app browse
```

![Acceder aplicación 1](./parte4/img/acceder_aplicacion_01.png)

![Acceder aplicación 2](./parte4/img/acceder_aplicacion_02.png)

Modificar el archivo [app.yaml](./parte4/app_2.yaml), estableciendo como nombre del servicio "*practica*":

* service: practica

```bash
nano ./app.yaml
```

![Modificar app.yaml 2](./parte4/img/modificar_app_yaml_02.png)

Desplegar la aplicación con nombre de servicio "*practica*" y versión "version-1-0-0":

```bash
gcloud app deploy -v version-1-0-0
```

![Desplegar aplicación 2](./parte4/img/desplegar_aplicacion_02.png)

Obtener la dirección de la aplicación desplegada, y visitar la dirección obtenida:

```bash
gcloud app browse -s practica
```

![Acceder aplicación 3](./parte4/img/acceder_aplicacion_03.png)

![Acceder aplicación 4](./parte4/img/acceder_aplicacion_04.png)

Desplegar la aplicación con nombre "*practica*" y versión "version-2-0-0":

```bash
gcloud app deploy -v version-2-0-0
```

![Desplegar aplicación 3](./parte4/img/desplegar_aplicacion_03.png)

Acceder a la sección de "*Versiones*" de "*App Engine*", seleccionar el servicio "*practica*", y marcar las dos versiones del servicio:

![Acceder sección versiones](./parte4/img/acceder_seccion_versiones.png)

Hacer clic en "*Distribuir Tráfico*":

![Distribuir tráfico 1](./parte4/img/distribuir_trafico_01.png)

Establecer la distribución aleatoria al 50% entre las dos versiones, y guardar la configuración:

![Distribuir tráfico 2](./parte4/img/distribuir_trafico_02.png)

Las dos versiones del servicio "*practica*" tienen el tráfico distribuido al 50% entre las dos versiones de la aplicación:

![Distribuir tráfico 3](./parte4/img/distribuir_trafico_03.png)
