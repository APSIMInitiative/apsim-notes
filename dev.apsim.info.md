# dev.apsim.info

The Linux GCP VM (dev-apsim-info) hosts a number of web APIs and pages:

* builds API
* jenkins
* registration
* vault

Each of these apps are deployed via docker-compose. In addition, apache2 is also deployed via a docker image and is not installed on the the host dev-apsim-info. 

All files for the server are in the repo: https://github.com/APSIMInitiative/apsim-web. This is cloned to ~/sources/apsim-web. Each of the above apps and apache (http) are contained in directories in the rep and are deployed via separate *deploy* scripts.

To ssh to dev.apsim.info use:
    ssh webmaster@dev.apsim.info