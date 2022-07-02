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


### set up the cluster
Follow the guides in [this repo](https://github.com/karname-interview/applications) to set up `GitOps` based infrastructure for your platform (APIs,DataBases, Backup_services, and test, dev, prd environment configuration)




