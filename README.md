![Statens Kartverk](https://github.com/kartverket/etinglysing-systembeskrivelse/blob/master/doc/kartverket.png)


# Systembeskrivelse for eksterne aktører


## Innledning

Kartverkets løsning for elektronisk innsending til tinglysing tilbyr et Web Service basert API for elektronisk innsending av dokumenter til tinglysing. Grensesnittet tilbyr tjenester for validering av dokumenter før innsending, tjenester for innsending til tinglysing samt tjenester for uthenting av tinglysingsresultat.

### Innholdsfortegnelse

1. [Konsepter](#1-konsepter)
2. [Innsendingsgrensesnitt](#2-innsendingsgrensesnitt)
3. [Innsending med Altinn Formidlingstjeneste](#3-innsending-med-altinn-formidlingstjeneste)
4. [Prosessflyt](#4-prosessflyt)
5. [Funksjonalitet og begrensninger](#5-funksjonalitet-og-begrensninger)
6. [Innsending av signerte dokumenter](#6-innsending-av-signerte-dokumenter)
7. [Hvilke identifikasjonsnumre følger med inn til signaturkontroll](#7-hvilke-identifikasjonsnumre-følger-med-inn-til-signaturkontroll)
8. [Hvordan behandles signaturene i et signert dokument](#8-hvordan-behandles-signaturene-i-et-signert-dokument)
9. [Bruk av BANKID grensesnitt for å signere SDO](#9-bruk-av-bankid-grensesnitt-for-å-signere-sdo)
10. [Statustjeneste for systemet](#10-statustjeneste-for-systemet)
11. [Endepunkter for test og produksjon](#11-endepunkter-for-test-og-produksjon)


## 1. Konsepter

### Melding

Den minste enhet som kan sendes til tinglysing er en melding inneholdende ett eller flere dokumenter, hvert bestående av en eller flere rettsstiftelser. Alle dokumenter i en melding behandles som en transaksjonell enhet, slik at alle dokumenter blir enten avvist, nektet eller tinglyst. Alle dokumenter i en melding får samme prioritet, men behandles i en gitt rekkefølge som må eksplisitt angis av innsender i et følgebrev ved innsending. 

### Dokument 

Den enheten som tinglyses kalles et dokument, og består av en eller flere rettsstiftelser som tinglyses i den rekkefølge de er oppført i dokumentet.

Før et dokument kan legges i en melding og sendes til tinglysing må det pakkes i en digital struktur og signeres digitalt av de personer eller juridiske enheter som måtte være nødvendig for å få dokumentet tinglyst. Alle signaturene for et dokument gjelder hele dokumentets innhold, det vil si at alle signatarene må signere på eksakt det samme innholdet. Det er ikke mulig å signere på enkeltelementer i dokumentet. 

Når et dokument har blitt signert av alle parter må innsender sørge for å kombinere signaturene samt forsegle strukturen. Da er dokumentet klart til å legges i en melding sammen med eventuelle andre signerte dokumenter.

Dokumenter som skal inngå i en melding kan opprettes og signeres uavhengig av hverandre. 

Signeringsteknologien sikrer integriteten til innholdet, det vil ikke være mulig å endre på innholdet av et signert dokument. Signeringsløsningen vil sjekke om noen av sertifikatene som ble brukt til å signere med har blitt tilbakekalt eller har utløpt, slik at kun gyldige sertifikater vil bli akseptert. 

Beskrivelse av hvordan man kan sette sammen en signert melding basert på det resultatet man får når man signerer noe med BankId er beskrevet i seksjonen Innsending av signerte dokumenter.

### Vedlegg 

For enkelte dokumenter vil det være nødvendig med informasjon i tillegg til den strukturerte informasjonen som finnes i selve dokumentet, for at dokumentet skal kunne tinglyses. 

Slik tilleggsinformasjon kan sendes med i meldingen i form av vedlegg til dokumentet.

### Følgebrev 

Før innsending må innsender lage et følgebrev som ved hjelp av interne referanser i meldingen lister opp alle dokumenter og eventuelle vedlegg som meldingen inneholder. Følgebrevet angir dermed entydig 
hvilke dokumenter og vedlegg som inngår i meldingen uten at disse selv må være en del av følgebrevet. Følgebrevet legges i en digital struktur som signeres av innsender. 

Det signerte følgebrevet, de signerte dokumentene samt eventuelle vedlegg legges i en åpen meldingsstruktur som sendes til tinglysing. Rekkefølgen på dokumentene i meldingen må være den samme som angitt i følgebrevet. 

### Hvem signerer dokument og følgebrev

En melding må alltid inneholde en liste av dokumenter sammen med et følgebrev. Normalt signeres dokumenter og følgebrev hver for seg, av forskjellige parter. 
Typisk vil dokumentet måtte signeres av rettighetshaver, for eksempel rettighetshaver til eiendomsrett, eller rettighetshaver i en Leierett som pantsettes. 
I sistnevnte tilfelle må denne rettighetshaveren signere når et pant i en slik rettighet skal tinglyses. 
Den som sender pantedokumentet sammen med følgebrevet, typisk en finansinstitusjon, vil normalt signere dette følgebrevet. 
Følgebrevet skal signeres med et virksomhetssertifikat.  

I dette eksempelet er det to ulike parter som signerer. I et annet tilfelle, for eksempel når man skal slette et pant, kan parten som skal 
signere dokument og følgebrev være den samme. Det er derfor tilrettelagt for at man skal kunne elektronisk signere dokument og følgebrev samlet. 
Det er ikke alltid dette gir mening, men i de tilfeller der organisasjon kan signere og dette er den samme part som signerer følgebrevet, kan 
det gir færre signeringer og dermed også reduserte kostnader.

Se for øvrig beskrivelsen [Signatur Fullmakt og Vitner](http://www.kartverket.no/eiendom/signatur-fullmakt-og-vitner/) for en mer utførlig beskrivelse av dette konseptet.


## 2. Innsendingsgrensesnitt


Innsendingsgrensesnittet baserer seg på en signeringsløsning, basert på SEID-SDO med bruk av BankID XML og XSL for visning 
av dokumenter til signering for sluttbruker.

Innsendingsapi for tinglysing støtter validering og tinglysing av signerte og usignerte meldinger som inneholder instruksjoner 
om dokumenter som skal tinglyses. Grensesnittet er tilpasset signeringsløsningen på en slik måte at datastrukturen som 
representerer dokumentene, og som sendes inn i tjenestene, er den samme som brukes som grunnlag for visning med XSL med 
BankID eller med andre eId-leverandører som støtter BIDXML. Presentasjonsmalen er laget slik at all informasjon 
som ligger i XML strukturen vil bli vist, således vil samtlige av elementene i det som signeres av rettighetshaver ha 
blitt forevist den rettighetshaveren som har signert.

**Som kontaktpunkt for BankID henviser vi til [BankId Bedrift](https://www.bankid.no/bedrift/)**

Se også egen seksjon litt senere: Innsending av signerte dokumenter for en beskrivelse av dette.

Innsendingsgrensesnittet er definert som en WebService med endepunkter og operasjoner for innsending av signerte og usignerte dokumenter, en valideringstjeneste som man kan bruke i test og produksjon for å verifisere innhold før man sender dem til signering, samt en tjeneste for å hente status på tidligere innsendte dokumenter. 

Samtlige av disse tjenestene er tilgjengelige som WebService i test for å gi en tettere og raskere integrasjon i test, der to av de tre 
tjenestene er synkrone og returnerer data, mens kun valideringsgrensesnittet for usignerte og signerte meldinger er 
tilgjengelige som WebService i produksjon. For all annen integrasjon skal filer til tinglysing og status på innsendte dokumenter 
formidles til Kartverket gjennom Altinn formidlingstjeneste.. 
Se egen seksjon om Altinn formidlingstjeneste for en beskrivelse av denne tjenesten. Kartverket har publisert eksempelklienter 
som open-source som viser hvordan denne integrasjonen kan etableres: [Altinn eksempelklient](https://github.com/kartverket/eksempelklient-etinglysing-altinn)

### Tjenester

Det er implementert tre funksjoner for innsending. Disse kan brukes uavhengig av grunnbokens øvrige tjenester, og de er uavhengige av hverandre.

Signerte data vil kunne brukes i tjenestene for å validere og å sende melding til tinglysing i produksjon og i test. For signerte meldinger i test kommuniserer grunnboksystemet med BankID i test, og aksepterer meldinger signert av sertifikater utstedt fra BankID test CA. For produksjon aksepterer vi signerte meldinger signert av BankID CA.

Følgende tjenester tilbys som WebService i test, men bare valider er tilgjengelig som WebService i produksjon. For elektronisk innsending til tinglysing må innsender benytte Altinn Formidlingstjeneste. Payloaden som sendes med til Altinn er den samme som i Webtjenestene. Dette er derfor å regne som et rent transportanliggende. Alle tre abstraksjonene er modellert også i formidlingstjenesten så dette bør oppleves som ganske likt.

#### valider

Tjenesten kan kalles for å validere en signert eller en usignert melding, både i test og i produksjon.  Valideringstjenesten 
gir ingen garanti for at dokumenter som validerer uten feil vil kunne tinglyses.  Formålet med tjenesten er å forsøke å finne 
åpenbare mangler i dokumenter før de sendes til signering, eller gjøre oppmerksom på eventuelle forhold på valideringstidspunktet 
som kan hindre tinglysing. Dette bør fortrinnsvis gjøres for å forsikre seg om at et dokument er gyldig før man sender det 
til signering. Et dokument som er signert kan aldri endres, men må signeres på nytt. 

Det er ikke noe krav om at man i det avleverende fagsystemet skal bruke valider før man sender data til tinglysing. 

Tjenesten vil for signerte meldinger slå opp data samt validere mot eId-leverandørers testsystemer for sjekke om sertifikat brukt til signering er gyldig, og hvis nødvendig hente identifikasjonsnummer tilknyttet sertifikatet fra BankId sin oppslagstjeneste.

#### sendTilTinglysing

En usignert (i test) eller signert melding (i test og prod) sendes inn elektronisk til tinglysing. Man sender med en intern referanse, forsendelsesreferanse, som kan brukes som referanse i avleverende fagsystem. Hvis meldingen blir lagret i Kartverkets systemer så vil den internt bli tilordnet en innsendingId. Det er denne innsendingId man bruker når man skal hente informasjon om meldingen. Signerte meldinger kan sendes til tinglysing både i test og i produksjon, mens usignerte meldinger kan sendes til tinglysing kun i test. I forkant kan det være nyttig å ha testet at meldingen validerer. Det vil kunne være de samme feilene som propageres fra sendTilTinglysing som fra valider, gitt at valideringen feiler på valideringer på tidspunktet den blir validert ved mottak.

Tjenesten vil i produksjon slå opp data samt validere mot eId-leverandørers produksjonssystemer. Tjenesten returnerer en forsendelsesstatus som inneholder informasjon om mottatt melding, eventuelle avvisninger samt en unik innsendingId hvis Kartverket har akseptert å motta meldingen videre prosessering.

#### hentStatus

Etter at en melding er mottatt av systemet kan man hente oppdatert behandlingsstatus for den innsendte meldingen. Input til 
denne tjenesten er en innsendingId som man har fått tilbake etter at man har sendt en melding med minst ett dokument til 
tinglysing som Kartverket har akseptert. Ved innsending med Altinn formidlingstjeneste vil man normalt ikke benytte hentStatus, 
da statusmeldingene automatisk pushes fra systemet og kan hentes ut via formidlingstjenesten, se egen seksjon om Altinn 
formidlingstjeneste.

#### Sekvensdiagram
 
![Sekvensdiagram](https://github.com/kartverket/etinglysing-systembeskrivelse/blob/master/doc/sekvensdiag.png)
 
#### Forsendelse
 
![Figur 1Forsendelse](https://github.com/kartverket/etinglysing-systembeskrivelse/blob/master/doc/forsendelse1.png)

Dokumenter som ønskes tinglyst som en enhet samles som en signert eller en usignert melding i en forsendelse. Meldingen inneholder ett eller flere dokumenter, samt et følgebrev som refererer til de individuelle dokumentene, og dermed også bekrefter hvor mange dokumenter forsendelsen inneholder og hvilken rekkefølge disse har.

Følgebrevet inneholder også innsenders identifikasjonsnummer, og det valideres at identifikasjonsnummeret er et gyldig organisasjonsnummer. 

Innsendingsgrensesnittet er utformet med henblikk på at dokumenter skal kunne opprettes og signeres uavhengig av grunnboksystemet, 
dersom innsender har tilstrekkelig informasjon. Innsender må blant annet ha informasjon 
om koder som skal anvendes i dokumentene som skal tinglyses. Informasjon om koder er statisk og vil kunne hentes på forhånd 
gjennom avgivergrensesnittet, eller kopieres inn og vedlikeholdes i et fagsystem på utsiden av kartverket. Innsendte koder 
som ikke kan oversettes til koder systemet kjenner igjen vil forårsake systemfeil. 

#### Vedlegg

En melding til tinglysing kan fra også inneholde vedlegg til dokumentene i meldingen. Vedlegg legges i forsendelsen på samme måte enten meldingen er signert eller usignert, og hvert vedlegg må ha en vedleggsreferanse som er unik innenfor meldingen, samt en base64-encodet pdf.

Følgebrevet må inneholde metadata for hvert vedlegg, der disse metadata kobler et vedlegg til et dokument og til en eller flere vedleggskategorier.

Vedlegg metadata inneholder også digest for den base64-encodede pdf-en (påkrevd kun ved signert melding, men valideres dersom finnes ved usignert melding), samt en digest-algoritme. Oppgitt digest blir kontrollert mot vedlegget via angitt algoritme. Dette er tilsvarende som digest for dokumentene i følgebrevets dokumentrekkefølge. Grunnlaget for beregningen er base64-formatet av vedlegget, slik det er innsendt i pdfVedlegg-elementet.

Signeringen av følgebrevet inkludert vedlegg metadata innebærer at innsender bekrefter ‘rett kopi’ av vedleggene. For øvrig henvises det til [vedleggsveilederen](https://github.com/kartverket/etinglysing-vedlegg/) for mer informasjon om vedlegg.


#### Forsendelsesstatus

![Figur 2 Forsendelsesstatus](https://github.com/kartverket/etinglysing-systembeskrivelse/blob/master/doc/forsendelse2.png)

Tjenestene for validering og tinglysing returnerer en forsendelsesstatus. 

For validering vil behandlingsinformasjon være satt dersom det ble funnet avvik, mens tinglysingsinformasjon aldri vil være satt. 
Eventuell behandlingsinformasjon vil inneholde et eller flere kontrollresultater som beskriver avvik funnet. 
Kontrollresultatene kan gjennomgås for å vurdere hva som vil kunne skje dersom saken sendes til tinglysing.

Validering er å anse som informasjon, og er ingen garanti for at det samme vil skje ved innsending til tinglysing. 
Eksempelvis kan omstendighetene ha endret seg mellom validering og tinglysing, slik at noe som har validert uten avvik vil avvises ved tinglysing. 
Løsninger som benytter validering må altså ta høyde for at resultatet ved tinglysing kan bli noe annet enn valideringen tilsa, en forsendelse kan bli avvist selv om valideringen ikke returnerte noen kontrollresultater.

For elektronisk innsendte meldinger er følgende verdier relevante:

| Saksstatus | Behandlingsutfall | Forklaring |
|---------|----------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| MOTTATT |	UAVKLART | Meldingen er mottatt og venter på registrering i grunnboken. Registrering av meldinger skjer i rekkefølge i henhold til tildelt registreringstidspunkt. |
| UNDER_BEHANDLING | UAVKLART | Meldingen er registrert i grunnboken og etterfølgende meldinger vil ta hensyn til den, men dens endelige behandlingsutfall er ikke avgjort. Meldingen kan enten bli tinglyst eller nektet. |
| AVSLUTTET | AVVIST | Meldingen er avvist og ble ikke registret i grunnboken |
| AVSLUTTET | TINGLYST | Meldingen er tinglyst i grunnboken. |
| AVSLUTTET | NEKTET | Meldingen er nektet i grunnboken. |
| UKJENT_TEKNISK_FEIL | UAVKLART | Meldingen har støtt på en ukjent teknisk feil og venter på videre avklaring./

Behandling av meldingen vil fortsette fra forrige tilstand på et senere tidspunkt. Formidler av meldingen trenger ikke å foreta seg noe.<ul>
<li>Hvis forrige status var MOTTATT er meldingen ennå ikke registrert i grunnboken og vil bli forsøkt registrert på et senere tidspunkt.</li>
<li>Hvis forrige status var UNDER_BEHANDLING er meldingen allerede registrert i grunnboken og videre behandling vil skje på et senere tidspunkt.</li></ul>
 
Hvis dokumentene i meldingen er avvist vil statusfeltene saksstatus og behandlingsutfall i forsendelsesstatus inneholde den overordnede tilstanden, mens behandlingsinformasjon vil inneholde kontrollresultat med begrunnelse for avvisningen i form av strukturert informasjon med koder, men også med lesbare tekster.

Dersom dokumentene i meldingen er blitt registrert i grunnboken inneholder forsendelsesstatus blant annet informasjon om registreringstidspunkt 
(prioritet), samt tildelte dokumentnummer og rettsstiftelsesnummer. Disse er angitt med en kobling til innsenders oppgitte referanser for hhv. 
dokumentene og rettsstiftelsene.

#### Kontrollresultater

Kontrollresultater kan ha følgende utfall:

| Utfall | Forklaring |
|---------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| IKKE_GODKJENT | Kontrollen er ikke godkjent. |
| UAVKLART | Det må vurderes manuelt av en saksbehandler om kontrollen kan godkjennes eller ikke. |

* Dersom man får minst ett IKKE_GODKJENT kontrollresultat ved validering er sjansen stor for at forsendelsen vil avvises ved innsending til tinglysing.
* Dersom man får minst ett IKKE_GODKJENT kontrollresultat ved innsending til tinglysing vil forsendelsen bli avvist.
* Dersom man får minst ett UAVKLART kontrollresultat ved validering og ingen IKKE_GODKJENT er sjansen stor for at forsendelsen vil gå til manuell behandling ved innsending til tinglysing.
* Dersom man får minst ett UAVKLART kontrollresultat ved innsending til tinglysing og ingen IKKE_GODKJENT vil forsendelsen gå til manuell behandling.

Dokumentene i en forsendelse som går til manuell behandling vil være registrert i grunnboken. Den manuelle behandlingen gir som resultat at dokumentene i forsendelsen enten blir tinglyst eller nektet.

### Bekreftet grunnboksutskrift

Når dokumentene i en melding er tinglyst, gjøres det tilgjengelig en bekreftet grunnboksutskrift for hver av registerenhetene
som er berørte av denne tinglysingen. Hver utskrift er representert ved en link i Forsendelsesstatus-strukturen, der linken peker til en 
bekreftet grunnboksutskrift beliggende på en server hos Kartverket.

En utprinting eller andre former for kopi av en bekreftet utskrift som aksesseres via linken vil i seg selv ikke være en bekreftet utskrift, 
men vil være å anse som en visning av den bekreftede utskriften som er lagret hos Kartverket.

### Feilhåndtering

For forventede feil som systemet håndterer, f.eks. valideringsfeil, returneres begrunnelsene i form av informasjon i behandlingsinformasjon, som beskrevet over. 

Ved uventede feil, eller dersom meldingen ikke er lesbar, vil slike feil propageres som en SOAPFault med en SystemException som underliggende årsak. I slike tilfeller må avleverende system forsøke å rette feilen i sitt fagsystem før meldingen sendes på nytt, eller vente til eventuelle feil er rettet i mottakende system.  Et eksempel på en slik feil kan være innsendt XML som ikke er i henhold til skjema.

### Kontekstinformasjon

I forbindelse med signering av dokumenter er det krav til å vise tilleggsinformasjon (kontekst-informasjon) for sluttbruker, slik som navn på personer, for at sluttbruker skal få bedre forståelse av hva det signeres på. Ved innsending må alltid fullstendig identifikator for hvert dataelement sendes inn, selv om denne ikke nødvendigvis er vist for sluttbruker. Også kontekstinformasjon vist for sluttbruker må sendes inn, selv om systemet i utgangspunktet ikke har behov for denne, men det er avgjørende for den visuelle inspeksjonen ved signering, og det er det denne informasjonen brukes til.

Det stilles krav til validering av innsendt kontekstinformasjon. Eksempelvis er det krav om at navn og identifikasjonsnummer 
på personer skal valideres mot data fra folkeregisteret, mens kodeverdi og navn på koder skal valideres mot systemets egne data. 
Systemet avviser en melding dersom validering av kontekstinformasjon feiler. 

#### Validering av navn fra folkeregisteret

Navn innsendt på kontekstinformasjon av typen:

```xml
<person>
   <navn>OLA NORMANN</navn>
   <identifikasjonsnummer>12345678911</identifikasjonsnummer>
</person>
```

Vil bli validert på to måter, først mot det som folkeregisteret representerer som Fullt Navn. Dette vil være et navn som 
forkortes i henhold til de regler som folkeregisteret definerer på 50 tegn. Deretter vil man validere opp mot den sammensatte 
fulle og sammensatte strengen av ETTERNAVN+FORNAVN+MELLOMNAVN. Sammenlikningen er case insensitiv.

#### Validering av navn fra enhetsregisteret

Validering av navn fra enhetsregisteret blir gjort mot det som heter «Redigert navn» som har en maksimal lengde på 50 tegn. 
Hvis ikke redigert navn stemmer med det innsendte navnet så validerer vi mot et navn bestående av organisasjonsnavn1..organisasjonsnavn5 
med mellomrom i mellom. Sammenlikningen er case insensitiv.

## 3. Innsending med Altinn Formidlingstjeneste

Altinn Formidlingstjeneste er den primære kanalen for å benytte innsendingsgrensesnittet. I produksjon vil sendTilTinglysing 
og hentStatus kun være tilgjengelig via Altinn grensesnittet mens valider vil være tilgjengelig både via Altinn og WebService 
grensesnittet. For detaljert dokumentasjon og kodeeksempler på hvordan man kan integrere mot innsendingsgrensesnittet via 
Altinn finnes det en eksempelklient tilgjengelig i GitHub. Det vil derfor kun bli gitt en overordnet beskrivelse av 
funksjonaliteten i dette dokumentet.

Formidlingstjenesten i Altinn baserer seg på at man laster opp en zip fil til Altinn som deretter blir formidlet til riktig mottaker(e). Innholdet i zip filen vil være en fil som inneholde en forsendesle. Det er kun støtte for å sende en forsendelse av gangen i zip filen. Tjenesten som Kartverket har satt opp er konfigurert sånn at det er kun mulig å sende og motta filer til/fra Kartverket. 

#### Bruk av Altinn grensesnittet

For å kalle Altinn formidlingstjenesten benyttes et sett SOAP endepunkter. For å kalle en av operasjonene i innsendingsgrensesnittet til Kartverket må man først initiere en overførsel i Altinn og angi hvilken operasjon man ønsker å kalle (sendTilTinglysing, hentStatus eller valider). Deretter laster man opp en zip fil som inneholder en (og bare en) fil med payload på formatet definert av innsending.xsd. For å følge med på status for en opplastet fil må det polles etter kvitteringen til filen. Der vil det være synlig om forsendelsen har blitt hentet ned fra Kartverket, og om den har blitt mottatt eller om noe har feilet. Se figur nedenfor for en beskrivelse av flyten og hvilke tjenester fra Altinn som benyttes.

![Figur 1 Altinn](https://github.com/kartverket/etinglysing-systembeskrivelse/blob/master/doc/altinn1.png)
 
Dersom forsendelsen har blitt mottatt og innsendingsgrensesnittet har klart å lese forsendelsen vil det bli sendt en eller 
flere filer tilbake via Altinn som respons på forsendelsen. Det må derfor kontinuerlig sjekkes mot Altinn om det har kommet nye filer. 
Operasjonene valider og hentStatus vil kun trigge at en fil blir sendt tilbake, mens dersom man kaller sendTilTinglysing 
vil man få en ny forsendelsesstatus hver gang det skjer en relevant endring i status. Ved
sendTilTinglysing trenger man altså normalt ikke benytte hentStatus operasjonen, ettersom status pushes automatisk. 

Man poller på filer 
basert på organisasjonsnummer. For å koble sammen forsendelseresponsen til riktig utgående forespørsel benyttes innsendingsId. 
Dersom innsendingsId ikke finnes kan en egen referanse man selv setter i det man initierer oversendelsen (sendersReference) 
benyttes. Se figuren nedenfor for en beskrivelse av flyten og av hvilke tjenester som blir benyttet.

![Figur 2 Altinn](https://github.com/kartverket/etinglysing-systembeskrivelse/blob/master/doc/altinn2.png)
 
Før man kan ta i bruk Altinn som kanal for inngående meldinger så må man få satt opp riktige tilganger hos Kartverket, samt at man selv må inn å opprette bruker og gi riktig tilgang i Altinn sluttbrukerløsning. Mer om hvordan dette kan gjøres er beskrevet i dokumentasjonen som følger med eksempelklienten. Ta kontakt med Kartverket via post@grunnbok.no for å få satt opp de tilgangene man trenger og får å få tildelt et fiktivt organisasjonsnummer til bruk i test.

### Altinn dokumentasjon

For informasjon om Altinn formidlingstjeneste se [dokumentasjon](https://altinn.github.io/docs/api/rest/formidling/).

## 4. Prosessflyt 

Systemet implementerer en asynkron prosessflyt, slik at tinglysingskallet kun sikrer at systemet har mottatt meldingen. 
Videre behandling av meldingen skjer asynkront. Når tjenesten for å tinglyse en melding har returnert uten feil, garanteres 
det at meldingen er mottatt av systemet og at Kartverket ivaretar videre behandling av meldingen.

## 5. Funksjonalitet og begrensninger 

Følgende typer rettsstiftelser er tilgjengelig for elektronisk innsending av eksterne aktører:

* Eierskifte matrikkelenhet (HJ_HJG)
* Overdragelse av festerett med og uten bygning (TF_OMB, TF_OUB)
* Eierskifte borettslagsandel (BH_HJA)
* Anmerkning på registerenhetsrettsandel, kun ved skifteoppgjør (EN_SAT)
* Annen heftelse, herunder urådighet samt rettigheter ved skifteoppgjør (AH_URK, AH_GAR, AH_PBF, AH_PBG, AH_PBH)
* Pantedokument, herunder bytte av bank (OB_PDO, OB_PDB)
* Sletting, herunder begrenset/delvis sletting (SL_SLE)
* Transport, herunder massetransport (TR_TRA, TR_TRP, TR_TAS, TR_MAS, TR_FUS)
* Nedkvittering (NE_NEK)
* Tinglysing på ny (TN_TPN)
* Prioritetsbestemmelse for dokumentnummer, herunder 'internvikelse' i samme forsendelse (PR_PRB, PR_PRV)
* Prioritetsbestemmelse for ikke tinglyst dokument (PB_PRB)
* Tvangsforretning (kun enkelte aktører) (TV_UTL, TV_ARR)
* Anmerkning på person (kun enkelte aktører) (ET_KKM)

I tillegg kommer rettsstiftelser som kun kan sendes inn fra matrikkelen.

Dokumentasjon av rettsstiftelsestypene finnes i [UML-modellen for innsending](https://etgltest.grunnbok.no/grunnbok/modell/grunnbok-innsending-v2-modell/index.html).

Innsendingsgrensesnittet krever at XML-delen tilhørende BIDXML kan valideres mot XSD for namespacet for innsendingsskjemaet. Dette må oppgis på standardisert vis med namespacehenvisning i XML-dokumentet. Det kreves også at XSL-delen tilhørende BIDXML kan valideres mot godkjente versjoner av XSL transformasjonen. Innpakket XSL må således være binært identisk med en versjon som er publisert av Kartverket.

Det er ingen validering av gyldig offisiell versjon av XSL når man skal validere BIDXML for følgebrev.

## 6. Innsending av signerte dokumenter

For å forenkle testingen av hvordan man må bygge opp rettsstiftelser støtter innsendings-api i test validering samt mottak og behandling av usignerte meldinger. 

En forsendelse med usignert melding vil representere grunnlaget for en signert melding. I de tilfellene så vil det som representerer henholdsvis `<dokument>` i en usignert melding mappes inn som en `<SDODokument>` i signert melding. Det samme gjelder for følgebrev. Denne beskrivelsen vil vise hvordan man med utgangspunkt i en forsendelse med usignert dokument kan etablere en forsendelse med ett eller flere signerte dokumenter samt et signert følgebrev. 
Normalt så vil vi har følgende elementer som kan opptre i signert melding avhengig av hvordan data er pakket sammen

| Element i signert melding | Rotnode for payload i BIDXML | Merknad |
|------------------------------|------------------------------|------------------------------|
| Dokumenter | `<dokument>` | For hvert signerte dokument i den signerte meldingen, så må hvert signerte dokument ha `<dokument>` som rotnode i BIDXML for den signerte meldingen |
| Foelgebrev | `<foelgebrev>` | Det signerte følgebrevet som omhandler ett eller flere dokumenter inkludert referanser og digest-data. XSL script trenger ikke å være offisiell versjon av XSLT, men scriptet må være maskinelt gyldig xsl. |
| dokumenterOgFoelgebrev | `<usignertMelding>` | `<usignertMelding>` er rotnoden som inneholder både dokumenter (ett eller flere), samt følgebrevet som refererer til dokumentene. I dette tilfellet så trenger man ikke å fylle inn digest og algoritme, da det kun er en melding som er signert og denne er komplett. |


### Mapping fra signert til usignert melding

Dette eksempelet viser mapping av en forsendelse med pant, inkludert følgebrev til en variant som inneholder signerte dokumenter i form av SEID-SDO og signerte objekter basert på BIDXML. Merk at pantedokumentet er forkortet noe for å redusere mengden tekst i eksempelet, men alle de andre relevante feltene er fylt ut. I Eksempelet er det oppgitt noen kommentarer i xml-strukturen markert som `<-- Ref 1 -->` for eksempel. Dette er gjort for å forenkle referansen til dette xml-elementet etterfølgende. Prosessen fra å konvertere en usignert melding til en signert melding vil bestå av følgende trinn:

1.	Hvert `<dokument>` under `<dokumenter>` i `<usignertMelding>` konverteres til et SDODokument som inneholder <dokument> uendret som en del av det signerte grunnlaget i `<SignersDocument>` i sdo på BIDXML-format.
2.	Signaturene i forsendelsen under `<ikkeDigitaleSignaturer>` fjernes. Disse signaturene representerer de parter som må signere dokumentene fra 1.
3.	For hvert dokument som refereres i følgebrevet, så henter man en referert Digest fra den assosierte signatur i SDO fra elementet <HashedData>. Dette hash/algoritmeparet angis som `<digest>` og `<digestAlgoritme>` under elementet `<referanse>`, der man har `<gjelderDokumentreferanse>` liggende fra det usignerte dokumentet.

Eksempler på denne konverteringen vises i etterfølgende avsnitt.

#### Usignert pant med følgebrev

```xml
<?xml version=»1.0» encoding=»ISO-8859-1»?>
<forsendelse xmlns=»http://kartverket.no/grunnbok/wsapi/v2/domain/innsending»>
    <forsendelsesreferanse>1</forsendelsesreferanse>
    <usignertMelding>
        <dokumenter>
            <dokument>
             <!-- Ref 1 -->
                <dokumentreferanse>1</dokumentreferanse>
                <rettsstiftelser>
                    <pant>
                        <rettsstiftelsesreferanse>pant.test</rettsstiftelsesreferanse>
                        <rettsstiftelsestype>
                            <kodeverdi>OB_PDO</kodeverdi>
                            <navn>PANTEDOKUMENT</navn>
                        </rettsstiftelsestype>
                        <rettighetshavere>
                       ....
                    </pant>
                </rettsstiftelser>
            </dokument>
        </dokumenter>
        <foelgebrev>
         <-- Ref 2 -->
            <innsendersIdentifikasjonsnummer>911044110</innsendersIdentifikasjonsnummer>
            <dokumentrekkefoelge>
                <referanse>
                    <gjelderDokumentreferanse>1</gjelderDokumentreferanse>
                </referanse>
            </dokumentrekkefoelge>
        </foelgebrev>
    </usignertMelding>
    <ikkeDigitaleSignaturer>
        <signatur>
            <gjelderDokumentreferanse>1</gjelderDokumentreferanse>
            <personidentifikasjonsnummer>11111111111</personidentifikasjonsnummer>
        </signatur>
    </ikkeDigitaleSignaturer>
</forsendelse>
```

#### Signert forsendelse med signert dokument for pant og signert følgebrev

Det signerte dokumentet opprettes ved at hvert enkelt element `<dokument>` etableres som en SDO signert av rettighetshavere. 
I tillegg til dette opprettes følgebrevet som en SDO signert med virksomhetssertifikat. 
Forsendelsen med de signerte dokumentene vil etter at de signerte elementene i form av SDODokument har blitt samlet inn fra signeringsprosessen se slik ut:

```xml
<?xml version=’1.0’ encoding=’ISO-8859-1’?>
<forsendelse xmlns=»http://kartverket.no/grunnbok/wsapi/v2/domain/innsending»>
    <forsendelsesreferanse>17</forsendelsesreferanse>
    <signertMelding>
        <dokumenter>
            <SDODokument>
                <signertDokument>
                <!-- Fra Ref 1, Base 64 encodet sdo med <dokument> som rotnode i BIDXML -->
                    PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0iaXNvL....
            </SDODokument>
        </dokumenter>
        <foelgebrev>
                <SDODokument>
        	 <!-- Fra Ref 2, Base 64 encodet sdo med <foelgebrev> som rotnode i BIDXML -->
                    PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0iaXNvL....
                </SDODokument>
        </foelgebrev>	
    </signertMelding>
    <ikkeDigitaleSignaturer/>
</forsendelse>
```

Merk her at hvert enkelt `<SDODokument>`, enten det er et følgebrev eller et dokument vil være et komplett SDO dokument som deretter har blitt base 64 encodet. `<SignersDokument>` i denne SDO-strukturen vil være en BIDXML struktur, der XML-delen er en identisk kopi av det som er hentet fra Ref 1 i det usignerte eksempelet. 

Det er ikke mulig å oppgi signaturer i signerte dokumenter, dette skal hentes fra VA-tjenestene i form av oppslag fra BankId som en del av valideringen av dokumentet. Signaturelementet `<ikkeDigitaleSignaturer>` er derfor tomt. SEID SDO dokumentet vil før base 64 encodingen for både `<dokument>` og `<foelgebrev>` likne på dette:

```xml
<?xml version=”1.0” encoding=”utf-8”?>
<SDOList xmlns=”http://www.npt.no/seid/xmlskjema/SDO_v1.0” xmlns:XAdES=”http://uri.etsi.org/01903/v1.2.2#”
         xmlns:ds=”http://www.w3.org/2000/09/xmldsig#” xmlns:xsi=”http://www.w3.org/2001/XMLSchema-instance”>
    <SDO>
        <SEIDSDOVersion>1.0</SEIDSDOVersion>
        <SDODataPart>
            <SignatureElement>
                <CMSSignatureElement>
                    <SDOProfile>SEID-SDO-Basic-V</SDOProfile>
                    <SignaturePolicyIdentifier>
                        <SignaturePolicyId>
                            <SigPolicyId>
                                <XAdES:Identifier>urn:oid:2.16.578.1.16.4.1</XAdES:Identifier>
                            </SigPolicyId>
                        </SignaturePolicyId>
                    </SignaturePolicyIdentifier>
                    <SignersDocumentFormat>
                        <MimeType>text/BIDXML</MimeType>
                    </SignersDocumentFormat>
                    <HashedData>
                        <ds:DigestMethod Algorithm=”SHA-256”/>
                                  <ds:DigestValue>miF0ND68NTmdhdFosjMqI8qbqyxRx3WUPF2rNQcfpPc=</ds:DigestValue>
                    </HashedData>
                    <CMSSignature>
                       MIAGCSqGSIb3DQ
                    </CMSSignature>
                    <UserCertificateAndRevocationData>
                        <RevocationValues>
                            <XAdES:OCSPValues>
                                <XAdES:EncapsulatedOCSPValue>
                                    MIIHGTCCAS6hX...
                                </XAdES:EncapsulatedOCSPValue>
                            </XAdES:OCSPValues>
                        </RevocationValues>
                    </UserCertificateAndRevocationData>
                </CMSSignatureElement>
            </SignatureElement>
        </SDODataPart>
        <SDOSeal>
            <SDOSignature>
                <CMSSignatureElement>
                    <SDOProfile>SEID-SDO-Basic-V</SDOProfile>
                    <SignaturePolicyIdentifier>
                        <SignaturePolicyId>
                            <SigPolicyId>
                                <XAdES:Identifier>urn:oid:2.16.578.1.16.4.1</XAdES:Identifier>
                            </SigPolicyId>
                        </SignaturePolicyId>
                    </SignaturePolicyIdentifier>
                    <SignersDocumentFormat>
                        <MimeType>text/plain</MimeType>
                    </SignersDocumentFormat>
                    <HashedData>
                        <ds:DigestMethod Algorithm=”SHA-256”/>
                      <ds:DigestValue>CfV9X7Jn6TOCfnP/zbbvaZXtkcDJwZ9lY2sys+cFydI=</ds:DigestValue>
                    </HashedData>
                    <CMSSignature>
                        MIAGCSqGSIb3DQEHAq...
                    </CMSSignature>
                    <UserCertificateAndRevocationData>
                        <RevocationValues>
                            <XAdES:OCSPValues>
                                <XAdES:EncapsulatedOCSPValue>
                                    MIIG8TCCAQahXj...
                                </XAdES:EncapsulatedOCSPValue>
                            </XAdES:OCSPValues>
                        </RevocationValues>
                    </UserCertificateAndRevocationData>
                </CMSSignatureElement>
            </SDOSignature>
        </SDOSeal>
        <Metadata>
            <ValuePair>
                <Name>MerchantDesc</Name>
                <Value>Merchant description?</Value>
            </ValuePair>
        </Metadata>
        <SignedObject>
            <SignersDocument>
                PEJhbmtJRFhNTD48QklEWE1MPjw/eG1sIHZlc...
            </SignersDocument>
        </SignedObject>
    </SDO>
</SDOList>
```

For `<dokument>` elementet med Ref 1 som vist i eksempelet for den usignerte meldingen så vil det som er oppgitt som <SignersDocument> i SDO være BIDXML. Grensesnittet som benyttes mot BankId vil sørge for at dette er formattert korrekt. Man skal ikke lage BIDXML strukturen selv. Det eneste man skal sende inn til denne signeringen er det som er payloaden for `<dokument>` samt en identisk binær kopi av det som har blitt levert ut av gyldig xsl for innsending som er hentet fra en av kartverkets releaseversjoner. Det er denne visningen som benyttes av BankId når man skal presentere dokumentet i signeringsdialogen.

Denne strukturen i `<SignersDocument>` vil etter base64 decoding likne på dette:
```xml
<BankIDXML><BIDXML><?xml version=”1.0” encoding=”ISO-8859-1”?>
            <dokument>
             <!-- Ref 1 -->
                <dokumentreferanse>1</dokumentreferanse>
                <rettsstiftelser>
                    <pant>
                        <rettsstiftelsesreferanse>pant.test</rettsstiftelsesreferanse>
                        <rettsstiftelsestype>
                            <kodeverdi>OB_PDO</kodeverdi>
                            <navn>PANTEDOKUMENT</navn>
                        </rettsstiftelsestype>
                        <rettighetshavere>
                       ....
                    </pant>
                </rettsstiftelser>
            </dokument>
</BIDXML><BIDXSL><?xml version=»1.0» encoding=»ISO-8859-1»?>
<xsl:stylesheet xmlns:xsl=”http://www.w3.org/1999/XSL/Transform”
                xmlns:xsi=”http://www.w3.org/2001/XMLSchema-instance”
                xmlns:inn=”http://kartverket.no/grunnbok/wsapi/v2/domain/innsending”
                version=”1.0”
                exclude-result-prefixes=”xsi inn”>

    <xsl:output method=”html” encoding=”ISO-8859-1” doctype-system=”http://www.w3.org/TR/html4/loose.dtd”
....
</xsl:stylesheet>
</BIDXSL></BankIDXML>
```

For følgebrevet så er det et spesielt hensyn som må tas. For at det ikke skal være mulig å bytte ut et signert dokument med et annet, så krever vi at det blir oppgitt en `<digest>` samt tilhørende `<digestAlgoritme>` i følgebrevet som refererer til dokumentet. Det betyr at det usignerte følgebrevet i `<foelgebrev>` elementet i det usignerte eksempelet må kompletteres for å bli gyldig. I det oppgitte eksempelet som vi har referert, så vil XML-biten i BIDXML for det signerte følgebrevet måtte utvides slik:
```xml
<?xml version=”1.0” encoding=”ISO-8859-1”?>
<foelgebrev xmlns=”http://kartverket.no/grunnbok/wsapi/v2/domain/innsending”>
    <!-- Ref 2 -->
    <innsendersIdentifikasjonsnummer>911044110</innsendersIdentifikasjonsnummer>
    <dokumentrekkefoelge>
        <referanse>
            <gjelderDokumentreferanse>1</gjelderDokumentreferanse>
            <digest>miF0ND68NTmdhdFosjMqI8qbqyxRx3WUPF2rNQcfpPc=</digest>
            <digestAlgoritme>SHA256</digestAlgoritme>
        </referanse>
    </dokumentrekkefoelge>
</foelgebrev>
```

Det er dette grunnlaget, inkludert digest-referanser som skal signeres med virksomhetssertifikat. Når følgebrevet valideres så vil vi validere dette opp mot den beregnede Digest for det signerte dokumentet. Man trenger ikke å beregne denne, denne hentes fra `<HashedData>` oppgitt i SDO for de signerte dokumentene. Man kan lage en enkel parser for dette selv, eller eventuelt bruke BankId sin implementasjon som inneholder klasser og metoder for å konvertere bytes med XML-data til en SDO-struktur i Java eller C#. Gyldigheten av disse refererte verdier vil valideres når dokumentet valideres, og så slipper man å lage programkode for å beregne Digest over det som er BIDXML strukturen som man normalt ikke vil se eller behandle selv.

Hvis det er flere dokumenter i en melding, så gjentas denne prosessen for hvert av de dokumentene som man skal signere. En SDO kan kun inneholde et `<dokument>` eller et `<foelgebrev>` som rotnode inne i BIDXML. Forutsetningen er at man signerer på et dokument i sin helhet. Disse kan signeres av rettighetshavere i forskjellige løsninger, slik som i en nettbank eller en signeringsportal. Når det er flere rettighetshavere som skal signere, så vil man normalt samle inn flere SDO fra BankId for disse signeringene for så å kombinere dette inn i en SDO med flere signaturer. Dette er mulig når det signerte innholdet er likt, men det er flere rettighetshavere som skal signere.
For følgebrevet som skal signeres med virksomhetssertifikat så er det ikke noen spesielle krav til xsl-transformasjonen som pakkes inn sammen med følgebrevet ut over de krav som eventuelt kommer fra BankId. Kartverket leverer ikke fra seg noen xsl for følgebrevet og har derfor heller ikke noen validering av dette innholdet.
 
## 7. Hvilke identifikasjonsnumre følger med inn til signaturkontroll

Identifikasjonsnumre som hentes ut fra signaturene på dokumentet følger med inn i signaturkontrollen. 
I tillegg følger identifikasjonsnummeret fra følgebrevet med inn i signaturkontroll for hvert dokument. 

I signaturkontrollen sjekkes det i henhold til spesifikke regler for hver type rettsstiftelse i dokumentet, at de påkrevde signaturene
som representerer rettighetshaverne har signert dokumentet. Det aksepteres overflødige signaturer på dokumentet.

### Eksempler

Vi bruker følgende konvensjoner:

**Rolle(B)** eller **Rolle(P)** for å betegne en rolle som signerer, for eksempel **Selger** og **B** for virksomhetssertifikat og **P** for personsertifikat. Rolle representerer et identifikasjonsnummer, når vi skriver **Selger** så mener vi identifikasjonsnummeret som tilhører denne rollen.

#### Dokumentpakke

Dokumentpakke, sletting av gammelt pant i DnB, eierskifte med Selger og Kjøper, pant for ny eier i Skandiabanken, sendt inn av Megler.
Sletting signeres av DnB med virksomhetssertifikat. Selger signerer Eierskifte i eBoks sendt fra meglerens system, ny eier signerer pantedokument i Skandiabankens nettbank. Megler samler inn dokumenter, signerer følgebrev med virksomhetssertifikat og sender inn.


| Signert objekt | Teknisk Signatur | Identifikasjonsnummer til signaturkontroll |
|-------------------------|-------------------------|-------------------------|
| Sletting | DnB(B) | DnB, Megler |
| Eierskifte | Selger(P), eBoks(B) | Selger, eBoks, Megler |
| Pantedokument | Kjøper(P), Skandiabanken(B) | Kjøper, Skandiabanken, Megler |
| Følgebrev | Megler(B) | Megler |

#### Sletting med følgebrev

DnB sender inn en sletting signert med virksomhetssertifikat, det samme sertifikatet brukes for å signere følgebrevet.

| Signert objekt | Teknisk Signatur | Identifikasjonsnummer til signaturkontroll |
|-------------------------|-------------------------|-------------------------|
| Sletting | DnB(B) | DnB |
| Følgebrev | DnB(B) | DnB |

#### Sletting og følgebrev sammen

DnB signerer sletting og følgebrev sammen

| Signert objekt | Teknisk Signatur | Identifikasjonsnummer til signaturkontroll |
|-------------------------|-------------------------|-------------------------|
| Sletting og Følgebrev (Dokument og Følgebrev sammen) | DnB(B) | DnB | 
 
## 8. Hvordan behandles signaturene i et signert dokument

Vi ønsker at alle tester som kjører med usignerte meldinger skal være så representative som mulig. Vi har tatt hensyn til dette gjennom at behandlingen av en melding med usignerte data er så lik som den med signerte data som mulig. Med signert melding mener vi her en forsendelse med der feltet `<signertMelding>` er fylt ut. 

Dette gjør vi ved å dele opp behandlingen av signerte og usignerte meldinger i to steg:
1.	Konverter en `<forsendelse>` med `<signertMelding>` til en `<forsendele>` med `<usignertMelding>`
2.	Rapporterer eventuelle feil med denne konverteringen, formatfeil, valideringsfeil fra BankId med mer. Ingen av regler som gjelder bruk av identifikasjonsnumrene i de signerte dokumentene blir kjørt her
3.	Validerer og behandler en forsendelse med `<usignertMelding>`.

For forsendelse med `<signertMelding>` og forsendelse med `<usignertMelding>` så er behandlingen helt lik.

Formidlingen av identifikasjonsnumre fra de signerte dokumentene og eventuelt følgebrev blir hentet fra SDO. 
Dette eller disse identifikasjonsnumrene blir deretter lagt inn i feltet som holder signaturer for et dokument.

For et signert dokument som sendes inn, så består konverteringen i steg 1) fra 
følgende dokument:

```xml
<?xml version=’1.0’ encoding=’ISO-8859-1’?>
<forsendelse xmlns=»http://kartverket.no/grunnbok/wsapi/v2/domain/innsending»>
    <forsendelsesreferanse>17</forsendelsesreferanse>
    <signertMelding>
        <dokumenter>
            <SDODokument>
                <signertDokument>
                <!-- Fra Ref 1, Base 64 encodet sdo med <dokument> som rotnode i BIDXML -->
                    PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0iaXNvL....
            </SDODokument>
        </dokumenter>
        <foelgebrev>
                <SDODokument>
        	 <!-- Base 64 encodet sdo med <foelgebrev> som rotnode i BIDXML -->
                    PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0iaXNvL....
               </SDODokument>
        </foelgebrev>	
    </signertMelding>
    <ikkeDigitaleSignaturer/>
</forsendelse>
```

I å først behandle og konvertere hver sdo til et dokument i en usignert melding, for deretter å lage et element `<ikkeDigitaleSignaturer>` som legger ved signaturene fra dokument og følgebrev. Signaturene fra følgebrevet vil alltid gjelde alle dokumentene, mens signaturene fra hvert dokument gjelder dokumentet selv, gjennom angitt `<dokumentreferanse>`.

I eksempelet over kan vi si at `<SDODokument>` under elementet `<dokumenter>` var signert av person med identifikasjon 123, og følgebrevet i `<SDODokument>` under `<foelgebrev>` var signert av virksomhet med organisasjonsnummer 456.

Det konverterte dokumentet vil da se slik ut:

```xml
<?xml version=»1.0» encoding=»ISO-8859-1»?>
<forsendelse xmlns=»http://kartverket.no/grunnbok/wsapi/v2/domain/innsending»>
    <forsendelsesreferanse>1</forsendelsesreferanse>
    <usignertMelding>
        <dokumenter>
            <dokument>
                <dokumentreferanse>1</dokumentreferanse>
                <rettsstiftelser>
                    <pant>
                        <rettsstiftelsesreferanse>pant.test</rettsstiftelsesreferanse>
                        <rettsstiftelsestype>
                            <kodeverdi>OB_PDO</kodeverdi>
                            <navn>PANTEDOKUMENT</navn>
                        </rettsstiftelsestype>
                        <rettighetshavere>
                       ....
                    </pant>
                </rettsstiftelser>
            </dokument>
        </dokumenter>
        <foelgebrev>
            <innsendersIdentifikasjonsnummer>911044110</innsendersIdentifikasjonsnummer>
            <dokumentrekkefoelge>
                <referanse>
                    <gjelderDokumentreferanse>1</gjelderDokumentreferanse>
                </referanse>
            </dokumentrekkefoelge>
        </foelgebrev>
    </usignertMelding>
    <ikkeDigitaleSignaturer>
        <signatur>
            <gjelderDokumentreferanse>1</gjelderDokumentreferanse> <!—Fra dokument -->
            <personidentifikasjonsnummer>123</personidentifikasjonsnummer> 
            <gjelderDokumentreferanse>1</gjelderDokumentreferanse> 
            <personidentifikasjonsnummer>456</personidentifikasjonsnummer> <!—fra følgebrev -->
        </signatur>
    </ikkeDigitaleSignaturer>
</forsendelse>
```

Her ser vi at forsendelsen med en signertMelding har blitt konvertert til en forsendelse med `<usignertMelding>`, legg spesielt merke til hvordan signaturene har blitt hentet fra `<dokument>` og `<følgebrev>` og plassert inn i seksjonen for `<ikkeDigitaleSignaturer>`. All behandling som gjelder signaturkontroll vil bli gjort på dette elementet med disse signaturene.

## 9. Bruk av BANKID grensesnitt for å signere SDO

BankId tilbyr klientgrensesnitt og eksempelklienter i Java og C# for å implementere signeringsdialog som brukersted, 
både for sluttbruker gjennom signering i nettleser, men også for serversidesignering som brukersted. 

For scenariet som dekker signering i nettleser så er det utenfor scopet til dette dokumentet å vise det i sin helhet. 
Vi anbefaler at brukerstedet gjør seg kjent med den dokumentasjonen som finnes på http://bankid.no på innlogget område. 
Her kan man laste ned eksempelkode og annen programvare samt dokumentasjon som viser hvordan dette fungerer. 
Vi kan ikke gi en uttømmende forklaring på hvordan dette fungerer her, men vi vil og noen henvisninger til de relevante 
grensesnittene som kan brukes for å generere SDO som etterfølgende kan pakkes som et eller flere `<SDODokument>` i en `<SignertMelding>`.

### Signering av xml og xsl med banklagret sertifikat

Dette er den signeringsdialogen som skal brukes når det er en rettighetshaver med et personnummer som skal signere. Det kan også brukes når man skal signere som organisasjon med banklagret ansattsertifikat der sertifikatet peker ut en organisasjon. Se beskrivelsen i Sertifikattyper for å se hvilke typer som kan brukes når.

Kodeeksemplene nedenfor benytter er skrevet i Java og benytter at klientbibliokteket for bankidservere-java er tilgjengelig. Eksemplene viser ikke hele kildekodefilen, men utvalgte og antatt relevante fragmenter. 

Eksempelet antar også at man har satt opp en BankId-klienten slik at denne kan vises korrekt, eksempelvis slik:
 
![Figur BankId](https://github.com/kartverket/etinglysing-systembeskrivelse/blob/master/doc/bankid.png)

Dette vil variere fra brukersted til brukersted. Kartverket stiller ingen krav til utforming her.

#### Initialisering av innhold til signeringsdialog for BIDXML

```java
        BIDSessionData sessionData = new BIDSessionData(session.getTransactionReference());
        String signType = session.getSignType(); 
        sessionData.setDataDescription(session.getDataDescription());
        if (“xml”.equals(signType)) {
            sessionData.setDataToBeSignedXMLFormat(getXsl());
            sessionData.setDataToBeSigned(getXml());
        } else {
            throw new ServletException(
                    “This BankID signature type is not supported: “ + signType);
        }
        session.setBidSessionData(sessionData);
        String resp2Client = bankIDFacade.initTransaction(bidInfo
                        .getOperation(), bidInfo.getEncKey(), bidInfo.getEncData(),
                bidInfo.getEncAuth(), bidInfo.getSid(), sessionData);

        session.setBidSessionData(sessionData);
        return resp2Client;
```

Dette vil sette opp nødvendig grunnlag for signeringen. Dette er serverkode som henter ut xml  og xsl gjennom lokale kall og setter dette på `sessionData. no.bbs.server.vos.BIDSessionData` er en BankId-spesifikk klasse.

#### Opprette SDO fra BIDXML

```java
//Hente sertifikatinfo
CertificateInfo certInfo = bankIDFacade.getCertificateInfo(bankIDFacade
                .getPKCS7Info(sessionData.getClientSignature())
                .getSignerCertificate());

//Lage SDO
SEID_SDO sdo = bankIDFacade.createSDO(sessionData.getClientSignatureBytes(),
                    sessionData.getMerchantSignatureBytes(),
                    sessionData.getSignedDataBytes(), sessionData.getDataToBeSignedMimeType(),
                    sessionData.getMerchantOCSPBytes(), sessionData.getClientOCSPBytes(),
                    sessionData.getDataDescription());

                //Sette SDO-xml på sesjonsobjekt
                 sdo.addSignedDataRaw(sessionData.getSignedDataBytes());
                session.setSdoFile(new String(sdo.toXML()));
```

#### Tilrettelegge SDO signert av flere rettighetshavere

Dette er scenariet som skal brukes når man har flere rettighetshavere som skal signere på det samme dokumentet. Eksempel kan være pant i fast eiendom med tre andeler der alle rettighetshavere for den personlige andelen må signere. Eksempelet antar at det allerede har blitt etablert tre SDO strukturer allerede som har blitt signert av BankId. Dette orkestreres i avleverende fagsystem.

```java
byte[] signedData = null;
final File[] signedSdos = new File[]{new File(“sdo-1.xml”), new File(“sdo-2.xml”)};
List<PKCS7WithOCSPResponse> cmsData = new LinkedList<>();
for (File signedSdo : signedSdos) {
    try (InputStream sdoData = new FileInputStream(signedSdo)) {
        final byte[] bytes = ByteStreams.toByteArray(sdoData);
        SEID_SDO sdo = new SEID_SDO(bytes);
        final SEID_SDOElement seid_sdoElement = sdo.getSEID_SDOElement(0);
        //Alle sdo-ene er i dette tilfellet signert over det samme grunnlagret, det er kravet.
        signedData = seid_sdoElement.getSignedObject().getSignedDataRaw();
        for (SEID_SignatureElement signatureElement : seid_sdoElement.getSeidSDODataPart().getSignatureElements()) {
            final PKCS7WithOCSPResponse pkcs7WithOCSPResponse = new PKCS7WithOCSPResponse();
            cmsData.add(pkcs7WithOCSPResponse);
            pkcs7WithOCSPResponse.setB64PKCS7(signatureElement.getCmsSignatureElement().getCmsSignature().getB64CMSSignature());
            pkcs7WithOCSPResponse.setB64OCSPResponse(signatureElement.getCmsSignatureElement().getCertAndRevokeData().getB64EncapsulatedOCSPValue());
        }
    }

}
final SEID_SDO dynamicSDO = bidFacade.createDynamicSDO(cmsData.toArray(new PKCS7WithOCSPResponse[cmsData.size()]), signedData, “text/BIDXML”, “test av dynamisk sdo”);
dynamicSDO.addSignedDataRaw(signedData);
final byte[] sdoXMLBytes = dynamicSDO.toXML();
```

Den resulterende SDO vil inneholde samtlige signaturer. 
Resultatet av dette kan formidles som en SDO i SDODokument i en egnet forsendelse. Dokumentene vil alltid inneholde mer 
enn en signatur. Det er påkrevet at slike dokumenter er forseglet med et `<SDOSeal>`, det vil bli sjekket som en del av 
valideringslogikken for disse SDO objekter.


#### Forsegle SDO

```java
SEID_SDO dynamicSDO = bidFacade.createDynamicSDO(cmsData.toArray(new PKCS7WithOCSPResponse[cmsData.size()]), signedData, «text/BIDXML», «test av dynamisk sdo»);
            final CertificateStatus ownCertificateStatus = bidFacade.getOwnCertificateStatus();
            dynamicSDO.addSignedDataRaw(signedData);
            dynamicSDO = bidFacade.createSDOSeal(dynamicSDO, ownCertificateStatus.getB64OCSPResponse());
```

Forseglingen av SDO må foretas etter at alle signaturene har blitt lagt inn.

### Signering av følgebrev med virksomhetssertifikat

Dette utføres som en ren serversignering uten brukerdialog i nettleser eller mobilenhet.

```java
final byte[] contentToBeSigned = ByteStreams.toByteArray(inputStreamBIDXMLFoelgebrev);
            signatureAndData = bidFacade.sign(contentToBeSigned);

            final no.bbs.server.vos.CertificateStatus ownCertificateStatus =      bidFacade.getOwnCertificateStatus();
            

            PKCS7WithOCSPResponse merchantData = new PKCS7WithOCSPResponse();

            merchantData.setB64PKCS7(signatureAndData.getB64SignatureBytes());
            merchantData.setB64OCSPResponse(ownCertificateStatus.getB64OCSPResponse());

            SEID_SDO serverSignedSDO = bidFacade.createDynamicSDO(new PKCS7WithOCSPResponse[]{merchantData}, contentToBeSigned, mimeType.getType(), “Signert Grunnboksutskrift”);
            serverSignedSDO.addSignedDataRaw(contentToBeSigned);
            serverSignedSDO = bidFacade.createSDOSeal(serverSignedSDO, merchantData.getB64OCSPResponse());
            final byte[] sdoXMLBytes = serverSignedSDO.toXML();
```

Dette vil gi en SDO som er signert med et virksomhetssertifikat. Det base64 encodede resulatet av dette kan benyttes som 
SDOElement i følgebrev. Det valideres at virksomheten som signerer er registrert i enhetsregisteret, og at sertifikatet har
gyldig sertifikatsstatus og er benyttet innenfor gyldighetsperioden.

Det er et krav om at det er virksomhetssertifikat som anvendes når følgebrevet signeres.
Det er også virksomhetssertifikat som skal benyttes når man skal signere følgebrev og dokument sammen. 

#### Merknad om virksomhetssertifikat

Løsningen støtter virksomhetssertifikater utstedt av BankId og av Buypass.

Virksomhetssertifikat utstedt av BankId må ha en av følgende oid (Object Identifier):

* 2.16.578.1.16.1.6.1.1 Soft Merchant
* 2.16.578.1.16.1.6.2.1 HSM Merchant

Virksomhetssertifikat utstedt av Buypass må ha en av følgende oid (Object Identifier):

* 2.16.578.1.26.1.3.2 Soft Enterprise
* 2.16.578.1.26.1.3.5 Hard Enterprise
* 2.16.578.1.26.1.0.3.2 Soft Enterprise Test

Hvis sertifikatet som har blitt brukt til å signere følgebrevet ikke er en av disse typene, vil forsendelsen bli avvist med egnet kontrollresultat. 

Det er ikke et krav om at virksomhetssertifikatet skal være registrert hos Kartverket.

## 10. Statustjeneste for systemet

Det er tilgjengeliggjort en tjeneste som skal gi status for om systemet er tilgjengelig eller ikke. Tjenesten gir status for tre deler av systemet: Innsendingstjenestene, Valideringstjenesten og Grunnbokstjenestene. Status for hver av systemene kan være en av følgende:

* UP: Alt fungerer som det skal
* REDUCED: Systemet er delvis tilgjengelig. 
* DOWN: Systemet er ikke tilgjengelig
* UNKNOWN: Status er ukjent på grunn av at statussjekk ikke har blitt utført.

Statustjenesten gir svar på JSON format og hver av de tre statusene kan være aggregert opp av flere underliggende statuser. Dersom systemet har redusert tilgjengelighet vil man kunne utlede hva som er tilgjengelig og ikke ved å se på underliggende statuser. 

Eksempel på response dersom alt er tilgjengelig:

```json
{
"innsendingstjenesteStatus":
     {
            "status": «UP»,
            "updated": "2017-01-18T10:52:35.261"
     },
 "valideringstjenesteStatus": 
    {
            "status": «UP»,
            "updated": "2017-01-18T10:52:36.469"
    },
 "grunnbokstjenesterStatus": 
    {
        "status": «UP»,
        "updated": "2017-01-18T10:52:34.265"
    }
}
```

Eksempel på response dersom tinglysingen ikke klarer å hente filer fra Altinn:

```json
{
"innsendingstjenesteStatus": {
        "status": «DOWN»,
        "updated": "2017-01-18T10:52:35.261"
        "Grunnboken server": {"status": «UP»},
        "Altinn innsending": {"status": «DOWN»},
        "BankId integrasjon": {"status": «UP»}
  },
 "valideringstjenesteStatus": {
        "status": «UP»,
        "updated": "2017-01-18T10:52:36.469"
  },
 "grunnbokstjenesterStatus": {
        "status": «UP»,
        "updated": "2017-01-18T10:52:34.265"
  }
}

```

### Tolking av status

Dersom en eller flere av sjekkene som blir utført gir svar at systemet er nede eller at det har redusert tilgjengelighet skal man kunne være i stand til å si hva dette vil ha for bruk av tjenesten. Mulige statuser hver enkelt sjekk kan gi er listet opp i parantes.

* InnsendingstjenesteStatus (UP/DOWN/REDUCED): Ved status DOWN betyr det at man ikke kan benytte sendTilTinglysing eller hentStatus. Dersom den har status REDUCED må man se på underliggende statuser for å se hvilke deler som ikke er tilgjengelig.
  * Altinn innsending (UP/DOWN): Ved status DOWN vil Kartverket ikke være i stand til å hente filer fra Altinn og det er ikke mulig å sende inn noe til elektronisk tinglysing eller sjekke status.
  * BankId integrasjon(UP/DOWN): Ved status DOWN vil det ikke være mulig å sende inn signerte dokumenter og det går heller ikke an å generere signert grunnboksutskrift.
  * Grunnboken server (UP/DOWN/REDUCED): Ved status DOWN vil det ikke være mulig å sende inn meldinger, ved status REDUCED se underliggende statuser for å finne ut av hva begrensningene er.
    * Sjekk av konsesjon(UP/REDUCED): Ved status REDUCED vil alle rettstiftelser som krever avklaring av konsesjon gå til manuell behandling.
    * Matrikkelenhetinfo (UP/REDUCED): Ved status REDUCED vil alle forsendelser som omhandler matrikkelenheter vil gå til manuell behandling, kun borettslagsandeler vil kunne tinglyses automatisk
    * Signaturkontroll (UP/REDUCED): Ved status REDUCED vil alle overdragelser som omhandler tilfeller der organisasjoner mister en rettighet gå til manuell behandling.
* Valideringstjeneste (UP/DOWN): Ved status DOWN vil ikke validering være tilgjengelig gjennom webservice grensesnittet.
* Grunnbokstjenester (UP/DOWN): Ved status DOWN vil ikke de andre grunnbokstjenestene være tilgjengelig gjennom webservice grensesnittet.
 
## 11. Endepunkter for test og produksjon

Dette avsnittet vil gi en oversikt over endepunkter for produksjon, samt for de to testmiljøene etgltest (produksjonsversjon) og betatest (kommende versjon), både for direkte aksess til Innsendingstjenestene våre og til Altinn sine endepunkter.

For beskrivelse av tjenestene som tilbys er innsending og validering dekket av inneværende dokument, mens en generell beskrivelse
av øvrige tjenester, som oppslagstjenester og endringslogg, finnes [her](https://github.com/kartverket/etinglysing-tjenester). 

### Forbehold 

Testsystemene representerer et system under utvikling der de ulike deler av systemet vil være gjenstand for forandringer. Løsningen kan ha mangelfull implementasjon samt inneholde feil.


### Produksjon

#### Innsendingstjeneste

https://www.grunnbok.no/grunnbok/wsapi/v2/InnsendingServiceWS

På produksjonssystemet er denne vanligvis ikke i bruk, da innsending skjer via Altinns formidlingstjeneste, som beskrevet i dette dokumentet.

#### Valideringstjeneste

https://www.grunnbok.no/grunnbok/wsapi/v2/ValideringServiceWS

#### Teknisk dokumentasjon alle tjenester

https://www.grunnbok.no/grunnbok/dokumentasjon.html

#### Endepunkter Altinn

| Tjeneste | URL |
|-----------------------------------|-------------------------------------------------------------------------------------------------------|
| BrokerServiceExternal | https://www.altinn.no/ServiceEngineExternal/BrokerServiceExternalBasic.svc |
| BrokerServiceExternalBasicStreamd | https://www.altinn.no/ServiceEngineExternal/BrokerServiceExternalBasicStreamed.svc |
| ReceiptExternalBasic | https://www.altinn.no/IntermediaryExternal/ReceiptExternalBasic.svc |


#### Konfigurasjonsparametre Altinn

| ServiceCode | ServiceEditionCode | Organisasjonsnummer Kartverket |
|-------------|--------------------|--------------------------------|
| 4433 | 1 | 971040238 |


 
### Testmiljø - etglTest

#### Innsendingstjeneste

https://etgltest.grunnbok.no/grunnbok/wsapi/v2/InnsendingServiceWS

I test er elektronisk innsending tilgjengelig både direkte via WS og via Altinns formidlingstjeneste.

#### Valideringstjeneste

https://etgltest.grunnbok.no/grunnbok/wsapi/v2/ValideringServiceWS

#### Teknisk dokumentasjon alle tjenester

https://etgltest.grunnbok.no/grunnbok/dokumentasjon.html

#### Endepunkter Altinn

| Tjeneste | URL |
|-----------------------------------|-------------------------------------------------------------------------------------------------------|
| BrokerServiceExternal | https://tt02.altinn.no/ServiceEngineExternal/BrokerServiceExternalBasic.svc |
| BrokerServiceExternalBasicStreamd | https://tt02.altinn.no/ServiceEngineExternal/BrokerServiceExternalBasicStreamed.svc |
| ReceiptExternalBasic | https://tt02.altinn.no/IntermediaryExternal/ReceiptExternalBasic.svc |

#### Konfigurasjonsparametre Altinn

| ServiceCode | ServiceEditionCode | Organisasjonsnummer Kartverket |
|-------------|--------------------|--------------------------------|
| 4433 | 3 | 910976168 |



### Testmiljø - betatest

#### Innsendingstjeneste

https://betatest.grunnbok.no/grunnbok/wsapi/v2/InnsendingServiceWS

I test er elektronisk innsending tilgjengelig både direkte via WS og via Altinns formidlingstjeneste.

#### Valideringstjeneste

https://betatest.grunnbok.no/grunnbok/wsapi/v2/ValideringServiceWS

#### Teknisk dokumentasjon alle tjenester

https://betatest.grunnbok.no/grunnbok/dokumentasjon.html

#### Endepunkter Altinn

| Tjeneste | URL |
|-----------------------------------|-------------------------------------------------------------------------------------------------------|
| BrokerServiceExternal | https://tt02.altinn.no/ServiceEngineExternal/BrokerServiceExternalBasic.svc |
| BrokerServiceExternalBasicStreamd | https://tt02.altinn.no/ServiceEngineExternal/BrokerServiceExternalBasicStreamed.svc |
| ReceiptExternalBasic | https://tt02.altinn.no/IntermediaryExternal/ReceiptExternalBasic.svc |

#### Konfigurasjonsparametre Altinn

| ServiceCode | ServiceEditionCode | Organisasjonsnummer Kartverket |
|-------------|--------------------|--------------------------------|
| 4433 | 5 | 910976168 |