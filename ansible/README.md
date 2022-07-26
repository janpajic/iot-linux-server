### STAŽENÍ A PŘÍPRAVA ANSIBLE-PLAYBOOK
# Zkopírujte sourbor z M:\IoT\05 Ansible-playbook\ansible_22.04TLS.tar.gz do lokálního uložiště NS

## Kopůrování přes SSH:
'cd /home/sidat/
scp <cesta k ember souboru na pc>\ember-ansible.tar.gz ember@<ip linux servra>:/home/ember/
tar –xvzf ansible_22.04TLS.tar.gz –C /home/sidat/'

## Získání douborů z Gitu:
'cd /home/sidat/
sudo apt install git
git clone https://github.com/janpajic/iot-linux-server.git'


### KONFIGURACE SERVERU
## Konfigurace statické IP adresy:
'Cd /etc/netplan/'
# zadejte příkaz ¨ls¨, který vám zobrazí soubory ve složce /etc/netplan/. Zobrazený soubor upravte příkazem:
Sudo nano <název souboru>.yaml

# Otevře se něco soubor s přibližně stejným obsahem (tj. tvar dynamické IP):
'network: 
 ethernets:
  enp0s3:
  addresses: []
  dhcp4: true
 version: 2'

# Obsah konfiguračního souboru upravte do poboby:
#Pozn. K odsazení řádku používejte [TAB].
'network: 
 version: 2
 ethernets: 
  enp0s3:
   dhcp4: no
   addresses: [192.168.YOUR.NET/SUB]  (např. 192.168.80.205/24)
   gateway4: 192.168.80.1
   nameservers:
    addresses: [192.168.SI.DAT,192.168.SI.DAT]'

# Potvrďte stisknutím [Ctrl+X], stiskněte “Y” jako Yes a [ENTER].
# Vaši konfiguraci následně nastavíte pomocí příkazu:
'Sudo netplan apply'
#Pokud budete mít nějaké potíže využijte příkazu:
Sudo netplan --debug apply

# Instalace ansible:
'sudo apt install ansible'
 
# Zahájení instalace podle ansible-playbook:
'cd /home/sidat/iot-linux-server/ansible/
ansible-playbook ember-connect.yml -i hosts --limit sidat --ask-become-pass'

# Pozn. Zadejte heslo "sidat"
#Pokud error s key host cheking:
'export ANSIBLE_HOST_KEY_CHECKING=False'

# Přidat MQTT port:
'sudo nano /etc/mosquitto/mosquitto.conf'

# Dopsat do conf: 
'listener 1883 
allow_anonymous true' 


# Restart mosquito: 
'sudo service mosquitto stop 
systemctl status mosquitto.service  (pro kontrolu)
sudo service mosquitto start -v'


# Kontrola a dodatečné povoleni portů: 
netstat -lntu    # (pro kontrolu, konfigurace mosquitto může v některých případech uzavřít ostatní otevřené porty (8080 a 1880)
'sudo ufw allow 1883
sudo ufw allow 8080
sudo ufw allow 1880'

### Porty služeb po instalaci:

ChirpStack:	 8080 (admin:admin)
Node-Red:	 1880 (admin:admin)
MQTT broker:	 1883

8.3.4.5.	Ansible trouble shooting
Pokud zpuštíte instlaci po druhé, třetí ... je nutné některé kroky v /home/ember/ansible/install.yml zakomentovat "#" (aby se při dalších instalacích neprováděly, protože jsou již plně vytvořeny a brání v úspěšném pokračování instalace). Nejčastěji se jedná o úlohu Node-Red a Chirpstark.
