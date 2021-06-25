# Inleiding, opzetten werkomgeving

## Motivatie

TODO:

- Belang van automatisering
- Casus: OVF brand

## Opzetten werkomgeving

Installeer eerst de nodige software, meer bepaald de laatste stabiele versie bij aanvang van het semester. De volledige neerslag van al wat je voor deze cursus doet wordt bijgehouden in het versiebeheersysteem [Git](https://git-scm.com/). Via Chamilo vind je een link die, als je er op doorklikt, een nieuwe repository creëert waar je in kan werken. Deze is zichtbaar voor jou en de lector. Naast de configuratie van de opgezette systemen zal je er ook je documentatie bijhouden, zoals testrapporten, procedures, cheat sheets en checklists.

Richtlijnen voor het opstarten van je Git project:

1. In principe moet je al een Github account hebben. Als je dit nog niet gedaan hebt, koppel dan zeker je @student.hogent.be adres aan het account. Je kan dan het [Github Student Developer Pack](https://education.github.com/pack) aanvragen met allerlei interessante aanbiedingen. Het is mogelijk om meerdere e-mailadressen te registreren, en het is zinvol om ook je privé-adres te koppelen. Op die manier kan je je Github-account nog gebruiken na je afstuderen.
2. Zorg dat je een SSH-sleutelpaar hebt aangemaakt. Op die manier kan je het pushen naar Github vereenvoudigen. Het is dan meer bepaald niet meer nodig een gebruikersnaam en wachtwoord op te geven. Voer in een Bash-terminal het commando `ssh-keygen` uit, en volg de richtlijnen die het geeft. Geef voor je gemak een lege passphrase op (zoniet moet je telkens je de sleutel gebruikt je passphrase intikken). Normaal zou er in de directory `~/.ssh` (`~` staat voor je home-directory: op Windows `C:\Gebruikers\Gebruikersnaam`, op MacOS `/Users/Gebruikersnaam`, op Windows `/home/gebruikersnaam`) twee bestanden aangemaakt moeten zijn: `id_rsa` en `id_rsa.pub`. Het eerste is je private sleutel (die je geheim moet houden), het tweede de publieke. Die laatste kan je op Github registreren via je profielinstellingen (klik op je avatar rechtsboven, volg *Settings* en dan *SSH and GPG keys*).
3. Controleer je Git basisconfiguratie (in `~/.gitconfig`) en als je dit nog niet gedaan hebt, maak je volgende aanpassingen:

    ```
    git config --global user.name ”VOORNAAM NAAM”
    git config --global user.email ”VOORNAAM.NAAM@student.hogent.be”
    git config --global push.default simple
    git config --global core.autocrlf input
    git config --global pull.rebase true
    ```

4. Maak lokaal een directory aan die je voorbehoudt voor al wat met deze cursus te maken heeft. Binnen deze directory kan je je Github-repository klonen. Klik op de Github-pagina van je repository op de groene knop rechts (Code), kies voor “SSH” kopieer de link van de vorm git@github.com:HoGentTIN/REPO_
NAAM-GEBRUIKERSNAAM.git.

    ```
    cd Documents/Courses/EnterpriseLinux/
    git clone git@github.com:HoGentTIN/REPONAAM-GEBRUIKERSNAAM.git
    ```

    Dit maakt een lokale kopie van de repository in een subdirectory. Je mag de naam van deze directory aanpassen en die verplaatsen, alles blijft gewoon werken.
5. Creëer een nieuwe branch met de naam `solution` om je eigen code en documentatie bij te houden. Verderop wordt duidelijk waarom dit belangrijk is.

    ```
    git switch -c solution
    ```

6. Open de directory in Visual Studio Code en open het bestand `README.md`. Vul bovenaan de gevraagde gegevens in (naam, klasgroep, enz.) en commit deze wijziging. Lees het README-bestand voor verdere instructies.

### Errata

Wanneer er errata in de opgave gepubliceerd worden, kan je die relatief eenvoudig binnen halen, maar enkel als je een eigen `solution`-branch aangemaakt hebt.

1. Eerst moet je zorgen dat je updates kan binnenhalen van de repository met de opdracht. Het volgende commando zorgt dat je kan synchroniseren met die repository. Dit moet slechts één keer gebeuren.

    ```
    git remote add upstream https://github.com/HoGentTIN/infra-labs.git
    ```

2. Wanneer er nieuwe commits gebeurd zijn in de opgave, kan je de wijzigingen telkens zo ophalen:

        ```
        git switch main
        git pull upstream main
        git switch solution
        git rebase main
        ```

    - De eerste twee regels zorgen er voor dat jouw versie van de main-branch up-to-date gebracht wordt met de nieuwe commits.
    - In de derde regel ga je opnieuw naar je eigen branch. Voorlopig is er daar nog niets gewijzigd
    - In de vierde regel, tenslotte, ga je de wijzigingen in de opgave op jouw eigen versie toepassen. Mogelijks komen er hier conflicten naar boven tussen bepaalde bestanden in de opgave en jouw wijzigingen. De command-line Git client geeft goede instructies om deze conflicten op te lossen. Zie hieronder voor een voorbeeld.

Deze procedure werkt niet als je geen branch voor je eigen oplossing gemaakt hebt.

In dat geval haal je jezelf een hoop ellende op de hals... Als er conflicten optreden in bestanden, blijft Git in rebase-modus” steken en kan je voorlopig niet verder werken. Voer eerst git status uit om een lijst te krijgen met alle betrokken bestanden.

```console
$ git status
rebase in progress; onto e5bd2df
You are currently rebasing branch 'master' on 'e5bd2df'.
    (fix conflicts and then run ”git rebase --continue”)
    (use ”git rebase --skip” to skip this patch)
    (use ”git rebase --abort” to check out the original branch)
Unmerged paths:
    (use ”git reset HEAD <file>...” to unstage)
    (use ”git add <file>...” to mark resolution)
both modified:

 README.md

no changes added to commit (use ”git add” and/or ”git commit -a”)
```

Open deze één voor één met je teksteditor en zoek naar de markeringen voor conflicten, bv.

```text
If you have questions, please
<<<<<< HEAD
open an issue
======
ask your question in IRC.
>>>>>> main
```

VSCode toont deze conflicten in een kleur zodat ze in het oog springen.

Bewerk alle bestanden met conflicten en verbeter de code. Vervolgens voeg je alle aangepaste bestanden toe met `git add .` en kan je het rebase-proces verder zetten met `git rebase --continue`. Gebruik tussendoor regelmatig git status om te zien hoe ver je staat en welke commando’s je nodig hebt om een stap verder te gaan.

## Tips voor efficiënt werken

In deze sectie vind je enkele algemene richtlijnen die je helpen vlotter en efficiënter te werken.

- Voor je aan een labo-opdracht begint, bereid je je eerst voor door alle aangereikte studiematerialen te bestuderen: handleidingen, screencasts, ... “Van buiten blokken” is helemaal niet nodig, maar zorg er in elk geval voor dat je er in die mate vertrouwd bent, dat je snel gericht kan zoeken naar juiste, relevante informatie. Je vindt die via de bronvermeldingen in deze syllabus, of via de opgave.
- Open **meerdere terminalvensters/consoles** naast elkaar (TODO: figuur invoegen). Elke terminal krijgt zijn eigen functie, bijvoorbeeld:
    - Vim editor (of VS Code/Sublime/Notepad/...in een apart venster);
    - doorvoeren van wijzigingen aan de configuratie;
    - ingelogd op VM, voor commando’s;
    - ingelogd op VM, voor tonen logbestanden.
- **Werk stap voor stap.** Schrijf niet teveel code ineens. Probeer eerst een minimaal werkende opstelling te verkrijgen en registreer meteen in Git. Maak minimale wijzigingen en test elke wijziging uit. Hoe groter en ingrijpender de wijzigingen, hoe meer kans op fouten en hoe moeilijker die te debuggen zijn. Zodra iets werkt, en je bent een stap verder, registreer je dit meteen in Git en geef je een duidelijke, beschrijvende commit-boodschap.
- **Gebruik Git op de command-line.** Bij de meeste Git commando’s krijg je gedetailleerde uitleg over hoe je een stap verder moet gaan en ook hoe je de laatste stap kan ongedaan maken. Dit geeft op de duur een beter inzicht in hoe Git precies werkt dan je via een GUI kan krijgen. De meeste GUIs voor Git verbergen belangrijke details waardoor je niet goed kan begrijpen wat je aan het doen bent.
- **Commit regelmatig wijzigingen in je code** en probeer **elke commit te beperken tot één enkele “reden”** om wijzigingen aan te brengen aan de bestaande code. Dit maakt de “geschiedenis” van je project transparanter en maakt ook dat je makkelijker kan terugkeren naar een bepaalde stap wanneer je de mist in gaat.
- **Maak backups** van originele, ongewijzigde configuratiebestanden zodat je er op kan terugvallen als er iets misloopt. Soms heb je zodanig zitten “prutsen” dat je er niet meer in slaagt de service te laten werken.
- **Gebruik `vagrant destroy`**. Wanneer je veel manuele wijzigingen hebt aangebracht in een VM, ben je op de duur niet meer zeker dat die zich in de gewenste toestand bevindt. Of je kan door experimenteren de VM onbruikbaar gemaakt hebben. Door de VM te verwijderen en opnieuw op te bouwen (met `vagrant up`) kan je opnieuw beginnen van een werkende versie (als je de vorige richtlijnen opvolgt, tenminste!).
- Ook wanneer je denkt klaar te zijn met een deelopdracht, genereer de VM nog eens helemaal opnieuw en voer alle acceptatietests uit (indien die voorzien zijn in de labo-opdracht).

### Bash tips

In deze sectie vind je een aantal tips voor het gebruik van de Bash-shell. In deze cursus maak je intensief gebruik van de shell, en het loont de moeite om Bash wat beter te leren kennen. Het zit immers vol met interessante features die je toelaten productiever te werken. Online is hier nog veel meer over te vinden (Rowe, 2009).

De tips hier gelden ook voor Windows- en MacOS-gebruikers! Windows-gebruikers hebben beschikking over een recente versie van Bash via Git (Git Bash). In MacOS X zit ook een Bash-shell, weliswaar een zeer oude versie. Je kan een recente versie installeren via HomeBrew.

- **Gebruik TAB-completion.** Je kan een deel van een commando of pad intikken en dan de TAB-toets indrukken. Indien mogelijk zal Bash het woord vervolledigen, of mogelijke alternatieven tonen. Als je de package bash-completion installeert zijn er nog veel meer mogelijkheden.
- **Gebruik de command-history.** Bash houdt de commando’s die je eerder gebruikt hebt bij. Met de pijltjestoetsen kan je eerdere commando’s terug halen. Gebruik `Ctrl-R` om te zoeken in de command history. Je kan dan een fragment van het commando intikken, Bash toont dan het laatste commando waar dat tekstfragment in voorkomt.
- **Bang-bang!!**  In Bash zijn er enkele shortcuts voorzien voor (delen van) het vorige commando. `!!` staat voor het gehele vorige commando, `!$` voor het laatste argument en `!*` voor alle argumenten. Een voorbeeldje van het gebruik:

```console
$ dnf update
You need to be root to perform this command.
$ sudo !!
sudo dnf update
[...]
$ mkdir -p some/long/path/i/dont/want/to/repeat
$ cd !$
cd some/long/path/i/dont/want/to/repeat
$
```

- **Toetsenbordcombinaties.** Naast `Ctrl-R` van hierboven zijn er nog een aantal nuttige toetsenbordcombinaties.
    - `Ctrl-C` onderbreekt het huidige proces
    - `Ctrl-Z` pauzeert het huidige proces (haal het terug met fg)
    - `Ctrl-D` sluit de shell af (enkel op lege regel)
    - `Ctrl-K` verwijdert de tekst rechts van de cursor
    - `Ctrl-U` verwijdert de tekst links van de cursor
    - `Ctrl-T` verwisselt letterteken onder de cursor met dat voor de cursor
    - `Alt-T` verwisselt woord voor met woord na de cursor

    Er bestaan nog veel meer combinaties. Deze kan je vinden in de man-pagina `bash(1)` in de sectie READLINE (subsectie Readline Command Names en volgende).

- **Personaliseer de shell.** Je kan het gedrag van de shell in hoge mate aan je eigen wensen aanpassen door het bestand `~/.bashrc` aan te passen. Je kan het gedrag van tab-completion of de command history aanpassen, een eigen commandoprompt instellen, aliassen (zie verder) of functies definiëren, enz. Het zou ons te ver leiden om alle mogelijkheden uit te diepen, maar je kan een relatief eenvoudig voorbeeld vinden in <https://github.com/bertvv/server-dotfiles/> (installatie-instructies te vinden in de README), en een meer uitgebreid voorbeeld in <https://github.com/bertvv/dotfiles/> (of gelijknamige reposi-
tories van andere Github-gebruikers).
- **Aliassen** zijn een soort shortcuts voor commando’s die je vaak gebruikt. Dit kan enorm veel typwerk besparen. Je kan ze toevoegen in je `~/.bashrc`. Enkele voorbeelden ter inspiratie:

```bash
alias l='ls -l --si --time-style=long-iso --color'
alias a='git add'
alias c='git commit -m'
alias h='git log --pretty=”format:%C(yellow)%h %C(blue)%ad %C(reset)%sC(red)%d %C(green)%an%C(reset), %C(cyan)%ar" --date=short --graph --all'
alias s='git status'
alias vu='vagrant up'
alias vD='vagrant destroy'
alias infra='cd /home/bert/Documents/Courses/InfraAutomation/21-22'
```

Nog meer voorbeelden vind je in <https://github.com/bertvv/dotfiles/blob/master/.bash.d/aliases.sh>.


Voor meer complexere shortcuts is `alias` minder geschikt, maar daarvoor kan je in `.bashrc` een functie definiëren, bv.

```bash
# Usage: dip CONTAINER
# This function will get the IP-address of the specified Docker container
dip() {
    docker container inspect "${1}" | jq '.[]|.NetworkSettings.Networks|.[]|.IPAddress'
}
```

## Vagrant

Vagrant is een command line tool die het aanmaken en configureren van virtuele machines automatiseert. Het ondersteunt een aantal virtualisatieplatforms, o.a. VirtualBox, Hyper-V, libvirt, enz. Wij zullen het gebruiken in combinatie met VirtualBox.

Je Git repository bevat een aantal Vagrant-omgevingen voor het opzetten van de VMs voor
je labo-opdrachten. Je kan een overzicht van de VMs opvragen met `vagrant status`. Dit commando moet uitgevoerd worden in een directory waar een `Vagrantfile` te vinden is. In dat geval ziet de uitvoer er bijvoorbeeld zo uit:

```console
$ vagrant status
Current machine states:

srv001              not created (virtualbox)

The environment has not yet been created. Run `vagrant up` to
create the environment. If a machine is not created, only the
default provider will be shown. So if a provider is not listed,
then the machine is not created for that environment.
$
```

Deze omgeving bevat één virtuele machine met naam `srv001`. Deze VM kan opgestart worden met `vagrant up srv001`. De eerste keer dat je dit doet wordt er een basis-VM gedownload met een minimale Linux-installatie. Doe dit best op een performant netwerk, bv. het bekabelde netwerk op de campus of thuis/op kot, en niet via WiFi. Deze “base box” wordt bijgehouden en zal telkens dienst doen als basis voor het opzetten van alle hosts in onze opstelling. Het downloaden gebeurt dus slechts één keer.

Na opstarten kan je inloggen met `vagrant ssh srv001`. Je bent ingelogd als gebruiker vagrant en kan commando’s uitvoeren met root-rechten door er `sudo` voor te plaatsen (geen wachtwoord vereist). Als het nodig mocht zijn: het wachtwoord van de gebruikers `vagrant` en `root` is telkens `vagrant`.

Als je `ls /` uitvoert, zal je merken dat er een directory `/vagrant` bestaat. Dit is je lokale repository op je fysieke systeem die gemount is binnen de VM. Dit is een eenvoudige manier om bestanden te delen tussen VM en het fysieke systeem.

Let er op dat je VMs niet meer vanuit je VirtualBox GUI opstart of bewerkt. Doe dit
nu enkel met Vagrant en vanuit een terminal. Het commando `vagrant` moet altijd
uitgevoerd worden vanuit de directory waar het bestand `Vagrantfile` zich bevindt.

De belangrijkste Vagrant commando’s worden opgesomd in de tabel hieronder. Daar waar
`[VM]` tussen rechte haken staat, is dat een optioneel argument. Als je het weglaat,
wordt de actie op alle VMs tegelijk uitgevoerd.

| Commando                 | Functie                                    |
| :----------------------- | :----------------------------------------- |
| vagrant status           | Geef een overzicht van de Vagrant omgeving |
| `vagrant up [VM]`        | Start VM op                                |
| `vagrant provision [VM]` | Voer het configuratiescript voor VM uit    |
| `vagrant ssh VM`         | Log in op VM als gebruiker `vagrant`       |
| `vagrant halt [VM]`      | Zet VM uit                                 |
| `vagrant reload [VM]`    | Herstart VM                                |
| `vagrant destroy [VM]`   | Vernietig VM                               |

## Markdown

[Markdown](https://daringfireball.net/projects/markdown/) is een syntax die toelaat om geformatteerde tekst in een eenvoudige platte tekstformaat te schrijven en deze vervolgens om te zetten naar HTML (zodat je het kan bekijken in de webbrowser), PDF (om af te drukken), LaTeX, enz. Markdown is de standaard geworden voor het schrijven van [handleidingen en documentatie op Github](https://guides.github.com/features/mastering-markdown/).

Je labo-verslagen en documentatie moeten ook in Markdown geschreven worden. Dit heeft een aantal voordelen: omdat Markdown tekstgebaseerd is, kan het in een versiebeheersysteem opgeslagen worden. Met Word-documenten gaat dit niet (Git is niet geschikt om wijzigingen in binaire bestanden te traceren). Een verzameling tekstbestanden is ook makkelijker te doorzoeken dan een verzameling Word-documenten, bijvoorbeeld met een tool als [The Silver Searcher](https://geoff.greer.fm/ag/).

**Leer tekst correct opmaken met Markdown.** [Dit is heel eenvoudig te leren](https://www.markdownguide.org/) en kost nauwelijks inspanning. Maar als je je niet houdt aan enkele basisregels, wordt je tekst niet correct omgezet naar HTML met een moeilijk leesbaar verslag als gevolg.

Goede teksteditors bieden ondersteuning voor Markdown en kunnen je een preview laten zien van hoe het bestand er in HTML zou uitzien. Er bestaan VSCode plugins die de basis-ondersteuning uitbreidt met enkele features die je productiviteit verhogen. Markdown All in One (door Yu Zhang) en markdownlint (door David Anson) zijn de interessantste:

- Shortcut `Ctrl+Shift+V` toont een preview van het bestand in HTML
- Shortcut `Alt+Shift+F` (Linux: `Ctrl+Shift+I`) formatteert tabellen zodat de verticale lijnen altijd mooi uitgelijnd zijn.
- Opmaak van wiskundige formules in LaTeX-notatie
- Markdownlint toont fouten in je Markdown-code aan de hand van een gekleurde golvende lijnt onder de tekst (zoals een spell-checker) en vat ze onderaan in het "Problems"-paneel samen in een lijst.

Met deze hulpmiddelen is het eenvoudig om Markdown correct te gebruiken en een mooi opgemaakte HTML-pagina te krijgen als je je verslagen naar Github pusht.