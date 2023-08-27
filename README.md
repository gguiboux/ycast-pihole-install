# ycast-pihole-install
Documentation qui permet d'installer ycast + pihole sur un raspberry pi

# pihole

Installation : 
curl -sSL https://install.pi-hole.net | bash

On se laisse guider, on choisit une ip fixe (ou router modem l'assigne automatiquement ou le configurer dans le router)
On choisi le dns par défaut de google 
On fait yes à tout le reste

#config-pihole:

On créé une conf spécifique pour la redirection des requêtes de notre ampli marantz:
sudo nano /etc/dnsmasq.d/vtuner.conf

Le fichier (on pointe vers notre raspbery-pi):
address=/radiomarantz.vtuner.com/192.168.1.51
address=/vtuner.com/192.168.1.51

On relance les services pi-hole:
pihole restardns

On regarde si Ok:

pihole status

#ycast

Les librairies :

sudo apt-get install libtiff5
sudo apt-get install libopenjp2-7

Ycast :

sudo apt install python3-pip
sudo pip3 install ycast

#config-ycast:

on ajoute un user: 
sudo useradd ycast

on créé un répertoire qui va contenir nos stations:
sudo nano /etc/ycast/stations.yml

on choisi ycast comme propriétaire:
sudo chown ycast:ycast /etc/ycast/stations.yml

#ycast-service

Voici la config qui permet de lancer le service ycast avec le user pointant sur le fichier stations.yml

[Unit]
Description=YCast internet radio service
After=network.target

[Service]
Type=simple
User=ycast
Group=ycast
ExecStart=/usr/bin/python3 -m ycast -l 127.0.0.1 -p 8010 -c /etc/ycast/stations.yml

[Install]
WantedBy=multi-user.target

#light-pdd (le serveur):

On active le service:
sudo lighttpd-enable-mod

On créer le fichier de conf:
sudo nano /etc/lighttpd/conf-enabled/10-proxy.conf

Le fichier de conf qui permet de catch les requêtes vtuner:

#YCAST
$HTTP["host"] =~ "vtuner.com" {
        proxy.server = ( "" =>  ( (
                                "host" => "127.0.0.1",
                                "port" => "8010"
                                 ) ) )
        proxy.forwarded = (
                        "for"          => 1,
                        "proto"        => 1,
                        "host"        => 1,
                        #"by"          => 1,
                        #"remote_user" => 1
    )

}

On relance les services : 

sudo service lighttpd force-reload
systemctl status lighttpd.service

#router

Il faut absolument aller dans la conf dns (dhcp) de son router et configurer comme dns principal l'adresse de notre raspberry (192.168.1.51)

Restart toutes les machines sur le réseau (ampli, raspberry pi) 

#les stations (stations.yml)

Attention à l'indentation du yml 

Fip:
  FipMain: https://icecast.radiofrance.fr/fip-midfi.mp3?id=radiofrance
  FipRock: https://icecast.radiofrance.fr/fiprock-midfi.mp3?id=radiofrance
  FipElectro: https://icecast.radiofrance.fr/fipelectro-hifi.aac?id=radiofrance
  FipNouveaute: https://icecast.radiofrance.fr/fipnouveautes-hifi.aac?id=radiofrance

Autres:
  Nova: http://radionova.ice.infomaniak.ch/radionova-256.aac
  Meuh: http://radiomeuh.ice.infomaniak.ch/radiomeuh-128.mp3
  Phenix: https://live.radio-campus.org:8002/caen.mp3
  CanalB: http://stream.levillage.org/canalb

