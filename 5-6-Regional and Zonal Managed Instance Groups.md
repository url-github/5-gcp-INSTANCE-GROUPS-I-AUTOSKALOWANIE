# 5-6-Regional and Zonal Managed Instance Groups

- Klasyczny MIG to Instance groups zonal czyli wszystkie VM są umieszczone w jednym zonal.
- W przypadku Regional MIG korzystamy z zalety globalnych sieci czyli VM są porozrzucane pomiędzy zonal. Dla regionów, które mają więcej niż trzy zonal Instance groups wrzuci maszyny do trzech zonal. Jeżeli chcemy zwiększyć lub wybrać specyficzne zonal to mogę to zrobić dodając parametry w CLI.

### [Distributing instances by using regional MIGs](https://cloud.google.com/compute/docs/instance-groups/distributing-instances-with-regional-instance-groups)

### Zmienne:
- template-pm = nazwa Instance Temlate
- rmig1 = nazwa Instance Groups
- instances-pm = nazwy Instancji VM

### Tworzenie Instance groups w modelu Regional (automatyczny wybór Zonal). 

1. Aby stworzyć jakąkolwiek Instance Groups muszę najpierw stworzyć Instance Templates.

> gcloud compute instance-templates create template-pm

2. To polecenie tworzy regionalną grupę instancji zarządzanych w trzech strefach w regionie us-east1.

> gcloud compute instance-groups managed create rmig1 --template template-pm --base-instance-name instances-pm --size 3 --region us-east1

### Tworzenie Instance groups w modelu Regional (wskazanie na konkretne Zonal za pomocą flagi --zones)

> gcloud compute instance-groups managed create rmig2 --template template-pm --base-instance-name instances-pm --size 3 --zones us-east1-b,us-east1-c

### Tworzenie Instance groups w modelu Regional (wyłączona autodystrybucja VM)

> gcloud beta compute instance-groups managed create rmig3 --template template-pm --base-instance-name instances --size 3 --zones us-east1-b,us-east1-c --instance-redistribution-type NONE


