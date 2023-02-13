
# Práctica 4.4: deployment of an architecture EFS-EC2-MultiAZ in the CLoud (AWS)


## Adrián Cordovero Crespo


### ÍNDICE
- Creación grupos de seguridad
- Creación de instancias
- Creación del EFS
- Cambios de los grupos de seguridad
- Conexión a las instancias
- Comandos
- Creación y conexión al cluster


1. Comenzamos creando los siguientes 3 grupos de seguridad.
   - SGWEB (Puerto 80 desde IP del Balanceador, 22 desde Todas)
 
   - SGEFS (Puerto 2049 desde IP de Servidores WEBs, 22 desde Todas)

   - Load Balancer (Puerto 80 y puerto 22 desde Todas)


2. Crearemos los 2 servidores WEB:
La configuración será la siguiente:
   - Ubuntu 22.04
   - t2.micro
   - Par claves para entrar por SSH
   - VPCs Distintas para mayor estabilidad ante fallos.
   - Grupo de Seguridad WEB
   - Detalles Avanzados > Datos de Usuario
  
	```#!/bin/bash
	 apt update
	 apt install apache2 -y
	 apt install nfs-utils
	 systemctl reboot
	 ```

1. Por ultimo crearemos la EC2 del balanceador con la misma configuración anterior con los siguientes cambios:
   - Grupo de Seguridad Load Balancer
   - Asignamos IP Elástica

2. Creamos el Sistema de Archivos EFS:
   - Guardamos el nombre de DNS
   - Red > Administrar > Grupos de Seguridad > Cambiamos "Default" por "FS"

3. Entramos en cada uno de los nodos del cluster mediante SSH:

   - Creamos la carpteta nfs-mount en la dirección /var/www/html/
   - Sincronizamos el contenido del contenedor NFS con esta carpeta
     - ```sudo mount -t nfs -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport fs-03689917b85f6eef4.efs.us-east-1.amazonaws.com:/ /var/www/html/nfs-mount``` Sustituyendo la DNS con la respectiva del EFS.
   - Modificamos el archivo /etc/apache2/sites-enabled/000-default.conf
     - Cambiamos el Document Root, agregando el directorio /nfs-mount al final de la línea para evitar que nos salga en la barra de navegación información innecesaria.

4. Entramos al balanceador de carga mediante SSH:
   - Instalamos los siguientes plugins de apache
     - ```sudo a2enmod proxy proxy_http proxy_balancer lbmethod_bytraffic lbmethor_byrequests```
   - Modificamos el fichero /etc/apache2/sites-enabled/000-default.conf añadiendo la siguiente configuración. En IP deberemos poner la IP de nuestras máquinas independientes.
  
```<Location /balancer-manager>
 			SetHandler balancer-manager
   	</Location>
   	ProxyPass /balancer-manager !
   	<Proxy balancer://mycluster>
   			BalancerMember http://IP
   			BalancerMember http://IP
   			ProxySet lbmethod=byrequests
   	</Proxy>
   	ProxyPass / balancer://mycluster/
   	ProxyPassReverse / balancer://mycluster/
```


Bibliografía: https://github.com/santos-pardos/Hands-On-Lab-in-AWS/blob/main/Storage/EFS/requirements.md
https://www.youtube.com/watch?v=CCfuASU73Jo

