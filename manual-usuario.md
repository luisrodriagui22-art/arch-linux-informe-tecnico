# Manual de usuario: servidor Arch Linux

## 1. Objetivo y alcance

Este manual está dirigido al usuario y administrador del servidor web Arch Linux del grupo Arch Linux. Describe las operaciones habituales sin sustituir la documentación oficial ni los procedimientos de cambio de la organización.

**Usuario:** Luis Alberto Rodriguez Aguilar  
**Escenario:** servidor web en máquina virtual  
**Arquitectura:** x86_64

## 2. Acceso al servidor

Desde Linux, macOS o Windows con OpenSSH:

```bash
ssh luis@IP_DEL_SERVIDOR
```

Usar preferentemente una clave SSH ed25519. Nunca enviar contraseñas o claves privadas por correo, chat o repositorios. Para salir:

```bash
exit
```

Si no hay conexión, comprobar primero la IP, la ruta, el puerto 22, el estado de `sshd` y el firewall.

## 3. Comandos esenciales

| Necesidad | Comando |
|---|---|
| Ver usuario actual | `whoami` |
| Ver hostname | `hostnamectl` |
| Ver IP y enlaces | `ip address` |
| Ver espacio | `df -h` |
| Ver memoria | `free -h` |
| Ver procesos | `top` o `htop` |
| Ver servicios | `systemctl --type=service` |
| Consultar ayuda | `man comando` |
| Ejecutar como administrador | `sudo comando` |

No ejecutar comandos destructivos como `rm -rf`, formateos o cambios de particiones sin verificar el objetivo y disponer de una copia recuperable.

## 4. Actualizar Arch Linux

Arch Linux es rolling release. Antes de actualizar:

1. Confirmar que existe una copia reciente y restaurable.
2. Revisar las noticias y avisos de Arch Linux.
3. Disponer de una consola alternativa o acceso a la VM.
4. Ejecutar una actualización completa, nunca una actualización parcial.

```bash
sudo pacman -Syu
```

Limpiar caché solo con criterio y sin eliminar paquetes necesarios:

```bash
paccache -r
```

Si la actualización muestra un conflicto, detenerse, leer el mensaje y consultar la documentación. No forzar opciones de `pacman` sin entender sus efectos.

## 5. Gestionar servicios

Comandos habituales:

```bash
systemctl status nginx
sudo systemctl restart nginx
sudo systemctl reload nginx
sudo systemctl enable --now nginx
sudo systemctl disable --now servicio
```

- `start`: inicia ahora.
- `stop`: detiene ahora.
- `restart`: reinicia.
- `reload`: recarga configuración sin reinicio completo, si el servicio lo permite.
- `enable`: activa el arranque automático.
- `status`: muestra estado y últimos mensajes.

Antes de recargar Nginx, comprobar la configuración:

```bash
sudo nginx -t
```

## 6. Servidor web

El contenido web debe ubicarse en la ruta definida por la configuración de Nginx, normalmente dentro de `/srv/http` o una ruta documentada. Revisar propietario y permisos para que el proceso web pueda leer, pero no modificar innecesariamente.

Prueba local:

```bash
curl -I http://127.0.0.1
```

Logs habituales:

```bash
sudo journalctl -u nginx -e
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log
```

En producción, publicar HTTPS, renovar certificados y redirigir HTTP según la política del sitio.

## 7. Red y DNS

Consultar la red:

```bash
ip address
ip route
resolvectl status
nmcli connection show
```

Probar conectividad por etapas:

```bash
ping -c 4 IP_DE_LA_PUERTA_DE_ENLACE
ping -c 4 1.1.1.1
getent hosts archlinux.org
```

Interpretación básica:

- Si falla la puerta de enlace, revisar interfaz, cableado/red virtual y dirección IP.
- Si funciona la puerta de enlace pero no una IP externa, revisar ruta o firewall.
- Si funciona una IP pero no un nombre, revisar DNS.

No modificar la IP de producción sin registrar la nueva dirección y preparar una sesión de acceso alternativa.

## 8. Firewall

Consultar estado de UFW:

```bash
sudo ufw status verbose
```

Reglas de referencia para un servidor web:

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
```

Restringir SSH a la red de administración cuando sea posible. Antes de activar o modificar el firewall, confirmar que la sesión actual y una sesión de emergencia seguirán funcionando.

## 9. Usuarios y permisos

Crear una cuenta nominal:

```bash
sudo useradd -m -G wheel -s /bin/bash nombre_usuario
sudo passwd nombre_usuario
```

Editar permisos de sudo de forma segura:

```bash
sudo visudo
```

La cuenta root no debe usarse para tareas cotidianas. Aplicar el principio de mínimo privilegio y no compartir cuentas. Revisar usuarios y grupos:

```bash
getent passwd
getent group
```

## 10. Logs y diagnóstico

Consultar el registro del sistema:

```bash
journalctl -b
journalctl -p warning..alert
journalctl -u sshd
journalctl -u nginx
```

Seguir el registro en tiempo real:

```bash
journalctl -f
```

Comprobar recursos cuando el servidor esté lento:

```bash
df -h
free -h
uptime
ss -tulpn
systemctl --failed
```

Si `/var` se llena, no borrar logs a ciegas. Identificar qué servicio crece, conservar evidencias necesarias y corregir la rotación o retención.

## 11. Copias de seguridad

Una copia útil debe poder restaurarse. Como mínimo, incluir:

- Configuración de Nginx y servicios.
- Contenido de `/srv` o directorio web.
- Bases de datos, si existen.
- Lista de paquetes instalados y documentación de red.
- Claves y certificados según una política segura, nunca dentro de Git público.

Registrar fecha, alcance, ubicación, cifrado y resultado de la última prueba de restauración. Mantener al menos una copia fuera del servidor.

## 12. Mantenimiento periódico

- Aplicar actualizaciones completas de forma planificada.
- Revisar avisos de Arch Linux y errores de servicios.
- Comprobar espacio, memoria, certificados y sincronización horaria.
- Verificar backups y hacer pruebas de recuperación.
- Eliminar servicios y puertos que ya no se necesiten.
- Revisar accesos SSH y claves autorizadas.
- Documentar cada cambio relevante.

## 13. Problemas frecuentes

### No puedo conectarme por SSH

Comprobar IP y ruta; desde la consola del hipervisor ejecutar `systemctl status sshd`, comprobar `ss -tlnp | grep :22` y revisar `ufw status`. Confirmar que el usuario y la clave corresponden al servidor.

### Nginx no arranca

Ejecutar `sudo nginx -t` y `sudo journalctl -u nginx -e`. Corregir primero el error de configuración; después reiniciar.

### No resuelven los nombres

Comprobar `resolvectl status`, DNS configurado y conectividad hacia el servidor DNS. No confundir un fallo DNS con un fallo general de red.

### El disco está lleno

Ejecutar `df -h`, localizar directorios grandes con `du`, revisar logs y cachés. No borrar archivos del sistema sin identificar su función.

### Una actualización falla

No realizar actualizaciones parciales ni apagar durante una operación de paquetes. Leer el error, revisar noticias oficiales, conservar la salida del comando y aplicar el procedimiento documentado.

## 14. Reglas de seguridad

- No publicar credenciales, tokens, claves privadas ni datos personales innecesarios.
- Usar SSH con claves y limitar el acceso administrativo.
- Mantener firewall activo y abrir solo puertos necesarios.
- Aplicar actualizaciones completas con copias previas.
- Probar cambios en una máquina virtual antes de producción.
- No ejecutar scripts descargados sin revisar su contenido y procedencia.
- Registrar cambios y conservar un método de recuperación por consola.

## Fuentes

- [Arch Linux — Downloads](https://archlinux.org/download/)
- [ArchWiki — Guía de instalación](https://wiki.archlinux.org/title/Installation_guide_(Español))
- [ArchWiki — Arch Linux](https://wiki.archlinux.org/title/Arch_Linux)
