# H5

http://terokarvinen.com/2017/aikataulu-palvelinten-hallinta-ict4tn022-3-5-op-uusi-ops-loppusyksy-2017-p5

## a) Asenna Puppetin orjaksi vähintään kaksi eri käyttöjärjestelmää. (Tee alusta, pelkkä tunnilla tehdyn muistelu ei riitä).

Käytän tehtävän orjina Microsoft Windows [Version 10.0.15063], sekä Xubuntu 16.04 LTS Vagrant virtuaalikoneella.

# Masterin asetukset

Asensin puppet masterin koneelle komennolla

`sudo apt-get -y install puppetmaster`

Tästä sain virheilmoituksen:

```
● puppetmaster.service - Puppet master
   Loaded: loaded (/lib/systemd/system/puppetmaster.service; enabled; vendor preset: enabled)
   Active: failed (Result: exit-code) since ke 2017-11-22 12:12:59 EET; 8ms ago
  Process: 10684 ExecStart=/usr/bin/puppet master (code=exited, status=1/FAILURE)

marras 22 12:12:59 einopuucee puppet-master[10684]: On the master:
marras 22 12:12:59 einopuucee puppet-master[10684]:   puppet cert clean einopuucee.lan
marras 22 12:12:59 einopuucee puppet-master[10684]: On the agent:
marras 22 12:12:59 einopuucee puppet-master[10684]:   1a. On most platforms: find /var/lib/puppet/ssl -name ei...lete
marras 22 12:12:59 einopuucee puppet-master[10684]:   1b. On Windows: del "/var/lib/puppet/ssl/einopuucee.lan.pem" /f
marras 22 12:12:59 einopuucee puppet-master[10684]:   2. puppet agent -t
marras 22 12:12:59 einopuucee systemd[1]: puppetmaster.service: Control process exited, code=exited status=1
marras 22 12:12:59 einopuucee systemd[1]: Failed to start Puppet master.
marras 22 12:12:59 einopuucee systemd[1]: puppetmaster.service: Unit entered failed state.
marras 22 12:12:59 einopuucee systemd[1]: puppetmaster.service: Failed with result 'exit-code'.
Hint: Some lines were ellipsized, use -l to show in full.
dpkg: error processing package puppetmaster (--configure):
 subprocess installed post-installation script returned error exit status 1
Setting up ruby-atomic (1.1.16-2build5) ...
Setting up ruby-thread-safe (0.3.5-3) ...
Setting up ruby-tzinfo (1.2.2-1) ...
Setting up ruby-activesupport (2:4.2.6-1) ...
Setting up ruby-blankslate (3.1.3-1) ...
Setting up ruby-builder (3.2.2-4) ...
Setting up ruby-activemodel (2:4.2.6-1) ...
Setting up ruby-arel (6.0.3-2) ...
Setting up ruby-activerecord (2:4.2.6-1) ...
Setting up ruby-activerecord-deprecated-finders (1.0.4-1) ...
Processing triggers for systemd (229-4ubuntu21) ...
Processing triggers for ureadahead (0.100.0-19) ...
Errors were encountered while processing:
 puppetmaster
```

Tämä johtuunee siitä, että olin hetkeä aikaisemmin purgennut puppetin koneelta?!?!?
Palaan virheilmoituksiin vain, jos puppetin käytössä ilmenee ongelmia.

Lisäsin puppet.conf tiedostoon dns alt namen

```
sudoedit /etc/puppet/puppet.conf

dns_alt_names = puppet, einopuucee, einopuucee.lan
```

Tämän koneen kohdalla minun ei pitäisi joutua vaihtelemaan nimiä, tai poistelemaan avaimia.

edit: windowssia varten joudutaan lisäämään hosts kansioon .local pääte

```sudoedit /etx/hosts
127.0.1.1       einopuucee einopuucee.local
```

Loin Hello World moduulin

```
$ cd /etc/puppet/
$ sudo mkdir -p /etc/manifests/ modules/helloworld/manifests/
$ sudoedit modules/helloworld/manifests/init.pp
```
```
  
class helloworld {
        file { '/tmp/helloFromMaster':
                content => "See you at http://terokarvinen.com/tag/puppet\n"
        }
}
```

Tämän lisäksi loin site.pp tiedoston

```
sudoedit manifests/site.pp

include helloworld
```

Testasin moduulin masterilla

```
$ sudo puppet apply -e 'class{'helloworld':}'
Notice: Compiled catalog for einopuucee.lan in environment production in 0.18 seconds
Notice: /Stage[main]/Helloworld/File[/tmp/helloFromMaster]/ensure: defined content as '{md5}d69e1336f25776f557a2ee309841751b'
Notice: Finished catalog run in 0.25 seconds
```
```
$ cat /tmp/helloFromMaster
See you at http://terokarvinen.com/tag/puppet
```

# Vagrant orjan asetukset

Asensin vagrantin ja loin virtuaalixubuntun

`sudo apt-get -y install vagrant virtual`

```
$ vagrant init bento/ubuntu-16.04
$ vagrant up
$ vagrant ssh
```
Asensin virtuaalikoneelle puppetin

```
$ sudo apt-get update && sudo apt-get -y install puppet
```

Asetin virtuaalikoneen hosts tiedostoon masterin masterin iipparin osoittamaan nimeen

```
sudoedit /etc/hosts

192.168.1.222   einopuucee.lan

```

Ja puppet.conffiin masterin nimi

```
sudoedit /etc/puppet/puppet.conf

[agent]
server = einopuucee.lan
```

Käynnistin orjan 

```
sudo puppet agent --enable
sudo puppet agent -t
```

Tästä virheilmoitus

```
Error: Could not request certificate: Failed to open TCP connection to einopuucee.lan:8140 (Connection refused - connect(2) for "einopuucee.lan" port 8140)
Exiting; failed to retrieve certificate and waitforcert is disabled
```

Masterilla yrittäessäni listata cert avaimia, niin sain saman virheilmoituksen kuin ylhäällä, joten seurasin ohjeita:

Masterilla:

```
sudo puppet cert clean einopuucee.lan
To fix this, remove the certificate from both the master and the agent and then start a puppet run, which will automatically regenerate a certficate.
```

Joudun ilmeisesti jokatapauksessa postelemaan certit..

Pysäytin masterin, poistin certin ja käyynistin uudelleen

```
$ sudo service puppetmaster stop
$ sudo rm -r /var/lib/puppet/ssl
$ sudo service puppetmaster start
```

Ja sama orjalla

```
$ sudo service puppet stop
$ sudo rm -r /var/lib/puppet/ssl
$ sudo service puppet start
```
 Ja nyt yritin käynnistää orjan uusiksi
 
 ```
 $ sudo puppet agent -t
Exiting; no certificate found and waitforcert is disabled
```

## Masterilla certtien hyväksyntä

Listasin certit komennolla

```
$ sudo puppet cert --list
  "vagrant.vm" (SHA256) FA:44:0E:11:F8:66:0E:52:9A:63:82:EC:63:2F:70:5B:57:C0:BF:85:9D:99:B1:04:78:7C:AE:2E:5A:E6:5C:CF
```

ja hyväksyin

```
$ sudo puppet cert --sign vagrant.vm
Notice: Signed certificate request for vagrant.vm
Notice: Removing file Puppet::SSL::CertificateRequest vagrant.vm at '/var/lib/puppet/ssl/ca/requests/vagrant.vm.pem'
```

Jostain syystä agentti kiukuttteli edelleen

```
vagrant@vagrant:~$ sudo puppet agent -t
Info: Caching certificate for vagrant.vm
Error: Could not request certificate: The certificate retrieved from the master does not match the agent's private key.
Certificate fingerprint: 8D:72:F0:0E:D7:65:12:62:83:B9:87:68:39:B4:7D:97:71:99:B2:54:DA:D3:6F:DD:80:C0:ED:04:35:A4:61:49
To fix this, remove the certificate from both the master and the agent and then start a puppet run, which will automatically regenerate a certficate.
On the master:
  puppet cert clean vagrant.vm
On the agent:
  1a. On most platforms: find /var/lib/puppet/ssl -name vagrant.vm.pem -delete
  1b. On Windows: del "/var/lib/puppet/ssl/vagrant.vm.pem" /f
  2. puppet agent -t
```

Poistin kummatkin certit uusiksi, ja nyt yhteys toimi.

Testasin orjalla Helloworld moduulin ja se toimi.


## b) Säädä Windows-työpöytää. Voit esimerkiksi asentaa jonkin sovelluksen ja tehdä sille asetukset.

Säädin väliaikaisesti "User Account Control Settings" kohtaan Never notify ja käynnistelin koneen uusiksi.

Katsoin Masteriltani puppet version komennolla `puppet --version`, jotta osaan asentaa Windowssille vastaavan version.

Latasin osoitteesta https://downloads.puppetlabs.com/windows/ version puppet-3.8.5-x64.msi.

Asentelin puppetin install wizardista.

Asetin jo asnnusvaiheessa masterin nimeksi "einopuucee".

Tarkistin vielä puppet.confista (C:\ProgramData\PuppetLabs\puppet\etc\puppet.conf), että masteri oli oikein.

Latasin ja aasensin Bonjour Print Services for Windows, jotta .local osoite toimisi.

https://support.apple.com/kb/dl999?locale=fi_FI

Testasin vielä .local osoitteen toimivuuden syöttämällä Command Promttiin

```ping einopuucee.local```

mihin sain vastauksen.

Tässä välissä loin masterilla HelloWindows moduulin

```
cd /etc/puppet
sudo mkdir -p /etc/manifests/ modules/hellowin/manifests/
sudoedit module/hellowin/manifests/init.pp
```
```
class hellowindows {
        file {"C:/hellotero":
                content => "Hello Master\n",
        }
}

```

Sekä site.pp teidoston, jonne lisäsin:

```
sudoedit manidfests/site.pp

class {hellowin:}
```

Takaisin Windowsille

Avasin "puppet terminaalin" avaamalla "Start Command Prompt with Puppet" "Run as ADministrator"

Syötin komennon

```
puppet agent -tdv
```

Ei niin yllättäen puppet kiukutteli certistä

```
Error: Could not request certificate: A connection attempt failed because the connected party did not properly respond after a period of time, or established connection failed because connected host has failed to respond. - connect(2)
Exiting; failed to retrieve certificate and waitforcert is disabled
```

Löytämässäni ohjeessa
