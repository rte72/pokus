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
        "clear": <boolean>,
        "set": [
          {
            "port": <number>,
            "x1": <number>,
            "x2": <number>,
            "style": <number>,
            "color": <number>
          },
          {
            "port": <number>,
            "x1": <number>,
            "x2": <number>,
            "style": <number>,
            "color": <number>
          }
        ]
      }
    }
    ```
    **"clear"**:true zhasne všechny led před provedením requestu
    
    Example:    
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

2. Nastaveni barvy
    ```jsonc
    {
      "cmd": "set_color",
      "args": {
        "color_id": <number>,
        "color": <number>
        }
    }
    ```
    **color_id**: číslo barvy\
    **color**: barva ve formatu RGB dekadicky\
   
    Example:    
    ```jsonc
    {
      "cmd": "set_color",
      "args": {
        "color_id": 1,
        "color": 16711680   //0xFF0000 RED
        }
    }
    ```
# Prikazy co posila jednotka serveru
Prikazy se posilaji na topic ve tvaru: `I/<server_id>/<device_id>/[request_id]/CMD`\
Odpovedi od serveru jdou na: `O/<server_id>/<device_id>/<request_id>/CMD`\
**server_id**: ID serveru (zatim konfigurovatelne v nastaveni appky)\
**device_id**: ID jednotky (zatim konfigurovatelne v nastaveni appky)\
**request_id**: optional ID requestu, pokud neni, znamena to ze volajici neceka odpoved

Od serveru se ocekava odpoved, pokud nedojde odesila se request na server znova, momentalne 3x s odstupem 1s.

---
1. Alive
    ```jsonc
    {
        "cmd": "alive",
        "args": {
            "id": <string>,
            "mac": <string>
            "battery": <number>
        }
    }
    ```
    **Alive se posílá bez `[request_id]` neočekává odpověď**
    
2. Log
```jsonc
{
    "cmd":"log",
    "args":{
        "id": <string>,
        "level":<string>,
        "text": <string>
    }
```
**level**: DEBUG, INFO, WARNING, ERROR\
**text**: Popis
