Helm basic 
---------------

Helm repo --help
Helm repo list
Helm repo update
helm repo add bitnami https://charts.bitnami.com/bitnami
Helm repo remove bitnami 

helm search repo wordpress
helm search repo wordpress --versions


helm  show chart  bitnami/wordpress
Helm show readme bitnami/wordpress
Helm show values bitnami/wordpress
helm show values bitnami/wordpress --version 24.2.3  > values.yaml


helm install myblogging  bitnami/wordpress --version 24.2.3

helm install local-wp bitnami/wordpress --version 24.2.3 --set mariadb.auth.rootPassword=goodpassword --set mariadb.auth.username=mariadbuser --set mariadb.auth.password=badpassword

helm install local-wp bitnami/wordpress --version 24.2.3 -f custom-values.yaml

Helm uninstall local-wp


helm get values local-wp
helm get values local-wp --all

Upgrading & Rollback helm 
----------------------

helm upgrade --reuse-values --values custom-values.yaml local-wp bitnami/wordpress --version 24.2.3
helm get values local-wp 
helm history local-wp 
helm get values local-wp --revision 1

helm rollback local-wp 3

helm upgrade --reuse-values --values custom-values.yaml --set image.tag="nonexistence" local-wp bitnami/wordpress --version 24.2.2  --atomic --cleanup-on-fail  --debug --timeout 2m
![Uploading image.png…]()
