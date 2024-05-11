# Magnum Opus Ansible Configuratie

## Introductie
Deze repository is ontworpen om OpenSCAP te configureren voor beveiligingsscans en om de beveiliging van servers te automatiseren met behulp van Ansible. Het is zo gestructureerd dat het beheersbaar en begrijpelijk is, zelfs voor degenen die nieuw zijn bij Ansible of serverbeheer.

## Project Structuur Overzicht
In deze sectie wordt de lay-out van de projectmap beschreven:

```
.
├── ansible.cfg            # Configuratie instellingen voor Ansible
├── inventory              # Hosts file waar je alle servers kunt definiëren
├── playbooks
│   └── setup_openscap.yml # Main playbook om OpenSCAP te installeren en te configureren
├── README.md              # Documentatie over het project
├── roles                  # Bevat alle Ansible-roles
│   ├── cron_management    # Beheert cron-taken op de servers
│   ├── openscap           # Installeert en configureert OpenSCAP
│   ├── package_management # Zorgt ervoor dat de pakketten up-to-date zijn
│   └── security_hardening # Bevat taken voor het beveiligen van de servers
└── vars
    └── main.yml           # Variabelen die worden gebruikt in de Ansible-roles
```

## Aan de slag
**Vereisten**
- Ansible
- Toegang tot beheerhosts via SSH
- Sudo-rechten op de doelhosts

### Te bewerken configuratiebestanden

Voordat u de playbooks uitvoert, moet u twee belangrijke bestanden configureren:

1. **Inventory (`inventory`):** Dit bestand lijst alle hosts op waarop u de configuraties wilt toepassen. Bewerk dit bestand om uw server-IP-adressen of hostnamen op te nemen onder de juiste groep. Bijvoorbeeld:

   ```ini
   [rocky_servers]
   192.168.1.60
   ```

   Voeg indien nodig meer servers toe onder de groep `[rocky_servers]`.

2. **Variabelen (`vars/main.yml`):** Dit bestand bevat essentiële configuratievariabelen die gebruikt worden door verschillende rollen in het playbook. Het is cruciaal om deze variabelen correct in te stellen om te zorgen dat de Ansible rollen naar verwachting functioneren. Hieronder volgt een gedetailleerde uitleg van elke variabele:

- **`host`**: Dit is het IP-adres of de hostnaam van de doelmachine waarop de Ansible-rollen worden toegepast. 
- **`openscap_output_dir`**: Specificeert de directory op de doelhosts waar de OpenSCAP rapporten worden opgeslagen. Standaard ingesteld op "/home/scap_reports". Zorg ervoor dat de directory beschikbaar is en schrijfrechten heeft op de doelhost.

- **`openscap_profile`**: Definieert het beveiligingsprofiel dat gebruikt wordt voor de OpenSCAP scans. Dit voorbeeld gebruikt "xccdf_org.ssgproject.content_profile_anssi_bp28_high", een hoog beveiligingsniveau dat geschikt is voor strenge beveiligingseisen.

- **`openscap_content`**: Het pad naar de SCAP content-bestanden die gebruikt worden voor de scans. In dit geval is "/usr/share/xml/scap/ssg/content/ssg-rl9-ds.xml" aangegeven, wat specifiek is voor Red Hat Enterprise Linux 9.

- **`openscap_cron_hour`**, **`openscap_cron_minute`**, **`openscap_cron_day`**, **`openscap_cron_month`**, **`openscap_cron_weekday`**: Deze instellingen bepalen het schema van de cron job die de OpenSCAP scan uitvoert. Hier ingesteld om dagelijks om 3:00 uur te draaien. Pas deze waarden aan op basis van gewenste frequentie en tijdstip.

- **`http_auth_username`** en **`http_auth_password`**: Gebruikersnaam en wachtwoord voor toegang tot de lokaal gehoste webpagina waarop de OpenSCAP rapporten worden weergegeven. Dit is cruciaal voor het beveiligen van de toegang tot de rapporten.

- **`port_reports`**: De netwerkpoort waarop de rapporten worden geserveerd door de HTTP-server, hier ingesteld op 8000. Zorg dat deze poort open staat op de firewall indien nodig.

- **`RootLogin`** en **`PasswordAuth`**: Configuraties voor het verharden van SSH-toegang. `RootLogin` staat hier toe dat root direct kan inloggen (`yes`). `PasswordAuth` staat toe dat authenticatie met een wachtwoord mogelijk is (`yes`). Afhankelijk van je beveiligingsbeleid wil je deze waarden mogelijk aanpassen.


Om de beschikbare profielen uit je OpenSCAP datastream te includeren in je documentatie of configuratiebestanden, kun je een sectie toevoegen in `vars/main.yml` of een aparte documentatiegedeelte creëren om de verschillende profielen te beschrijven die gebruikt kunnen worden. Hieronder een voorstel hoe je dit kunt opnemen in je documentatie:

### Beschikbare OpenSCAP Profielen

Dit project gebruikt de OpenSCAP tool voor beveiligingsscans op Red Hat Enterprise Linux 9. De volgende profielen zijn beschikbaar voor gebruik en kunnen worden gespecificeerd in de `openscap_profile` variabele in `vars/main.yml`. Kies een profiel dat past bij de beveiligingseisen van je organisatie:

- **ANSSI-BP-028 Profielen:**
  - `xccdf_org.ssgproject.content_profile_anssi_bp28_enhanced` - Verhoogde beveiligingsinstellingen.
  - `xccdf_org.ssgproject.content_profile_anssi_bp28_high` - Hoge beveiligingsinstellingen.
  - `xccdf_org.ssgproject.content_profile_anssi_bp28_intermediary` - Tussenliggende beveiligingsinstellingen.
  - `xccdf_org.ssgproject.content_profile_anssi_bp28_minimal` - Minimale beveiligingsinstellingen.

- **CCN Red Hat Enterprise Linux 9 Profielen:**
  - `xccdf_org.ssgproject.content_profile_ccn_advanced` - Gevorderde beveiligingsinstellingen.
  - `xccdf_org.ssgproject.content_profile_ccn_basic` - Basis beveiligingsinstellingen.
  - `xccdf_org.ssgproject.content_profile_ccn_intermediate` - Tussenliggende beveiligingsinstellingen.

- **CIS Red Hat Enterprise Linux 9 Benchmarks:**
  - `xccdf_org.ssgproject.content_profile_cis` - CIS Benchmark Niveau 2 voor servers.
  - `xccdf_org.ssgproject.content_profile_cis_server_l1` - CIS Benchmark Niveau 1 voor servers.
  - `xccdf_org.ssgproject.content_profile_cis_workstation_l1` - CIS Benchmark Niveau 1 voor werkstations.
  - `xccdf_org.ssgproject.content_profile_cis_workstation_l2` - CIS Benchmark Niveau 2 voor werkstations.

- **Andere belangrijke profielen:**
  - `xccdf_org.ssgproject.content_profile_cui` - Profiel voor het beveiligen van niet-federale informatiesystemen (NIST 800-171).
  - `xccdf_org.ssgproject.content_profile_e8` - Australian Cyber Security Centre Essential Eight.
  - `xccdf_org.ssgproject.content_profile_hipaa` - Profiel voor Health Insurance Portability and Accountability Act.
  - `xccdf_org.ssgproject.content_profile_ism_o` - Australian Cyber Security Centre ISM Official.
  - `xccdf_org.ssgproject.content_profile_ospp` - Protection Profile for General Purpose Operating Systems.
  - `xccdf_org.ssgproject.content_profile_pci-dss` - PCI-DSS v4.0 Control Baseline.
  - `xccdf_org.ssgproject.content_profile_stig` - DISA STIG voor Red Hat Enterprise Linux 9.
  - `xccdf_org.ssgproject.content_profile_stig_gui` - DISA STIG met GUI voor Red Hat Enterprise Linux 9.

Zorg ervoor dat je de juiste `openscap_profile` instelt in `vars/main.yml` op basis van het gewenste beveiligingsniveau en de compliance vereisten van jouw organisatie. Dit zorgt ervoor dat de scans aansluiten bij de specifieke behoeften en beleidsregels van je bedrijf.

### Het Playbook Uitvoeren

Nadat u uw `inventory` en `vars/main.yml` hebt geconfigureerd, kunt u het playbook uitvoeren met het volgende commando:

```bash
ansible-playbook -i inventory playbooks/setup_openscap.yml
```

Dit commando start het playbook dat uw servers configureert op basis van de gespecificeerde rollen.

## Gedetailleerde Rolbeschrijvingen

Elke rol in de directory `roles` heeft een specifieke functie:

### Package Management

Zorgt ervoor dat alle pakketten zijn bijgewerkt naar hun laatste versie om uw systeem up-to-date te houden met alle beveiligingspatches en updates.

### OpenSCAP

Installeert de benodigde pakketten voor OpenSCAP, configureert het scanscript, zet een server op om de rapporten te hosten en configureert de firewall om toegang tot de report server toe te staan.

### Cron Management

Configureert een cron job om regelmatig het OpenSCAP-scan script uit te voeren. Het schema voor deze job wordt gecontroleerd door de variabelen in `vars/main.yml`.

### Security Hardening

Past verschillende beveiligingsinstellingen toe zoals het configureren van de firewall om al het onnodige verkeer te laten vallen, het verharden van SSH-configuraties, en het zorgen voor instellingen voor veilige bediening.

Hier is een sectie die je kunt toevoegen aan je `README.md` om uit te leggen hoe je project uitgebreid kan worden met Semaphore, een tool die het mogelijk maakt om Ansible playbooks via een grafische gebruikersinterface (GUI) te beheren en uit te voeren. Deze sectie bevat ook stap-voor-stap instructies voor de installatie van Semaphore.

### Uitbreiding met Semaphore in Rocky Linux 9

Semaphore is een open-source automatiseringsserver voor Ansible, waarmee je playbooks kunt beheren en uitvoeren via een gebruiksvriendelijke grafische gebruikersinterface (GUI). Hieronder vind je de stappen om Semaphore te installeren en te configureren op een systeem met Rocky Linux 9 (Red Hat Enterprise Linux 9).

#### Installatie van Semaphore

1. **Voeg het EPEL-repository toe:**
   Semaphore is afhankelijk van pakketten die beschikbaar zijn in het Extra Packages for Enterprise Linux (EPEL) repository. Voer de volgende commando uit om het EPEL-repository toe te voegen:

   ```bash
   sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
   ```

2. **Upgrade het systeem:**
   Zorg ervoor dat je systeem volledig is bijgewerkt:

   ```bash
   sudo dnf upgrade
   ```

3. **Schakel optionele en extra repositories in:**
   Het is aanbevolen om additionele repositories in te schakelen voor extra pakketten die door Semaphore gebruikt kunnen worden:

   ```bash
   sudo subscription-manager repos --enable "rhel-*-optional-rpms" --enable "rhel-*-extras-rpms"
   ```

4. **Update de pakketlijst:**
   Gebruik `yum` om de pakketlijst te updaten:

   ```bash
   sudo yum update
   ```

5. **Installeer Snapd:**
   Semaphore kan geïnstalleerd worden via Snap, een applicatiepakket en runtime omgeving. Installeer Snapd:

   ```bash
   sudo yum install snapd
   sudo systemctl enable --now snapd.socket
   sudo ln -s /var/lib/snapd/snap /snap
   ```

6. **Installeer Semaphore:**
   Ten slotte, installeer Semaphore via Snap:

   ```bash
   sudo snap install semaphore
   ```

Hier is een verbeterde versie van de sectie "Configuratie en Gebruik" voor Semaphore in je `README.md`. Deze versie is meer gestructureerd en duidelijk over de verschillende stappen:

### Configuratie en Gebruik van Semaphore

#### Toegang tot Semaphore
Na de installatie kun je Semaphore bereiken via een webbrowser door te navigeren naar het IP-adres van de server en de specifieke poort (standaard is dit meestal poort 3000). Als je vanaf een andere machine toegang wilt, zorg er dan voor dat de firewallregels de toegang tot deze poort toestaan.

#### Eerste Configuratie
Bij het eerste gebruik van Semaphore wordt je gevraagd een initiële configuratie uit te voeren. Dit omvat het aanmaken van een admin account.

#### Aanmaken van een Admin Gebruiker
Voordat je begint met het beheer van projecten, moet je een admin gebruiker aanmaken met de volgende commando:
```bash
semaphore user add --admin --login user123 --name User123 --email user123@example.com --password 6YmJselO0P
```
Log vervolgens in met de zojuist aangemaakte inloggegevens.

#### Genereren en Toevoegen van SSH Sleutelparen
Indien nog niet gedaan, genereer een SSH sleutelpaar om de communicatie tussen Semaphore en de target machines te beveiligen.

#### Configuratie in de Semaphore Webinterface
1. **Key Store:**
   - Navigeer naar de 'Key Store' sectie en voeg je SSH private key en sudo-wachtwoord toe. Deze sleutels zullen gebruikt worden om verbinding te maken met de target hosts.

2. **Repositories:**
   - Voeg je repository toe in de 'Repositories' sectie. Als je repository privé is en je SSH sleutels gebruikt, gebruik dan een aangemaakt sleutelpaar om de repository te klonen. Specificeer ook de branch waar je Ansible-configuraties zich bevinden.

3. **Inventory:**
   - Maak een nieuwe inventory aan waarin je de target hosts specificeert. Voeg de eerder gemaakte SSH-sleutel en sudo-credentials toe indien nodig.

4. **Environment:**
   - Stel belangrijke variabelen in, zoals waar de rollen zich bevinden in de repository en onder welke gebruikersnaam de verbinding moet worden gemaakt:
   Extra variables:
   ```json
      {
         "host": "rocky_servers"
         "openscap_output_dir": "/home/scap_reports",
         "openscap_profile": "xccdf_org.ssgproject.content_profile_anssi_bp28_high",
         "openscap_content": "/usr/share/xml/scap/ssg/content/ssg-rl9-ds.xml",
         "openscap_cron_hour": "3",
         "openscap_cron_minute": "0",
         "openscap_cron_day": "*",
         "openscap_cron_month": "*",
         "openscap_cron_weekday": "*",
         "http_auth_username": "admin",
         "http_auth_password": "admin",
         "port_reports": 8000,
         "RootLogin": "yes",
         "PasswordAuth": "yes"
      }
   ```
   Environment variables:
     ```json
     {
       "ANSIBLE_ROLES_PATH": "roles/",
       "ANSIBLE_REMOTE_USER": "root"
     }
     ```

5. **Task Templates:**
   - Ga naar de 'Task Templates' sectie en maak een nieuwe template aan. Vul details in zoals de naam van de template, de playbook bestandsnaam, inventory, repository en environment.

#### Uitvoeren van de Playbook
Nadat alles is ingesteld, kun je de playbook uitvoeren door de gemaakte task template te selecteren en te starten. Dit zal je Ansible playbooks uitvoeren volgens de configuraties die je hebt ingesteld.

