.. _fiona:

Knihovna Fiona
==============

`Knihovna Fiona <http://toblerity.org/fiona/>`_ jako nadstavba nad
knihovnou *OGR* (součást knihovny `GDAL <http://gdal.org>`__)
představuje objektově orientovaný způsob práce s vektorovými
daty. Načtená vektorová data knihovna přemapovává do datové struktury
:wikipedia-en:`GeoJSON` a stejné struktury zapisuje zpět do výstupních
souborů. Uživatel se nemusí zabývat kurzory, vrstvami, geometrickými
operacemi a dalšími *odbornými* termíny.

Fiona **není** nástroj vhodný na **všechny** operace - jednoduchost práce
je vykoupena poněkud pomalejším během kódu a omezením velikosti
zpracovávaných dat.  Tam kde OGR používá pointery, Fiona zkopíruje
vektorová data do objektů jazyka Python, což samozřejmě vede k
intenzivnějšímu využívání paměti. Pokud potřebujete filtrovat
geoprvky, knihovna OGR je asi vhodnější. Pokud potřebujete zpracovat
postupně všechny geoprvky ve vrstvě, měla by být rychlejší Fiona.

.. note::
   
   Pro práci s daty ve formátu (Geo)JSON můžete použít knihovnu jazyka
   Python ``json``. Pokud budete pracovat s daty uloženými v
   databázích je vhodnější použít jejich vlastní knihovny.

Datovou vrstvu otevřeme pomocí funkce ``open()``, ta vytvoří objekt
tvz. *kolekce* geoprvků:

.. literalinclude:: ../../_static/skripty/fiona-example.py
   :language: python
   :lines: 1-2
                      
.. code-block:: python

    >>> chko
    <open Collection 'chko.shp:chko', mode 'r' at 0x7feea9595410>

Následně můžeme zjišťovat vlastnosti této kolekce geoprvků, viz
`dokumentace knihovny <http://toblerity.org/fiona/manual.html>`_.

.. code-block:: python

    >>> # driver
    >>> chko.driver
    u'ESRI Shapefile'

    >>> # souřadnicový systém
    >>> chko.crs
        {u'lon_0': 24.83333333333333, u'k': 0.9999, u'ellps': u'bessel', u'y_0': 0, u'no_defs': True,
         u'proj': u'krovak', u'x_0': 0, u'units': u'm', u'alpha': 30.28813972222222, u'lat_0': 49.5}

    >>> # jméno souboru
    >>> chko.path
    u'data/chko.shp'

    >>> # jméno vrstvy
    >>> chko.name
    u'chko'

    >>> # hraniční souřadnice
    >>> chko.bounds
    (-891817.1765, -1209945.3889999986, -440108.9158999994, -943075.1875)

    >>> # všechna metadata pohromadě
    >>> chko.meta
    >>> ...
    >>> ...
    >>> import json # naformátovat výstup
    >>> print(json.dumps(chko.meta, sort_keys=True, indent=4, separators=(',', ': ')))
    {
        "crs": {
            "alpha": 30.28813972222222,
            "ellps": "bessel",
            "k": 0.9999,
            "lat_0": 49.5,
            "lon_0": 24.83333333333333,
            "no_defs": true,
            "proj": "krovak",
            "units": "m",
            "x_0": 0,
            "y_0": 0
        },
        "driver": "ESRI Shapefile",
        "schema": {
            "geometry": "Polygon",
            "properties": {
                "IUCN": "str:2",
                "KAT": "str:4",
                "KOD": "int:10",
                "NAZEV": "str:27",
                "OBJECTID": "int:10",
                "OP_TYP": "str:3",
                "PREKRYV": "int:10",
                "ROZL": "float:24.15",
                "SHAPE.AREA": "float:24.15",
                "SHAPE.LEN": "float:24.15",
                "ZMENA_G": "int:10",
                "ZMENA_T": "int:10",
                "ZONA": "str:3",
                "gml_id": "str:80"
            }
        }
    }


Prvky uložené v kolekci můžeme standardním postupem iterovat a
zpracovávat je prvek po prvku. Nejprve ale zjistíme jejich počet:

.. code-block:: python

    >>> len(chko)
    5626

Souřadnicové systémy
--------------------

Na pozadí Fiony se používají nástroje knihovny `GDAL
<http://www.gdal.org>`_, proto ani práce se souřadnicovými systémy
není o tolik zjednodušena, jak by možná bylo potřeba. Pokud je
souřadnicový systém datové vrstvy definován pomocí kódu EPSG, je tento
kód dále využit, v našem případě se jedná o :epsg:`4326`.

.. literalinclude:: ../../_static/skripty/fiona-example.py
   :language: python
   :lines: 4-5

.. code-block:: python

    +init=epsg:4326

Při vytvoření nového geoprvku s definicí souřadnicového systému je postupováno
analogicky (zde S-JTSK, :epsg:`5514`):

.. literalinclude:: ../../_static/skripty/fiona-example.py
   :language: python
   :lines: 7-8

.. code-block:: python

    {'init': 'epsg:5514', 'no_defs': True}

Fiona těmito funkcemi pouze mapuje jednotlivé parametry souřadnicového
systému a stará se o jejich převod do textového řetězce a z textových
řetězců.


Procházení geoprvků
-------------------

Prvky v datovém souboru můžeme procházet postupně (sekvenčně):

.. literalinclude:: ../../_static/skripty/fiona-example.py
   :language: python
   :lines: 10-11

anebo si vybrat některý z geoprvků (náhodný přístup) a dále s ním
pracovat:

.. literalinclude:: ../../_static/skripty/fiona-example.py
   :language: python
   :lines: 13


Práce s daty
------------

Geometrie geoprvků a knihovna Shapely
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Knihovna `Shapely <http://toblerity.org/shapely/>`_ (stejného autora
jako Fiony) nám umožňuje pracovat s geometrickou složkou popisu
geoprvků opět ve stylu jazyka Python. Stejně jako Fiona, převádí
shapely geometrické vlastnosti na objekty typu JSON.

.. literalinclude:: ../../_static/skripty/fiona-example.py
   :language: python
   :lines: 15-19

.. code-block:: python

    (-683329.1875, -993228.75, -681265.625, -991528.0)

Shapely obsahuje i některé funkce pro modifikaci geometrií, například
generalizaci, obalovou zónu (buffer) nebo porovnání dvou geometrií.

.. literalinclude:: ../../_static/skripty/fiona-example.py
   :language: python
   :lines: 20-21

.. code-block:: python

    True

.. literalinclude:: ../../_static/skripty/fiona-example.py
   :language: python
   :lines: 23-24

.. code-block:: python
           
    True

Můžeme změnit některé vlastnosti geoprvků, např. upravit atribut `NAZEV`:

.. literalinclude:: ../../_static/skripty/fiona-example.py
   :language: python
   :lines: 26-33

.. code-block:: python

    5626            

.. literalinclude:: ../../_static/skripty/fiona-example.py
   :language: python
   :lines: 35-36

.. code-block:: python
                
    5627

Soubor uzavřeme:

.. literalinclude:: ../../_static/skripty/fiona-example.py
   :language: python
   :lines: 38

Načtení dat z webové služby
---------------------------

Pro načítání dat z webových služeb není Fiona zrovna stavěná. Nejlepší způsob je
uložení do dočasného souboru na disk a následném načtení. Pokud jsme v situaci,
kdy tuto možnost nemáme, můžeme zkusit vyrobit virtuální objekt typu soubor a
ten následně použít.

Další informace v kapitole o knihovně :doc:`OWSLib
<../../owslib/index>`.

..
   .. todo:: Opravit - nefunkční
          
   .. code-block:: python

    [...]
    >>> f = aopk.getfeatures(['UzemniOchrana_ChranUzemi:Zonace_velkoplošného_zvláště_chráněného_území'])

    Špinavý trik - načtení geoprvku pomocí `gdal.FileFromMemBuffer` objektu:

    .. code-block:: python

    >>> from osgeo import gdal
    >>> gdal.FileFromMemBuffer('/vsimem/temp', f.read())
    >>> ...
    >>> # malý trik
    >>> from fiona.collection import supported_drivers
    >>> supported_drivers['GML'] = 'r'
    >>> ...
    >>> # a čteme
    >>> c = fiona.open('/vsimem/temp', 'r')
    >>> ...
    >>> # počet geoprvků
    >>> len(c)
    3571
