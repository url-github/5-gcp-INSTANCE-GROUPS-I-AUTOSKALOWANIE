### [Treść zadania](https://szkolachmury.pl/google-cloud-platform-droga-architekta/tydzien-5-instance-groups-i-autoskalowanie/zadanie-domowe-nr-5/) 

MountKirk Games poprosiło cię, abyś przeprojektował aktualną architekturę, która będzie obejmowała nowe wytyczone firmowe standardy:

1. Na ten moment firma nie może zrezygnować z maszyn wirtualnych, dlatego nowa architektura musi korzystać z wirtualnych maszyn postawionych z niestandardowego obrazu, dostarczonego bezpośrednio przez firmę.
2. Rozwiązanie musi dynamicznie skalować się w górę lub w dół w zależności od aktywności w grze - bez większej ingerencji specjalistów.
3. Gracze korzystający z funkcjonalności firmy pochodzą z całego świata, a w szczególności z Stanów Zjednoczonych oraz Europy. Poprzez odpowiednie umiejscowienie rozwiązania MountKirk chce zredukować opóźnienie jakie występuje dla osób łączących się z US.
4. Rozwiązanie musi zapobiegać jakiejkolwiek przerwie w dostarczaniu funkcjonalności na wypadek awarii np. regionu Google Cloud.
5. Rozwiązanie musi umożliwość łatwe i bezpiecznie wdrażanie nowych wersji oprogramowania do instancji bez konieczności wpływania na całe środowisko.

## 1.1 Opis
Użycie [Managed Instance Groups](https://cloud.google.com/compute/docs/instance-groups/) pozwoli spełnić powyższe założenia:
* Zapewnienie **High availability**:
  * **Keeping instances running** - w przypadku niezamierzonego wyłączenia/usunięcia maszyny lub awarii, VM zostanie automatycznie odtworzona na nowo
  * [**Autohealing**](https://cloud.google.com/compute/docs/instance-groups/#autohealing) - w przypadku błędnego kodu odpowiedzi z aplikacji, maszyna zostanie usunięta i odtworzona na nowo
  * [**Regional (multiple zone) coverage**](https://cloud.google.com/compute/docs/instance-groups/#types_of_managed_instance_groups) - umieszczenie maszyn w różnych strefach pozwala na zabezpieczenie się przed awarią jednego z nich oraz rozłożeniem ruchu pomiędzy strefy. Natomiast umieszczenie maszyn w różnych regionach pomaga zmniejszyć opóźnienie w przypadku posiadania użytkowników z różnych części świata. W wyborze regionów pomóc może zbieranie metryk z informacją o lokalizacji użytkownika, ich ilości w danym regionie oraz występujących opóźnieniach.
  * [**Load balancing**](https://cloud.google.com/compute/docs/instance-groups/#load_balancing) - równomierne rozłożenie ruchu pomiędzy maszynami w danej strefie oraz pomiędzy samymi strefami. Wybór regionu do którego użytkownik zostanie przekierowany na podstawie najkrótszego opóźnienia. W przypadku awarii regionu/strefy tymczasowe przekierowanie ruchu do działającego regionu/strefy (zapewnienie **HA**).
* [**Scalability**](https://cloud.google.com/compute/docs/instance-groups/#autoscaling) - **MIG** automatycznie zeskaluje środowisko w zależności od obciążenia oraz naszej [polityki skalowania](https://cloud.google.com/compute/docs/autoscaler/#policies).
* [**Automated updates**](https://cloud.google.com/compute/docs/instance-groups/#automatic_updating) - umożliwia wykonanie **Rolling updates** oraz **Canary updates**, czyli wykonanie aktualizacji w dość bezpieczny sposób z możliwością łatwego przywrócenia poprzedniej wersji.

## 1.2 Opis aktualizacji wersji

1. Wybranie strefy na której testowana będzie nowa wersja aplikacji.
2. Przekierowanie 10% ruchu w wybranej strefie do nowej wersji, wraz z czasem zwiększanie tej wartości do 100% ([Canary update](https://cloud.google.com/compute/docs/instance-groups/rolling-out-updates-to-managed-instance-groups#starting_a_canary_update)). W międzyczasie zbierane będą informacje czy nie występują błędy, spowolnienie aplikacji, oraz zadowolenie użytkowników (porzez zbieranie zgłoszeń, monitorowanie portali społecznościowych oraz analizie KPI w Google Firebase).
3. Jeśli po określonym czasie w strefie nie zostaną odnotowane żadne nieprawidłowości, wykonanie **rolling update** w pozostałych strefach.
4. Jeśli wystąpią nieprawidłowości, zbyt wiele błędów lub za dużo niezadowolonych użytkowników nastapi cofnięcie wersji do poprzedniej.

# 2. Demo

### 2.1 Utworzenie `Health check`

- healthcheck-pm = nazwa Health check

```bash

# Utworzenie
gcloud compute health-checks create http healthcheck-pm \
    --check-interval 10 \
    --timeout 5 \
    --healthy-threshold 3 \
    --unhealthy-threshold 3 \
    --request-path "/health"

# Sprawdzenie
gcloud compute health-checks list

bigdata_pw_2020@cloudshell:~ (affable-doodad-259911)$ gcloud compute health-checks list
NAME            PROTOCOL
healthcheck-pm  HTTP
```
### 2.2 Utworzenie Instance template dla MIG

- template-pm = nazwa nr 1
- template-pm-new = nazwa nr 2

```bash
# Wersja 1

gcloud compute instance-templates create template-pm \
    --machine-type f1-micro \
    --tags http-server \
    --metadata startup-script='
  sudo apt-get update && sudo apt-get install git gunicorn3 python3-pip -y
  git clone https://github.com/GoogleCloudPlatform/python-docs-samples.git
  cd python-docs-samples/compute/managed-instances/demo
  sudo pip3 install -r requirements.txt
  sudo gunicorn3 --bind 0.0.0.0:80 app:app --daemon'
  
# Wersja 2
   
gcloud compute instance-templates create template-pm-new \
--image-family debian-9 \
--image-project debian-cloud \
--tags=http-server \
--machine-type=f1-micro \
--metadata=startup-script='
#!/bin/bash
sudo apt-get update 
sudo apt-get install -y nginx 
sudo service nginx start 
sudo sed -i -- "s/Welcome to nginx/Version:2 - Welcome to $HOSTNAME/g" /var/www/html/index.nginx-debian.html'
```
### 2.3 Utworzenie `Managed instance group` z włączonym autohealingiem

- mig1 = nazwa MIG 
- us-central1 = region dla MIG

```bash
gcloud compute instance-groups managed create mig1 \
    --region us-central1 \
    --template template-pm \
    --base-instance-name template-mig-pm \
    --size 3 \
    --health-check healthcheck-pm \
    --initial-delay 90 
    
```
### 2.4 Konfiguracja autoskalowania

```bash
gcloud compute instance-groups managed set-autoscaling mig1 \
    --region us-central1 \
    --min-num-replicas 3 \
    --max-num-replicas 8 \
    --target-cpu-utilization "0.5"
```

<details>
  <summary><b><i>Status</i></b></summary>

```bash

gcloud compute instance-groups managed list-instances mig1 --region us-central1

bigdata_pw_2020@cloudshell:~ (affable-doodad-259911)$ gcloud compute instance-groups managed list-instances mig1 --region us-central1
NAME                  ZONE           STATUS   HEALTH_STATE  ACTION  INSTANCE_TEMPLATE  VERSION_NAME  LAST_ERROR
template-mig-pm-x075  us-central1-b  RUNNING  HEALTHY       NONE    template-pm
template-mig-pm-3794  us-central1-c  RUNNING  HEALTHY       NONE    template-pm
template-mig-pm-jhk9  us-central1-f  RUNNING  HEALTHY       NONE    template-pm
```
</details>

### 2.5 Test autoskalowania (obciążenie środowiska)

<details>
  <summary><b><i>Wynik</i></b></summary>
 
 Niektóre VM wywalają się!

```bash

gcloud compute instance-groups managed list-instances mig1 --region us-central1

bigdata_pw_2020@cloudshell:~ (affable-doodad-259911)$ gcloud compute instance-groups managed list-instances mig1 --region us-central1
NAME                  ZONE           STATUS   HEALTH_STATE  ACTION    INSTANCE_TEMPLATE  VERSION_NAME  LAST_ERROR
template-mig-pm-jl23  us-central1-b           UNKNOWN       CREATING  template-pm                      Error QUOTA_EXCEEDED: Instance 'template-mig-pm-jl23' creation failed: Quota 'IN_USE_ADDRESSES' exceeded.  Limit: 4.0 in region us-central1.
template-mig-pm-pp0c  us-central1-b           UNKNOWN       CREATING  template-pm                      Error QUOTA_EXCEEDED: Instance 'template-mig-pm-pp0c' creation failed: Quota 'IN_USE_ADDRESSES' exceeded.  Limit: 4.0 in region us-central1.
template-mig-pm-x075  us-central1-b  RUNNING  HEALTHY       NONE      template-pm
template-mig-pm-3794  us-central1-c  RUNNING  HEALTHY       NONE      template-pm
template-mig-pm-tzds  us-central1-c  RUNNING  HEALTHY       NONE      template-pm
template-mig-pm-v64k  us-central1-c           UNKNOWN       CREATING  template-pm                      Error QUOTA_EXCEEDED: Instance 'template-mig-pm-v64k' creation failed: Quota 'IN_USE_ADDRESSES' exceeded.  Limit: 4.0 in region us-central1.
template-mig-pm-jhk9  us-central1-f  RUNNING  HEALTHY       NONE      template-pm
template-mig-pm-w5wb  us-central1-f           UNKNOWN       CREATING  template-pm                      Error QUOTA_EXCEEDED: Instance 'template-mig-pm-w5wb' creation failed: Quota 'IN_USE_ADDRESSES' exceeded.  Limit: 4.0 in region us-central1.
```
</details>

### 2.6 Test autohealing

<details>
  <summary><b><i>Status przed</i></b></summary>

```bash

gcloud compute instance-groups managed list-instances mig1 --region us-central1
```
</details>

<details>
  <summary><b><i>Status po</i></b></summary>

```bash
gcloud compute instance-groups managed list-instances mig1 --region us-central1

```
</details>

