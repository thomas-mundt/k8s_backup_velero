# Backup Kubernetes Clusters With Velero

## Links

- https://velero.io/
- https://min.io/


## Intruduction

Veloro Creates Backups of the K8s Cluster into a AWS S3 Storage.
You can backup/restore the entire cluster, namespaces or only some resources like pods, replicasets etc.
The default ttl of the backups are 30 days.


## Setup


### Install Minio

Create a S3 Storage for Testing with minio
```
docker pull minio/minio
docker run -d --name minio -p 9000:9000 -v data:/data minio/minio server /data

kubectl get nodes -o wide
# In webbrowser
<cluster-ip>:9000
Create a bucket kubedemo in Minio (Plus sign in the right bottom corner)

# Get access and secret key
docker exec -it minio cat /data/.minio.sys/config/config.json | egrep "(access|secret)_key"
>minioadmin
>minioadmin

```

Or with Helm
```
# Link: https://github.com/minio/charts

helm repo add minio https://helm.min.io/
helm install --namespace minio --generate-name minio/minio

helm install my-release minio/minio

# Access and Secret keys
helm install --set accessKey=myaccesskey,secretKey=mysecretkey --generate-name minio/minio
```

### Install Velero

```
wget https://github.com/heptio/velero/releases/download/v1.5.3/velero-v1.5.3-linux-amd64.tar.gz
tar zxf velero-v1.5.3-linux-amd64.tar.gz
sudo mv velero-v1.5.3-linux-amd64/velero /usr/local/bin/
rm -rf velero*
velero version
```


Create credentials file and change aws_access_key_id and aws_secret_access_key
```
cat <<EOF>> minio.credentials
[default]
aws_access_key_id=minioadmin
aws_secret_access_key=minioadmin
EOF
```



Install plugin
```
velero plugin add velero/velero-plugin-for-aws:v1.0.0

velero plugin add velero/velero-plugin-for-aws:v1.1.0
```


Setup Velero in the Kubernetes Cluster
```
velero install \
   --provider aws \
   --bucket kubedemo \
   --secret-file ./minio.credentials \
   --backup-location-config region=minio,s3ForcePathStyle=true,s3Url=http://<IP-ADDRESS>:9000 \
   --plugins velero-plugin-for-aws
```


With Helm
```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
velero install --provider aws --plugins velero-plugin-for-aws:v1.1.0 --bucket kubedemo --secret-file ./minio.credentials
```




## Backup

```
velero backup-location get

# List backups
velero backup get
```



```
kubectl get backups -n velero
kubectl get crds -n velero

# The cluster
velero backup create firstbackup

# A namespace
velero backup create firstbackup --include-namespace testing --exlude-resources pod
velero backup create firstbackup --exlude-namespaces testing
velero backup create firstbackup --exlude-resources configmaps
--exlude-resources
--include-resources
--exlude-namespace
--include-namespace

kubectl get backups -n velero
velero backups get

velero backup describe firstbackup 


# Restore
velero restore get
velero restore create firstbackup-restore1 --from-backup firstbackup
velero restore get
velero restore describe firstbackup-restore1 

# Restore parts from full backup
--exlude-resources
--include-resources
--exlude-namespace
--include-namespace

```










