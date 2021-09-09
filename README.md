# Syllabus Infrastructure Automation

In deze repository vind je de syllabus voor de cursus Infrastructure Automation (3e jaar bachelor toegepaste informatica aan Hogeschool Gent).

Gerelateerde repositories:

- Syllabus: <https://github.com/HoGentTIN/infra-syllabus>
- Slides van de lessen: <https://github.com/HoGentTIN/infra-slides>
- Vagrant demo-omgeving: <https://github.com/HoGentTIN/infra-demo>
- Opgave labo-opdrachten: <https://github.com/HoGentTIN/infra-labs>

## De syllabus lokaal genereren

De syllabus is opgemaakt in Markdown en omgezet in een website met mkdocs.

Om de syllabus zelf lokaal te genereren en te bekijken:

```console
mkdocs build
mkdocs serve
```

Publiceren op Github (kan enkel als je push-rechten hebt of in je eigen kloon van deze repo):

```console
mkdocs gh-deploy --site-dir ../gh-pages/
```

## Bijdragen

Bijdragen aan het hier gepubliceerde lesmateriaal zijn van harte welkom! verbeteren van tikfouten, toevoegingen, onduidelijkheden melden, enz. Je kan een Issue of een Pull Request openen.

## Licentie-informatie

Deze syllabus is samengesteld door [Bert Van Vreckem](https://github.com/bertvv/) en valt onder de [Creative Commons Naamsvermelding-GelijkDelen 4.0 Internationale Publieke Licentie](http://creativecommons.org/licenses/by-sa/4.0/).
