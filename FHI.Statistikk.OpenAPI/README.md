# FHI Statistikk Open API for Power Query 
***For Dummies by Dummies***

>[!IMPORTANT]
>Dette dokumentet er utarbeidet uten tilknytning til eller kvalitetsikring fra FHI, og er forfattet uten særlig kompetanse. Feil, mangler og merksnodigheter kan forekomme. Alle korreksjoner, innspill og andre tilbakemeldinger tas i mot med åpne armer. Alt innhold er fritt å bruke og gjenbruke.


* Mal for å bruke API-spørringen i Power Query finnes [her](FHI.Statistikk.OpenAPI/FHI_API_til_PQ_mal)
* Eksempelspørringer finnes [her](FHI.Statistikk.OpenAPI/FHI_API_til_PQ_eksempler)
* Dimensjonstabell for geografiske enheter tilpasset FHI-data finnes [her](FHI.Statistikk.OpenAPI/FHI_dim/dim_regioner_FHI.csv)

---


## Introduksjon

FHI Statistikk Open API er et åpent API for å hente data fra [Folkehelseinstituttets statistikkbanker](https://statistikk.fhi.no/). Dette notatet er et supplement til FHIs dokumentasjon, særskilt rettet mot å opprette kobling til dataene i MS Power BI/Excel/Fabric gjennom Power Query-editoren. Både målgruppe og avsender er helt gjennomsnittlige dataanalytikere med noe, men begrensede IT- og kodekompetanse. Dersom man har erfaring med å hente data fra SSB gjennom API bør det være håndterbart med FHI API også, selv om man får mindre hjelp og må gjøre mer selv. For mer avansert dokumentasjon eller bruk i andre kodespråk/verktøy henviser vi til FHIs GitHub.



FHI Statistikk Open API er fremdeles under utvikling og per dags dato (januar 2024) er det begrenset hvor mange datasett som er tilgjengeliggjort.

FHI dokumenterer API-et i [Swagger](https://statistikk-data.fhi.no/swagger/index.html) og [GitHub](https://github.com/folkehelseinstituttet/Fhi.Statistikk.OpenAPI#get-sources).

>[!TIP]
>### Swagger
>[Swagger](https://statistikk-data.fhi.no/swagger/index.html) er en nettbasert tjeneste som dokumenterer API-er og lar brukere teste, og bygge API-spørringer. For å bygge API-spørringen bruker vi [Swaggersiden til FHI](https://statistikk-data.fhi.no/swagger/index.html)  aktivt.
>
>### GitHub
>FHI Statistikk Open API er også dokumentert på [Folkehelseinstituttets GitHub](https://github.com/folkehelseinstituttet/Fhi.Statistikk.OpenAPI#get-sources). Her finner man også en rekke kode-eksempler for bruk i ulike verktøy (men ikke Power Query).

---




## #1 Hente oversikt over tabeller og variabler

Før vi bygger API-spørringen for å hente dataene må vi hente inn noen parametere som må inn i spørringen. Enkleste måten å hente oversikt over tabeller og metadata er ved hjelp av [FHIs Swagger](https://statistikk-data.fhi.no/swagger/index.html).



### Hente oversikt over statistikkbanker

Første punkt er å hente en oversikt over de ulike statistikkbankene («sources») som er støttet hos FHI. 
Klikk på tekstboksen under overskriften «Common» for å utvide den. Klik på «Try it out» og «Execute».  
I svaret man får vil man finne en web-adresse under «Request URL» som kan hentes inn i Power Query fra Web, dersom man ønske det.

```html
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


## #2 Bygge spørringen i Swagger ("Post")

For å bygge selve spørringen trenger vi altså:
* sourceId – kode for statistikkbank
* tableId – kode for tabell
* koder for alle variabler i tabellen
 Selve innholdet i spørringen følger denne semantikken: 

```json
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
```json
{
      "code": "string",
      "filter": "string",
      "values": [
        "string"
      ]
    }
```

Verdien for «code» er koden for variablene vi hentet vi fra dimensjonsspørringen.

«Filter» kan, som i SSB-spørringene, ha verdiene 
* item
* all
* top
  
Med «item» oppgir man verdien på varabelen (eller flere verdier adskilt med komma). Med «all» kombinert med asterisk (\*\) i «values» returneres data for alle verdier av variabelen. Man kan også kombinere asterisk med tekst for å avgrense spørringen (f. eks «20*» for alle verdier som begynner med 20).

I «values» oppgir vi hvilke verdier fra dimensjonstabellen vi vil spørre om. Verdiene må oppgis i formatet i «dimension values» i dimensjonsspørringen (f.eks «2016_2016» for årstallet 2016).

```json
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
        "RATE",
	"SMR"
      ]
    }
  ],
```

I den andre bolken i kodesnutten angir vi formatet vi vil ha på de returnerte dataene. Verdiene vi kan velge for format er «csv2», «csv3» og «json-stat2».
* csv2 returnerer data i csv-format med lesbare navn på verdiene for variablene
* csv3 returnerer data i csv-format med verdiene for variablene oppgitt som koder
* json-stat2 returnerer data i json-format som krever en del utpakking

Etter «maxRowCount» kan du angi hvor mange rader du vil sette som maks i responsen. Utelater du hele linjen settes verdien til uendelig.

```json
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
        "RATE",
	"SMR"
      ]
    }
  ],


  "response": {
    "format": "csv2",
    "maxRowCount": 50000
  }
}
```

---

>[!NOTE]
> ***Csv2 eller csv3 for Power Query?***
>
>Ett av csv-formatene er helt klart mest hensiktsmessig for bruk i Power Query. Dessverre er ingen av de to formatene helt ideelle. Csv2 gir forståelige tekster og krever lite bearbeiding. Problemet er at den geografiske variabelen blir oppgitt med kommunenavn, uten innbakt kommunenummer som hos SSB. Det kan bli problematisk hvis man ønsker å koble datasettet med et annet.  Med csv3 får man kommunenummer, men også lite menneskevennlige koder for de andre variablene. Muligens er det enkleste å laste inn dataene som csv2 for så å koble tabellen opp mot dimensjonsspørringen for regioner. En støtte-tabell for region-dimensjonen finner du også [her](FHI.Statistikk.OpenAPI/FHI_dim/dim_regioner_FHI.csv).

---

## #3 Sette sammen spørringen i Power Query

Selve spørringen i Power Query bygges opp ved a velg «Blank Query» og «Advanced Editor».
Først i spørringen defineres tre parametere. Det to første henviser til statistikkbanken og tabellen vi vil spørre mot, med parametrene sourceId og tableId vi fant tidligere.

```
let
	sourceId = "nokkel",
	tableId = "173",
```

Det siste parameteret kaller vi «body» og inneholder selve spørringen vi nettopp laget, men med alle anførselstegn erstattet med doble anførselstegn (klipp og lim inn i notisblokk og søk og erstatt alle). Kodesnutten innledes med  *body = "* og avsluttes med *"*. 

```
    body = "

		{
		  ""dimensions"": [

			{
			  ""code"": ""AAR"",
			  ""filter"": ""top"",
			  ""values"": [
				""3""
			  ]
			}
		,

			{
				  ""code"": ""GEO"",
				  ""filter"": ""all"",
				  ""values"": [
					""34*""
				  ]
			},

			{
				  ""code"": ""MEASURE_TYPE"",
				  ""filter"": ""item"",
				  ""values"": [
					""RATE"",
					""SMR""
				  ]
			}
		  ],

		  ""response"": {
			""format"": ""csv2"",
			""maxRowCount"": 50000
		  }
		}

	"
,

```


Deretter limer vi inn en kodefnutt som bruker disse parameterne for å foreta spørringen. Så lenge man har definert «url» og «body» er det ikke nødvendig å endre denne koden.

```m
    Response= 
    sv.Document(Web.Contents("https://statistikk-data.fhi.no/api/open/v1/" & sourceId & "/Table/" & tableId & "/data",
    [Content=Text.ToBinary(body), 
    Headers=[#"Content-Type"="application/json"]]),
    [Delimiter=";", 
    Encoding=65001, 
    QuoteStyle=QuoteStyle.None]),
    #"Promoted Headers" = Table.PromoteHeaders(Response, [PromoteAllScalars=true])
in
    #"Promoted Headers"
```


Sammensatt bør spørringen se omtrent slik ut:

```html
let
	sourceId = "nokkel",
	tableId = "173",

    body = "

		{
		  ""dimensions"": [

			{
			  ""code"": ""AAR"",
			  ""filter"": ""top"",
			  ""values"": [
				""3""
			  ]
			}
		,

			{
				  ""code"": ""GEO"",
				  ""filter"": ""all"",
				  ""values"": [
					""34*""
				  ]
			},

			{
				  ""code"": ""MEASURE_TYPE"",
				  ""filter"": ""item"",
				  ""values"": [
					""RATE"",
					""SMR""
				  ]
			}
		  ],

		  ""response"": {
			""format"": ""csv2"",
			""maxRowCount"": 50000
		  }
		}

	"
	
	
,
    Response= 
    Csv.Document(Web.Contents("https://statistikk-data.fhi.no/api/open/v1/" & sourceId & "/Table/" & tableId & "/data",
    [Content=Text.ToBinary(body), 
    Headers=[#"Content-Type"="application/json"]]),
    [Delimiter=";", 
    Encoding=65001, 
    QuoteStyle=QuoteStyle.None]),
    #"Promoted Headers" = Table.PromoteHeaders(Response, [PromoteAllScalars=true])
in
    #"Promoted Headers"
```

Det var det.


