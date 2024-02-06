#Lancer le Postgres
docker run --name mon-postgres-instance -p 5432:5432 -d -v mon-volume-postgres:/var/lib/postgresql/data mon-postgres

#Lancer Backend-API
docker build -t mon-backend-api .
docker run --name mon-backend-api-instance --network mon-reseau -d mon-backend-api

mon-postgres-instance
usr
pwd