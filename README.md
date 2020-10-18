## Deploy to Cloud Run with Travis CI

- install travis cli
`sudo gem install travis`

- login to travis account
`travis login`

- get your current GCP project id and save it as an env
`PROJECT_ID="$(gcloud config get-value project -q)" # fetch current GCP project ID`

- name your service account that will be used to deploy your cloud run services
`SVCACCT_NAME=travisci-deployer`

- create a service account
`gcloud iam service-accounts create "${SVCACCT_NAME?}"`

- find the email of the service account
`SVCACCT_EMAIL="$(gcloud iam service-accounts list \
  --filter="name:${SVCACCT_NAME?}@"  \
  --format=value\(email\))"`

- create a JSON key file for the service account

`gcloud iam service-accounts keys create "google-key.json" \
   --iam-account="${SVCACCT_EMAIL?}"`

- assign needed permissions to your service account

```
gcloud projects add-iam-policy-binding "${PROJECT_ID?}" \
   --member="serviceAccount:${SVCACCT_EMAIL?}" \
   --role="roles/storage.admin"
```

```
gcloud projects add-iam-policy-binding "${PROJECT_ID?}" \
   --member="serviceAccount:${SVCACCT_EMAIL?}" \
   --role="roles/run.admin"
```

```
gcloud projects add-iam-policy-binding "${PROJECT_ID?}" \
   --member="serviceAccount:${SVCACCT_EMAIL?}" \
   --role="roles/iam.serviceAccountUser"
```

- encrypt the service account key
`travis encrypt-file google-key.json`

- the encrypt command will generate a new encrypted file and show you a openssl command copy it and put in the .tavisci.yml file instead of the one that I have posted

- remove the service account key or ignore it in Dockerfile and .gitignore

- you are ready. just commit your changes and watch how travis ci will deploy automatically to Cloud Run 