# Infra Outline
## Network
### [Cilium](https://github.com/cilium/cilium)
Available FIPS 140-2 compliant from [Chainguard](https://images.chainguard.dev/directory/image/cilium-agent-fips/overview) if needed. This is the best network I could ever recommend. It's very fast, robust, secure, and has good observability capabilities through [Hubble](https://github.com/cilium/hubble). I used to use Istio a lot professionally for a service mesh, had some issues with it, tried Cilium, and have not looked back since. Does Gateways (better than ingress) and all of the service connecting/security components.

## CI/CD
### [Helm](https://github.com/helm/helm)
Helm is a packaging format for apps. It's a way to declare an app with all kinds of information on how it should be deployed as an installer. *Helm does not always play nicely with Argo because of the way it handles pre/post-installation steps*

### [Kustomize](https://github.com/kubernetes-sigs/kustomize)
Kustomize allows you to make changes to things before they hit the cluster. Super helpful for forcing changes to a helm chart when they don't have a values file.

### [ArgoCD](https://github.com/argoproj/argo-cd)
ArgoCD is what we use to install everything else. My usual approach is to bootstrap the cluster with Argo and then apply a bootstrap appset that loads an Application to manage Argo and other apps I want to deploy (in super simple environments). App of apps is possible and allows for ordering of installs much better, but it's usually more involved/harder to work with. Both will be demonstrated here.

### [External Secrets Operator](https://github.com/external-secrets/external-secrets)
ESO (not Elder Scrolls Online) is how you should re-think secrets. I use this in combination with Crunchy to make ephemeral password databases or to create client IDs for an IDP. Endless possibilities for syncing passwords that are maintained in a place like Vault and/or just are ephemeral keys for apps.

### [Harbor](https://github.com/harbor-framework/harbor)
Harbor is kind of the go-to for OCI image hosting in the projects I've worked on. It hooks in with Trivy for scanning and has a ton of neat automation capabilities. Highly recommend this, it also helps with those pesky docker pull limits when nodes don't cache enough.

## Identity Platforms
### [Keycloak](https://github.com/keycloak/keycloak)
Keycloak is a pretty standard IdP. They support [FIPS 140-2](https://www.keycloak.org/server/fips#_keycloak_server_in_fips_mode_in_containers) compliant modes and they can be very helpful as far as syncing users from somewhere (LDAP, etc) and managing an identity for other apps. A lot of the apps in K8s will sync via OIDC so having an IdP is helpful giving people a single sign-on for all the apps they use. I use passkeys for login in my stuff so everything is kept on a physical FIDO key I enroll that takes a pin (Yubikey, phones, browsers, lots of things support passkeys) so I have 2-factor (key = something you have + pin = something you know) auth. Supports multifactor using something like a Yubikey as the second factor to a password login with LDAP as well.

## Security
### [Copa](https://github.com/project-copacetic/copacetic)
Copa allows us to patch/remediate simple packages through upgrades in docker images using CVE reports--super helpful!

### [Trivy](https://github.com/aquasecurity/trivy)
Trivy helps you scan your images, configurations, secrets, and general cluster compliance in one, neat package! It's had its own bad vulnerability once because of a breach in their code base, but otherwise has been trustworthy. Worth knowing about the mistake in case someone brings it up, but this is my preferred package.

### [Kyverno](https://github.com/kyverno/kyverno)
The "easy/simple" method for denying access to your cluster for naughty developers. Don't want them to use root in a pod? Use Kyverno to stop them from being allowed to schedule root pods. Have a fix that requires all pods declare a certain feature or add a certain annotation? Kyverno mutation can make those changes to any admitted pod for you. If you need more power and you're a masochist, OPA Gatekeeper has its own language Rego and will help get those toothpicks under your fingernails.

### [Tetragon](https://github.com/cilium/tetragon)
Remember Cilium and Hubble? Now Tetragon. eBPF efficiency meets helpful runtime behavior policies. Rules like "detect-shell-in-pod" which allow you to either audit shell executions into pods or just automatically kill them. Falco is pretty popular in industry, but it mostly detects/alerts so I choose Tetragon. Less CPU utilization, active prevention, and they didn't scream Sysdig at me waving a $20 bill as a prize in a convention center.

## Monitoring
### [Prometheus](https://github.com/prometheus/prometheus) / [Thanos](https://github.com/thanos-io/thanos)
Prometheus collects events as a time series, allowing scraping/storing of large amounts of data. Thanos extends Prometheus to higher availability and better support for things like S3 backing storage.

### [Loki](https://github.com/grafana/loki)
Loki is a way to ingest logs from a variety of locations. Can be for pods/apps, firewalls, anything really. Recommend using it as a datasource for Grafana.

### [Grafana](https://github.com/grafana/grafana)
Grafana is how we look at our data. You can do it via explore or with dashboards using a variety of datasources like Prometheus/Thanos and Loki. Don't let your leaders buy cloud licenses and waste your money unless you can't manage simple software yourself.

## Storage
### [Crunchy PostgresOperator](https://github.com/CrunchyData/postgres-operator)
Crunchy is my preference for postgres. It's postgres, but has some helpful pieces for infra like if you change the password secret (via ESO for instance) it propagates to the DB and you can just mirror those out to another NS. Super cool setting up a DB cluster and then just creating new DBs/users via Argo and letting all the apps figure out the passwords on their own by loading a secret. I don't even know how to get into my DBs without loading up a secret to see the password.

### [Ceph](https://github.com/rook/rook)
If you ever move to having storage that you don't mind keeping attached to your K8s nodes, Rook Ceph is fantastic. OSDs are managed as pods, you can have HA for everything, and it basically handles everything while staying K8s native.
