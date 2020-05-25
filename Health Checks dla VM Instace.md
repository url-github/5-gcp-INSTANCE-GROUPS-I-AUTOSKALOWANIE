# [Health Checks](https://szkolachmury.pl/google-cloud-platform-droga-architekta/tydzien-5-instance-groups-i-autoskalowanie/health-checks-hands-on/) 5-5-Health Checks + Hands On

Compute Engine > Health checks:

1. Create Health checks.
2. Name: autohealing.
3. Request path np. /health Jest to ścieżka jaką Health checks będzie odpytywał.
4. Health criteria to parametry, które definiują sposób sprawdzania stanu aplikacji/node. 

Compute Engine > Instance templates:

1. Create Instance templates.
2. Name: webserver-template
3. Firewalls:
- Allow HTTP traffic
- Allow HTTPS traffic
4. Startup script:
>
git clone https://github.com/GoogleCloudPlatform/python-docs-samples.git 
cd python-docs-samples/compute/managed-instances/demo
sudo pip3 install -r requirements.txt
sudo gunicorn3 --bind 0.0.0.0:80 app:app --daemon
>








