## Docker compose i produktion
Der findes en guide til dette: https://docs.docker.com/compose/how-tos/production/

Vi kiggede på best-practice for Dockerfile i sidste kapitel. Man kunne sikkert lave en lignende gennemgang/hardning af et docker compose setup i produktion.
KIT har ikke erfaringer på dette område. Vi arbejder med best practise og zero-trust for Kubernetes.

Der skal laves to compose-setups:
1) "Platform": Indeholder infrastrukturkomponenter. I første omgang vil dette nok "bare" være en web server for et givent subdomæne. Måske kunne man tilføje noget monitorering senere, men det kan vi lige lade stå åben. Dette docker-compose setup kan startes som en systemd service
2) Applikationerne: De services, som I udvikler og gerne vil have driftet. Lave et cronjob, der trækker opdateret docker-compose ud fra git (i Azure) og kører docker compose på det (genstarter opdaterede komponenter). Burde nok monitorers, men det kan vi vende tilbage til. 
