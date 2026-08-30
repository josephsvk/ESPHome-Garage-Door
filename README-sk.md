# ESPHome Garage Door

[English version](README.md)

ESPHome balík na bezpečné ovládanie garážovej alebo inej motorickej brány cez Home Assistant. Projekt používa tri reléové vstupy pohonu (`OPEN`, `CLOSE`, `STOP`) a fyzický senzor úplne zatvorenej brány ako hlavný zdroj pravdy.

## Čo projekt rieši

- Ovládanie brány cez entitu `cover` v Home Assistante.
- Samostatné impulzy pre `OPEN`, `CLOSE` a `STOP` s interlock ochranou proti súčasnému zopnutiu relé.
- Bezpečný štart zariadenia: po boote sa všetky relé vypnú a stav brány sa zosynchronizuje podľa CLOSED senzora.
- Podpora WireGuard pripojenia, aby ESPHome zariadenie mohlo zostať dostupné v jednej bezpečnej sieti s Home Assistantom.
- Logika pre brány bez horného koncového snímača: otvorený stav sa po nastavenom čase predpokladá, zatvorený stav musí potvrdiť fyzický senzor.
- Štatistiky otvorenia: aktuálny čas otvorenia, posledné otvorenie, celkový čas otvorenia, počet otvorení a počet CLOSE príkazov.
- Diagnostické entity pre WireGuard, stav logiky, reštart a reset štatistík.

## Súbory

- `Garage-Door.yml` - hlavný ESPHome package s logikou brány.
- `esphome-sample.yml` - minimálna konfigurácia zariadenia, ktorá načíta package z GitHubu.
- `secrets.yaml.sample` - vzor potrebných hodnôt pre Wi-Fi, API, OTA, WireGuard, dosku a GPIO piny.

## Požiadavky

- ESPHome 2026.8.1 alebo novší.
- ESP32 doska podporovaná ESPHome.
- Home Assistant s ESPHome Builderom alebo lokálna ESPHome inštalácia.
- Pohon brány s oddelenými vstupmi pre `OPEN`, `CLOSE` a `STOP`.
- Fyzický senzor úplne zatvorenej brány pripojený na GPIO vstup.
- Funkčné WireGuard údaje, ak používaš aktuálnu verziu balíka.

## Inštalácia v Home Assistante

1. Otvor `Home Assistant > ESPHome Builder`.
2. Klikni na `+ New Device`.
3. Vyber `Continue` a potom `Skip this step` alebo `Advanced setup`, podľa verzie ESPHome Buildera.
4. Vytvor prázdnu konfiguráciu zariadenia.
5. Skopíruj obsah `esphome-sample.yml` do konfigurácie nového zariadenia.
6. Do svojho ESPHome `secrets.yaml` doplň hodnoty podľa `secrets.yaml.sample`.
7. Skontroluj GPIO piny, dosku, framework a WireGuard nastavenia.
8. Ulož konfiguráciu a spusti `Install`.
9. Po prvom nahratí over v Home Assistante stav entity garážovej brány a fyzicky otestuj `OPEN`, `STOP` a `CLOSE`.

## Nastavenie secretov

Minimálne musíš nastaviť:

- názov zariadenia a friendly name,
- typ dosky a ESP32 framework,
- GPIO piny pre `OPEN`, `CLOSE`, `STOP` a CLOSED senzor,
- Wi-Fi SSID, heslo a fallback AP heslo,
- ESPHome API kľúč a OTA heslo,
- WireGuard adresu, privátny kľúč zariadenia, verejný kľúč peeru, endpoint, port a povolené IP rozsahy.

Použi `secrets.yaml.sample` ako šablónu. Skutočný `secrets.yaml` necommituj do repozitára.

## Logika stavu brány

CLOSED senzor má najvyššiu prioritu. Keď je aktívny, brána sa považuje za zatvorenú a všetky aktívne pohybové sekvencie sa ukončia.

Keď CLOSED senzor nie je aktívny, brána sa považuje za otvorenú alebo v pohybe. Keďže projekt nemá horný koncový snímač, otvorenie sa po čase `assume_open_after` označí ako predpokladané. Pri zatváraní sa čaká na fyzické potvrdenie CLOSED senzora; ak nepríde do `close_timeout`, logika nastaví chybový stav `CLOSE timeout`.

## Plánované doplnenia

- Možnosť používať projekt aj bez WireGuard.
- Automatické testovanie funkčnosti pred inštaláciou pomocou ESP32 > ESP32.
- Farebný maják alebo indikátor stavu brány.
- Jednoduchšie pridávanie I2C modulov a základných rozšírení.

## Bezpečnostné poznámky

Pred pripojením k pohonu si over jeho elektrické požiadavky a typ vstupov. Relé výstupy z ESP32 modulu nesmú priamo spínať napätie alebo prúd mimo parametrov použitého relé modulu. Prvé testy rob s pohonom pod dohľadom a s možnosťou okamžite odpojiť napájanie.
