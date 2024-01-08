# FHI Statistikk Open API for Power Query 
***For Dummies by Dummies***


## Introduksjon

FHI Statistikk Open API er et åpent API for å hente data fra [Folkehelseinstituttets statistikkbanker](https://statistikk.fhi.no/). Dette notatet er et supplement til FHIs dokumentasjon, særskilt rettet mot å opprette kobling til dataene i MS Power BI/Excel/Fabric gjennom Power Query-editoren. Både målgruppe og avsender er helt gjennomsnittlige dataanalytikere med noe, men begrensede IT- og kodekompetanse. Dersom man har erfaring med å hente data fra SSB gjennom API bør det være håndterbart med FHI API også, selv om man får mindre hjelp og må gjøre mer selv. For mer avansert dokumentasjon eller bruk i andre kodespråk/verktøy henviser vi til FHIs GitHub.

FHI Statistikk Open API er fremdeles under utvikling og per dags dato (januar 2024) er det begrenset hvor mange datasett som er tilgjengeliggjort.

FHI dokumenterer API-et i [Swagger](https://statistikk-data.fhi.no/swagger/index.html) og [GitHub](https://github.com/folkehelseinstituttet/Fhi.Statistikk.OpenAPI#get-sources).

### Swagger
[Swagger](https://statistikk-data.fhi.no/swagger/index.html) er en nettbasert tjeneste som dokumenterer API-er og lar brukere teste, og bygge API-spørringer. For å bygge API-spørringen bruker vi Swaggersiden til FHI aktivt.

### GitHub
 FHI Statistikk Open API er også dokumentert på [Folkehelseinstituttets GitHub](https://github.com/folkehelseinstituttet/Fhi.Statistikk.OpenAPI#get-sources). Her finner man også en rekke kode-eksempler for bruk i ulike verktøy (men ikke Power Query).

---

## Hente oversikt over tabeller og variabler

Før vi bygger API-spørringen for å hente dataene må vi hente inn noen parametere som må inn i spørringen. Enkleste måten å hente oversikt over tabeller og metadata er ved hjelp av [FHIs Swagger](https://statistikk-data.fhi.no/swagger/index.html).


### Hente oversikt over statistikkbanker

Første punkt er å hente en oversikt over de ulike statistikkbankene («sources») som er støttet hos FHI. 
Klikk på tekstboksen under overskriften «Common» for å utvide den. Klik på «Try it out» og «Execute».  
I svaret man får vil man finne en web-adresse under «Request URL» som kan hentes inn i Power Query fra Web, dersom man ønske det.

```
 https://statistikk-data.fhi.no/api/open/v1/Common/source
```


I oversikten man får opp er det «source id» vi trenger å ta med oss videre. Foreløpig (januar 2024) er det stort sett bare under nøkkeltall («nokkel») det er publisert data. 


### Hente oversikt over tabeller ("Table")

I første blokk under «Table» (i Swagger) henter man en liste over de ulike datatabellene som ligger i hver statistikkbank. 
Under «sourceId» fyller du ut «id» fra oversikten vi nettopp fikk ut (husk å klikke på «Try it out» først!!!). 
I tabellen man får opp ser man de ulike tabellene som er publisert, med id («tableId») for hver tabell.

eks:
```
https://statistikk-data.fhi.no/api/open/v1/nokkel/Table
```


### Hente oversikt over variabler/dimensjoner

For å bygge spørringen må vi inkludere alle variabler (dimensjoner) i tabellen. Disse finner vi i tredje eller fjerde blokk under «Table». Hovedforskjellen er at i fjerde blokk er verdiene organisert som et hierarki fremfor som vertikal liste. I tillegg er både kodene og tekstverdiene inkludert. Dersom man åpner URL-en i Power Query, er denne oversikten mer oversiktlig, imo.

I oversikten må vi ta med oss kodene for variablene/dimensjonene («code» eller «dimension.code»). I tillegg bør vi ha oversikt over de tilhørende verdiene og formatet på disse.

eks:
```
https://statistikk-data.fhi.no/api/open/v1/nokkel/Table/173/query
https://statistikk-data.fhi.no/api/open/v1/nokkel/Table/173/dimension
```


## Bygge spørringen i Power Query ("Post")

For å bygge selve spørringen trenger vi altså:
•	sourceId – kode for statistikkbank
•	tableId – kode for tabell
•	koder for alle variabler i tabellen
 Selve innholdet i spørringen følger denne semantikken: 

```
{
  "dimensions": [
    {
      "code": "string",
      "filter": "string",
      "values": [
        "string"
      ]
    }
  ],
  "response": {
    "format": "string",
    "maxRowCount": 0
  }
}
```

I første omgang bygger vi opp spørringen i Swagger-vinduet.
Første bolk, «dimensions», definerer variablene, som defineres hver for seg, adskilt med komma. I kodesnutten må vi bytte ut hver forekomst av «string» for hver variabel.
```
{
      "code": "string",
      "filter": "string",
      "values": [
        "string"
      ]
    }
```

Verdien for «code» er koden for variablene vi hentet vi fra dimensjonsspørringen.

«Filter» kan, som i SSB-spørringene, ha verdiene «item», «all» eller «top». Med «item» oppgir man verdien eller verdiene op varabelen, adskilt med komme. Med «all» kombinert med asterisk (*) i «values» returneres data for alle verdier av variabelen. Man kan også kombinere asterisk med tekst for å avgrense spørringen (f. eks «20*» for alle verdier som begynner med 20).

I «values» oppgir vi hvilke verdier fra dimensjonstabellen vi vil spørre om. Verdiene må oppgis i formatet i «dimension values» i dimensjonsspørringen (f.eks «2016_2016» for årstallet 2016).

```
{
  "dimensions": [
    {
      "code": "AAR",
      "filter": "top",
      "values": [
        "3"
      ]
    },


    {
      "code": "GEO",
      "filter": "all",
      "values": [
        "34*"
      ]
    },


    {
      "code": "MEASURE_TYPE",
      "filter": "item",
      "values": [
        "RATE"
      ]
    }
  ],
```


