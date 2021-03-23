# iGo

No perdeu més el temps: iGo us ensenya el camí més ràpid! 🐰🚘


## Introducció

Aquesta pàgina descriu el projecte iGo, que correspon a la segona pràctica del
curs 2021 d'AP2 al GCED.

La vostra tasca consisteix en implementar en Python
un Bot de Telegram que permeti guiar als seus usuaris fins a la seva
destinació pel camí més ràpid en cotxe, utilitzant el novedós concepte d'*ispeed*
(velocitat intel·ligent) que té en compte l'estat del trànsit en temps real a
certs trams de la ciutat de Barcelona.

Per fer aquesta pràctica, haureu d'obtenir dades de diferents proveïdors
i enllaçar-les conjuntament. En particular, utilitzareu:

- El mapa de Barcelona d'[Open Street Map](https://www.openstreetmap.org).

- La [informació sobre l'estat del trànsit als trams](https://opendata-ajuntament.barcelona.cat/data/ca/dataset/trams)  de la ciutat de Barcelona.

- La [relació de trams de la via pública](https://opendata-ajuntament.barcelona.cat/data/ca/dataset/transit-relacio-trams)  de la ciutat de Barcelona.

Afortunadament, existeixen algunes llibreries ben documentades que us facilitaran molt la feina:

- Per utilitzar els Open Street Maps utilitzareu la llibreria [OSMnx](https://geoffboeing.com/2016/11/osmnx-python-street-networks/), que funciona
damunt de [NetworkX](https://networkx.github.io/documentation/stable/tutorial.html).

- Per llegir les dades del Servei de dades obertes de l'Ajuntament de Barcelona,
haureu de llegir fitxers en format `csv`, tot utilitzant el mòdul estàndard de Python
[csv](https://docs.python.org/3/library/csv.html).

- Els mapes que mostrareu els generareu amb [`staticmap`](https://github.com/komoot/staticmap), una llibreria de
Python que es connecta a Open Street Maps per generar mapes amb línies i marcadors.
Teniu informació sobre l'ús d'aquest mòdul en
[aquesta lliçó](https://lliçons.jutge.org/python/fitxers-i-formats.html)
(hi ha altres coses, centreu-vos en la darrera secció).

- Per escriure el Bot de Telegram podeu utilitzar
[aquesta lliçó](https://lliçons.jutge.org/python/telegram.html).




## Arquitectura del sistema

Els sistema consta dels mòduls següents:

- `igo.py` conté tot el codi i estrictures de dades relacionats amb
l'adquisició i l'enmagatzematge de grafs corresponents a mapes, congestions i
càlculs de rutes.  Aquest mòdul no en sap res del bot.

- `bot.py` conté tot el codi relacionat amb el bot de Telegram i utilitza el
  mòdul `igo`. La seva tasca és reaccionar a les comandes dels usuaris per poder-los guiar.

La raó per separar el bot de la resta de funcionalitats és poder adaptar
fàcilment el projecte a noves plataformes en el futur (web, whatsapp…), quan aquest triomfi i
tots plegats ens fem rics! 🤑



## Funcionalitat del mòdul `igo`

Heu de dissenyar el mòdul `igo`  per tal contingui tot el codi relacionat amb
l'adquisició, l'enmagatzematge i la consulta dels grafs de carrers de la
ciutat, dels seus trams, i del trànsit en aquests trams. A més, aquest mòdul
ha de ser capaç de calcular camins mínims entre parells de punts a la ciutat,
tot utilitzant el concepte de *ispeed* que més tard es concreta. Aquest mòdul
també ha de ser capaç de generar imatges amb els camins mínims que s'han
calculat.


## Funcionalitat del mòdul `bot`

El bot de Telegram ha de donar suport a les comandes següents:

- `/start`: inicia la conversa.
- `/help`: ofereix ajuda sobre les comandes disponibles.
- `/author`: mostra el nom dels autors del projecte.
- `/go destí`: mostra a l'usuari un mapa per arribar de la seva posició actual fins al punt de destí escollit
    pel camí més curt segons el concetpe de *ispeed*.
   Exemples:
    - `/go Campus Nord`
    - `/go Sagrada Família`
    - `/go Pla de Palau, 18` (Facultat de Nàutica)
- `/where`: mostra la posició actual de l'usuari.

També hi ha una comanda *secreta* que permet falsejar la posició actual de l'usuari,
de forma que es puguin fer proves facilment sense haver de sortir de casa:

- `/pos`: fixa la posició actual de l'usuari a una posició falsa.
   Exemples:
    - `/pos Campus Nord`
    - `/pos 41.38248 2.18511` (Facultat de Nàutica)


El bot hauria de començar carregant les dades necessàries. A partir d'aquell
moment esperarà connexions de diferents usuaris i els ajudarà a arribar a la
seva destinació tot mostrant la ruta òptima per anar des de la seva posició
actual (real o falsejada amb `/pos`) fins al seu destí. Quan sigui necessari,
el Bot recarregarà les dades sobre la congestió dels trams, per tal
d'adaptar-se al trànsit en temps real.

Totes les comandes han de funcionar per a diferents usuaris alhora.


## *ispeed:* La velocitat intel·ligent

Per tal de predir el camí més ràpid entre dos punts, iGo utilitza un concepte de
velocitat intel·ligent anomenat *ispeed*. Aquest concepte apareix de la integració de
diferents dades disponibles a les dades obertes que el projecte utilitza:

- Velocitat de cada carrer al graf d'OSMnx: Cada aresta del graf de la ciutat té
un atribut `speed` que indica la velocitat màxima en aquella aresta i un atribut
`length` que indica la seva llargada. Per tant, és fàcil deduir el temps necessari per
recórrer aquella aresta en condicions de circulació òptima.

- La relació de trams de la via pública de la ciutat de Barcelona
defineix un conjunt de trams amb les artèries de circulació més importants de la ciutat.
Cada tram ve definit per una seqüència de segments, dels quals es dónen les coordenades
dels seus extrems.

- La informació sobre l'estat del trànsit als trams de la ciutat de Barcelona
dóna (en temps real) la congestió existent a cadascun dels trams. Aquest estat
pot ser: sense dades, molt fluid, fluid, dens, molt dens, congestió, tallat.

Malauradament, la informació dels trams que ens proporciona l'ajuntament no quadra
completament amb els carrers d'OSM... I tampoc hi ha informació de la congestió
per a tots els carrers de la ciutat, només per a alguns.

Per tant, cal trobar alguna forma per transportar les dades de congestió dels
trams als carrers d'OSMnx.  Aquesta propagació de les congestions s'hauria
d'acabar materialitzant en un nou atribut `ispeed` a cada aresta del graf,
sobre el qual es calcularan els camins mínims.

Una forma de fer-ho és relacionant cada aresta del mapa amb al segment de tram més
pròxim i propagant-ne la congestió (a no ser que el segment més pròxim sigui
massa llunyà). Això es pot fer de forma senzilla considerant que cada aresta
del mapa i cada segment de tram es poden idealitzar a través del seu punt
mig, i llavors reduim el problema a trobar la distància del punt mig de cada
aresta al punt mig d'un segment de tram.

Hi ha moltes altres maneres raonables de calcular l'atribut  `ispeed` de les
arestes del graf. Sou lliures de fer-ho, a condició de deixar ben documentat
com ho feu i de raonar quins en són els seus avantatges i desavantatges.

A més, opcionalment, també podeu afegir un retard a cada cruïlla de carrers.
Aquest retard hauria de ser petit si la cruïlla s'atravessa del dret (gir de
menys de 15 graus, diguem) i hauria de ser més gran si cal girar a la cruïlla
(gir de més de 15 graus). A més, també ho podeu millorat tenint en compte que
girar cap a l'esquerra sol ser més lent que girar cap a la dreta. El grafs
d'OSMnx tenen atributs amb l'angle de les arestes (*bearings*) que us poden
ser útils per això.

Evidentment, cada implementació de la *ispeed* tindrà uns paràmetres associats
que vosaltres haureu de definir de forma adient (amb ús de constants, per
exemple).



## Llibreries

Utilitzeu les llibreries de Python següents:

- `csv` per llegir en format CSV.
- `pickle` per llegir/escriure dades en Python de/en fitxers.
- `networkx` per a manipular grafs.
- `osmnx` per a obtenir grafs de llocs.
- `urlib` per descarregar fitxers des de la web.
- `haversine` per a calcular distàncies entre coordenades.
- `staticmap` per pintar mapes.
- `python-telegram-bot` per interactuar amb Telegram.

Totes aquesta llibreries es poden instal·lar amb `pip3 install`, però `osmnx`
porta una mica de feina... En Ubuntu, cal primer fer un `apt install
libspatialindex-dev`. En Mac, cal fer un `brew install spatialindex gdal` i, a
més, posar les darreres versions dels instal·ladors:

1. `pip install --upgrade pip setuptools wheel`
2. `pip install --upgrade osmnx`
3. `pip install --upgrade staticmap`

Podeu utilitzar lliurament altres llibreries estàndards de Python, però si no
són estàndards, heu de demanar permís als vostres professors (que segurament no
us el donaran).


## Indicacions per treballar amb els grafs d'OSMnx

Els grafs d'OSMnx tenen molta informació i triguen molt a carregar. Per aquest
aplicació, demaneu-los per a cotxe i simplificats. A més, descarregeu-los el
primer cop i deseu-los amb Pickle:

```Python
graph = osmnx.graph_from_place("Barcelona, Catalonia", network_type='drive', simplify=True)
with open('barcelona.graph', 'wb') as file:
    pickle.dump(graph, file)
```

A partir d'aquest moment els podreu carregar des del fitxer enlloc de des de la xarxa:


```python
with open('barcelona.graph', 'rb') as file:
    graph = pickle.load(file)
```

Aquesta és la manera de recórrer tots els nodes i les arestes d'un graf:

```python
# for each node and its information...
for node1, info1 in graph.nodes.items():
    print(node1, info1)
    # for each adjacent node and its information...
    for node2, info2 in graph.adj[node1].items():
        print('    ', node2)
        # osmnx graphs are multigraphs, but we will just consider their first edge
        edge = info2[0]
        print('        ', edge)
```

De forma molt infreqüent, els grafs d'OSMnx tenen multi-arestes. El codi
anterior les ignora tot quedant-se amb la primera aresta; feu el mateix.
Compte: a vegades hi ha sorpreses: carrers amb més d'un nom,
valors absents o nuls...



# Instruccions

## Equips

Podeu fer aquest projecte sols o en equips de dos. En cas de fer-lo en equip,
la càrrega de treball dels dos membres de l'equip ha de ser semblant i el
resultat final és responsabilitat d'ambdós. Cada membre de l'equip ha de saber
què ha fet l'altre membre.

Els qui decidiu fer el segon projecte en un equip de dos estudiants, envieu
abans de les **23:59 del dia XYZ** un missatge al professor Jordi Petit
amb aquestes característiques:

- des del compte `@estudiantat.upc.edu` d'un dels dos membres,
- amb tema (subject) `Equips AP2 2021`,
- amb el nom dels dos estudiants de l'equip al cos del missatge,
- fent còpia (CC) al compte `@estudiantat.upc.edu` de l'altre estudiant.

Si no es reb cap missatge d'equip per aquesta data, es considerarà que feu la
pràctica sols. Si heu enviat aquest missatge, es considerarà que feu la
pràctica junts (i no s'admetràn "divorcis").


## Lliurament

Heu de lliurar la vostra pràctica al Racó. Si heu fet la pràctica en equip,
només el membre més jove ha de fer el lliurament. El termini de lliurament és
el **XYZ**.

Només heu de lliurar un fitxer ZIP que, al descomprimir-se,
generi els fitxers següents:

- `igo.py`,
- `bot.py`,
- `requirements.txt`,
- `README.md` i
- `*.png` si cal adjuntar imatges a la documentació.

Res més. Sense directoris ni subdirectoris.

Els vostres fitxers de codi en Python han de seguir
[les regles d'estíl PEP8](https://www.python.org/dev/peps/pep-0008/). Podeu
utilitzar el paquet `pep8` o http://pep8online.com/ per assegurar-vos
que seguiu aquestes regles d'estíl.
L'ús de tabuladors en el codi queda
prohibit (zero directe). Si voleu, podeu fer ratlles més llargues que les que dicta PEP8.

El projecte ha de contenir un fitxer `README.md`
que el documenti. Vegeu, per exemple, https://gist.github.com/PurpleBooth/109311bb0361f32d87a2.
Si us calen imatges al `README.md`, deseu-los com a fitxers PNG.

El projecte també ha de contenir un fitxer `requirements.txt`
amb les llibreries que utilitza el vostre projecte per poder-lo instal·lar.
Vegeu, per exemple, https://pip.pypa.io/en/stable/user_guide/#requirements-files.



## Consells

- Us suggerim seguir aquests passos per a dur a terme el vostre projecte:

    1. Seguiu el [tutorial de networkx](https://networkx.github.io/documentation/stable/tutorial.html).
    1. Seguiu el [tutorial de osmnx](https://geoffboeing.com/2016/11/osmnx-python-street-networks/).
    1. Estudieu el format de la relació de [trams de la via pública](https://opendata-ajuntament.barcelona.cat/data/ca/dataset/trams) de la ciutat de Barcelona.
    1. Estudieu la informació sobre [l'estat del trànsit als trams](https://opendata-ajuntament.barcelona.cat/data/ca/dataset/trams)  de la ciutat de Barcelona.
    1. Seguiu el [tutorial de staticmaps](https://lliçons.jutge.org/python/fitxers-i-formats.html).
    1. Proveu de generar una imatge dels trams de Barcelona, fent que el seu color mostri la seva congestió.
    1. Proveu de generar un camí mínim entre dos punts de Barcelona (sense usar ispeed) i pinteu-lo en un mapa.
    1. Penseu com implementareu el concepte de *ispeed*.
    1. Dissenyeu el mòdul `igo` tot definint els seus tipus de dades i les capçaleres de les seves funcions públiques.
    1. Implementeu el mòdul `igo` i provant les funcions a mesura que les escriviu.
    1. Seguiu el [tutorial de telegram](https://lliçons.jutge.org/python/telegram.html).
    1. Implementeu el mòdul `bot` i proveu-lo.

-  Documenteu el codi a mesura que l'escriviu.

- L'enunciat deixa obertes moltes qüestions expressament. Sou els responsables de prendre
  les vostres decisions de disseny i deixar-les reflectides adientment al codi i
  a la documentació.

- Podeu ampliar les capacitats del vostre projecte mentre manteniu les
  funcionalitats mínimes previstes en aquest enunciat. Ara bé, aviseu abans als
  vostres professors i deixeu-ho tot ben documentat.

- Per evitar problemes de còpies, no pengeu el vostre projecte en repositoris
  públics.


## Autors

Jordi Cortadella i Jordi Petit

Universitat Politècnica de Catalunya, 2021

