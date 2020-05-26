# 5-6-Regional and Zonal Managed Instance Groups

- Klasyczny MIG to Instance groups zonal czyli wszystkie VM są umieszczone w jednym zonal.
- W przypadku Regional MIG korzystamy z zalety globalnych sieci czyli VM są porozrzucane pomiędzy zonal. Dla regionów, które mają więcej niż trzy zonal Instance groups wrzuci maszyny do trzech zonal. Jeżeli chcemy zwiększyć lub wybrać specyficzne zonal to mogę to zrobić dodając parametry w CLI.

### Compute Engine > Health checks:
