# TPVMCA
Trabajo computacion aplicada UP

## Pasos:
# 1
### Cambio de contraseña:
- inicio por primera vez, y en el menu grub presiono `e`

- navego hasta
```
    linux    /boot/vmlinuz-... ... ro quiet
```
- agrego al final
```sh
init=/bin/bash
```
- guardo con `f10`
  
- monto el filesystem /
```sh
mount -o remount,rw /
```
- modifico la contraseña de root
```sh 
passwd root
```
- reinicio equipo
```sh
exec /sbin/init
```
- me logueo con usuario root y la contraseña nueva


### Cambio de hostname:
- Edito hostname
```sh
hostanamectl --static set-hostname TPServer
```
- Reinicio para aplicar cambios



# 2



### Upgrade de version
- Para actualizar 11->12
```sh
vim /etc/apt/sources.list
```
 - Reemplazo repos del archivo por:
  ```
  deb http://deb.debian.org/debian bookworm main contrib non-free-firmware
  deb http://security.debian.org/ bookworm-security main contrib non-free-firmware
  deb http://deb.debian.org/debian bookworm-updates main contrib non-free-firmware
  ```
- Actualizo repos
```sh
apt update
```
- Hago upgrade
```sh
apt full-update
```
- Reinicio para aplicar cambios
    
### Instalar ssh:
```sh
apt-get update
apt-get install openssh-server
```
- Configuro claves
   - Extarer archivo comprimido:
    ```sh
    gunzip Material_Adicional_TPVMCA.tar.gz
    ```
    ```sh
    tar --extract -f Material_Adicional_TPVMCA.tar
    ```
   - Creacion y copia de claves publica/privada
    ```sh
    cat Material_Adicional_TPVMCA/clave_publica.pub >> /root/.ssh/authorized_keys
    ```

### Instalar e iniciar apache
```sh
apt install apache2 php libapache2-mod-php
```
```sh
systemctl enable apache2
```
```sh
systemctl start apache2
```
 - Edito configuracion de sitio
```sh
vim /etc/apache2/sites-available/000-default.conf
```
- Cambio la configuracion del archivo:
```
    DocumentRoot /root/Material_Adicional_TPVMCA

    <Directory /root/Material_Adicional_TPVMCA>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
```
 - Edito configuracion de apache
```sh
vim /etc/apache2/apache2.conf
```
- Agrego a la configuracion:
```sh
<Directory /root/Material_Adicional_TPVMCA>
     Options Indexes FollowSymLinks
     AllowOverride None
     Require all granted
</Directory>
```
```sh
systemctl restart apache2
```
- Como prueba cambiamos permisos de /root
```sh
chmod 755 /root/
```

### Base de datos:

```sh
apt install mariadb-server php-mysql
```
```sh
mysql -u root < Material_Adicional_TPVMCA/db.sql
```
- Ya podemos abrir la web!


# 3




# 4



### Agergar disco de 10GB y crear particiones:
- Creo disco en virtualizador y lo agrego
    
- En la vm reviso el nombre del disco
```
lsblk
```
- Configuro las particiones:
```
        fdisk /dev/sdb
        n   (nueva partición)
        p
        1
        ENTER
        +3G

        n
        p
        2
        ENTER
        +6G

        t   (tipo)
        1
        83

        t
        2
        83

        w   (guardar)
```     
- verifico con lsblk

- Formatear
```sh
mkfs.ext4 /dev/sdb1
```
```sh
mkfs.ext4 /dev/sdb2
```
- Crear directorios y montar
```sh
mkdir -p /www_dir /backup_dir
```
```sh
mount /dev/sdb1 /www_dir
```
```sh
mount /dev/sdb2 /backup_dir
```
- Mover archivos web
```sh
cp /root/index.php /root/logo.png /www_dir/
```
- Actualizar Apache (000-default.conf)
    - Edito configuracion de sitio
    ```sh
    vim /etc/apache2/sites-available/000-default.conf
    ```
    - Cambio la configuracion del archivo:
    ```
        DocumentRoot /www_dir
    
        <Directory /www_dir>
            Options Indexes FollowSymLinks
            AllowOverride All
            Require all granted
        </Directory>
    ```
     - Edito configuracion de apache
    ```sh
    vim /etc/apache2/apache2.conf
    ```
    - Agrego a la configuracion:
    ```sh
    <Directory /www_dir>
         Options Indexes FollowSymLinks
         AllowOverride None
         Require all granted
    </Directory>
    ```
    ```sh
    systemctl restart apache2
    ```
- Modifico permisos de /root
```sh
chmod 700 /root
```
- fstab (automontaje)
```sh
echo "/dev/sdb1  /www_dir    ext4  defaults  0 2" >> /etc/fstab
```
```sh
echo "/dev/sdb2  /backup_dir ext4  defaults  0 2" >> /etc/fstab
```
- Guardar tabla de particiones
```sh
cat /proc/partitions > /opt/particion
```


# 5 
### Script de back-up
- Crear directorio
```sh
sudo mkdir -p /opt/scripts
```
```
sudo vim /opt/scripts/backup_full.sh
```
    ```
    #!/bin/bash
    
    # backup_full.sh
    
    show_help() {
        echo "Uso: $0 <origen> <destino>"
        echo
        echo "Ejemplo:"
        echo "  $0 /var/log /backup_dir"
        echo
        echo "Opciones:"
        echo "  -help    Muestra esta ayuda"
    }
    
    # Ayuda
    if [[ "$1" == "-help" ]]; then
        show_help
        exit 0
    fi
    
    # Validación de parámetros
    if [[ $# -ne 2 ]]; then
        echo "Error: cantidad incorrecta de argumentos."
        show_help
        exit 1
    fi
    
    ORIGEN="$1"
    DESTINO="$2"
    
    # Verificar que el directorio origen exista
    if [[ ! -d "$ORIGEN" ]]; then
        echo "Error: el directorio origen no existe."
        exit 1
    fi
    
    # Verificar que el directorio destino exista
    if [[ ! -d "$DESTINO" ]]; then
        echo "Error: el directorio destino no existe."
        exit 1
    fi
    
    # Verificar que los sistemas de archivos estén montados
    if ! mountpoint -q "$ORIGEN"; then
            echo "Advertencia: $ORIGEN no es un punto de montaje."
            exit 1
    fi
    
    if ! mountpoint -q "$DESTINO"; then
        echo "Error: el sistema de archivos destino no está montado."
        exit 1
    fi
    
    # Fecha en formato ANSI YYYYMMDD
    FECHA=$(date +%Y%m%d)
    
    # Obtener nombre base del directorio
    DIR_NAME=$(basename "$ORIGEN")
    
    # Nombre del archivo backup
    ARCHIVO_BACKUP="${DIR_NAME}_bkp_${FECHA}.tar.gz"
    
    # Crear backup
    tar -czf "${DESTINO}/${ARCHIVO_BACKUP}" "$ORIGEN"
    
    # Verificar resultado
    if [[ $? -eq 0 ]]; then
        echo "Backup realizado correctamente:"
        echo "${DESTINO}/${ARCHIVO_BACKUP}"
    else
        echo "Error al generar el backup."
        exit 1
    fi
    
    ```
Asignar permisos:
```
sudo chmod 755 /opt/scripts/backup_full.sh
```



