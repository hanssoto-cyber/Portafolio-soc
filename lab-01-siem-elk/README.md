# 🔵 Lab 01 — Detección de Escaneo de Puertos con SIEM ELK Stack

> **Laboratorio SOC L1 · IPLACEX · Ingeniería en Ciberseguridad · 2026**

## Descripción

Implementación de un entorno SIEM completo con ELK Stack 8.19 para la detección en tiempo real de un ataque de reconocimiento de red (port scan) ejecutado con Nmap. El laboratorio simula el flujo completo de un analista SOC L1: desde la configuración de la infraestructura de monitoreo hasta la documentación del incidente en formato NIST SP 800-61r2.

---

## 🏗️ Arquitectura del Laboratorio

```
┌─────────────────┐        ┌─────────────────┐        ┌─────────────────┐
│   Kali Linux    │        │  Ubuntu Server  │        │ Ubuntu Víctima  │
│  192.168.196.132│        │  192.168.196.131│        │ 192.168.196.134 │
│                 │        │                 │        │                 │
│  Atacante       │──────▶ │  ELK Stack 8.19 │ ◀───── │  Filebeat 8.19  │
│  Nmap 7.99      │        │  Elasticsearch  │        │  Apache + FTP   │
│  Wireshark      │        │  Logstash       │        │  SSH            │
│                 │        │  Kibana :5601   │        │                 │
└─────────────────┘        └─────────────────┘        └─────────────────┘
         │                          │                          │
         └──────────────────────────┴──────────────────────────┘
                          Red Host-Only VMware
                          192.168.196.0/24
```

---

## 🛠️ Herramientas Utilizadas

| Herramienta | Versión | Rol |
|---|---|---|
| Elasticsearch | 8.19.15 | Motor de búsqueda e indexación de logs |
| Logstash | 8.19.15 | Pipeline de procesamiento de logs |
| Kibana | 8.19.15 | Dashboard y visualización SIEM |
| Filebeat | 8.19.15 | Agente de ingesta de logs |
| Nmap | 7.99 | Herramienta de reconocimiento (atacante) |
| Wireshark | — | Análisis de tráfico de red |
| VMware Workstation | — | Virtualización del entorno |

---

## 📋 Fases del Laboratorio

### Fase 1 — Preparación del Entorno
- Despliegue de 3 VMs en red host-only aislada (192.168.196.0/24)
- Instalación y configuración de ELK Stack 8.19 en Ubuntu Server 22.04
- Configuración de pipeline: Filebeat → Logstash:5044 → Elasticsearch → Kibana

### Fase 2 — Configuración del SIEM
- Activación de módulo `system` en Filebeat para ingesta de logs del SO
- Configuración de Logstash con input Beats (puerto 5044) y output Elasticsearch
- Creación de Data View en Kibana: `filebeat-*`
- Verificación de índice `filebeat-8.19.15-2026.05.19` con eventos en tiempo real

### Fase 3 — Simulación del Ataque
```bash
# Escaneo completo desde Kali Linux
nmap -sV -sC -O -p- --open -T4 192.168.196.134 -oA lab01_victima_scan
```

**Resultados del escaneo:**
| Puerto | Servicio | Versión | Riesgo |
|---|---|---|---|
| 21/tcp | FTP | vsftpd | ⚠️ Sin cifrado |
| 22/tcp | SSH | OpenSSH 10.2p1 | ✅ Seguro |
| 80/tcp | HTTP | Apache httpd | ⚠️ Sin HTTPS |

### Fase 4 — Detección en Kibana
- Spike detectado: **~100 eventos/minuto** a las 15:57 (normal: 1-15 eventos/min)
- Visualización creada en Kibana Analytics con pico de tráfico visible
- Regla de alerta configurada: **Elasticsearch Query Rule > 50 eventos/minuto**

### Fase 5 — Documentación
- Incidente documentado en formato **NIST SP 800-61r2**
- Clasificación: **Verdadero Positivo — Reconocimiento de Red**
- Táctica MITRE ATT&CK: **T1046 — Network Service Discovery**

---

## 📸 Capturas del Laboratorio

| # | Captura | Descripción |
|---|---|---|
| 01 | `capturas/01-arquitectura-vms.png` | Configuración de red de las 3 VMs |
| 02 | `capturas/02-elk-servicios-activos.png` | Elasticsearch, Kibana y Logstash activos |
| 03 | `capturas/03-kibana-dashboard.png` | Dashboard principal de Kibana |
| 04 | `capturas/04-indice-filebeat.png` | Índice filebeat en Elasticsearch |
| 05 | `capturas/05-kibana-discover-logs.png` | Logs de la VM víctima en Kibana Discover |
| 06 | `capturas/06-nmap-corriendo.png` | Escaneo Nmap en progreso |
| 07 | `capturas/07-wireshark-syn-flood.png` | Tráfico SYN masivo en Wireshark |
| 08 | `capturas/08-kibana-spike-eventos.png` | Spike de eventos durante el ataque |
| 09 | `capturas/09-nmap-resultado-final.png` | Resultado completo del escaneo |
| 10 | `capturas/10-kibana-spike-scan2.png` | Segundo escaneo detectado |
| 11 | `capturas/11-kibana-visualizacion-ataque.png` | Gráfico del ataque en Kibana Analytics |
| 12 | `capturas/12-kibana-regla-alerta.png` | Regla de detección configurada |

---

## 📄 Documentación

- 📋 [Informe de Incidente NIST SP 800-61r2](./Informe_NIST_Lab01_SOC_Hans_Soto.docx)

**Resumen del incidente:**
- **ID:** INC-2026-001
- **Severidad:** MEDIO
- **Tipo:** Reconocimiento — Escaneo de Puertos
- **Clasificación:** Verdadero Positivo
- **MITRE ATT&CK:** T1046 — Network Service Discovery
- **Estado:** CERRADO

---

## 🔑 Lecciones Aprendidas

| Hallazgo | Acción Correctiva | Prioridad |
|---|---|---|
| FTP expuesto sin cifrado | Reemplazar por SFTP | 🔴 Alta |
| HTTP sin HTTPS | Implementar SSL en Apache | 🔴 Alta |
| Sin reglas de firewall | Configurar UFW con política restrictiva | 🔴 Alta |
| Sin alertas automáticas | Regla creada en Kibana ✅ | 🟡 Completado |
| Logs sin centralizar | ELK Stack implementado ✅ | 🟡 Completado |

---

## 🔗 Referencias

- [NIST SP 800-61r2](https://csrc.nist.gov/publications/detail/sp/800-61/rev-2/final)
- [MITRE ATT&CK T1046](https://attack.mitre.org/techniques/T1046/)
- [ELK Stack Documentación](https://www.elastic.co/guide/index.html)

---

*Hans Soto González · Analista SOC L1 en Formación · IPLACEX 2026*  
*[linkedin.com/in/hans-soto](https://linkedin.com/in/hans-soto) · [Portafolio](https://github.com/hanssoto-cyber/Portafolio-soc)*
