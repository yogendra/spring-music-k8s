# Deploy on Kubernetes

## (Optional) Build and Publish Docker Image

1.  Create the image for spring music application

    ```
    docker build -f Dockerfile.no-tools -t docker.io/yogendra/spring-music-k8s:latest .
    ```

1.  Pubish the image

    ```
    docker push docker.io/yogendra/spring-music-k8s:latest
    ```

## Deploy with Local Postgres

Just deploy everything from the `kubernetes` folder:

```
kubectl apply -f kubernetes
```

### Clean up

Delete all the resources

```
kubectl delete -f kubernetes
```

### Troubleshooting

Postgres may not run due to a missing storage class. Find out the storage class on your cluster

```
kubectl get sc
```

And updated the `kubernetes/04-portgres.yaml` (line 57 approx). Uncomment the line (remove `#`) and change value as per your cluster's available storage class.

```
...
        storageClassName: gp2
...
```

In case postgres pod is not coming up, you may need to change the storage class on the


## Deploy with Remote Greenplum

1.  Create namespace

    ```
    kubectl create namespace spring-music
    ```

1.  Create secret for database username and password. Replace USER and PASSWORD in the following command and execute.

    ```
    create secret generic \
      datasource-secret \
      -n spring-music  \
      --type=string \
      --from-literal "spring_datasource_username=<USER>" \
      --from-literal "spring_datasource_password=<PASSWORD>"
    ```

1.  Create application config with connection string. Replace GPDB_HOST and GPDB_DB in following and execute.

    ```
    kubectl create configmap \
      app-config \
      --from-literal=spring_profiles_active=postgres \
      --from-literal=spring_datasource_url=jdbc:postgresql://<GPDB_HOST>/<GPDDB_DB>
    ```

1.  Deploy application

    **(Optional)**: If you have built your own image, edit the file and update the image reference

    ```
    kubectl create -f kubernetes/05-spring-music.yaml
    ```
    Or deploy directly from this repo

    ```
    kubectl create -f https://raw.githubusercontent.com/yogendra/spring-music-k8s/master/kubernetes/05-spring-music.yaml
    ```


## Troubleshooting

### Docker hub image pull fails due to rate limits

This is a problem because Docker hub has a limit on the number of pull for image by anonymous users. But you can very easily fix it for your cluster very easily.

1.  Create a dockerhub secret

    ```
    kubectl create secret docker-registry \
      dockerhub \
      --docker-username=<your-name> \
      --docker-password=<your-pword> \
      --docker-email=<your-email>
    ```

1.  Patch the default service account in spring-music namespace, to use this secret

    ```
    kubectl patch serviceaccount -n spring-music default -p '{"imagePullSecrets": [{"name": "dockerhub"}]}'
    ```
You can use the same procedure for a private repository.






