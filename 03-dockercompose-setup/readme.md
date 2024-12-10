Nu vil vi bygge et setup bestående af en webserver og en række bagvedliggende komponenter.

Webserveren har til opgave at SSL terminere og route requests videre til den korrekte applikation.

Ideen er, at webserveren modtager requests til et specifikt subdomæne. I dette tilfælde hostnavne, der matcher *.msbapi.adm.aarhuskommune.dk
Dette skal konfigureres i organisationens DNS.

Webserveren modtager så et request på f.eks. xyz.msbapi.adm.aarhuskommune.dk
Routingen sker videre til en bagvedliggende komponent med navnet xyz på port 8080.
På denne måde kan man tilføje nye services (til et nyt subdomæne) dynamisk dvs. uden at skulle rette i webserverens konfiguration.


# Inden vi går i gang
Til SSL certifikat er der i dette testsetup blevet genereret et self-signed stjernecertifikat med openssl. I et rigtigt setup skal "nogen" (Aarhus kommune) skaffe sådan et.
Kommandoen der er brugt til at generere certifikat:
´´´
openssl req -x509 -newkey rsa:4096 -sha256 -days 3650 -nodes -keyout server.key -out server.cert -subj "/CN=msbapi.adm.aarhuskommune.dk" -addext "subjectAltName=DNS:msbapi.adm.aarhuskommune.dk,DNS:*.msbapi.adm.aarhuskommune.dk"
´´´

For at få requests, der matcher *.msbapi.adm.aarhuskommune.dk, til at ende inde i vores testsetup, skal der noget opsætning til. I dette testsetup tilføjes blot noget til din lokale hosts fil.
På windows ligger den vist under C:\Windows\System32\drivers\etc
På linux ligger den i /etc/hosts

Vi tilføjer en række subdomæner og lader dem pege på localhost (desværre kan man ikke bruge wildcards i hosts filen, men det kan men jo på en rigtig DNS server):
´´´
127.0.0.1  test.msbapi.adm.aarhuskommune.dk
127.0.0.1  hej.msbapi.adm.aarhuskommune.dk
127.0.0.1  ny.msbapi.adm.aarhuskommune.dk
´´´

# Testopsætning
Vi starter to bagvedliggende services (hej og test), der begge svarer på port 8080. Ingen af disse kan nås fra hosten (ingen portmappings).

Web server er sat op til at SSL terminere og route requests videre til den service med navnet subdomæne på port 8080 på det docker netværk, der udspændes.

Web serveren har en portmapping, så vi kan tilgå den fra hostmaskinen på port 443.

* Start compose setup og tjek, at de tre containere starter korrekt op.
* Tilgå følgende URL'er i en browser (man får en warning fordi man bruger et self-signed certifikat - man kan tilføje det som et trustet certifikat i browseren, hvis man er træt af det)
  * https://test.msbapi.adm.aarhuskommune.dk/ (giver en side, der siger noget med test - loggen skulle gerne afsløre, at den korrekte backend er i sving)
  * https://hej.msbapi.adm.aarhuskommune.dk/ (som ovenfor bare med den anden backend)
  * https://ny.msbapi.adm.aarhuskommune.dk/ (giver en fejlside fra webserveren, da der ikke findes en backend med navnet 'ny', der kan betjene requestet)


# Prøv selv
Tilføje en ny service (kaldet ny), så URL'en ovenfor kan komme til at virke

# Overvejelser
Er det her et "godt" setup? Det kan nok bruges i begrænset omfang
* Setup'et er dynamisk nok til at det bliver let at tilføje nye services (hvis man holder sig til konventionen om navne og porte)
* En service kører i en instans
* Et subdomæne -> en server
* Routing sker i DNS opsætningen, hvilket nok kan blive lidt uoverskueligt
* Ingen logopsamling, monitorering, alarmering og andre rare ting
* "Hjemmebrygget" i en eller anden form. God nok til at få erfaringer, men evt problemer kan enten skyldes, at setup'et er for simpelt (og ikke nødvendigvis, at containere er en dårlig ide)
