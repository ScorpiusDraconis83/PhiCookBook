<!--
CO_OP_TRANSLATOR_METADATA:
{
  "original_hash": "62b2632720dd39ef391d6b60b9b4bfb8",
  "translation_date": "2025-07-16T17:04:41+00:00",
  "source_file": "code/07.Lab/01/Apple/phi3ext/vsc-extension-quickstart.md",
  "language_code": "sl"
}
-->
# Dobrodošli v vaši razširitvi za VS Code

## Kaj je v mapi

* Ta mapa vsebuje vse datoteke, potrebne za vašo razširitev.
* `package.json` - to je manifestna datoteka, v kateri deklarirate svojo razširitev in ukaz.
  * Vzorec vtičnika registrira ukaz in določi njegov naslov ter ime ukaza. S temi informacijami lahko VS Code prikaže ukaz v ukazni paleti. Vtičnika še ni treba naložiti.
* `src/extension.ts` - to je glavna datoteka, kjer boste implementirali svoj ukaz.
  * Datoteka izvozi eno funkcijo, `activate`, ki se pokliče prvič, ko je razširitev aktivirana (v tem primeru z izvajanjem ukaza). Znotraj funkcije `activate` pokličemo `registerCommand`.
  * Funkcijo, ki vsebuje implementacijo ukaza, posredujemo kot drugi parameter funkciji `registerCommand`.

## Namestitev

* Namestite priporočene razširitve (amodio.tsl-problem-matcher, ms-vscode.extension-test-runner in dbaeumer.vscode-eslint)

## Zaženite takoj

* Pritisnite `F5`, da odprete novo okno z naloženo vašo razširitvijo.
* Zaženite svoj ukaz iz ukazne palete s pritiskom na (`Ctrl+Shift+P` ali `Cmd+Shift+P` na Macu) in vtipkajte `Hello World`.
* Nastavite točke prekinitve v kodi znotraj `src/extension.ts` za razhroščevanje razširitve.
* Izhod vaše razširitve poiščite v konzoli za razhroščevanje.

## Naredite spremembe

* Razširitev lahko ponovno zaženete iz orodne vrstice za razhroščevanje po spremembah kode v `src/extension.ts`.
* Prav tako lahko osvežite (`Ctrl+R` ali `Cmd+R` na Macu) okno VS Code z vašo razširitvijo, da naložite spremembe.

## Raziščite API

* Celoten nabor našega API-ja lahko odprete z odpiranjem datoteke `node_modules/@types/vscode/index.d.ts`.

## Zaženite teste

* Namestite [Extension Test Runner](https://marketplace.visualstudio.com/items?itemName=ms-vscode.extension-test-runner)
* Zaženite nalogo "watch" preko ukaza **Tasks: Run Task**. Poskrbite, da teče, sicer testi morda ne bodo odkriti.
* Odprite pogled Testing iz stranske vrstice in kliknite gumb "Run Test" ali uporabite bližnjico `Ctrl/Cmd + ; A`
* Rezultate testov si oglejte v pogledu Test Results.
* Spremembe naredite v `src/test/extension.test.ts` ali ustvarite nove testne datoteke znotraj mape `test`.
  * Zagotovljeni testni izvajalec bo upošteval samo datoteke, ki ustrezajo vzorcu imena `**.test.ts`.
  * Znotraj mape `test` lahko ustvarite podmape za poljubno organizacijo testov.

## Pojdite korak dlje

* Zmanjšajte velikost razširitve in izboljšajte čas zagona z [združevanjem razširitve](https://code.visualstudio.com/api/working-with-extensions/bundling-extension).
* [Objavite svojo razširitev](https://code.visualstudio.com/api/working-with-extensions/publishing-extension) na tržnici razširitev VS Code.
* Avtomatizirajte gradnje z nastavitvijo [neprekinjene integracije](https://code.visualstudio.com/api/working-with-extensions/continuous-integration).

**Omejitev odgovornosti**:  
Ta dokument je bil preveden z uporabo storitve za avtomatski prevod AI [Co-op Translator](https://github.com/Azure/co-op-translator). Čeprav si prizadevamo za natančnost, vas opozarjamo, da lahko avtomatski prevodi vsebujejo napake ali netočnosti. Izvirni dokument v njegovem izvirnem jeziku velja za avtoritativni vir. Za pomembne informacije priporočamo strokovni človeški prevod. Za morebitna nesporazume ali napačne interpretacije, ki izhajajo iz uporabe tega prevoda, ne odgovarjamo.