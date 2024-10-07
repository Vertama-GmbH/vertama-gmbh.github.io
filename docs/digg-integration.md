# DIGG Integration

Diese Anleitung erklärt, wie Sie die Vertama DIGG API verwenden können um DIGG mit ihrem KIS zu integrieren. Die API bietet dabei die Möglichkeit mit Daten aus dem KIS System URLs/Links zu den Eingabeformularen zu generieren, die dann entsprechend mit allen Daten aus dem KIS System vorausgefüllt sind um somit die notwendigen manuellen Eingaben zu minimieren. Die generierten URLs sind cryptographisch gesichert und erlauben keinen Zugriff auf die Daten ausserhalb der DIGG Integration. Ein Eintrag solcher URLs in Logfiles oder sonstigen Protokollen oder Browsercaches ist somit unkritisch. Die eigentliche API Nutzung, sowie der Formular Zugriff durch HTTPS und TLS absichert. 

Die Bespielhafte Nutzung der API ist technologieneutral mittels `curl` demostriert. Für notwendigen Accounts für einen Test- oder Produktivbetrieb wenden sie sich bitte direkt an Ihren Ansprechpartner bei Vertama. 

_`TODO: overview of the complete flow here. KIS -- api --> DIGG --> KIS --> BROWSER --> DIGG --> RKI --> (BROWSER | KIS)`_

## 1) Memento String erhalten

Vertama DIGG Formulare können im Webbrowser vorausgefüllt aufgerufen werden. Diese Daten werden vom Vertama Server aus einem `Memento` ermittelt welcher der Formular URL in form the `m` Parameters mitgegeben werden kann. Das Memento ist verschluesselt und kann nur vom Vertama Server ausgelesen werden. Der Vertama Server speichert keine Daten, die Daten werden ausschlieslich im Memento gespeichert (client-side).

### Endpunkt

Um einen Memento String zu erhalten, werden die Daten im JSON Format an folgenden Endpunkt
gesendet: https://elim.dev.vertamob.com/api/v3/memento/params

Die Authentifizierung erfolgt durch `Basic Auth`.

Als Response gibt die API ein JSON Objekt zurück in folgender Form:

```json
{
  "m": "MementoString"
}
```

Sie benötigen den value vom "m" key. Das ist Ihr Memento String. 

### Datenstruktur

Hier sehen Sie einen kompletten Datensatz. Wenn Sie möchten, dass alle Eingabefelder vorausgefüllt sind, muss ein 
JSON Datensatz gesendet werden, der diese Daten enthält.

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

### Beispiel

Um das ganze nun zu veranschaulichen, hier beispielhaft eine Memento Anfrage mit `curl`: 

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

## 2) Formular öffnen

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
