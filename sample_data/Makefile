NBIA_POD := $(shell kubectl get pod -l prism=nbia -o jsonpath="{.items[0].metadata.name}")
PATH_POD := $(shell kubectl get pod -l prism=imageloader -o jsonpath="{.items[0].metadata.name}")

download: sample_data.tar.gz
	wget https://github.com/UAMS-DBMI/PRISM_sample_data/releases/download/1.0/sample_data.tar.gz
	tar xf sample_data.tar.gz

load_path:
	kubectl cp ./images/ $(PATH_POD):/data/
	kubectl exec deployment/imageloader -- imageloader -src /data/images/example.csv -username admin -password bluecheese2018 -collectionname Public

load_radiology:
	@echo $(NBIA_POD)
	kubectl cp ./radiology/ $(NBIA_POD):/opt/dicoms/
	kubectl port-forward svc/nbia 8080:8080 &
	python3 insert_dicom.py
	@echo "Log in as admin and mark as visible through the QC tool"

load_semantic:
	@echo "Open http://localhost:7200 and load rdf"
	kubectl port-forward svc/triplestore 7200:7200
