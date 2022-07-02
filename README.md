# KARNAME INTERVIEW (KI)

> For my random `non-karname` friend out there: This is not a production ready environment setup, as i had just a day to build this up.
so don't try it on your production cluster.

> For my `karname` friend out there though:

## Assumptions 
I assumed that the followings are provided in your infrastructure
- A Gitlab where you can hosts "to be mentioned repos" 
- Atleast one Gitlab runner, configured to run CI on "to be mentioned repos"
- A fully functional k8s cluster
- A BlockStorage provider for k8s (if not, search for `persist` in `values.yaml` files and set `enable` to `false`)
- A S3 provider visible from k8s cluster (like minio, ceph S3, ...)
- Ingress Controller for k8s (this setup was tested with NginxIngressController but theoretically should work for anything)
- A Docker registry with no user/pass (for simplicity!). I suggest using docker-registry helm chart to deploy it inside of the k8s cluster as it manages the insecure/secure situation)

## Preperations

### Gitlab 
set the following environment variables in the top group where you are going to put the repos (needed for CICD)
- `DOCKER_REG_URL`: <Docker Registry Domain> 
- `DAG_REPO`: <a valid gitlab repo with proper pull & push accesses(i.e https://user:password@git_domain.com/path/to/dag/repo)>
- `rules`: {} 
- `CD_REPO_BASE_PATH`: <a valid gitlab group path with proper pull & push accesses(i.e https://user:password@git_domain.com/path/to/dag/repo)>

Explanations: 
- `DOCKER_REG_URL`: Domain for the docker registry used to push docker images
- `DAG_REPO`: Repo where Airflow Dag files are going to reside
- `rules`: although there is no specific use case for our scenario, it does need to be valid `json` for my codes to run (i didn't have enough time!!!).
- `CD_REPO_BASE_PATH`: base path for repo that contains manifests for a service.
lets say you have a API repo that has the path: `git.mysupercompany.com/karname/some_service/some_api.git` and you set `CD_REPO_BASE_PATH` to `https://user:pass@git.mysupercompany.com/infra/`. Then it is assumed by the CI/CD modules that the repo containing your API's `CD` manifests is located at: `https://user:pass@git.mysupercompany.com/infra/karname/some_servie_some_apit.git`.
please hold this contract as it is crucial for the CI/CD to work.

With that out of the way, lets look at the road map.

## RoadMap

### installations 
Install ArgoCD following guides in [this repo](https://github.com/karname-interview/argocd)
> Installation provided above differs from normal installation. so if you already have a working ArgoCD instance, make sure to check README.md

Install Airflow following guides in [this repo](https://github.com/karname-interview/airflow)

Install Logging following guides in [this repo](https://github.com/karname-interview/logging)

Install monitoring following `QuickStart` guides in [this repo](https://github.com/karname-interview/monitoring)


### Docker Images 
build and push images from following repos onto your docker registry
[dagger: a tool for formatting airflow dags](https://github.com/karname-interview/dagger)

`gcr.io/kaniko-project/executor:debug` as `$DOCKER_REG_URL/kaniko:debug`

`k8s.gcr.io/kustomize/kustomize:v4.4.1` as `$DOCKER_REG_URL/infra/kustomize:1.0.0`

### Repos to clone 
clone following repos and put them in your gitlab
- [dummy-api](https://github.com/karname-interview/dummy-api): repo contains simple CRUD api and uses postgres as a database (workload repo) (should run CICD atleast once)
- [psql_backup](https://github.com/karname-interview/psql_backup): repo for making backups from psql of the prd environment (should run CICD atleat once)
- [apps](https://github.com/karname-interview/apps): ArgoCD's App of Apps repo
- [manifests](https://github.com/karname-interview/manifests): manifests for dummy-api (repo name should match the description given Preparation section of this Readme )
- [dummy-rsc](https://github.com/karname-interview/dummy-rsc): manifest for dummy service resources (i.e. postgres for each environment)

### set up the cluster
Follow the guides in [this repo](https://github.com/karname-interview/applications) to set up `GitOps` based infrastructure for your platform (APIs,DataBases, Backup_services, and test, dev, prd environment configuration)


### notes about this setup 

- I had only one day and few hours of sleep, so i may have forget to mention a thing or two! so I suggest a meeting in which i can describe the platform and how it works

- in this scenario all three environments (prd,stg,dev) are assumed to be in the same cluster (seperated by their namespaces). but this platform has no such limit and you can easily deploy them on different clustes should you want to, only need to start a seperate ArgoCD in each cluster

- take a good care at domains set for different modules (domains set in helm chart or configured in ingress-patchs).

- I tried to be as platform-agnostic as i could, but then again, i only had one day so i may have forgot to consider a thing or two!


### contact me 
email: letmemakenewone@gmail.com 


