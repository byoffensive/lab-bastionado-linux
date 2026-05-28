# Hardening y bastionado de servidores Linux

Auditoría, control de acceso y securización de un servidor web corporativo

## Contexto

Parto de un servidor web recién desplegado sobre Ubuntu Server 22.04 LTS con la configuración de fábrica: servicios en texto plano, puertos de administración expuestos y contraseñas débiles. En ese estado es un objetivo fácil para escaneos y ataques automatizados.

El objetivo es bastionar ese servidor —cifrar las comunicaciones, cerrar la superficie de exposición y endurecer el acceso— sin tumbar el servicio web.

## Stack

![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)

![Apache](https://img.shields.io/badge/Apache_HTTP_Server-D22128?style=for-the-badge&logo=apache&logoColor=white)

![OpenSSH](https://img.shields.io/badge/OpenSSH-231F20?style=for-the-badge&logo=openssh&logoColor=white)

![Nmap](https://img.shields.io/badge/Nmap-2B445A?style=for-the-badge&logo=nmap&logoColor=white)

---

## Fases de securización

### 1. Cifrado del tráfico web (Apache + SSL)

El primer punto débil es el tráfico en claro. Deshabilité el puerto 80 y forcé todo el tráfico por HTTPS con el módulo SSL de Apache. Generé un certificado autofirmado y configuré el VirtualHost con restricciones de directorio.

```apache
<VirtualHost *:443>
    ServerName ejemplo.prueba
    DocumentRoot /var/www/ejemplo.prueba
    SSLEngine on
    SSLCertificateFile /etc/ssl/private/ejemplo.prueba
    SSLCertificateKeyFile /etc/ssl/private/ejemplo.prueba
    <Directory /var/www/ejemplo.prueba>
        Options FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

### 2. Hardening de OpenSSH

Con la capa web asegurada, endurecí el acceso por SSH editando /etc/ssh/sshd_config:

- Moví el servicio del puerto 22 al 2222.

- Deshabilité el inicio de sesión directo de root PermitRootLogin no).

- Resolví un conflicto con la activación por socket de systemd, que mantenía el puerto 22 abierto en las versiones recientes de Ubuntu: deshabilité el socket y recargué el demonio principal.

![Estado del servicio SSH escuchando en el puerto 2222](https://github.com/user-attachments/assets/ba77495f-1707-4658-a2b5-c418d432fe29)

### 3. Políticas de contraseñas con PAM

Instalé libpam-pwquality para impedir el uso de credenciales débiles:

- Mínimo de 10 caracteres minlen=10).

- Mayúsculas, minúsculas y números obligatorios ucredit=-1, lcredit=-1, dcredit=-1).

- Tres reintentos por cambio de contraseña retry=3).

![Configuración de libpam-pwquality](https://github.com/user-attachments/assets/e77202bf-e6bf-4c26-9145-b7a5523410ec)

![Rechazo de una contraseña débil](https://github.com/user-attachments/assets/ea0c58ba-2708-4deb-b926-deaebb6aff40)

![Aceptación de una contraseña conforme a la política](https://github.com/user-attachments/assets/1416ed1b-41c8-4464-81b9-b310eed9518f)

### 4. Auditoría con Nmap

Para comprobar el resultado, escaneé el servidor desde una máquina cliente en la misma red local:

- SYN scan con detección de versiones y sistema operativo -sS -sV -O).

- Escaneo de los 65535 puertos -p-).

- Cierre de puertos innecesarios (FTP en claro en el 21) y reglas en el firewall UFW.

![Salida del escaneo Nmap tras el bastionado](https://github.com/user-attachments/assets/373da8ed-b67d-4a3b-9adb-962d80cbdba7)

---

## Resultado

El servidor quedó con la superficie de exposición reducida a lo necesario: sin protocolos en claro, con SSH fuera del puerto estándar y con el acceso protegido por políticas de contraseña. El servicio web sigue operativo en HTTPS.

Parte de mi portfolio técnico. Más casos de estudio en mi [perfil de GitHub](https://github.com/byoffensive).
