#Lancer le Postgres
docker run --name mon-postgres-instance -p 5432:5432 -d -v mon-volume-postgres:/var/lib/postgresql/data mon-postgres

#Lancer Backend-API
docker build -t mon-backend-api .
docker run --name mon-backend-api-instance --network mon-reseau -d mon-backend-api

mon-postgres-instance
usr
pwd

matiascpe
MatiasCPE_TP_Github_Actions

Token_sonar : 6a537f7f08ad3dde97cb03d46d98c024d90f4d9e