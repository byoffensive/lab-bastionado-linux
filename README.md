<div align="center">
  <h1>🛡️ Hardening y Bastionado de Servidores Linux</h1>
  <p><i>Auditoría, control de acceso y securización perimetral de un entorno corporativo</i></p>
</div>

<br>

## 🏢 Contexto del Escenario
Una empresa hipotética despliega un nuevo servidor web corporativo basado en **Ubuntu Server 22.04 LTS**. Al realizar una auditoría preliminar, se detecta que el servidor cuenta con la configuración de fábrica (servicios de texto plano activados, puertos de administración expuestos y políticas de contraseñas débiles), lo que lo hace altamente vulnerable a ataques automatizados y botnets.

**Objetivo:** Transformar este servidor vulnerable en un sitio seguro (bastionado), garantizando la confidencialidad de las comunicaciones y mitigando riesgos de acceso no autorizado, manteniendo la operatividad del servicio web.

## 🛠️ Stack Tecnológico
<p>
  <img src="https://img.shields.io/badge/Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white" alt="Ubuntu" />
  <img src="https://img.shields.io/badge/Apache_HTTP_Server-D22128?style=for-the-badge&logo=apache&logoColor=white" alt="Apache" />
  <img src="https://img.shields.io/badge/OpenSSH-231F20?style=for-the-badge&logo=OpenSSH&logoColor=white" alt="SSH" />
  <img src="https://img.shields.io/badge/Nmap-2B445A?style=for-the-badge&logo=nmap&logoColor=white" alt="Nmap" />
</p>

---

## 🔒 Ejecución Técnica y Fases de Securización

### 1. Garantizando la Confidencialidad: Cifrado Web (Apache + SSL)
El primer vector de ataque a mitigar es la interceptación de tráfico (Man-in-the-Middle). Se procedió a deshabilitar el puerto 80 (HTTP) y forzar que toda la comunicación se realice mediante túneles seguros utilizando el módulo SSL de Apache2. Se generó un certificado autofirmado y se configuró el *VirtualHost* para imponer restricciones de directorio.

<details>
  <summary><b>📸(Código del VirtualHost)</b></summary>
  
  <br>

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
</details>

<br>

### 2. Blindando la Administración: Hardening de OpenSSH
Una vez asegurada la capa web, el siguiente paso crítico fue proteger la puerta de entrada de los administradores. Se intervino el archivo `/etc/ssh/sshd_config` aplicando políticas de mínimo privilegio:
* **Cambio de Puerto:** Traslado del servicio del puerto 22 al **2222**.
* **Bloqueo de Root:** Deshabilitación del inicio de sesión directo para el superusuario (`PermitRootLogin no`).
* **Troubleshooting:** Resolución de conflictos con `systemd sockets` (activación por socket introducida en versiones recientes de Ubuntu) que mantenían el puerto 22 abierto, forzando la deshabilitación del socket y recarga del demonio principal.

<details>
  <summary><b>📸 (Status SSH)</b></summary>
  
  <br>
  <p align="center">
    <img src="https://github.com/user-attachments/assets/ba77495f-1707-4658-a2b5-c418d432fe29" width="800">
  </p>
</details>

<br>

### 3. Políticas de Contraseñas Estrictas (PAM)
Se implementó el módulo Pluggable Authentication Modules (`libpam-pwquality`) para erradicar el uso de credenciales débiles, forzando matemáticamente una alta entropía:
* Requisito de **10 caracteres mínimos** (`minlen=10`).
* Obligatoriedad de combinación de mayúsculas, minúsculas y números (`ucredit=-1`, `lcredit=-1`, `dcredit=-1`).
* Bloqueo temporal del cambio tras **3 intentos fallidos** (`retry=3`).

<details>
  <summary><b>📸 (Políticas PAM)</b></summary>
  
  <br>
  <p align="center">
    <img src="https://github.com/user-attachments/assets/e77202bf-e6bf-4c26-9145-b7a5523410ec" width="800">
    <br><br>
    <img src="https://github.com/user-attachments/assets/ea0c58ba-2708-4deb-b926-deaebb6aff40" width="800">
    <br><br>
    <img src="https://github.com/user-attachments/assets/1416ed1b-41c8-4464-81b9-b310eed9518f" width="800">
  </p>
</details>

<br>

### 4. Auditoría de Superficie de Ataque (Nmap)
Para validar la eficacia del bastionado, se asumió el rol de auditor lanzando escaneos desde una máquina cliente externa en la red local:
* **TCP SYN Scan & Fingerprinting:** Escaneo sigiloso (`-sS`), detección de versiones (`-sV`) y estimación del sistema operativo (`-O`).
* **Escaneo Completo:** Auditoría exhaustiva a los 65535 puertos lógicos (`-p-`).
* **Mitigación:** Verificación del cierre de puertos innecesarios (como FTP plano en puerto 21) e implementación de reglas en el firewall perimetral `UFW`.

<details>
  <summary><b>📸 (Salida de Nmap)</b></summary>
  
  <br>
  <p align="center">
    <img src="https://github.com/user-attachments/assets/373da8ed-b67d-4a3b-9adb-962d80cbdba7" width="800">
  </p>
</details>

---

## 🎯 Impacto y Resultados
Tras la intervención, la superficie de exposición del servidor se redujo a la mínima expresión. Se eliminaron protocolos obsoletos de texto plano, se dificultó el éxito de escaneos automatizados de botnets mediante la ofuscación del puerto SSH, y se garantizó la integridad del acceso local mediante políticas PAM inquebrantables. El servidor ahora cumple con estándares de seguridad corporativos.

<br>

*Este proyecto forma parte de mi portfolio técnico. Puedes ver más casos de estudio en mi [perfil principal de GitHub](https://github.com/byoffensive).*
