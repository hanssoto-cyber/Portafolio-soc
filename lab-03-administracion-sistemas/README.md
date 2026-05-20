# 🖥️ Lab 03 — Administración de Sistemas y Hardening

> **Examen Final: Administración de Sistemas Operativos de Redes · IPLACEX · 2025**

## Descripción

Implementación de una infraestructura de servidores empresarial completa en entorno virtualizado con VMware. El laboratorio cubre la instalación y configuración de Windows Server 2016 y Red Hat Linux, implementación de servicios de red críticos (Active Directory, DHCP, DNS) y unión de clientes al dominio corporativo.

---

## 🏗️ Arquitectura del Entorno

```
┌─────────────────────────────────────────────────────────┐
│                    Red 192.168.1.0/24                   │
│                                                         │
│  ┌──────────────────────┐   ┌──────────────────────┐   │
│  │  Windows Server 2016 │   │   Red Hat Linux      │   │
│  │  hans-soto-gonzalez  │   │  hans-soto-gonzalez  │   │
│  │  WinSer              │   │  Redhat              │   │
│  │                      │   │                      │   │
│  │  IP: 192.168.1.2     │   │  IP: 192.168.1.3     │   │
│  │  Roles:              │   │  Disco 1: SO         │   │
│  │  • Active Directory  │   │  Disco 2: 20 GB      │   │
│  │  • DNS (Iplacex.cl)  │   │                      │   │
│  │  • DHCP              │   │                      │   │
│  │  Disco 1: SO         │   │                      │   │
│  │  Disco 2: E: (NTFS)  │   │                      │   │
│  └──────────────────────┘   └──────────────────────┘   │
│             │                                           │
│  ┌──────────▼───────────┐                              │
│  │    Windows 10 Client │                              │
│  │    IP: DHCP          │                              │
│  │    DNS: 192.168.1.2  │                              │
│  │    Dominio: Iplacex.cl│                             │
│  │    Usuario: hans.s   │                              │
│  └──────────────────────┘                              │
└─────────────────────────────────────────────────────────┘
```

---

## ⚙️ Actividades Implementadas

### Actividad 1 — Instalación Red Hat Linux + Configuración de Red

**Objetivo:** Instalar Red Hat Linux en VM con red estática.

| Parámetro | Valor |
|---|---|
| Nombre de la VM | hans-soto-gonzalez Redhat |
| Dirección IP | 192.168.1.3 |
| Máscara | 255.255.255.0 |
| Gateway | 192.168.1.1 |

**Procedimiento:**
- Creación de VM en VMware con recursos adecuados
- Instalación de Red Hat Linux con puntos de montaje personalizados
- Configuración de red estática mediante interfaz gráfica
- Verificación con `ifconfig`

---

### Actividad 2 — Segundo Disco en Red Hat Linux

**Objetivo:** Agregar y configurar disco adicional de 20 GB.

- Disco SATA agregado a la VM apagada
- Identificación con `gnome-disks`
- Formateo y montaje de la nueva partición
- Verificación del montaje correcto

---

### Actividad 3 — Instalación Windows Server 2016 + Red Estática

| Parámetro | Valor |
|---|---|
| Nombre de la VM | hans-soto-gonzalez WinSer |
| Dirección IP | 192.168.1.2 |
| Máscara | 255.255.255.0 |
| Gateway | 192.168.1.1 |
| DNS | 192.168.1.2 (apunta a sí mismo) |

**Verificación:** `ipconfig /all` en CMD

---

### Actividad 4 — Segundo Disco en Windows Server 2016

- Disco agregado a la VM vía VMware Settings
- Inicializado como MBR en Administrador de Discos
- Volumen simple creado con letra **E:**
- Formato **NTFS** — listo para almacenamiento

---

### Actividad 5 — Active Directory: Dominio Iplacex.cl

**Objetivo:** Promover Windows Server 2016 a Controlador de Dominio.

```
Dominio raíz: Iplacex.cl
Tipo: Nuevo bosque
NetBIOS: IPLACEX
Usuario de dominio: hans.s@iplacex.cl
```

**Proceso:**
1. Instalación del rol **AD DS** desde Administrador del Servidor
2. Promoción a Controlador de Dominio — opción "Agregar nuevo bosque"
3. Configuración de contraseña DSRM
4. Reinicio automático del servidor
5. Verificación en Administrador del Servidor → dominio `Iplacex.cl` activo

---

### Actividad 6 — Servicio DHCP

**Objetivo:** Asignar 30 IPs dinámicas a clientes de red.

| Parámetro | Valor |
|---|---|
| Inicio del ámbito | 192.168.1.20 |
| Fin del ámbito | 192.168.1.49 |
| Máscara | 255.255.255.0 |
| Gateway | 192.168.1.1 |
| DNS | 192.168.1.2 |
| Total IPs | 30 direcciones |

**Proceso:**
1. Instalación del rol DHCP
2. Autorización del servidor en Active Directory
3. Creación y activación del ámbito

---

### Actividad 7 — Servicio DNS: Dominio Iplacex.cl

**Registros tipo A creados:**

| Nombre | FQDN | IP |
|---|---|---|
| @ (raíz) | Iplacex.cl | 192.168.1.2 |
| www | www.iplacex.cl | 192.168.1.2 |
| ftp | ftp.iplacex.cl | 192.168.1.2 |

**Verificación:** `nslookup www.iplacex.cl 192.168.1.2`

---

### Actividad 8 — Windows 10 unido al Dominio

**Objetivo:** Integrar cliente Windows 10 al dominio Iplacex.cl.

**Configuración del cliente:**
- IP: DHCP (recibe 192.168.1.20–49)
- DNS: 192.168.1.2

**Proceso:**
1. VM Windows 10 creada y configurada con DNS apuntando al DC
2. Unión al dominio desde Propiedades del Sistema → Cambiar → Dominio: `Iplacex.cl`
3. Autenticación con credenciales del administrador de dominio
4. Reinicio para aplicar cambios
5. Inicio de sesión con usuario de dominio: `hans.s@iplacex.cl`

---

## 📸 Capturas del Laboratorio

| # | Archivo | Descripción |
|---|---|---|
| 01 | `capturas/01-redhat-instalacion.png` | Instalación Red Hat Linux en VMware |
| 02 | `capturas/02-redhat-red-estatica.png` | Configuración IP estática en Red Hat |
| 03 | `capturas/03-redhat-ifconfig.png` | Verificación con ifconfig |
| 04 | `capturas/04-redhat-disco2.png` | Segundo disco montado en Red Hat |
| 05 | `capturas/05-winserver-instalacion.png` | Instalación Windows Server 2016 |
| 06 | `capturas/06-winserver-red-estatica.png` | Configuración red estática Windows |
| 07 | `capturas/07-winserver-ipconfig.png` | Verificación con ipconfig /all |
| 08 | `capturas/08-winserver-disco2.png` | Disco E: NTFS en administrador de discos |
| 09 | `capturas/09-ad-instalacion.png` | Instalación rol AD DS |
| 10 | `capturas/10-ad-dominio-iplacex.png` | Promoción a DC — dominio Iplacex.cl |
| 11 | `capturas/11-ad-verificacion.png` | Verificación AD activo |
| 12 | `capturas/12-dhcp-ambito.png` | Ámbito DHCP 192.168.1.20-49 |
| 13 | `capturas/13-dns-registros.png` | Registros A www y ftp creados |
| 14 | `capturas/14-dns-nslookup.png` | Verificación con nslookup |
| 15 | `capturas/15-w10-union-dominio.png` | Windows 10 unido a Iplacex.cl |
| 16 | `capturas/16-w10-login-dominio.png` | Login con hans.s@iplacex.cl |

---

## ✅ Checklist de Hardening Aplicado

### Windows Server 2016
- [x] Contraseña de administrador robusta configurada
- [x] Firewall de Windows desactivado en red controlada (lab)
- [x] DNS apuntando a sí mismo como servidor autoritativo
- [x] DHCP autorizado en Active Directory
- [x] Políticas de grupo (GPO) aplicadas al dominio
- [x] Segundo disco con formato NTFS para separación de datos
- [ ] Actualizaciones de seguridad pendientes (recomendado en producción)
- [ ] Auditoría de eventos habilitada (recomendado en producción)

### Red Hat Linux
- [x] IP estática configurada — sin DHCP no autorizado
- [x] Contraseña de root definida durante instalación
- [x] Particionado con puntos de montaje separados
- [x] Disco secundario montado correctamente
- [ ] SELinux en modo enforcing (recomendado en producción)
- [ ] Actualizaciones de seguridad aplicadas (recomendado)

---

## 🔑 Lecciones Aprendidas

| Hallazgo | Recomendación para Producción |
|---|---|
| Firewall desactivado | Habilitar y configurar reglas específicas por rol |
| DNS único punto de falla | Implementar DC secundario como DNS alternativo |
| Sin auditoría de eventos | Habilitar GPO de auditoría y centralizar en SIEM |
| Contraseñas simples en lab | Implementar política de complejidad via GPO |
| Sin separación de roles | Separar DC, DNS y DHCP en servidores distintos |

---

## 🔗 Tecnologías Utilizadas

`Windows Server 2016` `Red Hat Linux` `VMware` `Active Directory` `AD DS` `DHCP` `DNS` `GPO` `NTFS` `IPv4` `Windows 10`

---

## 📁 Archivos del Laboratorio

```
lab-03-administracion-sistemas/
├── README.md
├── Informe_Examen_Administracion_Servidores.docx
└── capturas/
    ├── 01-redhat-instalacion.png
    ├── 02-redhat-red-estatica.png
    └── ...
```

---

*Hans Soto González · IPLACEX · Ingeniería en Ciberseguridad · 2025*  
*[linkedin.com/in/hans-soto](https://linkedin.com/in/hans-soto) · [Portafolio](https://github.com/hanssoto-cyber/Portafolio-soc)*
