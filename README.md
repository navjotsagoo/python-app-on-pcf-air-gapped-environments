# Deploying Python Apps in Air-Gapped Pivotal Cloud Foundry

If you are building a python application, you likely need access to PyPi and other repositories that host application dependencies. Python buildpacks in Cloud Foundry can download these dependencies from the internet on-demand when python apps are deployed to the platform.

However, in many cases, the same python application, when needs to be deployed in air-gapped, disconnected cloud foundry environments that do not have access to external PyPi repositories. In such cases, use the following instructions to deploy your python application in disconnected environments.

Deploy python application in Cloud Foundry that has access to PyPi online-repositories

```bash
cf push python-app
```

Once the application is successfully staged and running on the platform, then download the application droplet.

Identify the application id guid
```bash
cf env python-app
```
```json
{
 "VCAP_APPLICATION": {
  "application_id": "3d5e4b38-47a5-4b65-8c73-186a2c60aeb9",
  "application_name": "python-app"
}...
```

Download the application droplet

```bash
cf curl v2/apps/<application_id_guid>/droplet/download > python-app-droplet.tgz
```
