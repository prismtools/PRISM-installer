# Install
Clone this repo locally

`git clone https://github.com/UAMS-DBMI/prism_installer`

Create web/secret.yaml and coreapi/secret.yaml, there are example files in the directory to show the structure.

Ensure your kubectl can parse the yaml files with no errors

`make`

Then actually apply

`make apply`

Now wait for everything to come up, you can monitor status with.

# Install Posda
`git clone https://code.imphub.org/scm/pt/k8s`

`make deploy`

# Check Status

`make status`

After all pods are running, expose the main web container.

`make serve`

Finally open your browser and head to http://127.0.0.1.nip.io:8080/ this is a nip.io proxy to allow subdomains on your local machine without editing your hosts file.

# Load sample images -> from PRISM Sample data
See the instructions in the sample_data directory of this repo
