# TPVMCA
Trabajo computacion aplicada UP

## Pasos:

### Cambio de contraseña:
- inicio por primera vez
```
e
```
- navego hasta
```
x
```
- agrego al final
```sh
init=/bin/bash
```
- guardo con
```
f10
```
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

### Upgrade de version
- Para actualizar 11->12
```
vim /etc/apt/sources.list
```
 - Reemplazo repos del archivo por:
  ```
  deb http://deb.debian.org/debian bookworm main contrib non-free-firmware
  deb http://security.debian.org/ bookworm-security main contrib non-free-firmware
  deb http://deb.debian.org/debian bookworm-updates main contrib non-free-firmware
  ```
- Actualizo repos
```
apt update
```
- Hago upgrade
```  
apt full-update
```
    
### Instalar ssh:
```
apt-get install openssh-server
```
- Configuro claves
   - Extarer archivo comprimido:
    ```
    gunzip Material_Adicional_TPVMCA.tar.gz
    ```
    ```
    tar --extract -f Material_Adicional_TPVMCA.tar
    ```
   - Creacion y copia de claves publica/privada
    ```
    cat Material_Adicional_TPVMCA/clave_publica.pub >> /root/.ssh/authorized_keys
    ```

### Instalar e iniciar apache
```
apt install apache2 php
```
```
systemctl enable apache2
```
```
systemctl start apache2
```
 - Edito config
```
vim /etc/apache2/sites-available/000-default.conf
```
```
    DocumentRoot /root

    <Directory /root>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
``` 

### Base de datos:




    
    
    
    
    
    
Agergar disco de 10GB y crear particiones:
    Creo disco en virtualizador y lo agrego
    
    En la vm reviso el nombre del disco
        lsblk

    Configuro las particiones:
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
        
    verifico con lsblk

