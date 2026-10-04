# SOP: Despliegue de Proxmox VE sobre Hyper-V (Nested Virtualization)

## 1. Preparación de Red (Hyper-V)
Para garantizar que el hipervisor Proxmox tenga visibilidad en la red local (LAN) y podamos acceder a su interfaz de gestión, es necesario crear un enlace en Capa 2 con la tarjeta de red física.

1. Abrir **Administrador de Conmutadores Virtuales** en Hyper-V.

![Captura de Adminsitrador de Conmutadores Virtuales. Crear Comutador Virtual](../0-Infraestructura%20Base/assets/img/Captura%20de%20pantalla%202026-10-04%20014135.png)

2. Crear un nuevo conmutador de tipo **Externo**.
3. **Nombre asignado:** `Red Externa`.
4. **Tipo de conexión:** Red externa (seleccionando la tarjeta de red física del equipo anfitrión).
5. Mantener marcada la opción: *"Permitir que el sistema operativo de administración comparta este adaptador de red"*.

![Captura de Adminsitrador de Conmutadores Virtuales. Crear Comutador Virtual](../0-Infraestructura%20Base/assets/img/Captura%20de%20pantalla%202026-10-04%20015012.png)

### Verificación de conectividad de red del Host
Tras la creación del conmutador virtual, se verifica que la máquina anfitriona (Windows) conserva la resolución DNS y la salida a Internet.

**⚠️ Troubleshooting (Problemas conocidos y soluciones):** 
Al crear el conmutador externo, la conexión a internet del host puede perderse temporalmente (pérdida de IP y caída de DNS) mientras Windows transfiere la pila de red al nuevo adaptador virtual `vEthernet`.
* **Solución 1:** Deshabilitar y volver a habilitar físicamente el adaptador de red en el Administrador de Dispositivos o Conexiones de Red para forzar la reasignación por DHCP.
* **Solución 2:** Si persiste el fallo de resolución de nombres, configurar manualmente los servidores DNS públicos y fiables en las propiedades IPv4 del adaptador de red virtual de Hyper-V (`vEthernet`):
  * **DNS Primario:** `8.8.8.8` (Google)
  * **DNS Secundario:** `1.1.1.1` (Cloudflare)

![Verificación de Red Host (nslookup y tracert)](../0-Infraestructura%20Base/assets/img/Captura%20de%20pantalla%202026-10-04%20020010.png)

## 2. Aprovisionamiento de la Máquina Virtual
Se procede a crear la máquina virtual que actuará como hipervisor anfitrión (Proxmox VE) sobre el hipervisor de Tipo 1 (Hyper-V).

1. En el **Administrador de Hyper-V**, hacer clic en **Nuevo > Máquina virtual**.
2. **Configuración del asistente:**
   * **Nombre:** `Proxmox Server`
   * **Generación:** **Generación 1** (Imprescindible para la compatibilidad del arranque con Proxmox).
   * **Memoria asignada:** `12288 MB` (Desmarcar la opción de memoria dinámica).
   * **Red:** Seleccionar el conmutador virtual `Red Externa`.
   * **Disco duro virtual:** Crear un disco virtual de `250 GB` (.vhdx).
   * **Opciones de instalación:** Seleccionar la ISO de Proxmox VE descargada previamente.
   * **Características avanzadas del adaptador de red:** Habilitar suplantación de dirección MAC.

![Resumen de Aprovisionamiento de la VM](../0-Infraestructura%20Base/assets/img/Captura%20de%20pantalla%202026-10-04%20021317.png)

## 3. Habilitar Virtualización Anidada (Nested Virtualization)
1. Abrir una consola de **PowerShell como Administrador** en el sistema anfitrión.
2. Ejecutar el siguiente comando para habilitar el *passthrough* de la CPU:

```powershell
Set-VMProcessor -VMName "Proxmox Server" -ExposeVirtualizationExtensions $true
```

3. Verificar que haya quedado habilitado.

```powershell
Get-VMProcessor -VMName "Proxmox Server" | Select-Object VMName, ExposeVirtualizationExtensions
```
4. El resultado debería ser el siguiente:

```powershell
VMName         ExposeVirtualizationExtensions
------         ------------------------------
Proxmox Server                           True

PS C:\Windows\System32>
```

## 4. Instalación del Sistema Operativo (Proxmox VE)
1. Encender la máquina virtual `Proxmox Server` en Hyper-V.
2. Seleccionar la opción de instalación por defecto (*Install Proxmox VE*).
3. **Puntos clave durante el asistente:**
   * **Disco de destino:** Seleccionar el disco virtual de 250 GB configurado.
   * **Configuración de red:** Asignar una IP estática dentro del rango de tu red local (o DHCP si tu router lo maneja bien), el Gateway de tu red y un DNS válido (ej: `8.8.8.8`).
4. Tras finalizar la instalación, el sistema se reiniciará y mostrará la URL de acceso web (por ejemplo: `https://192.168.X.X:8006`).

**⚠️ Nota técnica sobre Generación 1 e Instalador Gráfico:**
Al desplegar Proxmox VE en una máquina virtual de **Generación 1** en Hyper-V, el instalador gráfico por defecto puede fallar al cargar el adaptador de vídeo emulado (pantalla negra o congelación).
* **Solución:** En el menú de arranque de la ISO de Proxmox, seleccionar la opción de instalación por **Terminal UI (Text mode)** o utilizar el modo de depuración para completar la instalación de forma fluida a través de la consola de comandos del instalador.

## 5. Post-Instalación, Repositorios y Verificación de Operatividad
1. **Configuración de Red Definitiva:** Establecer reserva estática de IP en el router para el adaptador virtual y verificar conectividad.
2. **Gestión de Repositorios (*No-Subscription*):** 
   * Desactivar los repositorios comerciales corporativos (*Enterprise*).
   * Añadir los repositorios públicos de Proxmox VE sin suscripción para permitir actualizaciones de paquetes gratuitas.
3. **Verificación final:** Acceder a la interfaz web de gestión a través de `https://<tu-ip>:8006` y comprobar el registro de tareas.

![Registro de Tareas y Operatividad del Nodo](../0-Infraestructura%20Base/assets/img/Captura%20de%20pantalla%202026-10-04%20030251.png)