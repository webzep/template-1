# Template One

## **Overview**

This template monorepo is made of two main products:

- `/apps/app` - A react app with firebase authentication.
- `/services/api` - A fastify api which consumes a mysql database.

There are a few other `/packages` to keep the code organized. The `/packages` are:

- `/packages/colors` - A small colour library for manipulating colours.
- `/packages/common` - A place to put common code that is used by both the app and api.
- `/packages/ui` - A collection of UI components.

--------------------------------------------------------------------------------

## **Tech stack**

The tech stack includes the following main technologies.

### **`apps/app`**

- [Emotion](https://emotion.sh/)
- [Firebase](https://firebase.google.com/)
- [React Router](https://reactrouter.com/)
- [React](https://reactjs.org/)
- [Redux Toolkit](https://redux-toolkit.js.org/)
- [TurboRepo](https://turbo.build/)
- [Vite](https://vitejs.dev/)

### **`services/api`**

- [Docker](https://www.docker.com/)
- [Fastify](https://www.fastify.io/)
- [Firebase Admin](https://firebase.google.com/docs/admin/setup)
- [MySQL](https://www.mysql.com/)
- [Typeorm](https://typeorm.io/)

### **global**

- [Prettier](https://prettier.io/)
- [ESLint](https://eslint.org/)
- [Husky](https://typicode.github.io/husky/#/)
- [Yarn](https://yarnpkg.com/)
- [Typescript](https://www.typescriptlang.org/)
- [Commitizen](https://commitizen.github.io/cz-cli/)

> **Note**: To make commits, use `yarn commit` instead of `git commit` to trigger commitizen.

--------------------------------------------------------------------------------

## **Getting started**

### **Prerequisites**

- Replace occurences of `template-1` and `Template One` with the name of the project.

### **`/apps/app`**

1. Go to firebase console and create a new project.
2. Go to the authentication tab and enable email, google authentication and anonymous authentication.
3. Create `/apps/app/.env` and copy the values from the comment in `apps/app/src/core/configs/environment.ts`
4. Go to the project settings and create a new web app. Copy the relevant values from the object in the code block, into `/apps/app/.env`. See `/apps/app/src/core/configs/firebase.ts` to understand how the env variables are then loaded in.
5. Run `yarn && yarn start:turbo` to start the app. At this stage you should be able to log in with your gmail account. You will get knocked back to the error page though because the server isn't running.
6. Kill the app for now.

### **`/services/api`**

1. In the firebase console go to `Settings > Service Accounts > Generate New Private Key` and download the file.
2. Rename it to `firebase-service-account.json` and put it in the `services/api/src/configs` directory. This name is already in the `.gitignore`.
3. Create `/services/api/.env` and copy the values from the comment in `services/api/src/configs/environment.ts`.
4. Run `yarn start`, this will take a minute or two to build. It will also start the frontend using concurrently.
5. Go to the browser and sign into the app.

**Other notes:**

- Use `yarn start:build` to also rebuild the api container if you make changes to the api.
- Use `yarn stop` to stop the container.

### **Finally**

Search the repo for `TODO` and follow instructions.

--------------------------------------------------------------------------------

## **Deployment**

- [Vercel](https://vercel.com/) for the frontend.
- [GCP](https://cloud.google.com/) for the backend.

### **GCP**

**Cloud SQL**

1. Go to the [Cloud SQL](https://console.cloud.google.com/sql/instances) page and click `Create Instance`.
2. Choose `MySQL` as the database type. **Be careful if using "Generate password" as it may include escape characters that will break the pipeline commands.**
3. In connections you need to allow access to any network as the IP for Cloud Run is dynamic. Click `Add network` and enter `0.0.0.0/0`. You might need to come back and do this once the instance is created.
4. Choose appropriate options and click `Create Instance`. This will take 10 minutes.

#### **Using GH Actions**

The steps below roughly follow [this article](https://cloud.google.com/community/tutorials/cicd-cloud-run-github-actions).

Open the Cloud Shell and follow through the steps below.

```sh
# To make things easier, set the project ID and account name as variables:
export PROJECT_ID="project-id-232323" # You can find this in the query string of the URL
export ACCOUNT_NAME="your-account-name" # This can be anything you want but it must be 6-30 letters to be used as a service account

# Now roll through the rest pasting the commands as is

# Enable the necessary services:
gcloud services enable cloudbuild.googleapis.com run.googleapis.com containerregistry.googleapis.com

# Create a service account:
gcloud iam service-accounts create $ACCOUNT_NAME \
  --description="Cloud Run deploy account" \
  --display-name="Cloud-Run-Deploy"

# Give the service account Cloud Run Admin, Storage Admin, and Service Account User roles. You can't set all of them at once, so you have to run separate commands:
gcloud projects add-iam-policy-binding $PROJECT_ID \
--member=serviceAccount:$ACCOUNT_NAME@$PROJECT_ID.iam.gserviceaccount.com \
--role=roles/run.admin

gcloud projects add-iam-policy-binding $PROJECT_ID \
--member=serviceAccount:$ACCOUNT_NAME@$PROJECT_ID.iam.gserviceaccount.com \
--role=roles/storage.admin

gcloud projects add-iam-policy-binding $PROJECT_ID \
--member=serviceAccount:$ACCOUNT_NAME@$PROJECT_ID.iam.gserviceaccount.com \
--role=roles/iam.serviceAccountUser

gcloud projects add-iam-policy-binding $PROJECT_ID \
--member=serviceAccount:$ACCOUNT_NAME@$PROJECT_ID.iam.gserviceaccount.com \
--role=roles/cloudsql.client

# Generate a key.json file with your credentials, so your GitHub workflow can authenticate with Google Cloud:
gcloud iam service-accounts keys create key.json \
--iam-account $ACCOUNT_NAME@$PROJECT_ID.iam.gserviceaccount.com

# Run the following to print the contents of the key.json file and copy it to somewhere handy for now.
cat key.json

# Delete the file. You'll need to paste the contents into a GitHub secret next.
rm key.json
```

**Head over to GitHub**

1. Go to your GitHub repository and click `Settings > Secrets and variables > Actions >New repository secret`.
2. Add the following secrets:

3. `GCP_PROJECT_ID` - The project ID you used in GCP.

4. `GCP_EMAIL` - The email address of the service account you created in GCP. It should look something like `$ACCOUNT_NAME@$PROJECT_ID.iam.gserviceaccount.com`. Find it inside of key.json under the property `client_email`.

5. `GCP_CREDENTIALS` - The contents of the `key.json` file you created in GCP.

6. `MYSQL_DB` - The name of the database you want to use. e.g. `template-1`

7. `MYSQL_USER` - The username of the database user. e.g. `root`

8. `MYSQL_PASSWORD` - The password of the database user the you chose in Cloud SQL.

9. `MYSQL_HOST` - The host of the database. Get the public IP of your Cloud SQL instance.

10. Click the variables tab and app the following variable:

11. `GCP_SERVICE_NAME` - The name of the app you want to deploy to Cloud Run. e.g. `template-1-api`

Continuous deployment to Cloud Run is now set up. Any time you push to the `main` branch, the app will be deployed to Cloud Run.

Do that now to allow Cloud Build to generate a url for the api.

If you get an ambiguous error, `/home/runner/work/_temp/7d...71b7.sh: line 1: OFv**: No such file or directory` try................

### **Vercel**

#### **Create a project**

Create a new project and connect the GitHub repo. You can then set up the following environment variables:

- `VITE_SERVICES_API_URL`
- `VITE_FIREBASE_PROJECT_ID`
- `VITE_FIREBASE_MESSAGING_SENDER_ID`
- `VITE_FIREBASE_MEASUREMENT_ID`
- `VITE_FIREBASE_APP_ID`
- `VITE_FIREBASE_API_KEY`
- `VITE_ENVIRONMENT`

#### **Get Vercel API access token**

1. Go to <https://vercel.com/account/tokens> and create a token, record the token.
2. In the terminal, run `npx vercel login` and sign in with the preferred method.
3. Run `mkdir vercel && cd vercel && npx vercel link` and follow the instructions to link the existing project.
4. In the sidebar, navigate to `vercel/.vercel/project.json` and copy the `orgId` and `projectId` values.
5. Add the secrets to the GitHub repo. Delete the vercel directory you created.

```
VERCEL_TOKEN=xxxx
VERCEL_ORG_ID=xxxx
VERCEL_PROJECT_ID=prj_xxxx
```

### **Finally**

Go to `services/api/src/configs/corsWhitelist.ts` and whitelist any sites for CORS. A value of `null` will allow all sites.

Commit to the main branch and the app should be deployed.
