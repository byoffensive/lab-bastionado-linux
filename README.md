<div align="center">
  <h1>🛡️ Hardening y Bastionado de Servidores Linux</h1>
  <p><i>Configuración integral de seguridad, control de acceso y auditoría sobre Ubuntu Server</i></p>
</div>

## 📝 Descripción del Proyecto
Este repositorio documenta el proceso completo de securización de un servidor **Ubuntu Server 22.04 LTS**. El objetivo de este laboratorio es reducir drásticamente la superficie de ataque, cifrar las comunicaciones y fortificar los accesos mediante políticas estrictas, superando con éxito auditorías de red.

## 🛠️ Stack Tecnológico
<p>
  <img src="https://img.shields.io/badge/Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white" alt="Ubuntu" />
  <img src="https://img.shields.io/badge/Apache_HTTP_Server-D22128?style=for-the-badge&logo=apache&logoColor=white" alt="Apache" />
  <img src="https://img.shields.io/badge/OpenSSH-231F20?style=for-the-badge&logo=OpenSSH&logoColor=white" alt="SSH" />
  <img src="https://img.shields.io/badge/Nmap-2B445A?style=for-the-badge&logo=nmap&logoColor=white" alt="Nmap" />
</p>

---

## 🔒 Fases del Despliegue y Securización

### 1. Cifrado Web y VirtualHosts (Apache + SSL)
Para garantizar la confidencialidad de los datos en tránsito, se configuró Apache2 para servir contenido exclusivamente a través del puerto seguro 443.
* **Módulo SSL:** Activación mediante `a2enmod ssl`.
* **Certificados:** Generación de un certificado autofirmado (clave pública y privada) asignado al dominio `ejemplo.prueba`.
* **VirtualHost:** Redirección estricta y configuración del archivo de sitio seguro.

**Fragmento de la configuración aplicada (`ejemplo.prueba-ssl.conf`):**
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

### 2. Hardening de Acceso Remoto (SSH)
La configuración por defecto de SSH supone un riesgo crítico frente a ataques de fuerza bruta y diccionarios. Se editó el archivo `/etc/ssh/sshd_config`:
* **Cambio de Puerto:** Traslado del servicio del puerto 22 al **2222**.
* **Bloqueo de Root:** Deshabilitación del inicio de sesión directo para el superusuario (`PermitRootLogin no`).
* **Troubleshooting:** Resolución de conflictos con `systemd sockets` (activación por socket introducida en versiones recientes de Ubuntu) que mantenían el puerto 22 abierto, forzando la deshabilitación del socket y recarga del demonio principal.

> <img width="1890" height="1178" alt="image" src="https://github.com/user-attachments/assets/ba77495f-1707-4658-a2b5-c418d432fe29" />
 

### 3. Políticas de Contraseñas Estrictas (PAM)
Se implementó el módulo Pluggable Authentication Modules (`libpam-pwquality`) para erradicar el uso de credenciales débiles, forzando matemáticamente una alta entropía:
* Requisito de **10 caracteres mínimos** (`minlen=10`).
* Obligatoriedad de combinación de mayúsculas, minúsculas y números (`ucredit=-1`, `lcredit=-1`, `dcredit=-1`).
* Bloqueo temporal del cambio tras **3 intentos fallidos** (`retry=3`).

> <img width="919" height="524" alt="image" src="https://github.com/user-attachments/assets/e77202bf-e6bf-4c26-9145-b7a5523410ec" />

> <img width="964" height="472" alt="image" src="https://github.com/user-attachments/assets/ea0c58ba-2708-4deb-b926-deaebb6aff40" />

> <img width="976" height="465" alt="image" src="https://github.com/user-attachments/assets/1416ed1b-41c8-4464-81b9-b310eed9518f" />


### 4. Auditoría de Superficie de Ataque (Nmap)
Para validar la eficacia del bastionado, se asumió el rol de auditor lanzando escaneos desde una máquina cliente externa en la red local:
* **TCP SYN Scan & Fingerprinting:** Escaneo sigiloso (`-sS`), detección de versiones (`-sV`) y estimación del sistema operativo (`-O`).
* **Escaneo Completo:** Auditoría exhaustiva a los 65535 puertos lógicos (`-p-`).
* **Mitigación:** Verificación del cierre de puertos innecesarios (como FTP plano en puerto 21) e implementación de reglas en el firewall perimetral `UFW`.

> <img width="852" height="664" alt="image" src="https://github.com/user-attachments/assets/373da8ed-b67d-4a3b-9adb-962d80cbdba7" />


---
*Este proyecto forma parte de mi portfolio técnico. Puedes ver más casos de estudio en mi [perfil principal de GitHub](https://github.com/byoffensive).*
