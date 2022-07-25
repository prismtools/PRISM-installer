![PRISM Logo](https://prismtools.dev/wp-content/uploads/2019/12/prism_logo-2x-drkbkgd.png)
# PRISM Installer
Platform for Imaging in Precision Medicine (PRISM) is a UAMS-centric repository for biomedical image-based software tools. The tools in PRISM focus on digital pathology, interactive deep learning, and data management. This installer will create automatically deploy the selected PRISM components. 


## Requirements

+ 10GB of free memory
+ 30GB of free disk space
+ At least 4 CPUs available for minikube to use
+ Internet connection
+ [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/install) (if running Windows)
+ [Docker](https://minikube.sigs.k8s.io/docs/drivers/docker/)
+ If using WSL then follow [these instructions](https://docs.docker.com/desktop/windows/wsl/#download) to configure Docker with WSL
+ python3
+ wget

## Supported OS and Architecture
+ Windows 10 (using Windows Subsystem for Linux)
+ MacOS x86-64 (Intel based Macs)
+ Linux x86-64


## Simple Install Instructions
+ If you don't already have Docker installed on your test machine then download Docker (if using on Windows then follow the steps to get WSL and Docker working together which was posted above) and make sure that you change the max resource settings (located under preferences > resources) to at least 4 CPUs and 10GB of RAM
+ Download prism_installer.py from [the PRISM webpage](https://prismtools.dev)
+ Navigate to the directory you want to use to install PRISM
+ Run prism_installer.py and answer the questions when prompted `python prism_installer.py`
+ If you're using Linux or Windows: after prism_installer.py exits, open one new terminal window in the PRISM installation directory (Windows and Linux) and run `python portforward.py`. This terminal window must remain open for the PRISM instance to function correctly.
+ If you're using Mac: after prism_installer.py exits, open two new terminal windows in the PRISM installation directory (Mac) and run `python tunnel.py` in one terminal window and run `python portforward.py` in the other terminal window. These terminal windows must remain open for the PRISM instance to function correctly.
+ Open a web browser and navigate to http://127.0.0.1.nip.io:8181/

## Loading data
### Example data
First you need to download the actual sample data (located in /sample_data) and extract it. If you used the advanced installer instructions, this file should have been downloaded if you cloned the entire repo.

```bash
wget https://github.com/ /releases/download/latest/sample_data.tar.gz
tar -zxf sample_data.tar.gz
```

### PathDB
#### Loading
The pathology slides are in the `images` directory and need to be copied into the PVC that is shared between most of the PathDB containers.

```bash
POD=$(kubectl get pod -l prism=imageloader -o jsonpath="{.items[0].metadata.name}")
kubectl cp ./images/ $POD:/data/
```
After the images are in place the imageloader container is used to load the images into PathDB proper.

```bash
kubectl exec deployment/imageloader -- imageloader -src /data/images/example.csv -username admin -password bluecheese2018 -collectionname Public
```
#### Testing

You can now log into PathDB (default user: `admin`, password: `bluecheese2018`) and check to see if the images have appeared properly in the Public collection.

#### Releasing Images
Once you are in the PathDB instance, the following steps are how to release images to the public collection.

1. Click on "Image Review" on the right side
2. Change the "Moderation State" to "Draft" and Apply
3. Select all images and for action choose "Modify field values" and click "Apply to Selected Items"
4. Scroll down and check "Moderation State" then choose "Published" for the "Save as" field. Click Apply
5. The images will now be available under the Public Collection

### NBIA
#### Loading

The radiology files are in the extracted `radiology` directory.
These can either be loaded through Posda which is out of scope for this document, or directly with a small script.

First the files need to be copied into the NBIA containers dicom storage PVC.

```bash
POD=$(kubectl get pod -l prism=nbia -o jsonpath="{.items[0].metadata.name}")
kubectl cp ./radiology/ $POD:/opt/dicoms/
```

Then the included `insert_dicom.py` script can be used to load them into the NBIA database. Note: if you downloaded the simple installer, you will need to download this file from the sample_data folder in this repo.
To use the included script, the NBIA service needs to be directly accessable over 8080, so you may need to take down any other port forwards.
Then run the following two commads in different terminals.

```bash
kubectl port-forward svc/nbia 8080:8080
python3 insert_dicom.py
```

This will walk the radiology directory and POST a request for each file to the nbia-api, which will parse the already copied dicom files into the NBIA database.

#### Testing
Before taking down the port-forwards, head to http://localhost:8080/nbia-search and log in as the `nbiaAdmin` user (default password `admin`).
* Click Data Admin->Perform Quality Control
* Select the Public collection from the filter on the left.
* Select all series on the right.
* Change the status to `Visible` and the Release to `Yes`.

Now when you return the nbia-search page, you should be able to select the Public collection from the left and see results.

### Semantic RDF
#### Loading
To populate the cohort builder with the sample data for these radiology/pathology subjects, the included `rdf/statements.ttl` needs to be loaded into the triplestore.
Port forward directly into the triplestore.

```bash
kubectl port-forward svc/triplestore 7200:7200
```

Points your browser at http://localhost:7200 and ensure you have selected the PRISM repository from the top right, then click the "Import RDF Data".
Then click "Upload RDF Files" and select the `rdf/statements.ttl` file to upload.
After uploading click the orange "Import" button on the right, and accept all of the default options.

#### Testing
Take down the various port-forwards and go to the cohort builder from the main PRISM landing page.
It should now not only load but when clicking "show collections" have a collection with iformation displayed.


## Advanced Install Instructions
+ You will need a running instance of Kubernetes
+ Clone this repo
+ Modify web/secret.yaml and coreapi/secret.yaml to set a custom username and password
+ If you want to modify the default ports used in PRISM, all port entries in all files of this repo are provided in port_map.csv
+ If you are doing a large installation, we recommend you modify the storage claims in the PVCs for each component
+ If you don't want to have all the PRISM components installed, you can simply remove the directory for the component. Note that only the directories named POSDA, NBIA, PATHDB, and semapi can be removed without breaking the PRISM instance.
+ If you remove components, you should modify prism/web/landing-configmap.yaml and change the corresponding removed service to false
+ If you want to keep all PRISM components in their own namespace, create a new namespace named prism `kubectl create namespace prism`
+ Once you have made all the modifications you want, simply run `kubectl apply --recursive -f .` to deploy all PRISM components. If you created a new namespace for PRISM then run `kubectl apply --recursive -f . --namespace=prism`



