# Enable-u-auth
Referentieimplementaties in .NET Core en .NET 4.x voor enable-u authN

## Prerequisites

### Framework

.NET Core en/of .NET 4.x

### Operating system

Public facing website/api Windows of Linux (VM) met een Fully Qualified Domain Name (FQDN) om authN af te handelen.

### Self signed certificates

Installeer openssl via chocolatey: https://chocolatey.org/packages/openssl


De referentieimplementatie gebruikt self signed certificates. Voer in powershell (of commandline) het volgende (unattended) commando uit (na het wijzigen van de FQDN's en IP nummers):

```
openssl req -x509 -newkey rsa:4096 -sha256 -days 3650 -nodes -keyout app.key -out app.crt -subj "/C=NL/ST=Zuid-Holland/L=Den Haag/O=wigo4it/OU=Org/CN=appname.azurewebsites.net" -addext "subjectAltName=DNS:appname1.azurewebsites.net,DNS:appname2.azurewebsites.net,IP:10.0.0.1"
```

Het creëert een certificaat dat

* geldig is voor de domeinen appname1.azurewebsites.net en appname2.azurewebsites.net (SAN),
* ook geldig is voor het IP-adres 10.0.0.1 (SAN),
* relatief sterk is (vanaf 2020) en
* geldig is voor 3650 dagen (~ 10 jaar).

Het creëert de volgende bestanden:

1. Privésleutel: app.key
2. Certificaat: app.crt

## AuthN minimale requirements

In het kader van SSD beproeven of een bestaand .NET auth framework (zoals openIdConnect) ingezet kan worden in de referentieimplementatie. Deze zorgen tevens voor login/logout flow/sliding expiration.

1.	Aanpassen applicatie voor koppeling Enable-u DigiD
2.	Relatieve hyperlinks/verwijzingen gebruiken i.p.v absoluut, zodat de browserurl gedurende de DigiD sessie gelijk blijft. 

Bijvoorbeeld: 
```
belastingsloket/formulier/inkomensbelasting = goed
/belastingsloket/formulier/inkomensbelasting = fout bij routering op resourcePath, Goed bij routering op basis van (sub)domein
https://externeCloudUrl.nl/formulier/inkomensbelasting = fout
```
## AuthZ minimale requirements

Claims (scope) in de applicatie toevoegen voor:

1. DigiD claim (BSN) uit http-headers/parameters
2. Betrouwbaarheidsniveau Claim (Digid) uit http-headers/parameters
3. Optioneel: Gemeente claim (naam gemeente/gemeentecode)

## App minimale requirements:

1. Login flow (login url)
2. Logout flow (logout url)
3. Toevoegen van claims
4. Lezen van claims voor attestatie/applicatie logica
5. Logout knop op alle protected webpagina’s, die verwijzen naar een logout url uit punt 2.
6. Pagina's (requirements uit Logius/DigiD):
    * Landingspage/homepage
    * ‘Inloggen is geannuleerd’
    * Url:
    * 'Errorpagina’
    * ‘Opnieuwinloggen’ pagina (sessie verlopen)
    *  Overige openbare pagina’s (bijvoorbeeld static content)
    *  Protected webpagina Url’s (bijv. alles beginnend met het path belastingsloket/formulieren/* of belastingsloket/protected/*). Om deze pagina’s te benaderen is een DigiD sessie vereist.

## API minimale requirements

1. Aanleveren protected API Url’s voor die bijvoorbeeld gebruikt worden voor asynchrone (bijv. ajax) http requests (bijv. alles beginnend met het path belastingsloket/api/*). Om deze pagina’s te benaderen is een DigiD sessie vereist.
Url: