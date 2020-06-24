In my app, I have the structure:

```
config/
  - deploy/
    - partials/
      - env.yml.erb
    - prod/
      - secrets.ejson
      - web-deploy.yml.erb
      - web-svc.yml.erb
      - migrate-pod.yml.erb
      - ingress.yml
      - # anything else, like a background worker
    - staging/
      - secrets.ejson
      - web-deploy.yml.erb
      - web-svc.yml.erb
      - migrate-pod.yml.erb
      - ingress.yml
      - # anything else, like a background worker
    # directories in this level are environment-specific, except `partials/`
    - traefik-helm.yml
    - redis-helm.yml
    # files in this level are helm charts that are used in all environments
```

In this repo I included `config/deploy/prod`

Assuming you can connect to your cluster with `kubectl`,

1. `kubectl create namespace prod-namespace`
I like to have a different namespace per application per stage (production, staging, etc)

2. gem install krane
This is for deployments

3. See how the k8s config files will look like by rendering them:
```sh
export ENVIRONMENT=prod NS=prod-namespace SHA=$(git rev-parse --verify master) && \
  krane render --current-sha=$SHA \
  --filenames=config/deploy/$ENVIRONMENT
```

Krane will render the config files (we're using embedded Ruby / erb after all)
and print them out to stdout. If that looks okay (note: you probably
won't need to just render again), then deploy:

```sh
export ENVIRONMENT=prod NS=prod-namespace SHA=$(git rev-parse --verify master) && \
  krane render --current-sha=$SHA \
  --filenames=config/deploy/$ENVIRONMENT | \
  krane deploy $NS $CLUSTER --stdin \
  --filenames=config/deploy/$ENVIRONMENT/secrets.ejson
```

Note: `$CLUSTER` is your GKE cluster name that looks like `gke_project-id_us-central1-c_cluster-name`.

`bin/kdeploy` includes a helper script for deployments. I do want to turn this into a gem but haven't gotten around to it yet.
