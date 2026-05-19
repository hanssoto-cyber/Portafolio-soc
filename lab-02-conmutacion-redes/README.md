# 🌐 Lab 02 — Examen Final: Conmutación de Redes

> **Laboratorio de Redes · IPLACEX · Ingeniería en Ciberseguridad · 2025**

## Descripción

Implementación y resolución de problemas de una red empresarial compleja en Cisco Packet Tracer. El laboratorio cubre configuración de OSPF Multiarea con autenticación MD5, seguridad de capa 2 y capa 3, NAT con sobrecarga, VLANs con port security y servicios de administración (NTP/Syslog).

---

## 🏗️ Topología de Red

```
                        Internet
                       8.8.8.8/32
                    3001:ABCD:8:8/128
                           │
                        Gig0/0
                     200.0.0.0/30
                   3001:ABCD:CD:200::/64
                           │
                        Gig0/0
                    ┌──────R2──────┐
                    │   AREA 0    │
              Se0/0/0              Se0/0/1
           10.0.0.0/24          10.1.0.0/24
        3001:ABCD:10::/64    3001:ABCD:20::/64
              │                       │
           Se0/0/1                 Se0/0/0
    ┌────────R1────────┐     ┌────────R3────────┐
    │     AREA 1       │     │     AREA 2       │
    │  OSPFv2 ID 1     │     │  OSPFv2 ID 3     │
    │  OSPFv3 ID 2     │     │  OSPFv3 ID 2     │
    │                  │     │                  │
   Gig0/0            Gig0/0  Gig0/0
    │                           │
   SWB ══ Port-Channel ══ SWA  SWC ══ Port-Channel ══ SWD
    │                    │      │                    │
  Fa0/3                Fa0/10  Fa0/3               Fa0/10
    │                    │      │                    │
    │              PC VLAN10   Server VLAN30    PC VLAN40
    │              (RRHH)      (TI - NTP/Syslog)(GERENCIA)
  Fa0/20
    │
  PC VLAN20
  (VENTAS)
```

---

## 📊 Esquema de Direccionamiento

### IPv4
| VLAN | Nombre | Red | Gateway |
|---|---|---|---|
| 10 | RRHH | 172.16.2.0/24 | 172.16.2.1 |
| 20 | VENTAS | 172.16.3.0/24 | 172.16.3.1 |
| 30 | TI | 192.168.4.0/24 | 192.168.4.1 |
| 40 | GERENCIA | 192.168.5.0/24 | 192.168.5.1 |

### IPv6
| VLAN | Red IPv6 |
|---|---|
| 10 | 3001:ABCD:1:A::/80 |
| 20 | 3001:ABCD:1:B::/80 |
| 30 | 3001:ABCD:1:C::/80 |
| 40 | 3001:ABCD:1:D::/80 |

---

## ⚙️ Configuraciones Implementadas

### A. Detección y Corrección de Problemas

#### Port-Channel (EtherChannel estático)
```cisco
! Configuración estática persistente en SWA/SWB/SWC/SWD
interface range fa0/1 - 2
 channel-group 1 mode on
!
interface port-channel 1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40
 switchport nonegotiate
```
**Verificación:** `show etherchannel summary` → estado **SU**

#### DHCP en R2
```cisco
ip dhcp excluded-address 172.16.2.1 172.16.2.4
ip dhcp excluded-address 172.16.3.1 172.16.3.4
ip dhcp excluded-address 192.168.5.1 192.168.5.4
!
ip dhcp pool VLAN10
 network 172.16.2.0 255.255.255.0
 default-router 172.16.2.1
 dns-server 8.8.8.8
!
ip dhcp pool VLAN20
 network 172.16.3.0 255.255.255.0
 default-router 172.16.3.1
 dns-server 8.8.8.8
!
ip dhcp pool VLAN40
 network 192.168.5.0 255.255.255.0
 default-router 192.168.5.1
 dns-server 8.8.8.8
```
**Verificación:** `show ip dhcp binding`

---

### B. Seguridad Capa 2

#### VLANs
```cisco
vlan 10
 name RRHH
vlan 20
 name VENTAS
vlan 30
 name TI
vlan 40
 name GERENCIA
```
**Verificación:** `show vlan brief`

#### Port Security (Sticky)
```cisco
interface fastEthernet 0/10
 switchport mode access
 switchport access vlan 10
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown
```
**Verificación:** `show port-security interface fa0/10`
- Port Security: **Enabled**
- Port Status: **Secure-up**
- Sticky MAC Addresses: **1**

#### Troncales sin DTP
```cisco
interface range fa0/1 - 2
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40
 switchport nonegotiate
```
**Verificación:** `show interfaces trunk`

---

### C. Seguridad Capa 3

#### OSPF Multiarea con autenticación MD5
```cisco
! R1 - Area 1
router ospf 1
 router-id 1.1.1.1
 area 1 authentication message-digest
 passive-interface gig0/0.10
 passive-interface gig0/0.20
 network 172.16.2.0 0.0.0.255 area 1
 network 172.16.3.0 0.0.0.255 area 1
 network 10.0.0.0 0.0.0.255 area 0
!
! Autenticación MD5 en interfaces
interface serial 0/0/1
 ip ospf message-digest-key 1 md5 prueba3
 ip ospf authentication message-digest
```
**Verificación:** `show ip ospf neighbor` → estado **FULL**

#### NAT con sobrecarga + ACL INTERNET
```cisco
ip access-list standard INTERNET
 permit 172.16.0.0 0.0.255.255
 permit 192.168.4.0 0.0.0.255
 permit 192.168.5.0 0.0.0.255
!
ip nat inside source list INTERNET interface gig0/0 overload
!
interface gig0/0
 ip nat outside
interface gig0/0.10
 ip nat inside
```
**Verificación:** `show ip nat translations`

#### NTP y Syslog
```cisco
! En todos los routers (servidor: IP VLAN30)
ntp server 192.168.4.10
logging 192.168.4.10
logging trap informational
```
**Verificación:** `show ntp status` → **Clock is synchronized**

---

## 📸 Capturas del Laboratorio

| # | Archivo | Descripción |
|---|---|---|
| 01 | `capturas/01-topologia-completa.png` | Topología completa del lab en Packet Tracer |
| 02 | `capturas/02-etherchannel-summary.png` | Port-channels en estado SU |
| 03 | `capturas/03-dhcp-binding.png` | PCs recibiendo IP dinámica por DHCP |
| 04 | `capturas/04-vlan-brief.png` | VLANs creadas y activas |
| 05 | `capturas/05-port-security-swa.png` | Port security en SWA (sticky, shutdown) |
| 06 | `capturas/06-interfaces-trunk.png` | Troncales sin DTP configurados |
| 07 | `capturas/07-ospf-neighbor.png` | Vecinos OSPF en estado FULL con MD5 |
| 08 | `capturas/08-ip-route-ospf.png` | Rutas OSPF en tabla de enrutamiento |
| 09 | `capturas/09-nat-translations.png` | Traducciones NAT activas |
| 10 | `capturas/10-acl-internet.png` | ACL INTERNET configurada |
| 11 | `capturas/11-ntp-status.png` | NTP sincronizado con servidor VLAN30 |
| 12 | `capturas/12-syslog-server.png` | Logs recibidos en servidor Syslog |
| 13 | `capturas/13-ping-ipv4.png` | Conectividad IPv4 extremo a extremo |
| 14 | `capturas/14-ping-ipv6.png` | Conectividad IPv6 extremo a extremo |
| 15 | `capturas/15-port-security-swc.png` | Port security en SWC |
| 16 | `capturas/16-spanning-tree.png` | STP estabilizado por VLAN |

---

## 🧪 Pruebas de Conectividad

| Origen | Destino | IPv4 | IPv6 |
|---|---|---|---|
| PC VLAN10 (RRHH) | PC VLAN40 (GERENCIA) | ✅ | ✅ |
| PC VLAN20 (VENTAS) | Server VLAN30 (TI) | ✅ | ✅ |
| PC VLAN10 | Internet (8.8.8.8) | ✅ | ✅ |
| PC VLAN40 | Internet | ✅ | ✅ |

---

## 📁 Archivos del Laboratorio

```
lab-02-conmutacion-redes/
├── README.md
├── examen-final-conmutacion.pkt     ← Archivo Cisco Packet Tracer
└── capturas/
    ├── 01-topologia-completa.png
    ├── 02-etherchannel-summary.png
    └── ...
```

---

## 🔗 Tecnologías y Protocolos

`Cisco IOS` `OSPFv2` `OSPFv3` `OSPF Multiarea` `MD5 Auth` `NAT Overload` `ACL` `VLANs` `Port Security` `EtherChannel` `STP` `DHCP` `NTP` `Syslog` `IPv4` `IPv6` `Dual Stack`

---

*Hans Soto González · IPLACEX · Ingeniería en Ciberseguridad · 2025*  
*[linkedin.com/in/hans-soto](https://linkedin.com/in/hans-soto) · [Portafolio](https://github.com/hanssoto-cyber/Portafolio-soc)*
