# homelab-pfsense
Déploiement et configuration d'un firewall pfSense sur VirtualBox pour segmenter le réseau du homelab SOC. Intégration des logs dans Wazuh pour centraliser les alertes réseau et de sécurité.

# Homelab SOC - Déploiement pfSense

## Objectif
Déploiement d'un firewall open source pfSense sur VirtualBox pour segmenter le réseau du homelab SOC. L'objectif est d'isoler les VMs agents du réseau domestique tout en maintenant la communication avec le Wazuh Manager.

## Architecture réseau
Internet / Box
|
pfSense (WAN 192.168.1.X / LAN 192.168.100.1)
|                         |
Bridge (192.168.1.x)      lab-network (192.168.100.x)
|                         |
Wazuh Manager              VMs agents W11
192.168.1.19               192.168.100.x (DHCP)

## Installation
### Prérequis
- Oracle VirtualBox
- ISO pfSense 2.8.1 AMD64 — téléchargeable sur pfsense.org
- VM dédiée avec 2 cartes réseau :
  - Adaptateur 1 : Mode Bridge (WAN)
  - Adaptateur 2 : Réseau interne "lab-network" (LAN)
- RAM : 1Go minimum
- Stockage : 10Go minimum

### Étapes
1) Créer la VM dans VirtualBox — Type BSD, Version FreeBSD 64-bit
2) Monter l'ISO pfSense et démarrer la VM
3) Suivre l'installation guidée — accepter les valeurs par défaut
4) Éjecter l'ISO avant le reboot final
5) Au premier démarrage, sélectionner les interfaces :
   - WAN → em0 (Adaptateur 1 Bridge)
   - LAN → em1 (Adaptateur 2 lab-network)
6) Noter l'IP WAN affichée dans le menu texte (ex: 192.168.1.34)
7) Depuis une VM sur le lab-network, accéder à l'interface web : https://192.168.100.1
8) Identifiants par défaut : admin / pfsense
9) Changer l'IP du LAN : Interfaces → LAN → IPv4 Address → 192.168.100.1/24
10) Activer le DHCP : Services → DHCP Server → LAN → Enable → Plage 192.168.100.100 à 192.168.100.200
11) Vérifier le routage depuis une VM agent :
```cmd
ping 192.168.1.19
```

## Configuration
### Interfaces réseau

| Interface | Mode VirtualBox | IP | Rôle |
|-----------|----------------|-----|------|
| WAN (em0) | Bridge | 192.168.1.34 | Accès internet / réseau domestique |
| LAN (em1) | Réseau interne lab-network | 192.168.100.1 | Réseau isolé agents |

### DHCP
- Interface : LAN
- Plage : 192.168.100.100 → 192.168.100.200
- Passerelle distribuée : 192.168.100.1

### Versions
- pfSense : 2.8.1-RELEASE (amd64)
- FreeBSD : 15.0-CURRENT

## Problèmes rencontrés et solutions
1) Interface web inaccessible depuis la machine hôte après installation — Cause : pfSense bloque l'accès web depuis le WAN par défaut. Solution : accéder uniquement depuis une VM sur le LAN via https://192.168.100.1
2) Erreur lors du changement d'IP du LAN : "DHCPv6 Server is active" — Solution : désactiver le DHCPv6 et les Router Advertisements sur le LAN avant de modifier l'IP. Services → DHCPv6 Server & RA → LAN → tout désactiver
3) DHCP ne distribue pas d'IP aux VMs — Solution : vérifier que le service dhcpd est bien running dans Status → Services. Si absent, retourner dans Services → DHCP Server → LAN et re-sauvegarder la configuration
4) Accès perdu à l'interface web après changement d'IP du LAN — Solution : configurer manuellement une IP statique en 192.168.100.x sur la carte lab-network de la VM Windows pour retrouver l'accès

## Résultats et tests
- Routage entre lab-network et Wazuh Manager opérationnel ✅
- VMs agents isolées du réseau domestique ✅
- DHCP fonctionnel sur le LAN ✅
- Test ping depuis lab-network vers Wazuh Manager (192.168.1.19) : succès ✅

## Prochaines étapes
- Intégrer les logs pfSense dans Wazuh ✅ => https://github.com/Bangalolz/homelab-wazuh.git
