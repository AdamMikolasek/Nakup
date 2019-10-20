## Určovanie sentimentu filmových recenzií

Adam Mikolašek, Veronika Žatková

Projekt z predmetu NSiete

Študijný program: Inteligentné softvérové systémy

Ak. rok: 2019/2020, zimný semester

Cvičiaci: Ing. Matúš Pikuliak

## Klasifikácia sentimentu vo filmových recenziách



#### 1. Motivácia

Veľké množstvo ľudí sa pri výbere filmov riadi práve názormi a recenziami iných. Určenie sentimentu sa neobmedzuje iba na recenzie, ale je využiteľné pri drvivej väčšine rečových prejavov.

Schopnosť určiť sentiment recenzií nemusí byť len finálnym produktom, ale aj základom pre ďalšie analýzy. Pokiaľ vieme určiť sentiment recenzií filmu, agregáciou sentimentov identifikovaných pri filme vieme odhadnúť celkový názor verejnosti na daný film a využívať ho ako alternatívny zdroj hodnotenia filmov. Určenie negatívneho sentimentu je zároveň jedným z prvých krokov pre pomoc pri identifikovaní úmyselne nenávistných komentárov.

#### 2. Súvisiace práce

Dataset, ktorý sme pre danú problematiku našli, bol už využitý v inej práci1. Cieľom tejto práce bolo určovanie sentimentu a sémantiky textu, a využitie týchto informácií na klasifikáciu komentárov do dvoch skupín a to podľa toho či komentár patril k pozitívnemu alebo negatívnemu hodnoteniu. Výsledky navrhnutého modelu boli testované na dvoch datasetoch, &quot;Pang and Lee Movie Review Dataset&quot; a &quot;Large Movie Review Dataset&quot;, pričom druhý z týchto datasetov je bližšie opísaný v kapitole 3. Výsledky, ktoré navrhnutý model dosiahol boli porovnané s rôznymi inými technikami na spracovanie prirodzeného jazyka, ako sú Bag-of-words, Latent semantic analysis (LSA) a Latent Drichlet allocation (LDA). Navrhnutý model dosiahol najlepšie výsledky práve v spojení s technikou Bag-of-Words a to 88.89, teda v 88.89% prípadoch kategorizoval komentár správne.







#### 3. Dataset

V rámci nášho projektu sme sa rozhodli využiť dataset filmových recenzií &quot;Large Movie Review Dataset v1.0&quot;2, publikovaný univerzitou Stanford. Dataset pozostáva z 50 000 výrazne polárnych filmových recenzií, mapovateľných na URL adresu filmu, ku ktorému patria (URL sú súčasťou datasetu). Recenzie sú oanotované - každá recenzia má priradenú hodnotu &quot;pozitívna&quot; alebo &quot;negatívna&quot; na základe jej sentimentu. Celková početnosť týchto dvoch tried je vyvážená - 25 000 rezencií je pozitývnych a 25 000 negatívnych. Výrazná polarita recenzií znamená, že negatívne recenzie patria k hodnoteniam \&lt;= 4 z 10 a pozitívne recenzie k hodnoteniam \&gt;= 7 z 10.

Maximálny počet recenzií pre jeden film je 30 (pre predídenie predpovedania rovnakých hodnôt sentimentu pre jeden film). Dataset je taktiež rozdelený na trénovaciu a testovaciu množinu v rovnakom pomere (25 000 : 25 000), pričom platí, že žiaden z filmov v trénovacej množine sa nenachádza v testovacej množine. K recenziám je taktiež dostupné ich pôvodné hodnotenie vo formáte počtu bodov (maximum je 10).

Súčasťou datasetu je aj ďalších 50 000 recenzií filmov, ktoré sú však neanotované (napríklad pre využitie v rámci metód učenia bez učiteľa). Jedná sa najmä o neutrálne recenzie.

#### 4. Návrh riešenia

Naše riešenie pozostáva z viacerých krokov:

1. Exploratívna analýza dát.
2. Predspracovanie dát - v našom prípade výrazne využívajúce prístupy spracovania prirodzeného jazyka
3. Vytvorenie reprezentácie dát vhodné pre vstup do neurónovej siete (embedding) vrátane samotného výberu vhodného embeddingu.
4. Návrh architektúry a implementácia neurónovej siete, trénovanie a validácia.
5. Vyhodnotenie výsledkov.





1

#
Andrew L. Maas, Raymond E. Daly, Peter T. Pham, Dan Huang, Andrew Y. Ng, and Christopher Potts. (2011). [Learning Word Vectors for Sentiment Analysis.](http://ai.stanford.edu/~amaas/papers/wvSent_acl2011.pdf)  _The 49th Annual Meeting of the Association for Computational Linguistics (ACL 2011)._

2

#
[http://ai.stanford.edu/~amaas/data/sentiment/](http://ai.stanford.edu/~amaas/data/sentiment/)