# Informe técnico: Arch Linux

## 1. Datos del grupo

- **Nombre y apellidos:** Luis Alberto Rodriguez Aguilar
- **Grupo:** Arch Linux
- **Sistema operativo:** Arch Linux
- **Versión y arquitectura:** imagen ISO oficial vigente en la fecha de instalación; arquitectura x86_64/amd64. Arch Linux utiliza un modelo rolling release, por lo que no se fija una versión LTS tradicional.
- **Escenario:** servidor web Linux en máquina virtual, con Nginx, PHP-FPM y una base de datos opcional.

## 2. Información general del SO

### Identificación y licencia

Arch Linux es una distribución GNU/Linux independiente, de propósito general y x86-64. Su filosofía prioriza simplicidad, control del usuario y software actualizado. Los paquetes se distribuyen con licencias libres variadas; no existe una licencia única para todo el sistema. El núcleo Linux y cada paquete deben consultarse individualmente.

- **Tipo de licencia:** software libre/open source; algunos componentes pueden tener licencias distintas, siempre indicadas en sus paquetes.
- **Soporte:** documentación oficial y ArchWiki, comunidad, foros y listas de correo. No hay soporte comercial central obligatorio.
- **Ciclo de vida:** rolling release; no hay ediciones LTS oficiales como en Ubuntu LTS. Las actualizaciones se aplican de forma continua y deben realizarse con atención.
- **Arquitectura principal oficial:** x86-64. Para ARM existen proyectos independientes y no deben confundirse con la distribución oficial x86-64.

### Casos de uso

- Servidores web y de aplicaciones.
- DNS, DHCP, proxy y servicios de red.
- Contenedores y laboratorios de virtualización.
- Automatización, desarrollo y plataformas de pruebas.
- Servidores ligeros personalizados.

### Ventajas e inconvenientes

| Aspecto | Ventajas | Inconvenientes |
|---|---|---|
| Actualización | Software muy reciente y rolling release | Una actualización puede exigir intervención manual |
| Control | Instalación mínima y altamente configurable | Requiere más conocimientos que una distribución orientada a servidor |
| Documentación | ArchWiki muy detallada | La responsabilidad de leer avisos y mantener el sistema recae en el administrador |
| Rendimiento | Pocos servicios instalados por defecto | El rendimiento depende mucho del diseño y la configuración |
| Coste | Sin coste de licencia del sistema | El soporte empresarial debe contratarse externamente si se necesita |

## 3. Requisitos de hardware

Arch Linux puede funcionar en hardware x86_64 con aproximadamente 512 MiB de RAM, aunque el medio live necesita más memoria. Esa cifra es un mínimo práctico para arrancar e instalar; no es adecuada para un servidor web de producción.

| Concepto | Mínimo oficial/práctico | Recomendado para producción | Justificación |
|---|---:|---:|---|
| CPU | x86_64, 1 núcleo | 2–4 vCPU o núcleos | Nginx consume poco; PHP y base de datos requieren margen |
| RAM | 512 MiB para el sistema instalado; más para el live | 4 GiB; 8 GiB con base de datos o contenedores | Evita swapping y permite caché |
| Almacenamiento | 2 GiB para un sistema muy básico | SSD/NVMe de 40–80 GiB | Incluye sistema, paquetes, web, logs y margen de crecimiento |
| Tarjeta de red | NIC compatible con Linux | 1 GbE virtual o física, IP estable | Administración, actualizaciones y servicio web |
| Otros | Firmware compatible con x86_64 | UEFI, disco virtual respaldado, snapshots y RAID en hardware físico | Facilita recuperación y mantenimiento |

- **Espacio mínimo de instalación base:** reservar al menos 2 GiB; para una instalación cómoda, 10 GiB.
- **Espacio recomendado para el rol:** 40 GiB como mínimo, separado o ampliable para datos, logs y copias; 80 GiB si se almacenan contenidos o bases de datos localmente.
- **Firmware:** UEFI recomendado. La imagen oficial de instalación no debe asumirse compatible con Secure Boot; si se exige Secure Boot, hay que diseñar y firmar la cadena de arranque después de estudiar la configuración correspondiente. En una VM, activar virtualización asistida por hardware en el host.
- **TPM:** no es requisito general de Arch Linux; puede aprovecharse para casos concretos de cifrado o atestación.

## 4. Software y prerrequisitos

- **Medio:** ISO oficial, espejo HTTP/HTTPS, USB arrancable o ISO montada por el hipervisor.
- **Descarga oficial:** [archlinux.org/download](https://archlinux.org/download/).
- **Verificación:** comprobar la firma PGP y/o checksum publicado antes de usar la ISO.
- **Red:** DHCP facilita la instalación; para un servidor se debe disponer de IP estática o reserva DHCP, puerta de enlace, DNS funcional y acceso a los espejos. Configurar proxy si la red lo requiere.
- **Antes de instalar:** actualizar firmware del servidor/hipervisor, definir nombre de host, planificar direccionamiento, decidir BIOS/UEFI, particionado, cifrado y copias de seguridad.
- **RAID:** en físico, configurar RAID por hardware o software según el diseño; en VM se recomienda almacenamiento redundante en el host.
- **Claves/suscripción:** no se necesita clave de licencia ni cuenta de suscripción para Arch Linux.
- **Dependencias externas:** NTP, DNS, repositorios/espejos, sistema de copias y, si aplica, certificado TLS, DNS externo y servicio de monitorización.

## 5. Planificación de la instalación

### Tipo de despliegue

Máquina virtual sobre KVM/libvirt, VMware ESXi o Hyper-V, con 2 vCPU, 4 GiB de RAM, disco SSD virtual de 60 GiB y una NIC conectada a una red de servidor. Para producción real se debe añadir redundancia, copias verificadas y, preferiblemente, separar datos del disco del sistema.

### Particionado previsto

| Punto de montaje | Tamaño | Tipo |
|---|---:|---|
| EFI System Partition | 512 MiB–1 GiB | FAT32 |
| `/boot` | 1 GiB | ext4 o FAT32 según el diseño UEFI |
| `/` | 20 GiB | ext4 |
| `/var` | 15 GiB | ext4 o XFS; contiene caché, paquetes, web y logs |
| `/home` | 5 GiB | ext4; opcional en servidor |
| swap | 2–4 GiB | partición o archivo |
| Datos restantes | ampliable | `/srv` o volumen dedicado |

Una alternativa de producción es LVM sobre LUKS, con volúmenes separados para `/`, `/var`, `/srv` y swap. Separar `/var` limita el impacto de logs o cachés; separar `/srv` facilita copias y ampliación. No se debe llenar `/tmp`; puede usarse montaje temporal con opciones de seguridad según compatibilidad.

### Opciones de instalación

- Usar el instalador guiado `archinstall` solo si el perfil resultante se revisa; para aprendizaje, también es válido el procedimiento manual de la guía oficial.
- Sin entorno gráfico.
- Instalar `base`, `linux`, `linux-firmware`, `networkmanager`, `openssh`, `sudo`, editor, herramientas de diagnóstico y `ufw` o `nftables`.
- Configurar hostname completo, zona horaria `Europe/Madrid`, teclado español y sincronización horaria.
- Usar DHCP durante el arranque inicial y fijar después la red con reserva DHCP o configuración estática documentada.

### Cuentas y seguridad inicial

Crear un usuario administrador nominal, por ejemplo `luis`, miembro de `wheel`, con `sudo`. Mantener root bloqueado para acceso remoto por contraseña. Usar contraseñas largas y únicas; preferir claves SSH ed25519. No guardar contraseñas, claves privadas ni tokens en GitHub.

## 6. Configuración post-instalación

### Actualización y paquetes

```bash
sudo timedatectl set-timezone Europe/Madrid
sudo pacman -Syu
sudo pacman -S --needed networkmanager openssh sudo vim git curl man-db man-pages ufw nginx
sudo systemctl enable --now NetworkManager
```

En Arch Linux se recomienda evitar actualizaciones parciales: usar `pacman -Syu` y revisar noticias y avisos antes de actualizar.

### Red definitiva

Documentar IP, prefijo, puerta de enlace, DNS, hostname y dominio. Ejemplo de laboratorio: `192.168.10.20/24`, gateway `192.168.10.1`, DNS `192.168.10.1` y nombre `web01.lan`. Sustituir estos valores por los reales y aplicar la configuración con NetworkManager.

```bash
nmcli connection show
sudo nmcli connection modify "CONEXION" ipv4.method manual ipv4.addresses 192.168.10.20/24 ipv4.gateway 192.168.10.1 ipv4.dns "192.168.10.1" ipv6.method disabled
sudo nmcli connection up "CONEXION"
sudo hostnamectl set-hostname web01.lan
```

### Usuarios, servicios y firewall

```bash
sudo useradd -m -G wheel -s /bin/bash luis
sudo passwd luis
sudo EDITOR=vim visudo
sudo systemctl enable --now sshd
sudo systemctl enable --now nginx
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
sudo ufw status verbose
```

En `sudoers`, habilitar el grupo `wheel` mediante la línea correspondiente. Restringir SSH a la red de administración, desactivar login de root y preferir autenticación por clave. No abrir puertos innecesarios.

### Hardening mínimo

- Mantener el sistema actualizado con una ventana de mantenimiento y copia previa.
- Revisar `journalctl`, logs de Nginx y espacio disponible.
- Instalar y configurar monitorización y alertas.
- Deshabilitar servicios no utilizados.
- Usar TLS con un certificado válido en producción.
- Separar permisos del contenido web y evitar ejecutar aplicaciones como root.
- Probar restauraciones de backup, no solo la creación de copias.
- Validar cambios en una VM de pruebas antes de producción.

## Fuentes oficiales consultadas

- [Arch Linux — About](https://archlinux.org/about/)
- [Arch Linux — Downloads](https://archlinux.org/download/)
- [ArchWiki — Guía de instalación en español](https://wiki.archlinux.org/title/Installation_guide_(Español))
- [ArchWiki — Arch Linux](https://wiki.archlinux.org/title/Arch_Linux)
- [ArchWiki — UEFI/Secure Boot](https://wiki.archlinux.org/title/Unified_Extensible_Firmware_Interface/Secure_Boot)

> Las cifras de producción son recomendaciones de diseño, no requisitos oficiales. Deben validarse mediante pruebas de carga, disponibilidad, retención de logs y política de copias de la organización.
