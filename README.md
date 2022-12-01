# Aplikace pro automatické stahování smluvních podmínek z webových stránek
Aplikace se skládá ze dvou částí - crawleru a neuronových sítí. Crawler stahuje stránky, které obsahují informace o ochraně osobních údajů nebo podmínky užití na základě klíčových slov. Při použití klíčových slov očekáváme určitou míru falešné pozitivity. Tu se snažíme eliminovat použitím neuronových sítí, které stránky klasifikují jako podmínky užití, informace o ochraně osobních údajů nebo nerelevantní stránky. Pro trénování neuronových sítí je použit přímo výstup crawleru bez manuálního označování.

Bližší informace jsou k dispozici [zde](https://otik.uk.zcu.cz/handle/11025/44238).

## Vývojové prostředí
Aplikace byla testována na operačních systémech Windows 10 PRO 1909 a Ubuntu 20.04.1  
s kernelem 5.8.0-48-generic. V obou případech s Python 3.8.1. Je vhodné mít k dispozici alespoň 16 GB operační paměti, jinak může docházet k pádům aplikace.

## Instalace prerekvizit
### Dodatečné soubory
Pro správné fungování některých skriptů je potřeba stáhnout ZIP archiv [zde](https://drive.google.com/file/d/1aFUuOcKDjAd4NIKJvnr4LOoxtMXGhz9n/view?usp=sharing) a extrahovat jeho obsah do kořenového adresáře projektu (např. pomocí 7zip).

### Crawler
Všechny balíčky lze nainstalovat příkazem
<code>pip install -r requirements.txt</code>

Crawler podporuje pouze webový prohlížeč Google Chrome - je potřeba ho nainstalovat. Zároveň je potřeba stáhnout odpovídající verzi chromedriveru (ke stažení [zde](https://chromedriver.chromium.org/)). Aplikace očekává, že chromedriver bude umístěn v kořenovém adresáři projektu a bude se jmenovat <code>chromedriver.exe</code> při spuštění na OS Windows a <code>chromedriver</code> při spuštění na OS Linux. Cesta ke chromedriveru je definovaná v proměnné <code>chromedriver_path</code> souboru <code>crawler/config.py</code>.

Konfigurace crawleru je v souboru <code>crawler/config.py</code>, kde je možné specifikovat klíčová slova, seznam výchozích URL a další parametry crawleru.

### Neuronové sítě
Neuronové sítě využívají sémantické reprezentace [fasttext](https://fasttext.cc). Ty lze stáhnout pomocí skriptu <code>download_fasttext.py</code>.

## Spuštění crawleru
Crawler se spouští pomocí skriptu <code>run_crawler.py</code>. Crawler je do jisté míry odolný vůči neočekávanému zastavení. Crawler je možné opětovně spustit a bude pokračovat, kde přestal. Může ale dojít k situaci, kdy zastavení crawleru poruší soubory uložené na disku, a pak je nutné crawler spustit znovu od začátku.

Výstupem crawleru jsou soubory
* <code>nolinks.txt</code> - seznam stránek, ze kterých nebyly staženy žádné relevantní stránky,
* <code>nocookies.txt</code> - seznam stránek, ze kterých nebyly staženy žádné informace o ochraně osobních údajů,
* <code>noterms.txt</code> - seznam stránek, ze kterých nebyly staženy žádné podmínky užití.

Stažené stránky jsou uloženy ve složce <code>pages</code>, ve které každá doména má svou vlastní složku. Každá složka domény pak obsahuje složky
* <code>terms</code> - složka se stránkami, které obsahují podmínky užití,
* <code>cookies</code> - složka se stránkami, které obsahují informace o ochraně osobních údajů.

Běh crawleru se zaznamenává do souboru <code>log.txt</code>.

## Spuštění neuronové sítě
### Relevantní složky
Pro neuronové sítě jsou relevantní následující složky:
* <code>model_configurations</code> - konfigurace neuronových sítí,
* <code>neural_net</code> - zdrojové kódy pro předzpracování textů, neuronové sítě a validaci modelů,
* <code>raw_datasets</code> - obsahuje crawlerem shromážděné relevantní a nerelevantní stránky, použité v práci jako dataset,
* <code>split_datasets</code>  - obsahuje předzpracovaný dataset rozdělený na trénovací a testovací části,
* <code>validation_datasets</code> - obsahuje vývojové a kontrolní datasety s jejich anotacemi,
* <code>final_models</code> - obsahuje finální evaluované modely.

### Vytvoření datasetu
Pro vytvoření datasetu a jeho rozdělení na trénovací a testovací části slouží skript <code>create_dataset.py</code>.  Trénovací část se uloží do složky <code>train_created</code> a testovací část do složky <code>test_created</code>. Je vytvořen soubor <code>preprocessing-config</code>, který obsahuje data potřebná pro předzpracování.

### Trénování modelů
Skripty <code>train_saved_model.py</code> a <code>train_created_model.py</code> slouží pro trénování modelů. První z nich trénuje modely použité v práci nad daty použitými v práci. Druhý skript modely trénuje nad daty, které byly vytvořeny pomocí skriptu <code>create_dataset.py</code> (viz. výše). Skripty jinak fungují  
obdobně.

Skripty vyžadují jeden argument, a to konfiguraci modelu. Konfigurace modelů jsou ve složce <code>model_configurations</code>, která obsahuje následující složky:
* <code>zakladni_model</code> - konfigurace základního modelu,
* <code>dropout_model</code> - konfigurace modelu s dropoutem,
* <code>zmensene_modely</code> - konfigurace zmenšených modelů.

Složky obsahují konfigurace modelů, z jejichž názvu je zjevné, o jaký model z práce se jedná. Skripty jako argument vyžadují cestu ke konfiguraci ve tvaru <code>složka modelu/konfigurace</code>, tedy například

<code>zakladni_modely/s-klicovymi-slovy-8-epoch</code>

Protože skripty využívají relativní cesty, je nutné je nepřesouvat. Skript pak načte konfiguraci modelu a spustí trénování a validaci modelu. Průběh trénování lze sledovat v konzoli. Výsledky validace jsou pak také vypsány do konzole. Validace se provádí nad všemi ověřovacími datasety; jako výsledek je do konzole pro každý dataset vypsána matice záměn a spočtené metriky.

Skript <code>train_created_model.py</code> je konfigurován tak, aby automaticky pracoval s daty, vytvořenými pomocí skriptu <code>create_dataset.py</code> - jediné, co očekává, je existence složek <code>train_created</code> a <code>test_created</code> s daty v nich. Skript <code>train_created_model.py</code> používá konfigurace ze složky <code>model_configurations</code>.

Složka <code>final_models</code> obsahuje finální modely (celkem pět modelů) - nejlepší testovaný model je pojmenován <code>model-best</code>. K validaci výsledků těchto modelů slouží skript <code>validate_final_model.py</code>. Ten vyžaduje jeden argument, a to název modelu ze složky <code>final_models</code>. Skript pak provede validaci vybraného modelu nad všemi datasety a pro kontrolní dataset navíc provede validaci v kombinaci s klasifikací pomocí klíčových slov.

