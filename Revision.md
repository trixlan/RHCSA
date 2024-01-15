## Network
```shell
Para actualizar una red se debe modificar la conexion
# nmcli connection modify enp7s0 ipv4.addresses 172.24.12.10/24
Eliminar una caracteristica
# nmcli connection modify enp7s0 -ipv4.addresses 172.24.12.10/24
Agregar una caracteristica adicional
# nmcli connection modify enp7s0 +ipv4.addresses 172.24.12.10/24
Despues de hacer los cambios se recomienda reiniciar la conexion
# nmcli connection down enp7s0: nmcli connection up enp7s0
```
### Extra Network
```shell
Directorio
# /etc/sysconfig/network-scripts/ifcfg-enp1s0
Muestra las conexiones
# nmcli connection show
Mustra las interfases
# nmcli device status
Connectar una interfase
# nmcli device connect enp7s0 
Despues de aplicar los cambios
# sudo systemctl restart NetworkManager.service
# nmcli connection reload
```

## NTP
```shell
# timedatectl
Para activar la sincronizacion con el servidor y iniciar chronyd.service
# timedatectl set-ntp true
Muestra la informacion del servidor al que esta sincronizado
# chronyc tracking
Mostrar las fuentes
# chrony sources -v
El archivo de configuracion 
# vi /etc/chrony.conf
pool - es un conjunto de servidores
server - si es un solo servidor
comentamos los pool y agregamos
server rhel8master.labrhel.com iburst
Despues de modificar el archivo se debe reiniciar el servicio
# systemctl restart chronyd.service
El comando ya debe mostrar la url del servidor
# chronyc tracking
Ya debe mostrar el servidor que configuramos con un * 
# chronyc sources -v
Validamos las fechas
# timedatectl
```

## Firewall
```shell
# ls /etc/firewalld
Muestra solo las zonas activas
# firewall-cmd --list-all
# firewall-cmd --get-default-zone
Cambiar la zona por defecto
# firewall-cmd --set-default-zone --zone=dmz
Agregamos un servicio y lo hacemos permanente
# firewall-cmd --add-service=samba --permanent
Agregamos un puerto
# firewall-cmd --add-port=8080/tcp --permanent

Despues de cada cambio
# firewall-cmd --reload
```

## Usuarios
```shell
Analisar los datos del usuario
# id ger
uid=1001(Jerry) gid=1001(Jerry) groups=1001(Jerry),7878(conta)
uid = id del usuario
gid = grupo primario
groups = grupos secundarios

Crear un nuevo usuario
# useradd ger -g (grupo primario) -G (grupo secundario)
# usermod ger -aG grupo (agregar grupo secundario)

Restringir el acceso de un usuario
# usermod -L dev
# usermod -L -e 2023-11-13 dev
```

## Contenedores
```shell
Contenedores como servicio - Usuario
# mkdir /home/ger/.config/systemd/user
Este comando crear un archivo container-web.service
# podman generate systemd --name web --files --new
--name hace referencia al nombre del pod
# cp container-web.service /home/ger/.config/systemd/user
# systemctl --user daemon-reload
# systemctl --user enable container-web
# systemctl --user start container-web
# systemctl --user status container-web
# loginctl enable-linger ger
o
# loginctl enable-linger $USER 

Contenedores como servicio - Root
Este comando crear un archivo container-web.service
# podman generate systemd --name web --files --new
--name hace referencia al nombre del pod
# cp container-web.service /etc/systemd/system
# systemctl daemon-reload
# systemctl enable container-web
# systemctl start container-web
# systemctl status container-web
```

## Busquedas
```shell
Por nombre del archivo 
# find / -name Revision
Por nombre del archivo May y min
# find / -iname revision
Por usuario
# find / -user 1000/gercha
Por tamaño mayor que 400M y menor que 500M
# find / -size +400M -and -size -500M
Por tipo archivo, menor que 800M y mayor que 400M y ejecutamos un comando adicional en cada archivo encontrado
# find ~/Downloads/ -type f -size -800M -and -size +400M -exec ls -lah {} \;

```

## Busqueda de Texto dentro de los archivos
```shell
# grep -n Texto Archivo
-n Muestra el numero de la linea donde encontro el texto
-l Imprime el archivo donde encontro el Texto
-i Ignora mayusculas y minusculas
-r Recursivo en caso de directorios 
```

## Discos para Swap
```shell
# free -h
Creamos un disco swap, ya sea fisico o logico
# parted /dev/vdc mklabel gpt
# gdisk /dev/vdc
n
8200
w
# mkswap /dev/vdc1
o
# pvcreate /dev/vdc1
# vgcreate vg_linux /dev/vdc1
# lvcreate vg_linux -n lv_linux -L 200M
# mkswap /dev/vdc1
# lsblk -fp

# echo "/dev/mapper/vg_linux-lv_linux swap swap 0 0" >> /etc/fstab
# swapon /dev/mapper/vg_linux-lv_linux

Para reiniciar el daemon 
# systemctl daemon-reload
Para guardar los cambios
# udevadm settle
Si creamos una nueva particion y no aparece 
# partprobe -s
Para verificar 
# lsblk -fp 
```

## Extender un volumen
```shell
Para agregar 80M
lvextend /dev/vg_data/lv_data1 -L +80M -r -t
Para definir a 80M
lvextend /dev/vg_data/lv_data1 -L 80M -r -t
Para agregar 80M con extend
lvextend /dev/vg_data/lv_data1 -l +20 -r -t

Para mostrar el PE Physical Extend
# vgdisplay vg_data
Una vez que conocemos el PE size lo podemos multiplicar o dividir para que nos de la cantidad de megas que necesitamos
# echo "4*49" | bc  = 196M
# echo "scale=3; 256/4" | bc  = 196M
```

## Compresion de almacenamiento VDO
```shell
Instalar los paquetes para usar
# yum install vdo kmod-kvdo 
# vdo create -n vdo01 --device /dev/vdd --vdoLogicalSize 20G
# vdo status
# vdo list
# vdo stop -n vdo01
# vdo start -n vdo01
Le damos un formato 
# mkfs.xfs -K /dev/mapper/vdo01
# udevadm settle
# lsblk -fp
# lsblk --output=UUID /dev/mapper/cdo01
# echo "UUID=5aff20e0-67a6-47b3-ac32-dce034c26360 /dir_vdo xfs defaults,x-systemd.requires=vdo.service 0 0" >> /etc/fstab
# systemctl daemon-reload
# mount -a
Para ver estadisticas se usa
# vdostats --human-readable
# vdo status | grep -i compres
# vdo status ! grep -i dedupli
```

## Man
```shell
Para ver todos los documentos
# man -k find
Para buscar una palabra dentro de man usar / y n para las siguientes busquedas
```

## Nmap
```shell
Verificar si un puerto esta abierto
# nmap rhel8master.labrhel.com -p 123 
```
## Tuberias
```shell
Permite pasar el resultado de un comando a otro comando
# |
Entrada estandar, como si escribieras
# stdin 
Salida estandar
# stdout
Salida de error
# stderr
Ejemplos
# ls -l | grep doc

Permite ver la salida en la terminal y guardarlo en un archivo
# ls -l | tee listado
Permite pasar la respuesta al siguiente comando
# find ./ -name "thumbs.db" | xargs rm
```

## Puertos
```shell
Revisar los puertos abiertos
# netstat -an | grep tcp | grep LISTEN
-a muestra todos los sockets conectados
-n muestra el numero del puerto
```

## Compresion de archivos
```shell
Comprimir con tar
# tar cfv archivo.tar file1 file2
-c create
-f file
-v verbose
-j bzip2 - archivo.tar.bz2
# tar cvfj file1 file2
# tar xvfj archivo.tar.bz2
-z gzip - archivo.tar.gz
# tar cvfz file1 file2
# tar xvfz archivo.tar.gz
Descomprimir con tar
# tar xvf archivo.tar
-x extract
Mostrar el contenido del tar
# tar tvf archivo.tar

Comprimir gz file.gz
# gzip file
Descomprimir gz
# gunzip file.gz

Comprimit bz file.bz2
# bzip2 file
Descomprimit
# bunzip2 file.bz2 

Comprimir con zip
# zip archivo.zip file1 file2
Descomprimir con zip
# unzip archivo.zip
```