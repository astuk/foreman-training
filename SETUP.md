# Foreman Training

## Initialisieren:

Es werden zwei Vagrant Plugins eingesetzt:

    vagrant plugin install vagrant-hostmanager
    vagrnat plugin install vagrant-vbguest

Jetzt kann die VM instantiiert werden:

    cd vagrant
    vagrant up foreman.example42.training

Danach Login:

    vagrant ssh foreman.example42.training
    sudo -i
    foreman-installer -i

Den Output sichern. z.B.:

      Success!
      * Foreman is running at https://foreman.example42.training
          Initial credentials are admin / DhXNVncksVCTxSrF
      * Foreman Proxy is running at https://foreman.example42.training:8443
      * Puppetmaster is running at port 8140
      The full log is at /var/log/foreman-installer/foreman.log

Nun muessen die Dienste konfiguriert werden:

    puppet apply /vagrant_foreman/scripts/00_router_config.pp
    puppet apply /vagrant_foreman/scripts/01_install_service_dhcp.pp
    puppet apply /vagrant_foreman/scripts/02_install_service_bind.pp
    puppet apply /vagrant_foreman/scripts/03_foreman_proxy.pp

Achtung! Dieses Setup kann nur einmal durchgefuehrt werden.
Spaetestens das Provisionierungs Setup aendert diese Einstellungen grundlegend.

## Foreman Einrichtung fertigstellen

### Smart proxies

Foreman Login -> Infrastructure -> Smart proxies

Actions -> Refresh

Klick auf 'foreman.example42.training'

Auf aktive Services und Fehler pruefen.

### Puppet Environments

    puppet module install garethr-docker
    puppet module install puppetlabs-ntp

Foreman Login -> Configure -> Puppet Environments -> 'Import environments from foreman.example42.training'

Haken setzen -> Update

### Netzwerk

Foreman Login -> Infrastructure -> Subnets -> Create Subnet

Tab Subnet: Name: example42.training
Tab Subnet: Network Address: 10.100.10.0
Tab Subnet: Network Prefix: 24
Tab Subnet: Primary DNS Server: 10.100.10.101
Tab Subnet: IPAM: DHCP
Tab Subnet: Start of IP Range: 10.100.10.120
Tab Subnet: End of IP Range: 10.100.10.240

Tab Domains: Auswaehlen 'example42.training'

Tab Proxies: DHCP, TFTP und DNS Proxy (Smart Proxy): foreman.example42.training

Submit

### Domain

Foreman Login -> Infrastructure -> Domains

DNS Proxy -> 'foreman.example42.training' -> Submit

### Provisioning Templates

Foreman Login -> Hosts -> Provisioning Templates

Templates unterscheiden sich nach Funktionalitaet:

  - PXELinux, PXEGrub, PXEGrub2 : werden auf den TFTP Server deployed
  - Provision : Kickstart oder Preseed fuer unattended Installation
  - Finish : Post-Install Scripts
  - user_data: Post-Install fuer Cloud VMs
  - Script : generische Skripte
  - iPXE : {g,i}PXE anstelle von PXE


Normalerweise werden nur die ersten 3 benoetigt.

Standard Templates sind "Locked". Niemals Standard Templates editieren, immer Klonen!

Beispiel Suchen: kind=PXELinux

#### Provisionierungs Templates mit OS Assoziieren

Foreman Login -> Hosts -> Provisioning Templates

1. Stelle: Templates mit OS Assoziieren

"Kickstart default PXELinux" auswaehlen -> Association -> OS auswaehlen

"Kickstart default" auswaehlen -> Association -> OS auswaehlen

"Kickstart default finish" auswaehlen -> Association -> OS auswaehlen

"Build PXE Default"

Foreman Login -> Hosts -> Operating Systems

2. Stelle: OS mit Templates Assoziieren

Partition Table -> "Kickstart default" auswaehlen
Installation media -> "CentOS mirror" auswaehlen
Templates -> alle Templates auswaehlen

Submit

ACHTUNG: beim Verwenden der default templates muss safemode : false gesetzt werden.
Foreman Login -> Administrator -> Settings


### Provisionieren

Foreman Login -> Infrastructure -> Provisioning Setup

Pre-requisites: Provisioning Network: 10.100.10.101/24 -> Submit
Network config: -> Submit (Bei Fehler das Netzwerk umbenennen)
Foreman Installer: 'Install provisioning with DHCP' -> Kopieren und Ausfuehren -> Next
Installation Media: 'CentOS mirror' -> Submit

ACHTUNG: nach dem foreman installer Kommando unbedingt die /etc/named/options.conf forwarders pruefen!!

### Host Group

Host Group braucht das Puppet Environment

Foreman Login -> Configure -> Host Groups
"Provision from foreman.example42.training" auswaehlen
Host Group: Puppet Environment setzen (Production)
Operating System: PXE Loader auswaehlen (PXELinux BIOS)

Submit

## Docker Setup

New Host
Host: Host Group -> Provision from foreman.example42.training
Host: Deploy on -> Bare Metal
Host: Environment -> production
Host: Puppet Master: foreman.example42.training
Host: Puppet CA: foreman.example42.training

Puppet Classes: docker

Interfaces: Edit
Eintragen: Mac Addresse und DNS Name (FQDN)

Hinweis: VirtualBox erzeugen und MAC Adresse in Virtualbox auslesen.

Oprating System: root password

Hardware model: VirtualBox


Jetzt die VM booten....