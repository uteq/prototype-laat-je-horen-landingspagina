# Setup: e-mailadressen opslaan in Google Sheets

Volg deze stappen één keer. Daarna worden alle aanmeldingen automatisch in je spreadsheet opgeslagen.

---

## Stap 1 — Maak een Google Sheet aan

1. Ga naar [sheets.google.com](https://sheets.google.com) en maak een nieuwe spreadsheet aan.
2. Geef hem een duidelijke naam, bijv. **Laat je Horen - Aanmeldingen**.
3. Voeg een header toe in rij 1:
   - Cel A1: `Datum`
   - Cel B1: `E-mailadres`

---

## Stap 2 — Voeg het Apps Script toe

1. Klik in de spreadsheet op **Uitbreidingen** (of "Extensions") in het menu.
2. Klik op **Apps Script**.
3. Verwijder alle bestaande code in het editor-venster.
4. Plak de onderstaande code erin:

```javascript
function doPost(e) {
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();

  var email   = '';
  var website = ''; // honeypot-veld

  try {
    var data = JSON.parse(e.postData.contents);
    email   = data.email   || '';
    website = data.website || '';
  } catch (err) {
    email   = (e.parameter && e.parameter.email)   ? e.parameter.email   : '';
    website = (e.parameter && e.parameter.website) ? e.parameter.website : '';
  }

  // Honeypot: als 'website' ingevuld is, is het een bot — stil negeren
  if (website) {
    return ContentService
      .createTextOutput(JSON.stringify({ status: 'ok' }))
      .setMimeType(ContentService.MimeType.JSON);
  }

  if (email) {
    sheet.appendRow([new Date(), email]);
  }

  return ContentService
    .createTextOutput(JSON.stringify({ status: 'ok' }))
    .setMimeType(ContentService.MimeType.JSON);
}
```

5. Klik op het **opslaan-icoon** (of Ctrl/Cmd + S). Geef het project een naam, bijv. **LJH Signup**.

---

## Stap 3 — Deployen als web app

1. Klik rechtsboven op **Implementeren** > **Nieuwe implementatie**.
2. Klik op het tandwiel naast "Selecteer type" en kies **Web-app**.
3. Vul in:
   - **Beschrijving:** Laat je Horen signup
   - **Uitvoeren als:** Mij (jouw Google-account)
   - **Toegang:** Iedereen
4. Klik op **Implementeren**.
5. Google vraagt om toestemming — klik door en geef akkoord.
6. Kopieer de **URL van de web-app** die verschijnt. Die ziet er zo uit:
   ```
   https://script.google.com/macros/s/AKfycb.../exec
   ```

---

## Stap 4 — URL in de landingspagina plakken

1. Open `landingspagina/index.html` in een teksteditor.
2. Zoek deze regel (staat in het `<script>`-blok onderaan):
   ```javascript
   var SHEETS_URL = '';
   ```
3. Vervang de lege string door jouw URL:
   ```javascript
   var SHEETS_URL = 'https://script.google.com/macros/s/AKfycb.../exec';
   ```
4. Sla het bestand op en upload opnieuw naar je hosting.

---

## Testen

1. Open de pagina in de browser.
2. Vul een test-e-mailadres in en klik op **Aanmelden**.
3. Controleer je Google Sheet — het adres moet verschijnen in rij 2 (of later).

Als er na 30 seconden niets in de sheet staat:
- Controleer of de URL precies klopt (begint met `https://script.google.com/macros/s/`)
- Controleer of "Toegang" op **Iedereen** staat (niet "Alleen ikzelf")
- Open Apps Script opnieuw, klik op **Implementeren** > **Implementaties beheren** en controleer de instellingen

---

## Iedere nieuwe aanmelding

Iedere keer dat iemand zich aanmeldt via de landingspagina, verschijnt er automatisch een nieuwe rij in je spreadsheet met datum/tijd en e-mailadres. Je hoeft niets te doen.

Wil je een e-mailnotificatie per aanmelding? Voeg dit toe aan het Apps Script (na de `sheet.appendRow`-regel):

```javascript
MailApp.sendEmail('jouw@emailadres.nl', 'Nieuwe aanmelding', 'E-mail: ' + email);
```
