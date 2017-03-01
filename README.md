# Deploying Python Apps in Air-Gapped Pivotal Cloud Foundry

If you are building a python application, you likely need access to PyPi and other repositories that host application dependencies. Python buildpacks in Cloud Foundry can download these dependencies from the internet on-demand when python apps are deployed to the platform.

However, in many cases, the same python application needs to be deployed in air-gapped, disconnected cloud foundry environments that do not have access to external PyPi repositories. In such cases, use the following instructions to deploy your python application in disconnected environments.

Begin with deployment in the unclassified environment with internet access.
Deploy python application in Cloud Foundry that has access to PyPi and anaconda repositories available from the internet.

```bash
cf push python-app
```

Once the application is successfully staged and running on the platform, download the application droplet.

Identify the application id guid
```bash
cf env python-app
```
> ```json
{
 "VCAP_APPLICATION": {
  "application_id": "3d5e4b38-47a5-4b65-8c73-186a2c60aeb9",
  "application_name": "python-app"
}
```

Download the application droplet (use the `application_id` guid from the step above)
```bash
cf curl v2/apps/3d5e4b38-47a5-4b65-8c73-186a2c60aeb9/droplet/download > python-app-droplet.tgz
```

This droplet now contains your source code, all application dependencies needed by your app, and the python runtime. You can now easily transport this `python-app-droplet.tgz` file to disconnected environments as one complete artifact and deploy your python application to Cloud Foundry in disconnected environments. Also transfer your source code and cloud foundry manifest file along with this artifact to the other environment.

Login to the your target Cloud Foundry endpoint in the `disconnected` environment.
```bash
cf login -a https://api.disconnected-environments.cf.io
```

Deploy your original source code to this environment. The application will fail at the time of staging due to lack of app dependencies. This is okay and expected behavior. This should at-least register your application in Cloud Foundry with a name and associated metadata for parity.
```bash
cf push python-app-disconnected-environment
```

Retrieve the OAuth Web Token
```bash
cf oauth-token
```
Response would be a bearer web token used later to upload the droplet.
> `bearer eyJhbGciOiJSUzI1NiIs...`

Retrieve the `application_id` guid of this app in the disconnected environment.
```bash
cf env python-app-disconnected-environment
```
> ```json
{
 "VCAP_APPLICATION": {
  "application_id": "56e4a698-e1ee-4fdc-a2f5-9ad76a17299a",
  "application_name": "python-app-disconnected-environment"
}
```

Upload your python application droplet to the disconnected environment. When uploading the droplet, use the `application_id` guid and `web token` from the steps above. Make sure to either provide the absolute path in the `-F` argument for your droplet or change to directory so `python-app-droplet.tgz` artifact is in your local directory.
```bash
curl "https://api.disconnected-environments.cf.io/v2/apps/56e4a698-e1ee-4fdc-a2f5-9ad76a17299a/droplet/upload" \
        -F droplet=@"python-app-droplet.tgz" \
        -X PUT \
        -H "Authorization: bearer eyJhbGciOiJSUzI1NiIs..."
```

Wait for a response that signifies a successful droplet upload job execution.
> ```json
{
  "metadata": {
    "guid": "d780d827-e2c3-4bef-afdb-2b8e0cec2b26",
    "created_at": "2017-02-28T22:33:51Z",
    "url": "/v2/jobs/d780d827-e2c3-4bef-afdb-2b8e0cec2b26"
  },
  "entity": {
    "guid": "d780d827-e2c3-4bef-afdb-2b8e0cec2b26",
    "status": "queued"
  }
}
```

Make sure that the upload droplet job has completed before proceeding further. Use the `url` key from the step above to confirm change of `status` to `finished`
```bash
cf curl /v2/jobs/d780d827-e2c3-4bef-afdb-2b8e0cec2b26
```
> ```json
"entity": {
      "guid": "0",
      "status": "finished"
   }
```

Restart the application in your disconnected environment and the it should run without any errors as it did in the environments which originally had access to external PyPi and anaconda repositories.
```bash
cf restart python-app-disconnected-environment
```
