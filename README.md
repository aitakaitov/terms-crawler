# Aplikace pro automatické stahování smluvních podmínek z webových stránek
Aplikace se skládá ze dvou částí - crawleru a neuronových sítí. Crawler stahuje stránky, které obsahují informace o ochraně osobních údajů nebo podmínky užití na základě klíčových slov. Při použití klíčových slov očekáváme určitou míru falešné pozitivity. Tu se snažíme eliminovat použitím neuronových sítí, které stránky klasifikují jako podmínky užití, informace o ochraně osobních údajů nebo nerelevantní stránky. Pro trénování neuronových sítí je použit přímo výstup crawleru bez manuálního označování.

Bližší informace jsou k dispozici [zde](https://otik.uk.zcu.cz/handle/11025/44238).

## Vývojové prostředí
Aplikace byla testována na operačních systémech Windows 10 PRO 1909 a Ubuntu 20.04.1  
s kernelem 5.8.0-48-generic. V obou případech s Python 3.8.1. Je vhodné mít k dispozici alespoň 16 GB operační paměti, jinak může docházet k pádům aplikace.

## Instalace prerekvizit
### Dodatečné soubory
Finální modely jsou k dispozici v ZIP archivu [zde](https://drive.google.com/file/d/1tXZuauUrxIDNdb9E1l0TjnnHtXsPCHTw/view?usp=sharing). Archiv také obsahuje konfigurace tokenizeru a konfigurace ostatních modelů.

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

## Neuronové sítě
### Relevantní složky
Pro neuronové sítě jsou relevantní následující složky:
* <code>model_configurations</code> - konfigurace neuronových sítí,
* <code>neural_net</code> - zdrojové kódy pro předzpracování textů, neuronové sítě a validaci modelů,
* <code>final_models</code> - obsahuje finální evaluované TensorFlow modely.
	
### Vytvoření datasetu
Pro vytvoření datasetu a jeho rozdělení na trénovací a testovací části slouží skript <code>create_dataset.py</code>. Trénovací část se uloží do složky <code>train_created</code> a testovací část do složky <code>test_created</code>. Je vytvořen soubor <code>preprocessing-config</code>, který obsahuje data potřebná pro předzpracování.
Skript očekává existenci složky <code>raw_datasets</code>, která obsahuje složku <code>pages</code> (výstup crawleru) a <code>irrelevant</code> (stránky, které nejsou ani podmínky, ani informace o ochraně o. ú.).

### Modely
Složka <code>final_models</code> obsahuje finální TensorFlow modely - nejlepší testovaný model je pojmenován <code>model-best</code>. Tyto modely lze použít pro filtrování stránek stažených crawlerem.

