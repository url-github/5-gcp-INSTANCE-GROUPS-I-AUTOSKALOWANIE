# 5-gcp-INSTANCE-GROUPS-I-AUTOSKALOWANIE

#### Komenda do zasymulowania obciążenia procesora:

> dd if=/dev/zero of=/dev/null

#### Zmiana konfiguracja autoskalingu:

> gcloud compute instance-groups managed set-autoscaling instance-vm-group --min-num-replicas 5 --max-num-replicas 8 --zone us-central1-a

#### Tworzenie 1-ego szablonu:

> gcloud compute instance-templates create template1 --image-family debian-9 --image-project debian-cloud --machine-type=f1-micro

#### Tworzenie 2-ego szablonu:

> gcloud compute instance-templates create template2 --image-family debian-10 --image-project debian-cloud --machine-type=f1-micro

#### Tworzenie Instance groups:

> gcloud compute instance-groups managed create my-group-1 --base-instance-name=my-group-1 --template=template1 --size=3 --zone=us-central1-a

#### Aktualizacja grupy instancji na nową wersję szablonu template2, gdzie max 2 instancje mogą być wyłączone w momencie uruchamiania aktualizacji:

> gcloud compute instance-groups managed rolling-action start-update my-group-1 --version template=template2 --max-unavailable 2 --zone us-central1-a

### Canary Testing

#### Tworzę 1-szą Instance templates.

> gcloud compute instance-templates create template1 \
--image-family debian-9 \
--image-project debian-cloud \
--tags=http-server \
--machine-type=f1-micro \
--metadata=startup-script=\#\!/bin/bash$'\n'sudo\ apt-get\ update\ $'\n'sudo\ apt-get\ install\ -y\ nginx\ $'\n'sudo\ service\ nginx\ start\ $'\n'sudo\ sed\ -i\ --\ \"s/Welcome\ to\ nginx/Version:1\ -\ Welcome\ to\ \$HOSTNAME/g\"\ /var/www/html/index.nginx-debian.html

#### Tworzę 2-szą Instance templates.

gcloud compute instance-templates create template2 \
--image-family debian-9 \
--image-project debian-cloud \
--tags=http-server \
--machine-type=f1-micro \
--metadata=startup-script=\#\!/bin/bash$'\n'sudo\ apt-get\ update\ $'\n'sudo\ apt-get\ install\ -y\ nginx\ $'\n'sudo\ service\ nginx\ start\ $'\n'sudo\ sed\ -i\ --\ \"s/Welcome\ to\ nginx/Version:2\ -\ Welcome\ to\ \$HOSTNAME/g\"\ /var/www/html/index.nginx-debian.html

#### Tworzę Instance Groups z szablonu 1.

> gcloud compute instance-groups managed create my-group --base-instance-name=my-group --template=template1 --size=4 --zone us-central1-a

#### Podmieniam 50% VM na nowy szablon nr 2.

> gcloud compute instance-groups managed rolling-action start-update my-group --version template=template1 --canary-version template=template2,target-size=50% --zone us-central1-a

#### Kiedy nowa wersja jest stabilna robię aktualizację szablony nr 2 na 100%

> gcloud compute instance-groups managed rolling-action start-update my-group --version template=template2 --max-unavailable 100% --zone us-central1-a

### Health Checks dla VM Instace

