# Code-signing certificate

## How to generate a new code-signing certificate

1. You should receive an email with a link to generate a new certificate roughly 30 or more days before renewal. The email will contain a link that cannot be shared here that allows you to submit a form for an automatically generated key.
2. In the form you are requested to submit the contents of a csr file. This can be located in the team vault under `APSIM code signing certificate | attachments` in the file `apsim-cert.csr`.
3. Fill out the rest of the fields. The fields are straight forward. One note however, make sure you include additional email addresses of the other members of your team so other people will also see the reminder email when it gets close to expiry.
4. Once submitted, wait for your DigiCert certificate email. This should only take 5-10 minutes. Download the files from the email's link making sure to select the option to download the files as `.pem`. This is needed for subsequent steps.
5. Move the file `apsim-cert.key` file from the vault to the dev.apsim.info server. This can be found in the same location in the vault from step 2.
6. Copy the zip archive you downloaded from digicert to the dev.apsim.info server as well. Using tools like scp are helpful for this action.
7. Unzip the zip archive.
8. Copy the `apsim-cert.key` file to the unzipped archive.
9. `cd` into the unzipped archived then use the command:

```bash
openssl pkcs12 -export -out apsim.p12 -in apsim_info.pem -inkey apsim-cert.key
```

10. You will be prompted to enter a password. Use the password in the vault password field for `APSIM code signing certificate` as the password.
11. Reenter the password.
12. You should now have an apsim.p12 key.
13. Zip this directory and attach it to the `APSIM code signing certificate` vault item for safe keeping.
14. Where ever the key is used it will need to be updated to the new file. Currently this can be updated using the Jenkins GUI but is likely to be elsewhere in future.
