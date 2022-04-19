# APSIM.Builds API

APSIM.Builds is a websites and REST API tracking metadata around apsim upgrades/releases. The source code is [on github](https://github.com/APSIMInitiative/APSIM.Builds). The website is an asp .net core application connected to a MariaDB database. The site runs inside docker behind a reverse proxy (apache) on a Linux GCP VM (see [apsim-web](GCP.md)).

All of the write-access API endpoints require authentication in the form of a JWT. We don't currently have an official JWT server; I've manually generated the tokens from the private key used to validate them. There is code in the APSIM.Builds repo (`Startup.BuildToken()`) to facilitate creating more tokens if need be.

## Running the website

Running the website requires that several environment variables are set. Details can be found in the repository [readme](https://github.com/APSIMInitiative/APSIM.Builds/blob/master/README.md). A mariadb server must also be running and accessible. The repository contains a sample docker-compose file to simplify this for debug sessions.

## Updating the website

Currently, the website must be updated manually, by ssh-ing into the apsim-web server and cloning or navigating to the apsim-web repository and running the deploy script in the builds directory. This will rebuild the docker image from latest git sources, and update the running container from the updated image. Note that the aforementioned environment variables must be set before running the script, otherwise the script will fail.

## Callers/Users

The API is called from the following places:

- APSIM GUI: called by the upgrades area of the GUI to populate the list of available upgrades. Also used in the "run on cloud" feature to (again) list the available versions of apsim
- [Registrations website](REGISTRATION.md): Called from the downloads page to list the available versions of apsim for download
- Jenkins: Called by CI/CD infrastructure and scripts to upload installers, documentation, add new builds to the DB, etc
- [Netlify site](NETLIFY.md): Used to populate the model documentation (autodocs) page
- GitHub: A github webhook is sent upon most pull request events in order to conditionally trigger jenkins release builds

Old versions of apsim also hit the following endpoint:

```
http://apsimdev.apsim.info/APSIM.Builds.Service/Builds.svc?GetUpgradesSinceIssue?issueID={issue}
```

This is still accessible on the [apsimdev](GCP.md) server, which will proxy requests through to the apsim-web server, where the current builds API runs.
