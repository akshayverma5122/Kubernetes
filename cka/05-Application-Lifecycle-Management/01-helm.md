## Helm Commands 
- Helm repo management commands

		Helm repo --help
		helm repo list
		helm repo update
		helm repo add bitnami https://charts.bitnami.com/bitnami
		helm repo remove bitnami 

- searching the charts in added helm repository
  
		helm search repo wordpress
		helm search repo wordpress --versions

- installing the wordpress using helm charts

		helm install myblogging  bitnami/wordpress --version 24.2.3

- Getting the values and readme details of added helm repository.
  
		helm show chart  bitnami/wordpress
		helm show readme bitnami/wordpress
		helm show values bitnami/wordpress
		helm show values bitnami/wordpress --version 24.2.3

- install the helm charts with custom values.

		helm install local-wp bitnami/wordpress --version 24.2.3 --set mariadb.auth.rootPassword=goodpassword --set 			mariadb.auth.username=mariadbuser --set mariadb.auth.password=badpassword

		helm install local-wp bitnami/wordpress --version 24.2.3 -f custom-values.yaml

- uninstalling the helm charts

		helm uninstall local-wp

- Get the custom applied and all the values

		helm get values local-wp	
		helm get values local-wp --all

- Upgrading & Rollback helm 

		helm upgrade --reuse-values --values custom-values.yaml local-wp bitnami/wordpress --version 24.2.3
		helm get values local-wp 
		helm history local-wp 
		helm get values local-wp --revision 1
		helm rollback local-wp 3

		helm upgrade --reuse-values --values custom-values.yaml --set image.tag="nonexistence" local-wp bitnami/wordpress --version 24.2.2  --atomic --cleanup-on-fail  --debug --timeout 2m

