---
tags:
  - IT
---

# Ferjanove avtomatične čestitke - avtomatizacija čestitk za rojstni dan


![Cestitke](https://lh3.googleusercontent.com/pw/AP1GczOH2U2pMiCVPIgJOvfa65UkaDtIC-RN3SIPhbIqrEqC1LyrlAD0gihhQYcmUhFY74JssEkoYcS7ggAGeFhC4W-QLp_8jjSBTLqcBf4vVyyf9QujVDH2bxV8II663A4K6Q2ndITRMgFMcDW1O9a0_rDEFA=w1920-h480-s-no-gm?authuser=0 "Cestitke"){referrerpolicy="no-referrer"}

Saj veš kako to ponavadi poteka. Tvoj koledar, Facebook ali podobna storitev ti pošlje avtomatizirano obvestilo, da ima prijatelj rojstni dan. Ker si dober prijatelj mu pošlješ en izviren "Vse najboljše za rojstni dan!", čemur po navadi sledi en izviren "Hvala!". Prijateljev imaš seveda veliko zato ta korak ponoviš vsaj enkrat na teden. 

Kaj pa če ... bi zadevo avtomatizirali, tako da bi "izvirno" čestitko prejel vsak prijatelj, ki si to resnično želi.

Spoznaj **Ferjanove avtomatične čestitke<sup>TM</sup>**. Storitev, ki prijateljem omogoča, da se naročijo na avtomatizirane čestitke za svoj rojstni dan.

[:fontawesome-solid-envelope:  cestitke.ferjan.net](http://cestitke.ferjan.net){ .md-button .md-button--primary }


  >Da ne bo kdo narobe razumel. Svoje prijatelje za rojstni dan raje pokliči/obišči in jim čestitaj iz srca.
  >Pričujoča storitev je namenjena zgolj učenju Google APP scripta. Gre za nekakšen "proof of concept" primer, ki sem ga uporabil spoznavanje osnov Google APP scripta. 

## Kaj potrebujemo

- [x] [Spletni obrazec za prijavo](#spletni-obrazec-za-prijavo) (Google Forms)
- [x] [Zbirko prijavljenih uporabnikov](#zbirka-prijavljenih-uporabnikov) (Google Spreadsheets)
- [x] Predloge 
    * [x] [Predloga za potrditev prijave na storitev]() (Google Docs)
    * [x] [Predloga za pošiljanje čestitke](https://docs.google.com/document/d/1IacJM0pHSt1yHQYh9XFCMD1pdDNMS_2mi1MLa9uWbwg/edit?usp=sharing) (Google Docs)
- [x] Skripte
    * [x] [Skripta OnFormSubmit](#skripta-onformsubmit) (Google Apps Script)
    * [x] [Skripta z API-jem za prijavo/odjavo od storitve](#skripta-z-api-jem-za-prijavoodjavo-od-storitve) (Google Apps Scrip)
    * [x] [Skripta za pošiljanje čestitk](#skripta-za-posiljanje-cestitk) (Google Apps Scrip)
    * [x] [Skripta za "Garbage Collection"](#skripta-za-garbage-collection) (Google Apps Scrip)
- [X] Triggerji za skripte
    * [x] [Trigger za funkcijo `OnFormSubmit`](#trigger-za-funkcijo-onformsubmit)
    * [x] [Trigger za funkcijo `sendBdayWishes`](#trigger-za-funkcijo-sendbdaywishes)
    * [x] [Trigger za funkcijo `garbageCollection`](#trigger-za-funkcijo-garbagecollection)
- [ ] Opcijsko
    * [ ] [Promocijski naslov storitve](#promocijski-naslov-storitve-dns-preusmeritev) (DNS, preusmeritev)
    * [ ] [Poštni naslov storitve](#postni-naslov-storitve) (Google Workspace)

### Spletni obrazec za prijavo

[Spletni obrazec](https://docs.google.com/forms/d/e/1FAIpQLSeDGwYFHfKZaCR3yZq9OfqTnXkaSsmVFsIaJUB_bvB6G-pkCw/viewform) ima 5 obveznih polj:

* Polje za vnos e-mail naslova uporabnika.
* Polje za vnos Imena uporabnika.
* Polje za vnos datuma rojstva uporabnika.
* Polje za soglasje k obdelavi osebnih podatkov (GDPR pa take fore)
* Polje za soglasje s pogoji uporabe storitve.

### Zbirka prijavljenih uporabnikov

Zbirka prijavljenih uporabnikov je Google Spreadsheets datoteka, ki jo avtomatično ustvari Google Forms.

Preglednica ima sledeče stolpce:

`Časovni žig`

:   Čas oddaje Google Forms obrazca.
    Avtomatično zabeleženo ob dodaji obrazca.

`E-poštni naslov`	

:   E-poštni naslov uporabnika storitve.
    Avtomatično zabeleženo ob dodaji obrazca.

`Ime`

:   Ime uporabnika storitve.
    Avtomatično zabeleženo ob dodaji obrazca.

`Datum rojstva`	

:   Datum rojstva uporabnika storitve.
    Avtomatično zabeleženo ob dodaji obrazca.

`Varstvo osebnih podatkov`	

:   Soglasje z obdelavo osebnih podatkov uporabnika storitve.
    Avtomatično zabeleženo ob dodaji obrazca.

`Pogoji uporabe`	

:   Soglasje s pogoji uporabe storitve.
    Avtomatično zabeleženo ob dodaji obrazca.

Preglednico razširimo in dodamo tri dodatne stolpce:

`Poslano`	

:   Časovni žig zadnjega pošiljanja čestite
    Zabeleži skripta za pošiljanje čestitk ob pošiljanju.

`formId`	

:   ID številka obrazca za prijavo na storitev.
    Zabeleži skripta OnFormSubmit ob oddaji obrazca.

`ActiveStatus`

:   Polje v katerem beležimo, če je posamezna prijava aktivirana, t.j. je uporabnik potrdil lastništvo vnešenega e-mail naslova.
    Zabeleži skripta z API-jem za prijavo/odjavo od storitve ob aktivaciji prijave.

### Skripta OnFormSubmit

Skripta vsebuje funkcije `OnFormSubmit`, `sendMail` in `replaceMarkerWithLink`.

Vsakič, ko nekdo odda obrazec, se sproži [trigger za funkcijo OnFormSubmit](#trigger-za-funkcijo-onformsubmit).

``` JavaScript
function onFormSubmit(e) {
  // Odpri preglednico 
  var sheet = SpreadsheetApp.openById('1qV8b4QagcvoFoeq8hT4KqFJIq6U8DW9SbSpdYlk0hPI').getSheetByName('Odzivi na obrazec 1');
  
  //
  var templateId = '16Efta4XTKXsDsd-YKkrfA9iEBYITr-YXrIvL1py9p44'; // the template doc with placeholders
    
  // Pridobi ID številko oddanega obrazca
  var formId = e.response.getId();
  
  // Pridobi posredovani e-poštni naslov 
  var toMail = e.response.getRespondentEmail();
  
  // Preverimo, če je formId že zabeležen (potrebno zaradi tega, da se potrdilo ne pošlje ponovno ob popravljanju obrazca)
  var isCellBlank = sheet.getRange(sheet.getLastRow(), 8).isBlank();
  
  // Če formId še ni zabeležen
  if(isCellBlank) {
   
    // Pošlji ID Forme na posredovani e-poštni naslov
    sendMail(sheet,templateId,toMail,formId);
    
    // Zapiši ID številko obrazca v osmi stolpec preglednice
    sheet.getRange(sheet.getLastRow(), 8).setValue(formId);
   
    // Zapiši status aktivacije v deveti stolpec preglednice
    sheet.getRange(sheet.getLastRow(), 9).setValue(0);
  }

  // izbriši odgovor iz baze forme.
  var form = FormApp.openByUrl('https://docs.google.com/forms/d/1q3Zw-UMIk2xM_kHgbnbzSzBvcM4pC6ZGZthTfXuue-w/');
  form.deleteResponse(formId); 
  
}

function sendMail(sheet,templateId,toMail,formId){
 
  // Poiščemo kje v predlogi se nahaja marker in ga zamenjamo z oblikovano povezavo
  var docId = DriveApp.getFileById(templateId).makeCopy('temp').getId(); // Naredimo kopijo predloge
  var doc = DocumentApp.openById(docId); // Odpremo kopijo predloge
  
  // Pokličemo funkcijo, ki v besedilu dokumenta zamenja marker s povezavo
  replaceMarkerWithLink(doc, '#potrditev#', 'Potrdi prijavo', 'https://script.google.com/a/ferjan.net/macros/s/AKfycbx7GLQLhKnS00iI2SIUiIa81DBRAhMpZfFA1vo1w4dQ9St4WS8/exec?id=' +formId);
  replaceMarkerWithLink(doc, '#odjava#', 'Odjava', 'https://script.google.com/a/ferjan.net/macros/s/AKfycbx7GLQLhKnS00iI2SIUiIa81DBRAhMpZfFA1vo1w4dQ9St4WS8/exec?unsubscribe=1&id=' +formId);
  
  doc.saveAndClose(); // save changes before conversion
      
  var url = "https://docs.google.com/feeds/download/documents/export/Export?id="+docId+"&exportFormat=html";
  var param = {
    method: "get",
    headers: {"Authorization": "Bearer " + ScriptApp.getOAuthToken()}
  };	
  var htmlBody = UrlFetchApp.fetch(url,param).getContentText();	
  var trashed = DriveApp.getFileById(docId).setTrashed(true); // delete temp copy  
  var Rname = "Ferjanove avtomatične čestitke"; // Set real email-from name
	
  MailApp.sendEmail(toMail,'Potrditev prijave',' ' ,{htmlBody: htmlBody, name: Rname});
  
}

function replaceMarkerWithLink(doc, marker, replaceMarkerTxt, replaceMarkerUrl) {
  var element = doc.getBody().findText(marker);
  if(element){
    var start = element.getStartOffset();
    var text = element.getElement().asText();
    text.replaceText(marker,replaceMarkerTxt);
    text.setLinkUrl(start, start+replaceMarkerTxt.length-1, replaceMarkerUrl);
  }
}
```

### Skripta z API-jem za prijavo/odjavo od storitve

To skripto je potrebno objaviti (**Deploy this project**), da dobimo DeploymentID in URL.
Ta URL uporabimo v skriptah na delih, kjer potrebujemo URL za potrditev prijave in za odjavo od storitve.

``` JavaScript
// Povzeto po https://gist.github.com/mhawksey/1276293
// Popravljen bug https://issuetracker.google.com/issues/72798634 #13 in #18 dodan /a/ferjan.net/ v API URL

function doGet(e){
  return handleResponse(e);
}

function handleResponse(e) {
  // LockService prevents concurrent access overwritting data
  // http://googleappsdeveloper.blogspot.co.uk/2011/10/concurrency-and-google-apps-script.html
  var lock = LockService.getPublicLock();
  lock.waitLock(30000);  // wait 30 seconds before conceding defeat.
  
  try {
    // Pridobimo id iz GET zahtevka
    var id = e.parameter.id;

    // V primeru, da id parameter ni podan izpišemo napako
    if (id == null) {
      return ContentService
          .createTextOutput("Zahtevek ne vsebuje parametra z ID številko uporabnika.");    
    }
    
    // Pridobimo unsubscribe parameter
    var unsubscribe = e.parameter.unsubscribe;
    
    // Nastavimo kam bomo pisali
    var doc = SpreadsheetApp.openById('1qV8b4QagcvoFoeq8hT4KqFJIq6U8DW9SbSpdYlk0hPI');
    var sheet = doc.getSheetByName('Odzivi na obrazec 1');
    // Nastavimo pravilno vrstico glede na vrednost id. V ta namen uporabimo dodatno funkcijo findRowByValue.
    var row = findRowByValue(sheet,id);
    // Nastavimo stolpec, ki vsebuje podatek o statusu aktivacije
    var columnActiveStatus = 9;    

    // V primeru, da vrstice za zahtevani id ne najdemo izpišemo napako
    if (row == -1) {
      return ContentService
          .createTextOutput("Uporabnik s to ID številko ne obstaja.");
    }

    // Pohendlamo morebiten unsubscribe
    if (unsubscribe == 1) {
      sheet.deleteRow(row);
      return ContentService
          .createTextOutput("Vaša prijava na Ferjanove avtomatične čestitke je odstranjena.");
    }
    
    // Zapišemo vrednost 1, ki pomeni, da je prijava aktivna
    sheet.getRange(row,columnActiveStatus).setValue(1);
        
    // vrnemo pozitiven odgovor
    return ContentService
          .createTextOutput("Vaša prijava na Ferjanove avtomatične čestitke je aktivirana.");
  } catch(e){
    // v primeru napake vrnemo error
    return ContentService
          .createTextOutput("Napaka");
  } finally { //release lock
    lock.releaseLock();
  }
}

// Funkcija, ki poišče številko vrstice z določeno vrednostjo v stolpcu H2:H
function findRowByValue(sheet, value) {
  var lastRow = sheet.getLastRow();
  var dataRange = sheet.getRange('H2:H'+lastRow);
  var values = dataRange.getValues();

  for (var i = 0; i < values.length; i++) {
      if (values[i] == value) {
        return i+2;
      }
   }
  return -1;  
}
```

### Skripta za pošiljanje čestitk

Vsak dan med 9h in 10h se poganja [trigger, ki sproži funkcijo sendBdayWishes](#trigger-za-funkcijo-sendbdaywishes). Funkcija `sendBdayWishes` preveri, če se današnji datum ujema s kakšnim rojstnim datumom. V kolikor pride do ujemanja pokliče funkcijo `sendMail`, ki pošlje čestitko s pomočjo predloge.

``` JavaScript

function sendBdayWishes(){
  var ss = SpreadsheetApp.openByUrl("https://docs.google.com/spreadsheets/d/1qV8b4QagcvoFoeq8hT4KqFJIq6U8DW9SbSpdYlk0hPI/"); // Google SpreadSheet URL - To je naša baza uporabnikov
  var sheet = ss.getSheetByName("Odzivi na obrazec 1"); // Google SpreadSheet - Sheet name - Ime zavihka v preglednici
  var templateId = '1IacJM0pHSt1yHQYh9XFCMD1pdDNMS_2mi1MLa9uWbwg'; // Template doc - Predloga za pošiljanje čestitke
	
  var cDate = new Date(); //Present Day, 
  for(var i =2 ;i<=sheet.getLastRow(); i++){
  
    var bDate = sheet.getRange(i,4).getValue(); // Date from SpreadSheet 
  
    if(cDate.getDate()==bDate.getDate()){
      if(cDate.getMonth()==bDate.getMonth()){
        // Pošljemo le uporabnikom s potrjeno aktivacijo, t.j. tistim, ki imajo v 9. stolpcu vrednost 1
        if(sheet.getRange(i,9).getValue()==1){
          var name = sheet.getRange(i,3).getValue();
          var toMail = sheet.getRange(i,2).getValue();
          var formId = sheet.getRange(i,8).getValue();
          sendMail(sheet,templateId,name,toMail,formId);
          // V stolpec 7 zabeležimo kdaj smo poslali e-pošto
          sheet.getRange(i,7).setValue(cDate); 
        }
      }
    }
  }
}


function sendMail(sheet,templateId,name,toMail,formId){
 
  var docId = DriveApp.getFileById(templateId).makeCopy('temp').getId();
  var doc = DocumentApp.openById(docId); // the temp copy
  var body = doc.getBody();

  body.replaceText('#name#',name); // Zamenjaj marker za #name# z vrednostjo spremenjivke name
  
  // Poiščemo kje v predlogi se nahaja marker za odjavo in ga zamenjamo z oblikovano povezavo
  var marker = '#odjava#';
  var replaceMarkerTxt = 'Odjava';
  // Popravljen bug https://issuetracker.google.com/issues/72798634 #13 in #18 dodan /a/ferjan.net/ v API URL
  var replaceMarkerUrl = 'https://script.google.com/a/ferjan.net/macros/s/AKfycbx7GLQLhKnS00iI2SIUiIa81DBRAhMpZfFA1vo1w4dQ9St4WS8/exec?unsubscribe=1&id=' +formId;
  
  var element = doc.getBody().findText(marker);
  if(element){
    var start = element.getStartOffset();
    var text = element.getElement().asText();
    text.replaceText(marker,replaceMarkerTxt);
    text.setLinkUrl(start, start+replaceMarkerTxt.length-1, replaceMarkerUrl);
  }
  
  doc.saveAndClose(); // save changes before conversion
      
  var url = "https://docs.google.com/feeds/download/documents/export/Export?id="+docId+"&exportFormat=html";
  var param = {
    method: "get",
    headers: {"Authorization": "Bearer " + ScriptApp.getOAuthToken()}
  };	
  var htmlBody = UrlFetchApp.fetch(url,param).getContentText();	
  var trashed = DriveApp.getFileById(docId).setTrashed(true); // delete temp copy  
  var Rname = "Ferjanove avtomatične čestitke"; // Set real email-from name
	
  MailApp.sendEmail(toMail,'Vse najboljše '+name,' ' ,{htmlBody: htmlBody, name: Rname});
```

### Skripta za garbage collection

Med 4h in 5h zjutraj se poganja [trigger, ki sproži skripto garbageCollection](#trigger-za-funkcijo-garbagecollection) s katero počisti vse neaktivirane prijave.

``` JavaScript
// Skripta se poganja s triggerjem pozno ponoči, da ne bi sklučajno prezgodaj izbrisali še ne aktiviranih prijav.
  
function garbageCollection(){

var SS = SpreadsheetApp.openById("1qV8b4QagcvoFoeq8hT4KqFJIq6U8DW9SbSpdYlk0hPI");
var SHEET = SS.getSheetByName("Odzivi na obrazec 1");
var RANGE = SHEET.getDataRange();
 
 
var DELETE_VAL = 0; // Vrednost, ki je sprožilec za izbris
var COL_TO_SEARCH = 8; // The column to search for the DELETE_VAL (Zero is first)
  
  
  var rangeVals = RANGE.getValues();
  
  //Reverse the 'for' loop.
  for(var i = rangeVals.length-1; i >= 0; i--){
    if(rangeVals[i][COL_TO_SEARCH] === DELETE_VAL){
      
      SHEET.deleteRow(i+1); 
    }
  }
}
```

### Trigger za funkcijo `OnFormSubmit`

function: **onFormSubmit** <br />
runs at deployment: **Head** <br />
event source: **From form** <br />
event type **On form submit** <br />
failure notification: **Notify me daily**

### Trigger za funkcijo `sendBdayWishes`

function: **sendBdayWishes** <br />
runs at deployment: **Head** <br />
event source: **Time-driven** <br />
type of time based trigger: **Day timer** <br />
time of the day: **9am to 10am** <br />
failure notification: **Notify me daily**

### Trigger za funkcijo `garbageCollection`

function: **garbageCollection** <br />
runs at deployment: **Head** <br />
event source: **Time-driven** <br />
type of time based trigger: **Day timer** <br />
time of the day: **4am to 5am** <br />
failure notification: **Notify me daily**

### Promocijski naslov storitve (DNS, preusmeritev)

Svojo domeno gostujem pri Cloudflare. Med DNS zapisi imam vpisan CNAME zapis `cestitke` za domeno `ferjan.net`. Nastavljeno imam tudi pravilo za preusmeritve:

If incoming requests match… <br />
Custom filter expression

Field: `Hostname` <br />
Operator: `equals` <br />
Value: `cestitke.ferjan.net`

Then URL redirect

Type: `Static` <br />
URL: `https://docs.google.com/forms/d/e/1FAIpQLSeDGwYFHfKZaCR3yZq9OfqTnXkaSsmVFsIaJUB_bvB6G-pkCw/viewform` <br />
Status code: `302`

### Poštni naslov 

Storitev pošilja vse e-maile iz naslova cestitke@ferjan.net.

Pošto za domeno ferjan.net upravljam preko storitve **Google Workspace for nonprofits**. Za potrebe te storitve sem odprl Google Workspace račun cestitke@ferjan.net v katerem so shranjene tudi vse skripte za to storitev.

<br />
...torej kaj še čakaš. Prijavi se! Tvoj rojstni dan je lahko že jutri. :smile:

[:fontawesome-solid-envelope:  cestitke.ferjan.net](http://cestitke.ferjan.net){ .md-button .md-button--primary }