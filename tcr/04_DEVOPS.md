# TCR 04 — DevOps, Git, Docker, Kubernetes, CI/CD

> **Read this file carefully.** Both TCR sample questions printed in the official instruction PDF came from here:
> - *Q10: "What is a Jenkins Pipeline?"* -> answer: **CI/CD workflow**
> - *Q43: "A Kubernetes application is healthy internally but users cannot access it externally. Pods are running and passing readiness checks. What should be investigated first?"* -> answer: **Service and Ingress configuration**
>
> Most candidates prepare SQL and OOPs and skip this. That makes it the cheapest place to gain marks.

---

## 1. What DevOps is

DevOps is a **culture and set of practices** that unite development and operations to shorten the delivery cycle while keeping quality high. Its pillars: automation, continuous integration and delivery, infrastructure as code, monitoring, and shared ownership.

**The lifecycle:** Plan -> Code -> Build -> Test -> Release -> Deploy -> Operate -> Monitor -> (feedback back to Plan).

---

## 2. Git & version control

| Command | What it does |
|---|---|
| `git init` | create a new repository |
| `git clone <url>` | copy a remote repository locally |
| `git status` | show the working tree state |
| `git add <file>` | stage changes |
| `git commit -m "msg"` | record staged changes |
| `git push origin main` | upload commits to the remote |
| `git pull` | **fetch + merge** from the remote |
| `git fetch` | download remote changes **without** merging |
| `git branch <name>` | create a branch |
| `git checkout -b <name>` | create and switch to a branch |
| `git merge <branch>` | join another branch into the current one |
| `git rebase <branch>` | replay your commits on top of another branch |
| `git stash` | shelve uncommitted work temporarily |
| `git log --oneline` | compact history |
| `git revert <commit>` | create a **new** commit undoing an old one (safe on shared branches) |
| `git reset --hard <commit>` | move the branch pointer and discard work (**destructive**) |
| `git cherry-pick <commit>` | apply one specific commit onto the current branch |

**The distinctions they test:**
- **`git pull` vs `git fetch`:** *"fetch downloads remote changes without touching your working tree, letting you inspect them first; pull is fetch followed by an immediate merge."*
- **`merge` vs `rebase`:** *"Merge preserves the true history and creates a merge commit; rebase rewrites your commits onto a new base, producing a linear history. Never rebase commits that others have already pulled, because rewriting shared history breaks their copies."*
- **`revert` vs `reset`:** *"Revert adds a new commit that undoes the change, preserving history and remaining safe on shared branches. Reset moves the branch pointer and can discard commits, so it is only safe locally."*
- **The three areas:** working directory -> staging area (index) -> repository. `add` moves work to the second, `commit` to the third.
- **`.gitignore`** lists paths Git should not track — `node_modules`, build output, `.env` files.
- **Merge conflict:** occurs when two branches change the same lines. Git marks the region with `<<<<<<<`, `=======` and `>>>>>>>`; you edit to the desired result, `git add` the file, and commit.

**Branching strategies:** Git Flow (long-lived `develop` and `release` branches), GitHub Flow (short feature branches straight off `main`, merged via pull request), and trunk-based development (very short-lived branches, feature flags). GitHub Flow plus pull requests and code review is the common default.

---

## 3. CI/CD

- **Continuous Integration (CI):** developers merge into the shared main branch frequently, and every merge automatically triggers a **build and automated tests**. The point is to catch integration failures within minutes rather than at the end of a release.
- **Continuous Delivery (CD):** every change that passes the pipeline is automatically prepared to a **releasable** state; the final push to production remains a **manual** decision.
- **Continuous Deployment:** the same, but the release to production is **automatic** too, with no human gate.

> **The exam distinction:** *"Continuous Delivery keeps the software always ready to release with a manual approval step; Continuous Deployment removes that step and ships every passing change automatically."*

**A typical pipeline:**
```
Commit -> Trigger -> Build -> Unit tests -> Static analysis / lint
       -> Package (Docker image) -> Deploy to staging -> Integration & E2E tests
       -> Manual approval -> Deploy to production -> Monitor
```

**Benefits to name in a reasoning box:** faster feedback, fewer integration conflicts, repeatable and auditable releases, less manual error, and quicker rollback.

**Deployment strategies:**
- **Rolling** — replace instances gradually. The Kubernetes default.
- **Blue-Green** — run two identical environments and switch traffic in one step; instant rollback by switching back.
- **Canary** — send a small percentage of traffic to the new version, watch the metrics, then widen.
- **Recreate** — stop the old, start the new. Simple, but incurs downtime.

---

## 4. Jenkins

**Jenkins** is an open-source **automation server** used to build CI/CD pipelines.

**Key terms:**
- **Job / Project** — a configured unit of work.
- **Pipeline** — a **set of automated stages defined as code in a `Jenkinsfile`**, describing how software is built, tested and deployed. **This is the answer to the sample question: a Jenkins Pipeline represents a CI/CD workflow — not a container, a VM, or a repository.**
- **Declarative vs Scripted pipeline** — declarative uses a structured `pipeline { }` block and is the recommended style; scripted is free-form Groovy.
- **Stage** — a logical phase (Build, Test, Deploy). **Step** — a single command inside a stage.
- **Agent / Node** — the machine where the work executes. The **controller** (master) orchestrates; agents do the work.
- **Trigger** — what starts a build: an SCM webhook, a poll, a cron schedule, or a manual click.
- **Artifact** — the build's output (a jar, a binary, a Docker image) that later stages consume.
- **Plugins** — Jenkins's extension mechanism; almost every integration (Git, Docker, Slack, Kubernetes) is a plugin.

```groovy
pipeline {
    agent any
    stages {
        stage('Build')  { steps { sh 'npm install && npm run build' } }
        stage('Test')   { steps { sh 'npm test' } }
        stage('Deploy') { steps { sh './deploy.sh' } }
    }
    post {
        failure { mail to: 'team@example.com', subject: 'Build failed' }
        always  { junit 'reports/*.xml' }
    }
}
```

**"Pipeline as code" — why it matters:** the `Jenkinsfile` lives in the repository alongside the source, so the build process is version-controlled, reviewable, and evolves together with the code.

**Alternatives worth naming:** GitHub Actions, GitLab CI, CircleCI, Travis CI, Azure DevOps, ArgoCD (GitOps-style continuous delivery for Kubernetes).

---

## 5. Docker (containerisation)

**A container packages an application together with its dependencies, libraries and configuration**, so it runs identically on any machine with a container runtime. It solves "it works on my machine".

### Containers vs Virtual Machines — a guaranteed question

| | Container | Virtual Machine |
|---|---|---|
| Virtualises | the **operating system** | the **hardware** |
| Includes a guest OS? | **No** — shares the host kernel | **Yes** — a full guest OS |
| Size | megabytes | gigabytes |
| Startup | seconds or less | minutes |
| Isolation | process-level (weaker) | hardware-level (stronger) |
| Density per host | high | low |

**Reasoning sentence:** *"Containers share the host kernel and virtualise only the operating system, so they start in seconds and are megabytes in size, whereas a VM bundles an entire guest OS on virtualised hardware — giving stronger isolation but far more overhead."*

### Core Docker concepts
- **Dockerfile** — the recipe: instructions for building an image.
- **Image** — an immutable, layered, read-only template built from a Dockerfile.
- **Container** — a **running instance** of an image, with a thin writable layer on top.
- **Registry** — where images are stored and shared (Docker Hub, ECR, GCR).
- **Volume** — persistent storage that outlives the container. **Containers are ephemeral: data written inside one is lost when it is removed**, which is why databases need volumes.
- **Docker Compose** — defines and runs a multi-container application from a single `docker-compose.yml` (app + database + cache together).

> **Image vs container, in one line:** *"An image is the immutable blueprint; a container is a running instance of it. One image can spawn many containers, much as a class can produce many objects."* That analogy scores well.

```dockerfile
FROM node:18-alpine              # base image
WORKDIR /app
COPY package*.json ./            # copy manifests FIRST so this layer caches
RUN npm ci --only=production     # dependencies change rarely, so the layer is reused
COPY . .                         # then copy the source
EXPOSE 3000
CMD ["node", "server.js"]        # the process to run
```
**Why copy `package.json` before the source?** Docker caches each layer. If the source changes but the dependencies do not, the expensive install layer is reused. This layer-caching question appears often.

**`CMD` vs `ENTRYPOINT`:** `CMD` supplies default arguments that are easily overridden at run time; `ENTRYPOINT` sets the executable that always runs. **`COPY` vs `ADD`:** prefer `COPY`; `ADD` also unpacks archives and fetches URLs, which is usually surprising rather than useful.

| Command | Purpose |
|---|---|
| `docker build -t myapp .` | build an image from the Dockerfile |
| `docker run -d -p 8080:3000 myapp` | run detached, mapping host 8080 to container 3000 |
| `docker ps` / `docker ps -a` | list running / all containers |
| `docker logs <id>` | view a container's output |
| `docker exec -it <id> sh` | open a shell inside a running container |
| `docker images` | list images |
| `docker stop` / `docker rm` / `docker rmi` | stop a container / remove a container / remove an image |
| `docker-compose up -d` | start the whole multi-container stack |

---

## 6. Kubernetes (container orchestration)

**Kubernetes (K8s)** automates the deployment, scaling, healing and networking of containers across a cluster of machines. Docker packages one container; Kubernetes runs thousands of them reliably.

### The objects, from smallest outward

| Object | What it is |
|---|---|
| **Pod** | the **smallest deployable unit** — one or more tightly-coupled containers sharing a network namespace and storage. Pods are **ephemeral** and get a new IP whenever they are recreated. |
| **ReplicaSet** | keeps a specified number of identical pod replicas running |
| **Deployment** | manages ReplicaSets and provides declarative **rolling updates and rollbacks**. This is what you normally create. |
| **Service** | a **stable network endpoint** and internal load balancer in front of a changing set of pods. Because pod IPs change, the Service is what makes them reliably reachable. |
| **Ingress** | HTTP/HTTPS routing from **outside** the cluster into Services — host and path rules, TLS termination |
| **ConfigMap** | non-sensitive configuration injected as environment variables or files |
| **Secret** | sensitive values (base64-encoded, and encrypted at rest when configured) |
| **Namespace** | a virtual cluster used to isolate teams or environments |
| **Node** | a worker machine that runs pods |
| **StatefulSet** | for stateful applications needing stable identities and storage (databases) |
| **DaemonSet** | ensures exactly one pod per node (log collectors, monitoring agents) |
| **Job / CronJob** | run-to-completion and scheduled tasks |
| **HPA** | Horizontal Pod Autoscaler — scales the replica count on CPU or custom metrics |

### Service types — this is where the sample question lives

| Type | Exposure |
|---|---|
| `ClusterIP` | **default** — reachable only from **inside** the cluster |
| `NodePort` | opens a fixed port on every node, reachable from outside |
| `LoadBalancer` | provisions an external cloud load balancer |
| `ExternalName` | maps to an external DNS name |

> ### Working through the official sample question
> *"A Kubernetes application is healthy internally, but users report they cannot access it externally. Pods are running and passing readiness checks. What should be investigated first?"*
>
> **Answer: Service and Ingress configuration.**
>
> **Model reasoning:** *"Passing readiness probes confirms the pods themselves are healthy, so the fault lies on the path from outside the cluster to those pods. That path is defined by the Service and the Ingress — a Service left as the default ClusterIP is only reachable internally, and a misconfigured Ingress rule or missing controller breaks external routing. Deleting pods, upgrading the cluster or adding CPU do not touch the networking layer that the symptom points to."*
>
> Note the shape of that answer: **the symptom (internally healthy, externally broken) identifies the layer, and the layer identifies the object.** That is the reasoning style this exam rewards.

### Probes — know the difference
- **Liveness probe** — "is this container still alive?" On failure Kubernetes **restarts** it.
- **Readiness probe** — "is this container ready to serve traffic?" On failure the pod is **removed from the Service's endpoints** but is **not** restarted.
- **Startup probe** — for slow-starting applications; it suppresses the other probes until it first succeeds.

**A common trap:** a pod can be `Running` yet receive no traffic because its readiness probe is failing. Conversely — as in the sample question — **passing readiness probes rules the pods out and points at Service/Ingress.**

### Debugging ladder — the order a real engineer follows
```
kubectl get pods                    # Running? CrashLoopBackOff? Pending?
kubectl describe pod <name>         # events: image pull errors, scheduling failures, probe failures
kubectl logs <pod>                  # application-level errors
kubectl logs <pod> --previous       # logs from the crashed instance
kubectl get svc / kubectl get ingress
kubectl describe svc <name>         # are the endpoints populated? empty means a selector mismatch
kubectl exec -it <pod> -- sh        # test connectivity from inside the cluster
```
**Pod status meanings:** `Pending` = cannot be scheduled (insufficient resources or no matching node). `ImagePullBackOff` = wrong image name, tag or registry credentials. `CrashLoopBackOff` = the container starts then exits repeatedly, so read the application logs. `OOMKilled` = it exceeded its memory limit.

**A Service with no endpoints almost always means its `selector` labels do not match the pods' labels.** That is the most common Kubernetes networking bug and a strong, specific detail to cite in a reasoning box.

### Kubernetes architecture
- **Control plane:** `kube-apiserver` (the front door for all commands), `etcd` (the distributed key-value store holding all cluster state), `kube-scheduler` (assigns pods to nodes), `kube-controller-manager` (reconciliation loops).
- **Worker nodes:** `kubelet` (starts and supervises containers on that node), `kube-proxy` (networking and Service routing), and a container runtime (containerd).

**The central idea to state in any Kubernetes reasoning box:** *"Kubernetes is declarative — you describe the desired state in YAML, and controllers continuously reconcile the actual state towards it. That reconciliation loop is what delivers self-healing and rolling updates."*

---

## 7. Cloud, IaC and monitoring — enough to answer confidently

**Service models:**
- **IaaS** — you rent raw infrastructure (EC2, virtual machines). You manage the OS upward.
- **PaaS** — you deploy code onto a managed platform (Heroku, App Engine). The provider manages the OS and runtime.
- **SaaS** — you consume finished software (Gmail, Salesforce).
- **FaaS / Serverless** — you deploy individual functions (AWS Lambda); they scale to zero and you pay per invocation. Trade-offs: no server management and elastic cost, versus cold starts, execution time limits and vendor lock-in.

**Infrastructure as Code (IaC):** define infrastructure in version-controlled files (Terraform, CloudFormation, Ansible) rather than clicking through a console. Benefits: reproducibility, review, and disaster recovery. **Declarative** tools (Terraform) describe the desired end state; **imperative** ones describe the steps.

**Monitoring and observability:**
- **Metrics** — numeric time series (Prometheus + Grafana).
- **Logs** — event records (the ELK stack: Elasticsearch, Logstash, Kibana).
- **Traces** — a request's path across services (Jaeger, OpenTelemetry).
- **The three pillars of observability are metrics, logs and traces.** Monitoring tells you *that* something is wrong; observability helps you work out *why*.
- **SLI / SLO / SLA:** an *indicator* is what you measure (latency), an *objective* is your internal target (99.9%), and an *agreement* is the contractual promise with consequences.

---

## 8. Reasoning phrases you can reuse verbatim

- *"A Jenkins Pipeline is a set of automated stages defined as code in a Jenkinsfile that build, test and deploy an application — it represents the CI/CD workflow, not a runtime environment."*
- *"Containers virtualise the operating system and share the host kernel, so they are far lighter and faster to start than virtual machines, which virtualise hardware and carry a full guest OS."*
- *"An image is an immutable template; a container is a running instance of it, in the same relationship as a class and an object."*
- *"Pods are ephemeral and their IPs change, so a Service provides a stable endpoint and load-balances across the current set of pods."*
- *"Since readiness probes pass, the pods are healthy and the fault lies in external exposure — the Service type and the Ingress rules."*
- *"Kubernetes is declarative: controllers continuously reconcile actual state towards the desired state, which is what provides self-healing and rolling updates."*
- *"Continuous Delivery keeps every passing build releasable behind a manual approval; Continuous Deployment removes that approval and ships automatically."*
- *"Rebasing rewrites commit history, so it must never be applied to commits already shared with others; merge preserves history and is safe on shared branches."*
- *"Infrastructure as code makes environments reproducible and reviewable, eliminating the configuration drift that manual provisioning causes."*

---

## 9. Self test (cover the answers)

1. What is a Jenkins Pipeline? -> *A set of automated stages defined in a Jenkinsfile representing the CI/CD workflow.*
2. Container vs VM? -> *Container virtualises the OS and shares the host kernel; a VM virtualises hardware and carries a full guest OS.*
3. Image vs container? -> *Blueprint vs running instance.*
4. Smallest deployable unit in Kubernetes? -> *A Pod.*
5. Why does Kubernetes need a Service if pods have IPs? -> *Pod IPs are ephemeral; the Service gives a stable endpoint and load-balances across the live pods.*
6. Liveness vs readiness probe? -> *Liveness failure restarts the container; readiness failure only removes it from the Service endpoints.*
7. Application unreachable from outside but pods are healthy — first check? -> *Service type and Ingress configuration.*
8. Continuous Delivery vs Continuous Deployment? -> *Manual approval before production versus fully automatic release.*
9. `git merge` vs `git rebase`? -> *Merge preserves history with a merge commit; rebase rewrites commits onto a new base for a linear history and must not be used on shared commits.*
10. Why does a Dockerfile copy `package.json` before the source? -> *Layer caching — the dependency install layer is reused when only the source changes.*
11. A Service exists but has no endpoints. Most likely cause? -> *The Service selector labels do not match the pod labels.*
12. What does `CrashLoopBackOff` mean? -> *The container repeatedly starts and exits; inspect the application logs with `kubectl logs --previous`.*

Next: [05_REASONING_PLAYBOOK.md](05_REASONING_PLAYBOOK.md)
