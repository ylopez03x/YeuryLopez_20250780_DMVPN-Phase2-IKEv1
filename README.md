# DMVPN Fase 2 — IKEv1 con OSPF
**Autor:** Yeury Lopez | **Matrícula:** 20250780 | **Repositorio:** YeuryLopez_20250780_DMVPN-Phase2-IKEv1

---

## Video de demostración

[![Ver en YouTube](https://img.shields.io/badge/YouTube-Ver%20Video-red?logo=youtube)](https://youtu.be/4rDdH7w28Qo)

**Enlace directo:** https://youtu.be/4rDdH7w28Qo

---

## 1. Objetivo
Configurar una VPN Hub and Spoke punto a multipunto DMVPN Fase 2 con IKEv1 y OSPF. En la Fase 2, los Spokes pueden comunicarse directamente entre sí sin pasar por el Hub mediante NHRP (Next Hop Resolution Protocol).

---

## 2. Topología


> <img width="1323" height="1021" alt="image" src="https://github.com/user-attachments/assets/e1778720-1ac8-4e79-bd00-f18bf1033f4e" />


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
> <img width="1010" height="343" alt="image" src="https://github.com/user-attachments/assets/ea8a9644-aacd-4bde-9b1e-7936a5d167f1" />


### 6.2 Estado OSPF en HUB

Ejecutar en HUB:
```
show ip ospf neighbor
```
> <img width="925" height="271" alt="image" src="https://github.com/user-attachments/assets/29d01e86-3f16-4b57-b99a-be0b26a2dd17" />


### 6.3 Estado ISAKMP SA

Ejecutar en HUB:
```
show crypto isakmp sa
```
> <img width="940" height="195" alt="image" src="https://github.com/user-attachments/assets/2cf19a30-abba-4a37-9265-f98a32a734b8" />


### 6.4 Tabla de rutas

Ejecutar en HUB:
```
show ip route
```
> <img width="940" height="526" alt="image" src="https://github.com/user-attachments/assets/63fc8776-9c43-496f-b30f-969e138d8b59" />


### 6.5 Demostración de conectividad

Ejecutar en Linux-SP1:
```
ping -c 4 192.168.7.2
ping -c 4 192.168.78.2
```
> <img width="940" height="269" alt="image" src="https://github.com/user-attachments/assets/8c24c7fe-832d-4574-84f8-a1667189e462" />


---
