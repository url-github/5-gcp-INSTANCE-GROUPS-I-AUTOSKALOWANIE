# 5-6-Regional and Zonal Managed Instance Groups

- Klasyczny MIG to Instance groups zonal czyli wszystkie VM są umieszczone w jednym zonal.
- W przypadku Regional MIG korzystamy z zalety globalnych sieci czyli VM są porozrzucane pomiędzy zonal. Dla regionów, które mają więcej niż trzy zonal Instance groups wrzuci maszyny do trzech zonal. Jeżeli chcemy zwiększyć lub wybrać specyficzne zonal to mogę to zrobić dodając parametry w CLI.

### [Distributing instances by using regional MIGs](https://cloud.google.com/compute/docs/instance-groups/distributing-instances-with-regional-instance-groups)

### Tworzenie Instance groups w modelu Regional (automatyczny wybór Zonal)

1. Aby stworzyć jakąkolwiek Instance Groups muszę stworzyć Instance Templates.

bigdata_pw_2020@cloudshell:~ (affable-doodad-259911)$ gcloud compute instance-templates create example-template
Created [https://www.googleapis.com/compute/v1/projects/affable-doodad-259911/global/instanceTemplates/example-template].
NAME              MACHINE_TYPE   PREEMPTIBLE  CREATION_TIMESTAMP
example-template  n1-standard-1               2020-05-26T03:14:55.644-07:00

### Tworzenie Instance groups w modelu Regional (wskazanie na konkretne Zonal)

### Tworzenie Instance groups w modelu Regional (wyłączona autodystrybucja VM)


