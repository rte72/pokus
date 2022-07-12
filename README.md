# Posilani requestu na jednotku
Requesty se posilaji na topic ve tvaru: `I/<device_to>/<device_from>/[request_id]/CMD`\
Odpovedi na request jdou na: `O/<device_to>/<request_from>/<request_id>/CMD`\
**device_from**: ID odesilatele\
**device_to**: ID prijemce\
**request_ID**: optional ID requestu, pokud neni, znamena to ze volajici neceka odpoved\

# Format requestu
```jsonc
{
    "cmd":"<cmd_id>",
    "args": { // optional, pokud prikaz nema argumenty netreba vkladat
        "<arg_name>":<arg_value>
    }
}
```

# Format odpovedi na request
```jsonc
{
    "status":"<response_code>",
    "data":<data> // optional data
}
```
**response_code**:
`ok` = ok odpoved\
`err` = chyba

Jednoduche odpovedi bez dat mozne poslat jako jednoduchy string:
`{ "status": "ok" }` ekvivalentni s `"ok"`


---
1. Rozsvícení/Zhasnutí ledek
    ```jsonc
    {
      "cmd": "leds",
      "args": {
        "clear": true,
        "set": [
          {
            "port": 1,
            "x1": 1,
            "x2": 3,
            "style": 0,
            "color": 1
          },
          {
            "port": 1,
            "x1": 4,
            "x2": 100,
            "style": 1,
            "color": 2
          }
        ]
      }
    }
    ```
    "clear":true zhasne všechny led před provedením requestu
2. Nastaveni obrazovky rukavice
    ```jsonc
    {
        "cmd":"display"
        "args": {
            "id":"<template_id>",
            "fields":"<fields>",
            "separator":"<separator>", // optional
            "duration":<duration>, // optional
            "refresh":<refresh> // optional
        }
    }
    ```
    **template_id**: ID sablony dle [dokumentace](https://docs.proglove.com/en/screen-templates.html#templates-68-7760)\
    **fields**: Text pro vyplneni poli sablony, jeden string, format dle [dokumentace](https://docs.proglove.com/en/set-screens.html#intent-specific-parameters)\
    **separator**: Optional separator datovych poli, vychozi je `;`, pokud byl ve `fields` pouzit jiny je treba ho zde poslat\
    **duration**: Optional doba zobrazeni obrazovky v ms, vychozi 0 = navzdy\
    **refresh**: Optional typ obnoveni obrazovky (`"FULL_REFRESH"`,`"PARTAL_REFRESH"` nebo `"DEFAULT"`) dle [dokumentace](https://docs.proglove.com/en/set-screens.html#refreshtype-refresh_type--refreshtype-string)
   
    Example:
    ```jsonc
    {
        "cmd":"display",
        "args": {
            "id":"PG2",
            "fields":"1|ABC|DEF|2|123|456",
            "separator":"|",
            "duration":0,
            "refresh":"DEFAULT"
        }
    }
    ```
3. Vyvolani svetylek/zvuku na rukavici
    ```jsonc
    {
        "cmd":"feedback",
        "args": {
            "id":<feedback_ids>
        }
    }
    ```
    **feedback_ids**: Bud jedine feedback ID nebo muze byt i pole (sekvence) ktera se ma provest,
    ID dle [dokumentace](https://docs.proglove.com/en/worker-feedback---sdk-intent-api.html#trigger-feedback-using-intent). ID muzete posilat jako integer i string.
    
    Prehlidka
    ```jsonc
    {
        "cmd":"feedback",
        "args":{
            "id":[1,"2",3,"4",5]
        }
    }
    ```
    Jeden
    ```jsonc
    {
        "cmd":"feedback",
        "args":{
            "id":1 // ekvivalentni se [1]
        }
    }
    ```
4. Dotaz na stav androidu/rukavice
    ```jsonc
    { "cmd": "state" }
    ```
    Odpoved zpatky:
    ```jsonc
    {
        "status":"ok", // status odpovedi na request
        "data":{
            "device":<state_android>,
            "glove":<state_glove>
        }
    }
    ```
    **state_android**: Zatim posilam proste "ok", uvidime do budoucna co pripojime\
    **state_glove**: String jako stav pripojeni, odpovida tomu co posila Insight aplikace dle [dokumentace](https://docs.proglove.com/en/intent-api---basic-integration.html#receive-scanner-connection-state)

5. Pozadavek na zapis do NFC tagu<a id='nfc-write'/>
Mala vsuvka k tomu jak je NFC tag strukturovan z pohledu software, aby se vyresila fragmentace komunikacnich protokolu s tagy od ruznych vyrobcu existuje standard [NDEF](https://cs.wikipedia.org/wiki/NFC_Data_Exchange_Format) (NFC Data Exchange Format), takze se da s sirokou skalou NFC zarizeni komunikovat jednotne. Zapis a cteni probiha pomoci NDEF zpravy, ktera obsahuje zaznamy (records), ty obsahuji nejaky metadata o tom co v tom zaznamu je a potom samotny payload v bajtech. Realne pokud zapisujes text do tagu, tak v podstate zapisujes zpravu s jednim zaznamem textovym. Rozliseni zda, se jedna o textovy nebo binarni zaznam se dela na zaklade MIME typu dat, `text/*` = text, vse ostatni jako binarni.\
\
Android odpovi OK pokud doslo k zobrazeni vyzvy pro zapis pres NFC a tedy uzivatel byl informovan a muze zapsat. Neznamena to, ze k samotnemu zapisu uz doslo. Udalost o samotnem zapsani odesle Android jako oddeleny event na server, viz [Dokonceni zapisu do NFC tagu](#nfc-write-done). 

```jsonc
{
    "cmd":"nfc_write",
    "args":{
        "id":<tag_id>, // volitelne; fyzicke ID tagu, pokud uzivatel prilozi telefon k tagu s jinym ID zapis se neprovede, pokud neni argument definovan zapis je povolen do libovolneho tagu
        "records":[<text_record>/<binary_record>,...]
    }
```
**tag_id**: Fyzicke ID tagu, posilane jako hexadecimalni reprezentace jednotlivych bajtu (2 znaky = 1 bajt), prvni dva znaky zleva = prvni bajt v prectenem poli bajtu, kazdy bajt musi byt zarovnan na dva znaky\

Zapis jednotlivych zaznamu vypada v plne forme takto:
```jsonc
{
    "binary": true/false // volitelne, zda je obsah textovy nebo binarni (zakodovan jako Base64 retezec). Vychozi je false (=text).
    "data": "data k zapisu" // obsah k zapisu, pokud je nastaven binarni priznak, obash je Base64 dekodovan na pole bajtu
}
``` 
**text_record**: Zapis textoveho zaznamu\
priklad: `{"binary":false, "data":"ABCD"}` pro text je mozne pouzit i zkracenou variantu, ekvivalentni zapis by tedy vypadal jednoduse takto `"ABCD"`. Data se zapisou s MIME typem `text/plain`.\
**binary_record**: Posila se jako Base64 zakodovane pole bajtu, po dekodovani je zapsano s MIME typem `application/octet-stream`.\
priklad: `{"binary":true, "data":"QUJDREVGR0hJSktMTU4="}`

Zapis jednoho textoveho zaznamu bez omezeni na ID tagu ve 3 ekvivalentnich variantach:
```jsonc
{
    "cmd":"nfc_write",
    "args":{
        "records":["Hello NFC world!"]
    }
}
```
```jsonc
{
    "cmd":"nfc_write",
    "args":{
        "records":[{"data":"Hello NFC world!"}]
    }
}
```
```jsonc
{
    "cmd":"nfc_write",
    "args":{
        "records":[{"binary":false,"data":"Hello NFC world!"}]
    }
}
```

Zapis dvou zaznamu jeden textovy a jeden binarni s omezenim zapisu na ID tagu 
```jsonc
{
    "cmd":"nfc_write",
    "args":{
        "id":"04962EFAC64880",
        "records":["Hello NFC world!", {"binary":true, "data":"QUJDREVGR0hJSktMTU4="}]
    }
}
```
   
# Prikazy co posila Android serveru
Prikazy se posilaji na topic ve tvaru: `I/<server_id>/<device_id>/[request_id]/CMD`\
Odpovedi od serveru jdou na: `O/<server_id>/<device_id>/<request_id>/CMD`\
**server_id**: ID serveru (zatim konfigurovatelne v nastaveni appky)\
**device_id**: ID Android zarizeni (zatim konfigurovatelne v nastaveni appky)\
**request_id**: optional ID requestu, pokud neni, znamena to ze volajici neceka odpoved

Od serveru se ocekava odpoved, pokud nedojde odesila se request na server znova, momentalne 3x s odstupem 1s.

---
1. Scan rukavici / kamerou telefonu
    ```jsonc
    { 
        "cmd":"scan",
        "args":{
            "barcode":"<barcode>",
            "symbology":"<symbology>"
        }
    }
    ```
    **barcode**: Naskenovany text, zde [dokumentace](https://docs.proglove.com/en/intent-api---basic-integration.html#barcode-scan-intent-structure) ale nic zajimavyho tam k tomu neni\
    **symbology**: Pouzita symbologie, seznam podporovanych dle [dokumentace](https://docs.proglove.com/en/symbology-settings.html)
    
2. Precteni NFC tagu
Pro male vysvetleni struktury komunikace s NFC tagem viz server pozadavek [Zapis do tagu](#nfc-write).

```jsonc
{
    "cmd":"nfc_read",
    "args":{
        "id":<tag_id>, // fyzicke ID tagu
        "records": <records>
    }
```
**tag_id**: Fyzicke ID tagu\
**records**: Pole zaznamu, podle prectenych metadat (MIME) se rozhodne zda posilat jako text nebo binarni data.\
Format argumentu stejny jako u [Pozadavek na zapis do NFC tagu](#nfc-write).

3. Dokonceni zapisu do NFC tagu<a id='nfc-write-done'/>
```jsonc
{
    "cmd":"nfc_write_done",
    "args":{
        "id":<tag_id>,
        "original":<original_records>, // pole zaznamu pred zapisem
        "written":<new_records>, // pole nove zapsanych zaznamu
    }
}
```
**tag_id**: Fyzicke ID tagu\
**original_records**: Pole puvodnich zaznamu\
**new_records**: Pole nove zapsanych zaznamu\
Format argumentu stejny jako u [Pozadavek na zapis do NFC tagu](#nfc-write).
   
# Zaverecne poznamky
Zachytavam i udalost o double clicku rukavici, navazal jsem si to pokusne na refresh obrazovky,
ale kdyby jste nasli vyuziti tak to muzeme navazat na neco nebo odesilat na server :-)
    
