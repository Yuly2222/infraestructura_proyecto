# Proyecto Infraestructura de TI — AndesMiner S.A. (2026-1)

## Introducción
AndesMiner S.A. es una compañía colombiana dedicada a la extracción y procesamiento de minerales, con operación distribuida entre:

- **Corporate Headquarters (HQ): 2000 empleados**
- **Regional Office: 1000 empleados**
- **Remote Mining Site: 1000 empleados**

Debido a conectividad limitada e intermitente en sitios remotos, la compañía adopta un **modelo híbrido**: servicios críticos on-premise que funcionan localmente, complementados por servicios en la nube para exposición pública y capacidades específicas (por ejemplo, asistente IA).

Este entregable incluye:

- Diagrama de arquitectura con zonas y servicios.
- Segmentación de red con direccionamiento IP privado.
- Tabla de servidores con roles y exposición.

---

## Objetivos y Alcance

### Objetivos funcionales
- Implementar **Active Directory** como sistema central de identidad y aplicación de GPO.
- Implementar **Keycloak** como proveedor central de autenticación (SSO).
- Integrar **Zammad** delegando autenticación a Keycloak.
- Implementar **Enterprise File Sharing** con mínimo 15GB por usuario.
- Implementar **Jump Server/Bastion** como único punto administrativo.
- Implementar **sistema de monitoreo** para CPU, RAM, disco y red.
- Implementar en la nube:
  - Sitio web institucional público (HTTPS).
  - Asistente IA (LLaMA 2/3) en IaaS.
  - Storage serverless para repositorio documental.

### Objetivos de seguridad
- Separación en zonas de red:  
  **LAN, Internal Servers, DMZ, Management/Administration, Cloud Services, AI Services Zone**
- Control explícito de tráfico entre zonas mediante firewall.
- Uso obligatorio de FQDN + HTTPS.
- TLS 1.2/1.3 únicamente.
- Certificados internos (CA interna) y externos (CA pública).

### Fuera de alcance
- Alta disponibilidad obligatoria.
- Redundancia completa de hardware.

---

## Arquitectura General

La solución implementa un modelo híbrido compuesto por:

### On-Premise (HQ)
- Internal Servers:
  - Active Directory
  - Keycloak
  - Zammad
  - Enterprise File Sharing
- Management/Administration:
  - Jump Server
  - Monitoring
- LAN corporativa HQ

### Sedes Remotas
- Regional Office (LAN propia)
- Remote Mining Site (LAN propia)
- Conectadas mediante VPN Tunnel hacia HQ

### Cloud
- Website público (andeminer.com)
- DNS público
- Storage serverless
- AI Service (LLaMA) en zona aislada

---

## Segmentación de Red

### Criterios de diseño

1. Separación por zonas de confianza.
2. Crecimiento proyectado (5% a 5 años).
3. No solapamiento entre sedes.
4. Control granular mediante firewall.

---

## Plan de Direccionamiento IP

Rango privado base utilizado: `10.0.0.0/8`

### Bloques por sede

- HQ: `10.10.0.0/16`
- Regional Office: `10.20.0.0/16`
- Remote Mining Site: `10.30.0.0/16`
- Cloud: `10.40.0.0/16`

---

## Subredes por Zona

### HQ — 10.10.0.0/16

| Zona | Subred | Propósito |
|------|--------|------------|
| LAN HQ | 10.10.0.0/20 | Usuarios (2000+) y crecimiento |
| Internal Servers | 10.10.16.0/24 | AD, Keycloak, Zammad, EFS |
| DMZ | 10.10.17.0/24 | Servicios frontera |
| Management | 10.10.18.0/28 | Jump + Monitoring |
| VPN Clients | 10.10.19.0/24 | Pool usuarios remotos |

---

### Regional Office — 10.20.0.0/16

| Zona | Subred |
|------|--------|
| LAN Regional | 10.20.0.0/21 |

---

### Remote Mining Site — 10.30.0.0/16

| Zona | Subred |
|------|--------|
| LAN Mining | 10.30.0.0/21 |

---

### Cloud — 10.40.0.0/16

| Zona | Subred |
|------|--------|
| Cloud Services | 10.40.10.0/24 |
| AI Services Zone | 10.40.20.0/24 |

---

## Política de Seguridad entre Zonas

Política base: **deny all por defecto**.

Se habilita únicamente:

- LAN → Internal Servers: DNS, Kerberos/LDAP (según AD), HTTPS.
- Management → Internal: SSH/RDP exclusivamente desde Jump Server.
- Sedes → HQ: acceso por VPN únicamente.
- Internet → On-Prem: solo endpoint VPN.
- Cloud ↔ On-Prem: solo tráfico estrictamente necesario.

---

## Tabla de Servidores

| Server Name | Ubicación | IP | Rol |
|-------------|------------|----|-----|
| SRV-HQ-AD-001 | HQ | 10.10.16.10 | Active Directory + DNS |
| SRV-HQ-KC-001 | HQ | 10.10.16.20 | Keycloak |
| SRV-HQ-ZMD-001 | HQ | 10.10.16.30 | Zammad |
| SRV-HQ-EFS-001 | HQ | 10.10.16.40 | File Sharing |
| SRV-HQ-JMP-001 | HQ | 10.10.18.10 | Jump Server |
| SRV-HQ-MON-001 | HQ | 10.10.18.20 | Monitoring |
| SRV-CLD-WEB-001 | Cloud | 10.40.10.10 | Website Público |
| SRV-CLD-AI-001 | Cloud | 10.40.20.10 | LLaMA AI |
| SRV-CLD-STO-001 | Cloud | N/A | Storage Serverless |

---

## DNS

- Dominio público: `andeminer.com`
- Dominio interno: `internal.andeminer.local`
- Certificados internos mediante CA interna.
- Certificados externos mediante CA pública.

---

## Acceso Remoto

- Conectividad sedes y teletrabajo mediante VPN.
- Administración únicamente desde Jump Server.
- Soporte remoto mediante herramienta autohospedada (ej. RustDesk).
- Monitoreo centralizado para todos los servidores.

---

## Equipo de Trabajo

- Nicolas Esteban Munoz Sendoya — Rol
- Juan Jose Campos Covaleda — Rol
- Juan David Orozco Rodriguez — Rol
- Yuly Dayana Rodríguez Salcedo — Rol
