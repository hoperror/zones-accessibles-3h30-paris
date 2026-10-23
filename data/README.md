# Données sources

Les fichiers volumineux ne sont pas versionnés (voir `.gitignore`). Avant d'exécuter le notebook, placer les fichiers
ci-dessous dans ce dossier `data/`, avec exactement ces noms.

| Fichier ou dossier attendu | Source | Versionné |
|---|---|---|
| `gtfs_sncf/` | GTFS « Horaires SNCF », décompressé | non |
| `ADMIN-EXPRESS-COG.gpkg` | IGN, ADMIN EXPRESS COG, couche `commune` | non |
| `CNTR_RG_03M_2020_3035.geojson` | Eurostat GISCO, pays | non |
| `airports.csv` | OurAirports | non |
| `wiki_CDG.html`, `wiki_ORY.html` | copies datées des pages Wikipédia, créées par le notebook | oui |
| `calibrage_avion.csv` | durées de vol relevées à la main | oui |
| `villes_etrangeres.csv` | noms et populations des villes étrangères desservies en train | oui |

## Téléchargements (PowerShell, depuis la racine du dépôt)

**GTFS Horaires SNCF** (licence ODbL, page du jeu : <https://transport.data.gouv.fr/datasets/horaires-sncf>)

```powershell
Invoke-WebRequest -Uri "https://eu.ftp.opendatasoft.com/sncf/plandata/Export_OpenData_SNCF_GTFS_NewTripId.zip" -OutFile "data\sncf_gtfs.zip"
Expand-Archive "data\sncf_gtfs.zip" -DestinationPath "data\gtfs_sncf"
```

Ce flux ne couvre que les mois à venir. Le jour de référence du projet (06/10/2026) n'y figurera plus après cette
date : pour reproduire exactement les résultats, récupérer la version archivée de fin septembre 2026 dans
l'historique des ressources de transport.data.gouv.fr, ou adapter `JOUR` dans le notebook à un mardi couvert par le flux.

**ADMIN EXPRESS COG** (IGN, Licence Ouverte Etalab 2.0) : télécharger l'archive France métropolitaine au format
GeoPackage depuis la page ADMIN EXPRESS de [geoservices.ign.fr](https://geoservices.ign.fr), la décompresser avec
7-Zip, puis copier le fichier `.gpkg` sous le nom `data/ADMIN-EXPRESS-COG.gpkg`.

**Pays GISCO** (© EuroGeographics pour les limites administratives)

```powershell
Invoke-WebRequest -Uri "https://gisco-services.ec.europa.eu/distribution/v2/countries/geojson/CNTR_RG_03M_2020_3035.geojson" -OutFile "data\CNTR_RG_03M_2020_3035.geojson"
```

**OurAirports** (domaine public)

```powershell
Invoke-WebRequest -Uri "https://davidmegginson.github.io/ourairports-data/airports.csv" -OutFile "data\airports.csv"
```

**Pages Wikipédia** (CC BY-SA 4.0) : le notebook les télécharge à la première exécution s'il ne les trouve pas.
Les copies versionnées dans ce dossier sont celles du 8 octobre 2026, utilisées pour les résultats publiés.

## Fichiers saisis à la main

`calibrage_avion.csv` : six vols directs relevés sur Google Flights le 8 octobre 2026
(colonnes `origine`, `destination_iata`, `duree_min`, `source`).

`villes_etrangeres.csv` : une ligne par gare étrangère atteinte en train, en UTF-8
(colonnes `stop_name`, `ville_etr`, `pays`, `pop_etr`, `source_pop`). La colonne `stop_name` doit correspondre
exactement au nom de la gare dans le GTFS. Les populations sont celles de la commune (et non de l'agglomération) ;
celles de Mannheim et Karlsruhe sont arrondies.
