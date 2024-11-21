# Composer CI/CD Sample

Code in this directory is part of a tutorial found [in the Cloud docs](https://cloud.google.com/composer/docs/composer-2/dag-cicd-github). In this tutorial, you will learn how to use Cloud Build to sync DAGs from version control with your Cloud Composer environment.

While the DAGs found in this tutorial specifically are written for use in Airflow 2, this tutorial can be used in Airflow 1, Airflow 2, Composer 1, and Composer 2 environments.

# High Level Steps

- [Run local Airflow Environments](https://cloud.google.com/composer/docs/composer-2/run-local-airflow-environments#airflow-cli)
- [Test DAGs](https://cloud.google.com/composer/docs/composer-2/test-dags)
- [Test, synchronize, and deploy your DAGs from GitHub](https://cloud.google.com/composer/docs/composer-2/dag-cicd-github)

# Cloud Workstations Prep

```bash
sudo apt-get update
sudo apt-get install -y libbz2-dev libncurses-dev libffi-dev libreadline-dev libssl-dev libsqlite3-dev liblzma-dev

# install pyenv to install python on persistent home directory
curl https://pyenv.run | bash

# add to path
echo 'export PATH="$HOME/.pyenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(pyenv init -)"' >> ~/.bashrc
echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.bashrc

# updating bashrc
source ~/.bashrc

# install python 3.10.15 and make default
pyenv install 3.10.15
pyenv global 3.10.15

# execute
python --version
```

```
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
```

# Run local Airflow

## Environment Prep

Application Default Login

```bash
gcloud auth application-default login
```

Gcloud login

```bash
gcloud auth login
```

Gcloud Config set

```bash
PROJECT_ID="ktb-dhw-poc-internal"

gcloud config set project $PROJECT_ID
gcloud auth application-default set-quota-project $PROJECT_ID
```

Install Composer Local Development CLI tool

```bash
cd ~
git clone https://github.com/GoogleCloudPlatform/composer-local-dev.git

cd composer-local-dev
```

Edit setup.py as there is an issue in requests package
https://github.com/GoogleCloudPlatform/composer-local-dev/issues/61

On line 32 replace with this

```
    "docker==7.*",
    "requests==2.31.0"
```

Install Composer Local Development CLI tool

```bash
pip install .
```

## Create a local Airflow environment with a specific Cloud Composer version

To list available versions of Cloud Composer, run:

```bash
composer-dev list-available-versions --include-past-releases --limit 10
```

To create a local Airflow environment with default parameters, run:

```bash
export ENVIRONMENT_NAME="composer-instance"
export LOCATION="asia-southeast1"
export WEB_SERVER_PORT="8081"
export LOCAL_ENVIRONMENT_NAME="local-$ENVIRONMENT_NAME"
export LOCAL_DAGS_PATH="./dags"
export PROJECT_ID="ktb-dhw-poc-internal"
```

```bash
composer-dev create $LOCAL_ENVIRONMENT_NAME \
  --from-source-environment $ENVIRONMENT_NAME \
  --location $LOCATION \
  --project $PROJECT_ID \
  --dags-path $LOCAL_DAGS_PATH
```

Add dependencies in requirements.txt

```bash
cat <<EOF > composer/$LOCAL_ENVIRONMENT_NAME/requirements.txt
databricks-sql-connector
google-cloud-bigquery
google-cloud-datacatalog-lineage==0.3.1
google-cloud-secret-manager
google-cloud-storage
pandas==2.1.4
EOF
```

You can now start it using following command:

```bash
composer-dev start $LOCAL_ENVIRONMENT_NAME
```

To stop

```bash
composer-dev stop $LOCAL_ENVIRONMENT_NAME
```

# Test Dags

check mount

```bash
docker inspect -f '{{ .Mounts }}' $CONTAINER_ID
```

