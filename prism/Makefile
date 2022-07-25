default: dry

dry:
	kubectl apply --recursive --dry-run=client -f .
	@echo "\nThis was just a dry run, 'make apply' to actually create"
	@echo "Don't forget to create your secrets.yaml, see web/secret.example and coreapi/secret.example for format"

apply:
	kubectl apply --recursive -f .

serve:
	@echo open: http://127.0.0.1.nip.io:8080/
	kubectl port-forward svc/web 8080:8080

status:
	kubectl get all -l prism
