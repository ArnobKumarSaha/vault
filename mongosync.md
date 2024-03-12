
## Create source & destination db
```bash
kubectl create secret generic s7-auth --from-literal=username=root --from-literal=password=12345 -n demo
// Apply DB
kubectl port-forward -n demo svc/s7 27018:27017
```

## Make the Destination database empty
```bash
kubectl scale deploy -n kubedb kubedb-kubedb-provisioner --replicas=0
```

```bash
kubectl exec -it -n demo d7-0 -- bash
mongosh -u root -p $MONGO_INITDB_ROOT_PASSWORD
use kubedb-system
db.dropDatabase()
```

## Start syncing

```bash
./Downloads/mongosync-ubuntu2004-x86_64-1.7.1/bin/mongosync --cluster0 "mongodb://root:12345@localhost:27018/?directConnection=true" --cluster1 "mongodb://root:12345@localhost:27019/?directConnection=true"

# Initiate the sync process
curl localhost:27182/api/v1/start -X POST \
    --data '{ "source": "cluster0", "destination": "cluster1" }'
```


## TODO (check using mongo atlas)
./Downloads/mongosync-ubuntu2004-x86_64-1.7.1/bin/mongosync --cluster0 "mongodb+srv://root:<>@praromvik.ixnv2gn.mongodb.net/" --cluster1 "mongodb://root:12345@localhost:27019/?directConnection=true"

