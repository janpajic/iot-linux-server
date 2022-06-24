## 8.4.	Instalace Chirpstack pomocí Ansible
Instalace Chirpstack a všech jeho závislostí lze provést v polo-automaticky pomocí Ansible-plybook (ten obsahuje soubory *.yml, které pro nás instalaci a konfiguraci všech komponent zajistí).
### 8.4.4.	Předpoklady pro úspěšnou instalaci

### 8.4.4.1.	OS
Ubuntu Server 21.04 – ostatní verze OS ubuntu (vč. Desktop) obsahují odlišnou konfiguraci, která povede k desítkám chyb.

*.ISO file vhodný pro instalaci naleznete na G:\Pajic\Iot Ubuntu server.

### 8.4.4.2.	Soubory ansible
Soubory ansible, které nalzenete na G:\Pajic\Iot Ubuntu server. Pod názvem „ember-ansible.tar.gz“ najdete hlavní konfigurační soubor *.yaml a k němu náležící konfigurační *.yml jednotlivých úloh. Soubor „hosts“ slouží pro konfiguraci SSH a budeme do něj zapisovat níže v návodu.

### 8.4.5.	Postup instalace na VM / Raspberry Pi 4 / On-Logic ...

### 8.4.5.1.	Boot OS
Proveďte boot z SD karty (např u Raspberry) nebo vložením *.ISO souboru do VMware New Virtual Machine Wizzarda a proveďte instalaci OS (výběr časové zóny, jazyka, klávesnice). Pokud budete vyzvání k instalaci novějšího OS, zakažte/přestočte ji. Při inicializaci vašeho (virtuálního) Linux Serveru zadajte název serveru, uživatele a heslo:
'''
Machine/Workstation name: (zadejte jméno Vašeho serveru)
User name: ubuntu
Passwd: ubuntu
'''
V dalším kroku potvrďte instalaci, během které proběne také automatická aktualizace balíků a restart systému. Nyní je Váš Linux Server 21.04 nainstalován se dvěmi uživateli („ubuntu“ a deffaultní „root“).

### 8.4.5.2.	Vytvoření nového uživatele ember
Nyní musíme vytvořit uživatele „ember“ (vč. Jeho složky v dir /home/). V dalším příkazu nastavíme uživateli heslo „hardwario“. V posledním příkazu přidáme uživatele „ember“ do skupiny „sudo“:
'''
sudo useradd -d /home/ember -s /bin/bash -m ember 
echo 'ember:hardwario' | sudo chpasswd 
sudo usermod -aG sudo ember 
'''
### 8.4.5.2.	 Získání souborů ansible
V tomto bodu se přihlašte na nově vytvořenéh uživatele „ember“:
'''
sudo su – ember
hardwario [ENTER]
'''
Přihlášení na nového uživatele je nutné kvůli právů (Pokud importujeme ansible soubory z účtu “ubuntu” budou jejím majetkem a bude s nimi moci nakládat jen uživatel “ubuntu” a defaultní uživatel “root”).
'''
Cd /Home/ember/
sudo passwd -l ubuntu (Uzamknutí klienta ubuntu)
'''
Pokud jste připojeni k Vašemu serveru přes SSH zadejte příkaz:
scp <cesta k ember souboru na pc>\ember-ansible.tar.gz ember@<ip linux servra>:/home/ember/

Pokud nemáte možnost využít SSH a příkaz scp je vhodné stáhnout ansible soubory z Gitu (je možné udělat mount USB a zkopírovat ansible soubory příkazem cp, ale pro náš návod je to přílišné odklonění od tématu, proto využijeme stažení ansible souborů z Gitu):
'''
sudo apt install git
git clone https://github.com/janpajic/iot-linux-server.git
'''
###  8.4.5.3.	Konfigurace statické IP adresy
Pozn. Vaše současné nastavení sítě, příkaz: ip a
Konfigurace Ip adres probíhá přes *.yaml soubor netplan(u).
	'''
Cd /etc/netplan/
Sudo nano 50-cloud-init.yaml nebo 00-network-manager-all.yaml
'''
Otevře se něco jako:
'''
network: 
 ethernets:
  enp0s3:
  addresses: []
  dhcp4: true
 version: 2
'''
Pozn. K odsazení řádku používejte [MEZERNÍK] – [TAB] je povolen až od verze OS Ubuntu 22.04 TLS.
Změňte obsah *.yaml souboru na:

	'''
network: 
 version: 2
 ethernets: 
  enp0s3:
   dhcp4: no
   addresses: [192.168.YOUR.NET/SUB]  (např. 192.168.80.205/24)
   gateway4: 192.168.80.1
   nameservers:
    addresses: [8.8.8.8,8.8.4.4]
'''
-	Potvrďte stisknutím [Ctrl+X], stiskněte “Y” jako Yes a [ENTER].

Vaši konfiguraci nastavíte pomocí příkazu:
'''
Sudo netplan apply
Pokud budete mít nějaké potíže využijte příkazu:
Sudo netplan --debug apply
'''
### 8.4.5.2.	Konfigurace souboru hosts
V tomto kroku zapíšeme hosta do souboru hosts, který jste v minulém kroku naimportovali.
Po získání ansible souboru musíme vejít do jeho adresáře a soubor *.tar.gz rozbalit. Po rozbalení musíme vejít do nově vytvořeného adresáře, kde zapíšeme hosta do souboru hosts:
'''
Cd /home/ember/
tar -xf ember-ansible.tar.gz
Cd ./setup-main/
Echo 'ember@<ip linux serveru>' >> hosts
'''
Pozn. IP adresu serveru zjistíte příkazem ip a (anebo lépe rozluštitelným ip r).

### 8.4.5.3.	Instalace základních balíků a tvorba SSH klíče
Proveďte instalaci balíků nodeejs, npm, ansible-base a sshpass. Vygenerujte SSH klíč a vložte jej do složky ~/.ssh/.
'''
sudo apt -y install nodejs
sudo apt install npm
sudo apt install sshpass
sudo apt install ansible-base
ssh-keygen
	yes
	[3x ENTER]
Ssh-copy-id -i ~/.ssh/id_rsa.pub ember@192.168.YOUR.NET
'''
### 8.4.5.1.	Instalace ansible-playbook
'''
Cd /home/ember/setup-main/
ansible-playbook -i hosts install.yml --ask-pass -K
'''
Pozn. Zadejte heslo „Hardwario“
Pokud error s key host cheking:
'''export ANSIBLE_HOST_KEY_CHECKING=False'''
Následně zadejte: 
'''
pm2 startup
sudo env PATH=$PATH:/usr/bin /usr/local/lib/node_modules/pm2/bin/pm2 startup systemd -u ember --hp /home/ember
'''
### 8.4.5.2.	Konfigurace a restart MQTT brokera
Do MQTT konfiguračního souboru je nutné zadat port:'''
'''
sudo nano /etc/mosquitto/mosquitto.conf
'''
Dopište:
	
'''
listener 1883 
allow_anonymous true 
'''
	
	-Potvrďte stisknutím [Ctrl+X], stiskněte “Y” jako Yes a [ENTER].

Restartujte brokera, aby se projevily vaše změny (s kontrolou stavu):

'''
sudo service mosquitto stop
systemctl status mosquitto.service 
sudo service mosquitto start -v
'''

### 8.4.5.3.	Kontrola a dodatečné povolení portů
Kontrola a povolení portu 18

'''
netstat -ln
srviceudo ufw allow 188
'''

### 8.4.5.4.	Porty služeb po instalaci
ChirpStack:	 8080
Node-Red:	 1880 (admin:hardwario)
MQTT broker:	 1883 (admin:hardwario)

### 8.4.5.5.	Ansible trouble shooting
Pokud zpuštíte instlaci po druhé, třetí ... je nutné některé kroky v /home/ember/setup-main/install.yml # Zakomentovat (aby se při dalších instalacích neprováděly, protože jsou již plně vytvořeny a brání v úspěšném pokračování instalace). Nejčastěji se jedná o úlohu Node-Red a Chirpstark.
