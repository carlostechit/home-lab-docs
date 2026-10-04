# Runbook & SOP: Despliegue de Ubuntu Server y Fundamentos Linux (Fase 1)

Este documento detalla el procedimiento operativo estandarizado (SOP) y el runbook técnico correspondiente a la **Fase 1** del homelab: aprovisionamiento, despliegue desatendido/manual con particionado tradicional extendido (GPT/UEFI), configuración de red con direccionamiento estático/DHCP por reserva y validación de la jerarquía de ficheros (FHS) sobre **Proxmox VE**.

---

## 1. Descarga e Importación Automatizada de la ISO en Proxmox
Para agilizar el proceso y garantizar la integridad de los binarios, se aprovecha el gestor de descargas nativo del hipervisor Proxmox VE.

1. **Acceso al Almacenamiento:** Acceso a la interfaz web de Proxmox VE (`https://<tu-ip>:8006`), navegación al almacenamiento local (`local`) y selección de **ISO Images**.
2. **Descarga desde URL:** Selección de **Download from URL**, introducción del enlace oficial de la ISO de Ubuntu Server LTS y nombre descriptivo del archivo.

![Descarga desde URL de la ISO](../1-Fundamentos/assets/img/Captura%20de%20pantalla%202026-10-04%20132652.png)

3. **Verificación de Integridad (SHA-256):** Validación automatizada del hash criptográfico por parte del hipervisor durante el proceso de transferencia para descartar cualquier corrupción.

![Verificación de la Integridad](../1-Fundamentos/assets/img/Captura%20de%20pantalla%202026-10-04%20132842.png)

![ISO descargada en almacenamiento local](../1-Fundamentos/assets/img/Captura%20de%20pantalla%202026-10-04%20132859.png)
---

## 2. Aprovisionamiento de la VM en Proxmox (UEFI)
Configuración de la máquina virtual con tecnologías de virtualización modernas y soporte para UEFI (*OVMF*).

* **Resumen de parámetros en Proxmox:**
  * **`bios: ovmf`** (Arranque UEFI moderno basado en GPT).
  * **`cores: 2`** y **`cpu: host`** (Aprovechamiento óptimo de la virtualización anidada).
  * **`memory: 4096 MB`** (4 GB de RAM estática).
  * **`scsi0: local-lvm:32,iothread=on`** (Disco virtual de 32 GB con controlador VirtIO SCSI e hilos de E/S activados).
  * **`efidisk0`** (Mapeo de la partición de sistema UEFI).
  * **`net0`** (Interfaz de red VirtIO en el bridge `vmbr0`).

![Confirmación de Creación de VM en Proxmox](image_5f2d5a.png)

---

## 3. Instalación de Ubuntu Server y Particionado Tradicional Extendido (GPT / UEFI)

En el asistente del instalador se selecciona la opción de diseño de almacenamiento personalizado **`Custom storage layout`** para aplicar un esquema estrictamente segmentado en lugar de LVM por defecto.

![Configuración Personalizada de Almacenamiento](../1-Fundamentos/assets/img/Captura%20de%20pantalla%202026-10-04%20134830.png)

### Esquema de Particionamiento Estándar sobre 32 GB:
1. **`/boot/efi`**: `1.049G` (`vfat` / `fat32`, partición ESP para el gestor de arranque UEFI).
2. **`/` (Raíz)**: `10.000G` (`ext4`, binarios y directorios del sistema operativo).
3. **`/boot`**: `1.000G` (`ext4`, núcleos del kernel e imágenes `initrd`).
4. **`swap`**: `4.000G` (Espacio de intercambio alineado con la RAM).
5. **`/var`**: `8.000G` (`ext4`, aislamiento de logs y bases de datos).
6. **`/home`**: `~7.948G` (Resto del espacio disponible para directorios de usuarios).

![Resumen del Sistema de Archivos Configurado](../1-Fundamentos/assets/img/Captura%20de%20pantalla%202026-10-04%20135744.png)

Una vez completada la asignación de puntos de montaje, se inicia el proceso de instalación desatendida mediante **curtin**, formateando las particiones, configurando los UUIDs e instalando los paquetes de arranque (*grub-efi-amd64-signed*, *shim-signed*).

![Proceso de Instalación con Curtin](../1-Fundamentos/assets/img/Captura%20de%20pantalla%202026-10-04%20135940.png)

![Instalación completa](../1-Fundamentos/assets/img/Captura%20de%20pantalla%202026-10-04%20140100.png)

---

## 4. Acceso Remoto por SSH y Configuración de Red (Netplan & DHCP Estático)

Tras el primer reinicio, el sistema operativo arranca correctamente con el kernel de Linux y asigna una IP temporal por DHCP que posteriormente se consolida mediante reserva estática en el router de laboratorio.

![Pantalla de Login Inicial en el Servidor](../1-Fundamentos/assets/img/Captura%20de%20pantalla%202026-10-04%20140432.png)

![Pantalla de acceso desde wsl2 por ssh](../1-Fundamentos/assets/img/Captura%20de%20pantalla%202026-10-05%20005623.png)

![Reserva de IP en el router](../1-Fundamentos/assets/img/Captura%20de%20pantalla%202026-10-05%20011343.png)


### Validación de Red e Interfaces:
* **Asignación IP:** El servidor adopta la dirección IP reservada `192.168.0.170` en la interfaz `ens18`.
* **Pruebas de Conectividad:** Comprobación de latencias con la pasarela predeterminada (`192.168.0.1`), resolución de dominios mediante `nslookup` y salida a internet validada (`ping` y `traceroute`).

![Ejecutando comando sudo netplan try y haciendo ping](../1-Fundamentos/assets/img/Captura%20de%20pantalla%202026-10-04%20141130.png)

![Pruebas de Red y Direccionamiento IP nslookup](../1-Fundamentos/assets/img/Captura%20de%20pantalla%202026-10-04%20141211.png)

![Pruebas de Red y Direccionamiento IP traceroute](../1-Fundamentos/assets/img/Captura%20de%20pantalla%202026-10-04%20141613.png)

---

## 5. Auditoría de Sistemas, Estructura de Directorios (FHS) y Permisos

Se comprueba la integridad del despliegue mediante comandos de inspección a nivel de bloques, sistemas de ficheros, almacenamiento persistente y control de accesos.

### A. Comprobación de Bloques y Particiones (`lsblk` y `lsblk -f`)
Verificación visual de la correspondencia exacta entre los puntos de montaje, el uso de UUIDs persistentes en `/etc/fstab` y el espacio disponible en disco (`df -h`).

![Inspección de Bloques y UUIDs con lsblk](../1-Fundamentos/assets/img/Captura%20de%20pantalla%202026-10-04%20140517.png)
![Uso de Espacio en Disco con df -h](../1-Fundamentos/assets/img/Captura%20de%20pantalla%202026-10-04%20140605.png)

### B. Persistencia y Configuración del Sistema
* **Fichero Netplan:** Configuración de la interfaz de red gestionada por el servicio de cloud-init/networkd.

![Fichero Netplan](../1-Fundamentos/assets/img/Captura%20de%20pantalla%202026-10-04%20142029.png)

* **Fichero `/etc/fstab`:** Montajes estáticos enlazados de forma robusta por identificador universal único (**UUID**).

![Fichero Netplan](../1-Fundamentos/assets/img/Captura%20de%20pantalla%202026-10-04%20142112.png)

* **Control de Accesos (RBAC):** Verificación del alta del usuario administrador `carlos` integrado en el grupo `sudo` y bloqueo de accesos directos por la cuenta de `root`.

![Grupos del usuario carlos](../1-Fundamentos/assets/img/Captura%20de%20pantalla%202026-10-05%20010735.png)

![Auditoría de Permisos en Directorios Personales y FHS](../1-Fundamentos/assets/img/Captura%20de%20pantalla%202026-10-04%20142434.png)