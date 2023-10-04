# Initial set up
```bash
export INSTANCE_NAME=

# 1. Prepare a startup script
cat << STARTUP >/tmp/startup-script
sudo apt updatesudo apt upgrade
sudo apt install -y jupyter-core jupyter-notebook jupyter-client
sudo useradd jupyter || true
sudo mkdir /home/jupyter || true
sudo chown jupyter /home/jupyter
sudo -u jupyter rm -f /home/jupyter/.jupyter/jupyter_notebook_config.py
sudo -u jupyter jupyter notebook --generate-config
sudo cat /home/jupyter/.jupyter/jupyter_notebook_config.py > /tmp/config.py
cat << CONFIG >> /tmp/config.py
import os
os.environ['SHELL'] = '/bin/bash'
c = get_config()
c.NotebookApp.ip = '*'
c.NotebookApp.open_browser = False
c.NotebookApp.port = 8888
c.NoteBookApp.token = ''
c.NoteBookApp.password = ''
CONFIG
sudo -u jupyter cp /tmp/config.py /home/jupyter/.jupyter/jupyter_notebook_config.py
sudo chown jupyter /home/jupyter/.jupyter/jupyter_notebook_config.py
sudo -u jupyter nohup jupyter-notebook --ip=0.0.0.0 --port=8888 --no-browser --NotebookApp.token='' --NotebookApp.password=''
STARTUP

# 2. Create a Google Cloud instance
gcloud compute instances create ${INSTANCE_NAME} \
  --project=open-targets-eu-dev \
  --zone=europe-west1-d \
  --machine-type=e2-standard-4 \
  --service-account=426265110888-compute@developer.gserviceaccount.com \
  --scopes=https://www.googleapis.com/auth/cloud-platform \
  --create-disk=auto-delete=yes,boot=yes,device-name=${INSTANCE_NAME},image=projects/ubuntu-os-cloud/global/images/ubuntu-2204-jammy-v20230829,mode=rw,size=500,type=projects/open-targets-eu-dev/zones/europe-west1-d/diskTypes/pd-balanced \
  --metadata-from-file=startup-script=/tmp/startup-script

# 3. Create and attach an external IP address.
gcloud compute addresses create ${INSTANCE_NAME} --project=open-targets-eu-dev --region=europe-west1
IP_ADDRESS=$(gcloud compute addresses list ${INSTANCE_NAME} 2>/dev/null | tail -n1 | awk '{print $2}')
gcloud compute instances delete-access-config -q ${INSTANCE_NAME}
gcloud compute instances add-access-config ${INSTANCE_NAME} --project=open-targets-eu-dev --zone=europe-west1-d --address=${IP_ADDRESS}

# 4. Create firewall policy
gcloud compute --project=open-targets-eu-dev firewall-rules create ${INSTANCE_NAME} --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=all --source-ranges=$(curl -s ifconfig.me)/32
```

# Start the instance before working
```
export INSTANCE_NAME=
gcloud compute instances start ${INSTANCE_NAME}
gcloud compute firewall-rules update ${INSTANCE_NAME} --source-ranges=$(curl -s ifconfig.me)/32
IP_ADDRESS=$(gcloud compute addresses list ${INSTANCE_NAME} 2>/dev/null | tail -n1 | awk '{print $2}')
echo
echo "Jupyter notebook URL: http://${IP_ADDRESS}:8888"
```

# Shut down the instance once finished working
```
export INSTANCE_NAME=
gcloud compute instances stop ${INSTANCE_NAME}
```

# Clean up
```bash
export INSTANCE_NAME=
gcloud compute instances delete -q ${INSTANCE_NAME}
gcloud compute addresses delete -q ${INSTANCE_NAME}
gcloud compute --project=open-targets-eu-dev firewall-rules delete -q ${INSTANCE_NAME}
```
