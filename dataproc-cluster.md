### Create
```bash
gcloud dataproc clusters create tskir-test --enable-component-gateway --region europe-west1 --single-node --master-machine-type n2-standard-16 --master-boot-disk-size 500 --image-version 2.1-debian11 --optional-components JUPYTER --project open-targets-eu-dev
```

### Access
https://console.cloud.google.com/dataproc/clusters/tskir-test/interfaces?region=europe-west1&project=open-targets-eu-dev

### Delete
```bash
gcloud dataproc clusters delete tskir-test --region europe-west1 --project open-targets-eu-dev
```
