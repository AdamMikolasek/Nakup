# Projekt

*Autor: Adam Mikolášek*

*Cvičenie: Piatok 12:00*

## Big Picture
Aplikácia slúži na vyhľadávanie podnikov v Bratislave. Ide konkrétne o kaviarne, reštaurácie a krčmy. Aplikácia je teda určená pre ľudí, ktorí Bratislavu nepoznajú alebo navštevujú a potrebujú si íst niekam sadnúť alebo najesť sa. 

Prvý scenár slúži na vyhľadanie všetkých podnikov v rámci Bratislavskej mestskej časti. Podniky sú zobrazené na mape s ich názvom a označením o aký podnik ide.

Druhý scenár slúži na nájdenie najkratšej cesty k podniku. Vlastnú pozíciu treba zadať kliknutím na mapu a názov podniku treba napísať. Tento scenár som implementoval sám pomocou rekurzívneho query (ktoré bolo dosť dlhé a náročné), takže je obmedzené na kratšiu vzdialenosť. 

## Strucny opis architektury
Projekt sa skladá z front-end a back-end časti.

Backend je reprezentovaný serverom (subor server.js), ktorý slúži primárne na komunikaciu s databázou (query dopyty) a na posielanie vysledkov na klienta.

Frontend je implementovany v subore index.html (viem ze je to nestastne riesenie a malo by to byt rozdelene aspon na index, script a css ale v ramci projektu som to vyriesil iba takto). Slúži na zobrazovanie dát používateľovi a prijímanie vstupov od používateľa.


## Getting started
Databáza bola importovaná pomocou osm2pgsql a obsahuje data o Bratislave a blízkom okolí. Data boli importované pomocou návodu v demo projekte, takže to tu už nebudem opisovať.

Ďalej treba mať nainštalovaný node.js, pomocou ktorého treba spustiť server (vo windowse je to cmd node server.js v adresáry, kde sa server.js nachádza). Server beží na localhoste porte 3000 a je pripojený na databázu, takže treba pozrieť tieto údaje (moja databáza sa volá gis, port je 5432 a heslo nie je žiadne).

Nasledne je možné klienta spustiť kliknutím na index.html, čo by malo otvoriť browser, kde už sa dá s aplikáciou reálne pracovať.


## Použité extensions
Mapbox na zobrazovanie geografických dát
Axios na posielanie http requestov na server
Samozrejme postgis extension nad databázou 


## Optimalizácie
Prvou optimalizáciou boli použité indexy. Nad atribútom `way` boli indexy vytvorené automaticky vo všetkých troch tabuľkách, pričom som doplnil indexy na `name` a `osm_id`  a `z_order` v tabuľke `planet_osm_line` a nad `name` a `amenity` v tabuľke `planet_osm_point`. Rýchlosť nebola zvýšená vôbec výrazne (približne o 10-20%), avšak keďže do databázy žiadne nové dáta neukladajú, nevidel som dôvod nepoužiť tieto indexy. 
```sql
create index index_amenity on planet_osm_point (amenity)
create index index_name_point on planet_osm_point (name)
create index index_name_line on planet_osm_line (name)
create index index_z_order on planet_osm_line (z_order)
create index index_osm_id on planet_osm_line (osm_id)
```

Druhou optimalizáciou je, v rámci rekurzívneho query, pracovanie s id-čkami miesto názvov ulíc. Toto je viac fix funkcionality ako optimalizácia, keďže je to korektnejšie riešenie. Jeden názov ulice môže mať viacero rôznych čiar v tabuľke, ktoré väčšinou predstavujú časť celej ulice, preto pri join-ovaní pomocou názvov bola výsledná výpočtová zložitosť omnoho horšia. Čo som skúšal, tak nájdené riešenie bolo vždy rovnaké, ale pri pracovaní s názvami by som nevedel zaručiť, či ide o najkratšiu cestu alebo sa pospájali ulice, ktoré sa reálne nepretínajú.


## Query
Keďže query použité v projekte sú pomerne dlhé, sú priložené vo vlastnom súbore `query.md`. V tomto súbore je len zoznam použítých query, s krátkym opisom ich funkcionality.
1. Vyhľadanie všetkých podnikov v mestskej časti Bratislavy.
2. Nájdenie najbližšej ulice k pozícii používateľa.
3. Nájdenie najbližšej ulice k pozícii podniku.
4. Rekurízvne query na nájdenie najkratšej cesty od používateľa k podniku.