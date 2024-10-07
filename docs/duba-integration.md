# DUBA Integration

Diese Anleitung erklärt, wie Sie die Vertama DUBA API verwenden können um DUBA mit ihrem KIS zu integrieren. 
Die API bietet dabei die Möglichkeit mit Daten aus dem KIS System URLs/Links zu den Eingabeformularen zu generieren, 
die dann entsprechend mit allen Daten aus dem KIS System vorausgefüllt sind um somit die notwendigen manuellen 
Eingaben zu minimieren. Die generierten URLs sind cryptographisch gesichert und erlauben keinen Zugriff 
auf die Daten ausserhalb der DUBA Integration. Ein Eintrag solcher URLs in Logfiles oder sonstigen 
Protokollen oder Browsercaches ist somit unkritisch. Die eigentliche API Nutzung, sowie der Formular 
Zugriff durch HTTPS und TLS absichert. 

Die Bespielhafte Nutzung der API ist technologieneutral mittels `curl` demonstriert. Für einen notwendigen
Account für den Test- oder Produktivbetrieb wenden Sie sich bitte direkt an Ihren Ansprechpartner bei Vertama.

## 1) Memento String erhalten

Vertama DUBA Formulare können im Webbrowser vorausgefüllt aufgerufen werden. Diese Daten werden vom Vertama Server aus einem `Memento` ermittelt welcher der Formular URL in form the `m` Parameters mitgegeben werden kann. Das Memento ist verschluesselt und kann nur vom Vertama Server ausgelesen werden. Der Vertama Server speichert keine Daten, die Daten werden ausschlieslich im Memento gespeichert (client-side).

### Endpunkt

Um einen Memento String zu erhalten, werden die Daten im JSON Format an folgenden Endpunkt
gesendet: https://elim.dev.vertamob.com/api/v3/memento

Die Authentifizierung erfolgt durch `Basic Auth`.

Als Response gibt die API ein JSON Objekt zurück in folgender Form:

```json
{
  "m": "MementoString"
}
```

Sie benötigen den value vom "m" key. Das ist Ihr Memento String. 

### Datenstruktur AnmeldungGeschlosseneUnterbringungMinderjaehriger

Hier sehen Sie einen kompletten Datensatz für das Formular einer Anmeldung für eine geschlossene Unterbringung
Minderjaehriger. Wenn Sie möchten, dass alle Eingabefelder vorausgefüllt sind, muss ein 
JSON Datensatz gesendet werden, der diese Daten enthält.

```json
{
  "MeldeDatum": "2024-03-21",
  "Aktenzeichen": "2414-515/1124E22",
  "Patient": {
    "Name": {
      "Titel": "Frau",
      "Vorname": "Tiffany",
      "Nachname": "Taylor"
    },
    "Geburtsdatum": "2003-02-01",
    "Station": "Teststation",
    "GesetzlicherVertreter": {
      "Ja": "yes",
      "Name": {
        "Titel": "Frau",
        "Vorname": "Anika",
        "Nachname": "Weber"
      },
      "Adresse": {
        "Strasse": "Hauptstr. 3",
        "Plz": "12435",
        "Stadt": "Berlin"
      },
      "Telefon": "015711111111"
    }
  },
  "ReportType": {
    "AnmeldungGeschlosseneUnterbringung": {
      "stationsarzt": {
        "name": "Hans Meier",
        "telefon": "0157222222222"
      },
      "erziehungsberechtigter": {
        "name": {
          "vorname": "Lavinia",
          "nachname": "Wong"
        },
        "adresse": {
          "strasse": "Hauptstraße",
          "hausnummer": "99",
          "plz": "10117",
          "stadt": "Berlin"
        }
      },
      "diagnose": "Sed molestiae ut sed",
      "dauer": "3 Wochen",
      "aufnahmegrund": "Voluptatem Doloribu"
    }
  }
}
```

### Beispiel

Um das ganze nun zu veranschaulichen, hier beispielhaft eine Memento Anfrage mit `curl`: 

```bash
curl -X POST "https://elim.dev.vertamob.com/api/v3/memento" \
     -u "{username}:{password}" \
     -H "Content-Type: application/json" \
     -d '{
          "MeldeDatum": "2024-03-21",
          "Aktenzeichen": "2414-515/1124E22",
          "Patient": {
            "Name": {
              "Titel": "Frau",
              "Vorname": "Tiffany",
              "Nachname": "Taylor"
            },
            "Geburtsdatum": "2003-02-01",
            "Station": "Teststation",
            "GesetzlicherVertreter": {
              "Ja": "yes",
              "Name": {
                "Titel": "Frau",
                "Vorname": "Anika",
                "Nachname": "Weber"
              },
              "Adresse": {
                "Strasse": "Hauptstr. 3",
                "Plz": "12435",
                "Stadt": "Berlin"
              },
              "Telefon": "015711111111"
            }
          },
          "ReportType": {
            "AnmeldungGeschlosseneUnterbringung": {
              "stationsarzt": {
                "name": "Hans Meier",
                "telefon": "0157222222222"
              },
              "erziehungsberechtigter": {
                "name": {
                  "vorname": "Lavinia",
                  "nachname": "Wong"
                },
                "adresse": {
                  "strasse": "Hauptstraße",
                  "hausnummer": "99",
                  "plz": "10117",
                  "stadt": "Berlin"
                }
              },
              "diagnose": "Sed molestiae ut sed",
              "dauer": "3 Wochen",
              "aufnahmegrund": "Voluptatem Doloribu"
            }
          }
        }'
```

## 2) Formular öffnen

Das DUBA Formular befindet sich unter `https://elim.dev.vertamob.com/duba/{form}` und kann direkt im Brower aufgerufen werden. Die Authentifizierung durch `Basic Auth`. Für vorausgefüllte Formulare kann der URL der optionala Memento Parameter `m` angehängt werden.

* form: Das Formular, welches aufgerufen werden soll

### 1. Vorausgefüllt mit Memento String

Haben Sie zuvor den Memento String aus Schritt 1 erhalten, können Sie diesen Nutzen, damit das Formular vorausgefüllt
ist. Dafür öffnen Sie im Browser die URL:

[https://{username}:{password}@elim.dev.vertamob.com/duba/{form}}?m={memento}](https://{username}:{password}@elim.dev.vertamob.com/digg/Geburtsbescheinigung?m={memento})

* username: Ihr Benutzername
* password: Ihr Passwort zu dem Benutzer
* form: Das Formular, welches aufgerufen werden soll
* memento: Der aus Schritt 1 erhaltene Memento String

### 2. Leer ohne Memento String

Um das Formular ohne Memento String aufzurufen (leer - ohne vorausgefüllte Felder), öffnen Sie in Ihrem Browser die URL:

[https://{username}:{password}@elim.dev.vertamob.com/duba/{form}](https://{username}:{password}@elim.dev.vertamob.com/digg/Geburtsbescheinigung)

* username: Ihr Benutzername
* password: Ihr Passwort zu dem Benutzer
* form: Das Formular, welches aufgerufen werden soll

## 3) Formularnamen {form}
* AnmeldungGeschlosseneUnterbringungMinderjaehriger
* AbmeldungGeschlosseneUnterbringungMinderjaehriger
