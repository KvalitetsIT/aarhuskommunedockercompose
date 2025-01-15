# Hvad vi vil gerne opnå?
Vi vil gerne have et setup, hvor vi kan ændre konfigurationen for en server ved:
* At bygge og pushe et nyt image
* At påvirke den ønskede tilstand i et git repo

# Hvad venter vi med?
I første omgang antager vi, at hemmeligheder (passwords, certifikater osv) kopieres manuelt ud på de relevante servere.

I første omgang opsættes ingen fælles logopsamling eller monitorering & alarmering.

Adgang til logs kræver, at man tilgår den maskine, hvor loggen findes.

Monitorering og alarmering må ske eksternt f.eks. ved at kalde en bestemt URL for at tjekke, at servicen lever og svarer.

# Basissetup
Vi etablerer et git repo serverconf i azure med følgende struktur:

scripts/platform-deploy.sh
scripts/services-deploy.sh
platform/docker-compose.yaml
msbapi/docker-compose.yaml

Al generel opsætning (nginx til SSL terminering og proxying for et givent subdomæne kommer til at ligge i platform/docker-compose.yaml. 
Der vil være en række properties for denne, der varierer for de enkelte subdomæner. Disse vil ligge i mappen med subdomænets navn i platform.env.

Dernæst følger en mappe msbapi med en tilhørende docker-compose (det bliver de konkrete services, der kommer til at køre på den server der modtager requests til *.msbapi.adm.aarhuskommune.dk).
Hvis der bliver brug for flere subdomæner, så skal de tilføjes som ekstra docker-compose setups.

Der skal startes en linux maskine til domænet *.msbapi.adm.aarhuskommune.dk og opsættes DNS til denne.

Når denne linux maskine er klar skal der laves et par manuelle skridt.

## Oprettelse af bruger deploy
sudo useradd -m deploy

## SSL certifikat
For at serveren kan ssl terminere skal ssl certifikatet kopieres ud på serveren.
Certifikat og nøgle skal ligge her (ejer skal være vores deploy bruger):
/home/deploy/conf/ssl/server.cert
/home/deploy/conf/ssl/server.key

## Oprettelse af statusfolder (som deploy bruger)
mkdir /home/deploy/status

## Clone git repo (som deploy bruger)
Git repo skal clones i /home/deploy directory (opsæt en service account i Azure Devops, der må læse serverconf repo authenticate via ssh nøgler).

## Opsætning af auto-deploy
Der skal opsættes autodeploy af hhv. platforms opsætningen og opsætningen af de konkrete services
sudo crontab -e -u deploy
```
*/5 * * * * /home/deploy/serverconf/scripts/platform-deploy.sh msbapi
*/5 * * * * /home/deploy/serverconf/scripts/services-deploy.sh msbapi
```
hvor msbapi er navnet på miljøet

# Ekstern monitorering
Man kan kalde ind på status.msbapi.adm.aarhuskommune.dk for at få en status på om platformen er ok
Outputtet har følgende format:

application_information{environment="msbapi", gitcommit="938518a2d4360a350da969db6baa8b9ef0d921c4"} 1
platform_script_executed_time 1483228810000
platform_script_executed_status 1
service_script_executed_time 1483228810100
service_script_executes_status 0

Her kan man se hvilket miljø man har med at gøre og hvilken gitcommit man aktuelt har tjekket ud.
Man kan også se, hvornår hhv platform script og service script senest har kørt, og om det er gået godt eller skidt.
