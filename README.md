# DMVPN Fase 2 — IKEv1 con OSPF
**Autor:** Yeury Lopez | **Matrícula:** 20250780 | **Repositorio:** YeuryLopez_20250780_DMVPN-Phase2-IKEv1

---

## 1. Objetivo
Configurar una VPN Hub and Spoke punto a multipunto DMVPN Fase 2 con IKEv1 y OSPF. En la Fase 2, los Spokes pueden comunicarse directamente entre sí sin pasar por el Hub mediante NHRP (Next Hop Resolution Protocol).

---

## 2. Topología

```
              [HUB] (LAN: 192.168.7.0/24)
             /     \
          [ISP]
         /     \
[SPOKE1]       [SPOKE2]
(LAN:          (LAN:
192.168.80.0)  192.168.78.0)
```

> 📸 **SCREENSHOT:** Insertar captura de la topología DMVPN completa en EVE-NG

---

## 3. Direccionamiento IP

| Dispositivo | Interfaz | Dirección IP | Rol |
|---|---|---|---|
| HUB | Fa0/0 | 192.168.7.1/24 | LAN |
| HUB | Fa1/0 | 10.7.82.1/30 | WAN → ISP |
| HUB | Tunnel0 | 172.16.78.1/24 | DMVPN NHS |
| ISP | Fa0/0 | 10.7.82.2/30 | WAN → HUB |
| ISP | Fa3/0 | 10.7.83.1/30 | WAN → SPOKE1 |
| ISP | Fa1/0 | 10.7.84.1/30 | WAN → SPOKE2 |
| SPOKE1 | Fa0/0 | 10.7.83.2/30 | WAN → ISP |
| SPOKE1 | Fa1/0 | 192.168.80.1/24 | LAN |
| SPOKE1 | Tunnel0 | 172.16.78.2/24 | DMVPN |
| SPOKE2 | Fa0/0 | 10.7.84.2/30 | WAN → ISP |
| SPOKE2 | Fa1/0 | 192.168.78.1/24 | LAN |
| SPOKE2 | Tunnel0 | 172.16.78.3/24 | DMVPN |

---

## 4. Parámetros VPN

| Parámetro | Valor |
|---|---|
| Tipo | DMVPN Fase 2 IKEv1 |
| Protocolo Túnel | mGRE (Multipoint GRE) |
| NHRP Network-ID | 780 |
| NHRP Holdtime | 300 segundos |
| Hub NHS IP Tunnel | 172.16.78.1 |
| Enrutamiento Dinámico | OSPF Proceso 1 — Area 0 |
| Cifrado | AES-128 SHA-1 |
| Modo IPSec | Transport |
| Pre-Shared Key | Yeury0780 |

---

## 5. Configuración

### 5.1 Configuración HUB

> 📸 **SCREENSHOT:** Insertar captura del `show running-config` del HUB mostrando Tunnel0 con `tunnel mode gre multipoint`, `ip nhrp map multicast dynamic` y OSPF

```
interface Tunnel0
 ip address 172.16.78.1 255.255.255.0
 no ip redirects
 ip nhrp map multicast dynamic
 ip nhrp network-id 780
 ip nhrp holdtime 300
 tunnel source FastEthernet1/0
 tunnel mode gre multipoint
 tunnel protection ipsec profile DMVPN-PROFILE
 ip ospf network point-to-multipoint
 ip ospf priority 255
router ospf 1
 network 172.16.78.0 0.0.0.255 area 0
 network 192.168.7.0 0.0.0.255 area 0
```

### 5.2 Configuración SPOKE1

> 📸 **SCREENSHOT:** Insertar captura del `show running-config` del SPOKE1 mostrando `ip nhrp nhs` y OSPF

```
interface Tunnel0
 ip address 172.16.78.2 255.255.255.0
 ip nhrp map 172.16.78.1 10.7.82.1
 ip nhrp map multicast 10.7.82.1
 ip nhrp network-id 780
 ip nhrp nhs 172.16.78.1
 tunnel source FastEthernet0/0
 tunnel mode gre multipoint
 tunnel protection ipsec profile DMVPN-PROFILE
 ip ospf network point-to-multipoint
 ip ospf priority 0
```

---

## 6. Verificación y Funcionamiento

### 6.1 Estado DMVPN en HUB

Ejecutar en HUB:
```
show dmvpn
```
> 📸 **SCREENSHOT:** Insertar captura mostrando ambos Spokes registrados con estado **UP** y tipo **Hub**

### 6.2 Estado OSPF en HUB

Ejecutar en HUB:
```
show ip ospf neighbor
```
> 📸 **SCREENSHOT:** Insertar captura mostrando ambos Spokes en estado **FULL**

### 6.3 Estado ISAKMP SA

Ejecutar en HUB:
```
show crypto isakmp sa
```
> 📸 **SCREENSHOT:** Insertar captura mostrando sesiones IKEv1 **QM_IDLE ACTIVE** para cada Spoke

### 6.4 Tabla de rutas

Ejecutar en HUB:
```
show ip route
```
> 📸 **SCREENSHOT:** Insertar captura mostrando rutas **O (OSPF)** hacia las LANs de los Spokes

### 6.5 Demostración de conectividad

Ejecutar en Linux-SP1:
```
ping -c 4 192.168.7.2
ping -c 4 192.168.78.2
```
> 📸 **SCREENSHOT:** Insertar captura del ping exitoso hacia LAN del HUB y LAN de SPOKE2

---

## 7. Archivos del repositorio

| Archivo | Descripción |
|---|---|
| `YeuryLopez_20250780_Script_P7.txt` | Script de configuración |
| `YeuryLopez_20250780_Informe_P7.pdf` | Documentación técnica en PDF |
| `YeuryLopez_20250780_Links_P7.txt` | Enlace al video |
| `README.md` | Este archivo |

---

## Video de demostración — DMVPN Fase 2 IKEv1 con OSPF

[![Ver en YouTube](https://img.shields.io/badge/YouTube-Ver%20Video-red?logo=youtube)](https://youtu.be/4rDdH7w28Qo)

**Enlace directo:** https://youtu.be/4rDdH7w28Qo

**Playlist completa:** https://www.youtube.com/playlist?list=PLMmRZwuxbsUo
