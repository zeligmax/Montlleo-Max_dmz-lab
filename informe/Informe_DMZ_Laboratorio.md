# Informe de configuración de DMZ con Cisco Packet Tracer

### 1. Objetivo del laboratorio

El objetivo de este laboratorio fue configurar una **Zona Desmilitarizada (DMZ)** segura usando un router Cisco ISR, aplicando NAT estático y ACLs para controlar el tráfico entre la red interna (LAN), la DMZ y la red externa (Internet). Se buscó garantizar que el servidor web en la DMZ fuera accesible desde Internet por HTTP, mientras se protegía la LAN interna y se bloqueaban pings no deseados.

---

### 2. Topología implementada

* **Cantidad de redes:** 3 (LAN interna, DMZ, Externa/Internet)
* **Dispositivos usados:**

  * 1 Router Cisco ISR (Router\_FW)
  * 3 Switches Cisco 2960
  * 1 PC Interna (PC\_Internal)
  * 1 Servidor Web DMZ (Server\_DMZ)
  * 1 PC Externa (PC\_External)
* **Descripción de la función de cada zona:**

  * **LAN interna:** Red segura de la organización donde residen los usuarios.
  * **DMZ:** Red intermedia que aloja servicios públicos (servidor web) accesibles desde Internet.
  * **Red Externa/Internet:** Simula clientes externos que acceden a los servicios públicos.

### 3. Plan de direccionamiento IP

| Dispositivo            | IP           | Máscara       | Gateway     |
| ---------------------- | ------------ | ------------- | ----------- |
| PC\_Internal           | 192.168.1.10 | 255.255.255.0 | 192.168.1.1 |
| Server\_DMZ            | 192.168.2.10 | 255.255.255.0 | 192.168.2.1 |
| PC\_External           | 192.168.3.10 | 255.255.255.0 | 192.168.3.1 |
| Router\_FW Gi0/0 (LAN) | 192.168.1.1  | 255.255.255.0 | -           |
| Router\_FW Gi0/1 (DMZ) | 192.168.2.1  | 255.255.255.0 | -           |
| Router\_FW Gi0/2 (Ext) | 192.168.3.1  | 255.255.255.0 | -           |

### 4. Configuración aplicada (resumen)

* **Interfaces configuradas:**

```bash
interface GigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown
exit

interface GigabitEthernet0/1
 ip address 192.168.2.1 255.255.255.0
 ip nat inside
 no shutdown
exit

interface GigabitEthernet0/2
 ip address 192.168.3.1 255.255.255.0
 ip nat outside
 no shutdown
exit
```

* **NAT estático:**

```bash
ip nat inside source static 192.168.2.10 192.168.3.1
```

* **ACLs aplicadas:**

```bash
! ACL para bloquear ping de Internet y permitir HTTP
access-list 100 deny icmp any host 192.168.3.1
access-list 100 permit tcp any host 192.168.3.1 eq 80
interface GigabitEthernet0/2
 ip access-group 100 in
exit

! ACL para bloquear DMZ -> LAN
access-list 110 deny ip 192.168.2.0 0.0.0.255 192.168.1.0 0.0.0.255
access-list 110 permit ip any any
interface GigabitEthernet0/1
 ip access-group 110 in
exit
```

### 5. Verificaciones realizadas

* `ping` desde PC\_Internal al router LAN: ✔ OK
* `ping` desde PC\_External a 192.168.3.1: ❌ Bloqueado correctamente
* Acceso web desde PC\_External a 192.168.3.1: ✔ OK
* Acceso web desde PC\_Internal a 192.168.2.10: ✔ OK
* Ping desde Server\_DMZ a PC\_Internal: ❌ Bloqueado correctamente
* NAT estático funcionando: ✔ OK

### 6. Conclusiones y recomendaciones

* Aprendí a configurar una DMZ segura con **NAT y ACLs**, controlando correctamente el acceso externo e interno.
* Es importante **verificar la conectividad básica** antes de aplicar ACLs para no bloquear tráfico legítimo accidentalmente.
* Recomiendo **bloquear explícitamente ICMP** cuando se quiere impedir pings desde Internet, manteniendo accesibles los servicios autorizados.

### 7. Capturas de evidencia

Para ver las capturas ir a [evidencias]()