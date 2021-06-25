# Configuration Management

Een *configuration management system* stelt een systeembeheerder in staat om de gewenste toestand van een systeem te gedetailleerd te beschrijven en aan de hand daarvan dat systeem naar die toestand te brengen. Dit impliceert dat deze beschrijving **declaratief** is, d.w.z. dat je de eindtoestand beschrijft, niet de stappen of commando's die nodig zijn om die toestand te bereiken. Als je bijvoorbeeld in [Ansible](https://www.ansible.com) (het configuration management system dat we in deze cursus gebruiken) wil specifiëren dat de SSH-server moet draaien, schrijf je:

```yaml
- name: Ensure the SSH daemon is running
  service:
    name: sshd
    state: started
```

Ansible voert op de achtergrond de juiste commando's uit, afhankelijk van de Linux-distributie die op de machine draait (`systemctl start sshd` op moderne Systemd-gebaseerde distro's, `service sshd start` op bepaalde oudere systemen, enz.).

Een andere eigenschap van een configuration management system is **idempotentie**. Dat betekent dat na het succesvol uitvoeren het systeem gegarandeerd in één iteratie naar de gewenste eindtoestand gebracht is. Als het proces met succes afgesloten wordt, moet je dus achteraf niet controleren of de SSH-daemon effectief draait. Als het configuration management system er om de ene of de andere reden niet in slaagt om de SSH-server op te starten (bv. door een fout in het configuratiebestand), dan stopt het proces meteen met het uitvoeren en toont een foutboodschap. Een config management system zal ook enkel die wijzigingen doorvoeren die nodig zijn. Wanneer je de code dus een tweede keer uitvoert, zal er niets meer gewijzigd worden (want we weten dat het systeem zich al in de gewenste toestand bevindt).

Sommige config management systemen hebben een eigen taal ontwikkeld om de gewenste toestand in te beschrijven (bv. Puppet), andere gebruiken een bestaande taal (bv. Chef, configuratie in Ruby). Bij Ansible beschrijf je de configuratie van een systeem met YAML (een variant van JSON), een eenvoudig tekstformaat om data te structureren op een manier die makkelijk te interpreteren is zowel door mensen als computers. De bestanden die deze configuratiecode bevatten worden [playbooks](https://docs.ansible.com/ansible/latest/user_guide/playbooks_intro.html) genoemd. Als je deze playbooks op een specifieke manier structureert en herbruik baar maakt, spreekt men van een [rol](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html). Je kan rollen toekennen aan een host, en specifieke instellingen voor die host aanpassen door het invullen van [variabelen](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html). Op [Ansible Galaxy](https://galaxy.ansible.com/) kan je honderden rollen vinden die door hun auteurs gepubliceerd zijn.

De directorystructuur van een Ansible-project ligt vast en wordt [uitvoerig beschreven in de documentatie](https://docs.ansible.com/ansible/latest/user_guide/sample_setup.html).
