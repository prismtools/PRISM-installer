# PRISM Sample Data

Instructions on loading data into PRISM

## Prereq
First you need to download the actual sample data and extract it into this directory.

```bash
wget https://github.com/UAMS-DBMI/PRISM_sample_data/releases/download/1.0/sample_data.tar.gz
tar -zxf sample_data.tar.gz
```


## PathDB
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

## NBIA
#### Loading

The radiology files are in the extracted `radiology` directory.
These can either be loaded through Posda which is out of scope for this document, or directly with a small script.

First the files need to be copied into the NBIA containers dicom storage PVC.

```bash
POD=$(kubectl get pod -l prism=nbia -o jsonpath="{.items[0].metadata.name}")
kubectl cp ./radiology/ $POD:/opt/dicoms/
```

Then the included `insert_dicom.py` script can be used to load them into the NBIA database.
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


## Semantic RDF
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
