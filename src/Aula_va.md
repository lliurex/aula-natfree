# Aula
## Descripció de l'aula
![Aula](./rsrc/aula.svg)
 
L'aula està formada per un equip del professor (ADI) que està connectat a la vlan 40 del centre. Normalment està acompanyada per un panell tàctil, el qual també està connectat a la mateixa vlan 40. 

Els alumnes utilitzaran uns portàtils que podran estar a l'aula o bé en un carretó per compartir aquests equips entre diverses aules. Els portàtils estaran connectats a la wifi.

## Metapaquets

Els metapàquets que hauran d'estar instal·lats a cada equip són:
* Equip del professor (ADI):
  * lliurex-meta-adi
  * lliurex-meta-gva
* Equip de l'alumne (portàtil):
  * lliurex-meta-wifi-alu
  * lliurex-meta-gva

## Etiquetes d'autoupgrade

Les etiquetes d'autoupgrade que hauran de tenir els equips són:
* Equip del professor (ADI):
  * adi
  * gva
* Equip de l'alumne (portàtil):
  * wifi-alu
  * gva

## Funcionament de l'aula natfree
### Equip del professor (ADI)
 * L'equip estarà connectat per cable de xarxa.
 * Es crea la interfície natfree00 mitjançant un script de dispatcher.d del NetworkManager.
 * El professor prendrà el control de l'aula mitjançant el controlador de carros, indicant quin és el carro que vol controlar. 

 ![Controlador de carritos](./rsrc/controlador_carritos.png)

 * Això executarà natfree-adi configure X, on X és el número del carro. 
   * Basant-se en la configuració del fitxer /etc/natfree.d/ i buscant la variable NF_DEF_IP_NUMBER, obtindrà la ip que ha de configurar-se a la interfície natfree00. Segons el carro que estigui controlant sumarà un offset a la ip de la xarxa i s'assignarà aquesta ip. En el cas de controlar el carro 1 se li assignarà la 15a ip partint des de la base. Aquest és el valor per defecte, però es pot canviar amb el valor NF_DEF_IP_NUMBER. L'últim carro correspondrà a la primera ip usable de la xarxa en la qual està.
   * Actualitzarà el /etc/hosts i la ip calculada del carro resoldrà "server"
   * Configurara les variables CLASSROOM, SRV_IP, CLIENT_LDAP_URI i CLIENT_LDAP_URI_NOSSL.
      * CLASSROOM es configurarà amb el número del carro que s'està controlant.
      * SRV_IP, CLIENT_LDAP_URI i CLIENT_LDAP_URI_NOSSL es configuraran amb la ip que s'ha assignat a la interfície
### Equip de l'alumne (portàtil)
  * L'equip inicialment no estarà connectat a cap xarxa.
  * L'equip ha d'haver estat registrat mitjançant l'eina de registre de carro "client register".

  ![Client register](./rsrc/client_register.png)
  * De forma habitual en iniciar sessió es crearà una connexió wifi mitjançant el seu usuari d'identitat digital. Amb usuaris locals no es crearà la connexió wifi. Només en el cas d'usuari alumnat es connectarà a la wifi XXX.
  * Durant la connexió intentarà obtenir de la xarxa el valor lliurex_vlan per conèixer quina és la ip base de la xarxa vlan40. Si ho aconsegueix, crearà la variable n4d LLIUREX_VLAN amb aquest valor.
  * Es llança el dimoni sync-on-server-ready, el qual estarà comprovant si el servidor "server" respon.
     * Quan server respongui, executarà tot el que existeixi al directori /usr/share/sync-on-server-ready/actions, entre els quals es troba tornar a disparar aquells startups de n4d que tinguin a la seva classe la variable NATFREE_STARTUP = True.
     ![Startup n4d natfree](./rsrc/startup_natfree.png)
     * Sync-on-server-ready només s'executarà quan hi hagi un canvi d'ip. Si es connecta i es desconnecta per pèrdua d'ip o de xarxa no s'executarà més vegades.
  * Es configura i recarrega el servei de monit perquè estigui vigilant la ip que hagi calculat que serà el seu servidor. Per a aquest càlcul es basarà en els següents paràmetres:
    * Tinc la variable de n4d LLIUREX_VLAN amb la ip base de la vlan40. Llavors utilitzo aquesta ip base i li sumo un offset segons el carro que sóc basat en la variable NF_DEF_IP_NUMBER.
    * No he complert el cas anterior. Llavors busco la ip que he obtingut a la interfície que s'ha aixecat amb ip, obtinc la ip de la xarxa a la qual pertanyo i li sumo un offset segons el carro que sóc basat en la variable NF_DEF_IP_NUMBER.
  * Quan monit detecti que el servidor respon, executarà el comandament natfree-tie put x.x.x.x on x.x.x.x és l'adreça obtinguda en el pas anterior.
  * Si monit detecta que el servidor deixa de respondre, executarà el comandament natfree-tie destroy, el qual eliminarà del /etc/hosts l'entrada de server.

