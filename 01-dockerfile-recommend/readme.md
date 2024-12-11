# Hvad er en Dockerfile?
En Dockerfile er en opskrift på at bygge et Docker image. Består af en række instruktioner, der tilsammen definerer en række lag i imaget.

Starter med en FROM clause, der definerer udgangspunktet (base image).

Tilføjer lag i de følgende instruktioner - feks:
* Tilføjelse af den applikation, man gerne vil pakke ind
* Tilføjelse af tredjeparts biblioteker, som man er afhængig af

# Hvad er et godt Docker image?
Der er mange definitioner af, hvad det vil sige at være god.

Overordnet vil kriterierne falde i to overordnede kategorier:

* God at drifte
* Højt niveau af sikkerhed

Vi har i KIT lavet en template (godtnok for java, men principperne er de samme) som man kan se på: https://github.com/KvalitetsIT/kithugs

## Hvad gør et Docker image godt at drifte?
Her er et par punkter, som du skal overveje:
* Statelessness: Tilstand skal i database eller andet struktureret datastore. Containere kan opstå og dø på alle tænkelige tidspunkter, så lad være med at gemme filer inden i din container (visse drift miljøer vil faktisk ikke give dig mulighed for dette).
* Konfiguration: Som udgangspunkt, så skal det samme image kunne afvikles både på udviklingsmiljøet, test og produktion. Der skal ikke være miljøafhængige images - alt forskel fra miljø til miljø skal ske vha konfiguration (ENV variable).
* Helbreds/readiness tjek: En komponent skal kunne udstille sin egen sundhedstilstand (f.eks. dedikeret endpoint, der kan svare HTTP kode 200) så omgivelserne ved, om den er klar til at modtage arbejde (requests).
* Monitoreringssnitflade: Udstilling af metrikker, så der kan laves dashboard og alarmer (Prometheus er defakto standard - mest relevant, hvis man kører på en platform som f.eks. Kubernetes)
* Applikationslogning: Applikationslogs ud på stdout (ikke personhenførbare informationer her). Her kan de samles op (Loki er ofte brugt på Kubernetes)


## Hvordan sørger man for, at sikkerheden er i orden?
Mange vil nok sætte lighedtegn mellem god og sikker. En del sikkerhed relaterer sig til måden containerne driftes på, men man kan som udvikler gøre noget selv. Der er flere bud på best practise, men en fornuftig kilde er [OWASP Cheat Sheet Series: Docker Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html#docker-security-cheat-sheet).

Vi har taget de relevante (for udviklerne) rules her og beskrevet dem:

### RULE2
Du skal sætte en bruger (så du ikke kører som ROOT): https://github.com/KvalitetsIT/kithugs/blob/984275856f03582136ffbd63c38c108c9632c341/web/docker/Dockerfile#L20C1-L20C11

### RULE3
Du skal (helst) kunne leve med begrænsede privilegier. Det er som sådan noget, der sker på drifttidspunktet, men det er godt at tjekke, at din service kan overleve dette.

### RULE8
Du skal (helst) kunne leve med read-only filsystem. Det er som sådan noget, der sker på drifttidspunktet, men det er godt at tjekke, at din service kan overleve dette.

### RULE9 og RULE13
Et godt udgangspunkt is less-is-more: Dit image skal indeholde det mindste, der er nødvendigt. Vælg en så lille base image som muligt og tilføj så lidt som muligt for at begrænse angrebsfladen. Hvis du har brug for en compiler, så benyt dig af multistage builds, hvor du har flere faser (f.eks. et image, der bruges til at bygge kode med, og et andet image, der brugs ved afvikling).
Scan dine images regelmæssigt og sørg for regelmæssig opdatering, så du følger med patches osv.

