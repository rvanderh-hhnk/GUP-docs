########
Messages
########
********
Testzooi
********
- test1
- test2
- test3

*********
Origineel
*********
De belangrijkste messages: "*upload succesvol*" of "*upload niet gelukt, pas gef aan*" worden geregeld bij het opslaan van waypoint in functie "save_gef" in **TestGefData.py**.

Ook in functie "upload_gef" in views.py worden enkele foutmeldingen gegenereerd als daar een fout optreed. Bijvoorbeeld als er iets misgaat bij het maken van de dictionary d_GEF in  **UtlGefOpen.py** wordt hier een foutmelding gegeven: "*fout bij uitlezen gef, pas gef aan*"

**Home.html**

De weergave wordt geregeld in het {% block messages %} in home.html.

Messages hebben een tag met "*info*" of "*error*" en worden respectievelijk met groen of rood weergegeven.  Alle messages worden in een lijst opgeslagen in django per request en op het einde weergegeven.
