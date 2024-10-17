# DIGG: Geburtsmeldungen für Krankenhäuser

Mit DIGG erfassen Sie notwendige Daten zur Geburtsanzeige und können diese
automatisiert und sicher an das Standesamt übermitteln.

Der Aufwand für den zuständigen Mitarbeiter wird dabei durch Vorbefüllung der bekannten Information aus der KIS Integration möglichst sowie der automatischen digitalen Übermittlung ans Standesamt minimiert. Bei Vollständiger Verfügbarkeit aller Daten durch das KIS ist auch ein komplett automatisierte Meldung möglich, ohne das weitere Mitarbeiter Interaktion zwingend erforderlich ist. 

## Ablauf des Meldeprozesses

to be written

todo: insert form and page flow diagrams here

### Datenstruktur

Hier sehen Sie beispielhaft einen Datensatz der Geburtsvoranzeige. Mit allen oder auch nur Teilen dieser Daten aus dem KIS können die Formulare vorausgefüllt werden. 

**JSON** 

```json
{
  "kind": {
    "name": {
      "vorname": "Grant"
    },
    "geburtsdatum": "2024-01-02",
    "geburtsstunde": "12:34",
    "geburtsort": {
      "strasse": "Hauptstr.",
      "hausnummer": "1",
      "stadt": "Berlin",
      "ortsteil": "Tempelhof",
      "kreisbezeichnung": "Kreis Berlin"
    },
    "geschlecht": "WEIBLICH"
  },
  "mutter": {
    "name": {
      "nachname": "Anna",
      "vorname": "Müller"
    },
    "geschlecht": "WEIBLICH",
    "adresse": {
      "strasse": "Hauptstraße 9",
      "postleitzahl": "10111",
      "stadt": "Berlin"
    },
    "familienstand": "VERHEIRATET",
    "geburtsdatum": "1997-01-30",
    "kontakt": {
      "email": "joe@example.com",
      "telefon": "+49 172 1234567"
    }
  }
}
```

**application/x-www-form-urlencoded**

```http
POST {{ host }}/api/v3/memento/route/geburtsbescheinigung
Authorization: Basic {{username}} {{password}}
Content-Type: application/x-www-form-urlencoded
X-CSRF-TOKEN: {{csrfToken}}

mutter.name.vorname=Edan  &
mutter.geburtsdatum=1976-04-14  &
mutter.familienstand=VERHEIRATET  &
mutter.name.nachname=Richmond  &
mutter.geschlecht=MAENNLICH  &
mutter.adresse.strasse=Rocky Hague Drive 43c  &
mutter.adresse.postleitzahl=83059  &
mutter.adresse.stadt=Consectetur molliti  &
mutter.kontakt.telefon=Ea molestiae beatae  &
mutter.kontakt.email=lyhinodoga@example.com  &
kind.name.vorname=Lane  &
kind.geburtsdatum=1984-06-12  &
kind.geschlecht=DIVERS  &
kind.geburtsstunde=04:04  &
kind.geburtsort.strasse=White New Drive 41b  &
kind.geburtsort.ortsteil=Tempelhof &
kind.geburtsort.stadt=Laufbach &
kind.geburtsort.kreisbezeichnung=Beilingen
```


## API Integration zur Vorbefüllung aus dem KIS

Diese Anleitung erklärt, wie Sie die Vertama DIGG API verwenden können um DIGG mit ihrem KIS zu integrieren. Die API bietet dabei die Möglichkeit mit Daten aus dem KIS System URLs/Links zu den Eingabeformularen zu generieren, die dann entsprechend mit allen Daten aus dem KIS System vorausgefüllt sind um somit die notwendigen manuellen Eingaben zu minimieren. Die generierten URLs sind cryptographisch gesichert und erlauben keinen Zugriff auf die Daten ausserhalb der DIGG Integration. Ein Eintrag solcher URLs in Logfiles oder sonstigen Protokollen oder Browsercaches ist somit unkritisch. Die eigentliche API Nutzung, sowie der Formular Zugriff durch HTTPS und TLS absichert. 

Die Bespielhafte Nutzung der API ist technologieneutral mittels `curl` demostriert. Für notwendigen Accounts für einen Test- oder Produktivbetrieb wenden sie sich bitte direkt an Ihren Ansprechpartner bei Vertama. 

_`TODO: overview of the complete flow here. KIS -- api --> DIGG --> KIS --> BROWSER --> DIGG --> RKI --> (BROWSER | KIS)`_

## 1) Memento String erhalten

Vertama DIGG Formulare können im Webbrowser vorausgefüllt aufgerufen werden. Diese Daten werden vom Vertama Server aus einem `Memento` ermittelt welcher der Formular URL in form the `m` Parameters mitgegeben werden kann. Das Memento ist verschluesselt und kann nur vom Vertama Server ausgelesen werden. Der Vertama Server speichert keine Daten, die Daten werden ausschlieslich im Memento gespeichert (client-side).

### Beispiel

Zur Veranschaulichung, hier beispielhaft eine Memento Anfrage mit `curl`: 

```bash
curl -X POST "https://elim.dev.vertamob.com/api/v3/memento/params" \
     -u "{username}:{password}" \
     -H "Content-Type: application/json" \
     -d '{
    "kind": {
        "name": {
            "vorname": "Grant"
        },
        "geburtsdatum": "2024-01-02",
        "geburtsstunde": "12:34",
        "geburtsort": {
            "strasse": "Hauptstr.",
            "hausnummer": "1",
            "stadt": "Berlin",
            "ortsteil": "Tempelhof",
            "kreisbezeichnung": "Kreis Berlin"
        },
        "geschlecht": "WEIBLICH"
    },
    "mutter": {
        "name": {
            "nachname": "Anna",
            "vorname": "Müller"
        },
        "geschlecht": "WEIBLICH",
        "adresse": {
            "strasse": "Hauptstraße 9",
            "postleitzahl": "10111",
            "stadt": "Berlin"
        },
        "familienstand": "VERHEIRATET",
        "geburtsdatum": "1997-01-30",
        "kontakt": {
            "email": "joe@example.com",
            "telefon": "+49 172 1234567"
        }
    }
}'
```

### Endpunkt

Um einen Memento String zu erhalten, werden die Daten im JSON Format an folgenden Endpunkt
gesendet: https://elim.dev.vertamob.com/api/v3/memento/params

Die Authentifizierung erfolgt durch `Basic Auth`.

Als Response gibt die API ein JSON Objekt in folgender Form zurück:

```json
{
  "m": "MementoString"
}
```

Sie benötigen den value vom "m" key. Das ist Ihr Memento String. 

## 2) Formularaufruf aus dem KIS

Das DIGG Formular befindet sich unter `https://elim.dev.vertamob.com/digg/Geburtsbescheinigung` und kann direkt im Brower aufgerufen werden. Die Authentifizierung durch `Basic Auth`. Für vorausgefüllte Formulare kann der URL der optionala Memento Parameter `m` angehängt werden.

### 1. Vorausgefüllt mit Memento String

Haben Sie zuvor den Memento String aus Schritt 1 erhalten, können Sie diesen Nutzen, damit das Formular vorausgefüllt
ist. Dafür öffnen Sie im Browser die URL:

[https://{username}:{password}@elim.dev.vertamob.com/digg/Geburtsbescheinigung?m={memento}](https://{username}:{password}@elim.dev.vertamob.com/digg/Geburtsbescheinigung?m={memento})

* username: Ihr Benutzername
* password: Ihr Passwort zu dem Benutzer
* memento: Der aus Schritt 1 erhaltene Memento String

### 2. Leer ohne Memento String

Um das Formular ohne Memento String aufzurufen (leer - ohne vorausgefüllte Felder), öffnen Sie in Ihrem Browser die URL:

[https://{username}:{password}@elim.dev.vertamob.com/digg/Geburtsbescheinigung](https://{username}:{password}@elim.dev.vertamob.com/digg/Geburtsbescheinigung)

* username: Ihr Benutzername
* password: Ihr Passwort zu dem Benutzer
