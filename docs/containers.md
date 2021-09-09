# Containervirtualisatie

## Leerdoelen

- Het concept van containervirtualisatie begrijpen en kunnen vergelijken met klassieke vormen van servervirtualisatie.
- Docker kunnen gebruiken om een netwerkservice of webapplicatie op te zetten:
    - Docker images kunnen beheren en gebruiken
    - Docker containers kunnen opstarten en beheren, eigenschappen opvragen
    - De werking van volumes voor persistente data 
    - Een Dockerfile kunnen schrijven of aanpassen voor een specifieke situatie
    - De werking van het gelaagde bestandssysteem begrijpen
        - Begrijpen hoe de inhoud van een Dockerfile invloed heeft op het aantal lagen van een container image
    - `docker-compose` kunnen gebruiken om reproduceerbare omgevingen met meerdere, onderling afhankelijke, Docker containers op te zetten

## Inleiding

Containervirtualisatie is een vorm van servervirtualisatie die zich vooral onderscheidt van andere vormen door het feit dat virtuele machines, containers genaamd, enorm klein zijn en dus ook minder systeembronnen van het host-systeem gebruiken. Het concept bestaat al tientallen jaren, o.a. binnen de context van mainframes, maar is pas echt bekend geworden na de [demo van Docker](https://www.youtube.com/watch?v=wW9CAH9nSLs) op de conferentie PyCon 2013.

Bij "full virtualization" zal de virtuele machine een simulatie zijn van een fysieke computer met een cpu, geheugen, een harde schijf, enz. Om een en ander performanter te laten verlopen kan het host-systeem de VM rechtstreeks toegang geven tot bepaalde hardware-bronnen, bv. door een cpu-kern of een bepaald deel van het fysieke RAM-geheugen exclusief toe te kennen aan die VM. Op deze VM kan je een besturingssysteem installeren, programmabibliotheken en de nodige applicatie(s).

Containervirtualisatie werkt anders, in die zin dat er in een container enkel een applicatie zit (samen met eventuele afhankelijkheden zoals programmabibliotheken). Een container "hergebruikt" de kernel van het onderliggende besturingssysteem van de host.
