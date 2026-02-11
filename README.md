# DHCP-STARVATION

## Video (máx. 8 minutos)
[![Ver video en YouTube](https://img.youtube.com/vi/pKLjDihj_vw/0.jpg)](https://youtu.be/pKLjDihj_vw)

# Informe Técnico: Denegación de Servicio mediante DHCP Starvation

## I-) Introducción y Objetivos del Script.
En el presente documento veremos el script con el objetivo principal de causar una denegación de servicio (DoS) en el servidor DHCP de la red local, este script se ejecuta para agotar totalmente el espacio de direcciones disponibles (Address Pool). La herramienta emplea la biblioteca Scapy para llevar a cabo una inundación masiva de paquetes DHCP DISCOVER, los cuales se generan con direcciones MAC de origen único y aleatorias. El servidor (el router R1 en la topología) asigna una dirección IP a cada cliente ficticio, asumiendo que son dispositivos reales, al recibir estas solicitudes. Cuando el conjunto de direcciones alcanza su límite, el servidor legítimo no puede contestar a las demandas de los dispositivos auténticos y los usuarios nuevos quedan sin conexión a la red.

<img width="456" height="436" alt="image" src="https://github.com/user-attachments/assets/b85621cf-5130-44e5-8808-26ae3c1f1d53" />

---

## II-) Topología y Escenario de Laboratorio
Se ha implementado el entorno de pruebas en PNETLab, con una infraestructura que emula una topologia del ITLA. El núcleo de la red consiste en un router R1 que tiene la dirección IP 10.24.9.1 y sirve como servidor DHCP primario, conectado a una serie de switches Cisco (Switch 1 y Switch 2). La dirección IP 10.24.9.18 es usada por la máquina atacante (Kali Linux), que está conectada al puerto Gi0/1 del Switch 1.

<img width="829" height="879" alt="Screenshot 2026-02-10 213221" src="https://github.com/user-attachments/assets/691996a8-b036-4a84-8bc4-8a58d23557e0" />


El propósito final es dejar aislada a la víctima, conocida como "POBRE ALMA EN PENA", que al intentar actualizar su IP se topa con un servidor colapsado y sin capacidad para responder.
<img width="463" height="460" alt="Screenshot 2026-02-10 213413" src="https://github.com/user-attachments/assets/8f38f0e4-a007-48a8-b5a6-01759b18b627" />

---

## III-) Parámetros de Ejecución y Requisitos
El script solicita una interfaz de red activa en modo promiscuo y privilegios de superusuario (root) para operar adecuadamente, puesto que es esencial manejar paquetes a nivel de Capa 2. Los parámetros esenciales que se emplean son:

- **Interfaz (-i):** Especifica la tarjeta de red (ej. eth0) conectada al segmento objetivo.  
- **Número de paquetes (-n):** Define la cantidad de solicitudes; el valor por defecto es 300, diseñado para desbordar una subred estándar /24 (254 IPs).  
- **Modo Persistente (-p):** Es una función crítica que mantiene el ataque en bucle infinito, re-enviando solicitudes periódicamente para evitar que las concesiones (leases) del servidor expiren y las IPs vuelvan a estar libres.
<img width="319" height="22" alt="Screenshot 2026-02-10 213923" src="https://github.com/user-attachments/assets/ce47a343-934f-40ba-b389-7fad3307d78b" />

---

## IV-) Análisis de Resultados y Evidencia
Se comprueba la eficacia del ataque al examinar los registros en la terminal de Kali, que muestran ráfagas de paquetes que se enviaron con éxito.
<img width="319" height="418" alt="Screenshot 2026-02-10 214239" src="https://github.com/user-attachments/assets/17a81a6b-15e3-4440-b7fd-c668413e7525" />

En la consola del switch, el puerto de la víctima queda en una condición de espera indefinida y, al lanzar el comando `ipconfig /renew` en un equipo con Windows, se genera el error: **"No se puede contactar con su servidor DHCP". "Se ha agotado el tiempo de la solicitud".**
<img width="694" height="214" alt="Screenshot 2026-02-10 214451" src="https://github.com/user-attachments/assets/a54800ed-de68-4a75-99cf-af04d2ea7b57" />

Esto evidencia que el router R1 ya no está procesando solicitudes. Asimismo, utilizando comandos de validación en el router, como `show ip dhcp binding`, es posible verificar que la tabla está colmada de entradas relacionadas con las direcciones MAC aleatorias producidas por el script.
<img width="816" height="677" alt="Screenshot 2026-02-10 214754" src="https://github.com/user-attachments/assets/8b543801-5e0b-4684-a90e-757e4b6b037c" />

---

## V-) Medidas de Mitigación y Seguridad
Para evitar este tipo de ataques en los entornos de producción, es esencial que se aplique **DHCP Snooping** a los switches de acceso. Esta tecnología posibilita la designación de puertos como "confiables" (los que están conectados al servidor DHCP real) y como "no confiables" (puertos de usuario), impidiendo cualquier mensaje DHCP no autorizado que llegue desde un cliente. El uso de **Port Security** es crucial para restringir el número de direcciones MAC que cada puerto físico permite; si un puerto identifica cientos de MACs diferentes en segundos (como lo hace este script), el switch tiene la capacidad de desconectar automáticamente la interfaz, lo que elimina el ataque desde su raíz.

---

