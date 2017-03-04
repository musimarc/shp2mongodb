##Conversion des données shapefile en GeoJSON
docker run -v $(pwd):/data geodata/gdal ogr2ogr -f geoJSON parking.json parcs-de-stationnement-concedes-de-la-ville-de-paris.shp

##Formatter le GeoJSON pour insertion dans MongoDB
Supprimer ces deux premières lignes du fichier:
    {"type": "FeatureCollection",           
               "features": [ 

Ainsi que les deux derniières:
    ]
    }

Remplacer les virgules à la fin de chaque ligne (ici avec Vim)
    :%s/] } },/ ] } }

Les données sont désormais prêtes à être importées dans MongoDB

##Import des données dans MongoDB
#Lancement de la base MongoDB
docker run --name mongo-json -v $(pwd):/data -d mongo
docker exec -it mongo-json bash

#Import des données
mongoimport --drop --db geodatatest --collection parking < /data/parking.json

#Création de l'index (à partir du mongo-shell)
use geodatatest
db.parking.createIndex({"geometry" : "2dsphere"})

#Vérifier les index
db.parking.getIndexes() 
