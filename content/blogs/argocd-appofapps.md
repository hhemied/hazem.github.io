+++
title = 'ArgoCD Appofapps'
date = 2024-05-28T21:28:06+02:00
draft = false
+++

# App of Apps
We are going to answer four questions here as follow.
- What problems does the App of Apps pattern solve?
- When is it best to use App of Apps?
- How do we use the App of Apps pattern?
- How does App of Apps enable GitOps?

Argo CD's "native" config management tools are Helm, Jsonnet, and Kustomize

Usually we deploy one application by one application whether using ArgoCD UI or commandline.
But, what if we need to deploy more that one application? How can we handle applications manifests?, best practice here is to group related applications.

To define a group of applications, we need to define a `root` ArgoCD application that will define and sync multiple applications under the hood.

Instead of pointing to an application manifest, the `root` application points to a folder in Git. where we store all applications manifests.

One of the strengths of the App of Apps model is that ArgoCD treats child applications as Kubernetes resources (because they are). It monitors their health and will detect any discrepancy. It will optionally resync them if you choose to do so (automatic sync, self-heal enabled).

![appofapps](../../static/images/logo.png)

Then the `GIT` structure should looks like 
```bash
myapplications/
├── myapps
│   ├── child-app-1.yml
│   ├── child-app-2.yml
│   └── child-app-3.yml
├── myprojects
│   └── myproject.yml
└── rootapps
    └── rootapp.yml
```

Now, what about the `root` application definition?

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: rootapp
  namespace: argocd
  finalizers:
  - resources-finalizer.argocd.argoproj.io
spec:
  destination:
    namespace: myapp
    server: https://kubernets.default.svc
  project: myproject
  source:
    path: myapps
    repoURL: https://github.com/hhemied/app-of-apps
    targetRevision: HEAD
```

Then, myapps directory contains the application manifest for each child app. Within that manifest, your file might look like, 

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: child-app-1
  namespace: argocd
spec:
  destination:
    namespace: myapp
    server: https://kubernetes.default.svc
  project: myproject
  source:
    path: guestbook
    repoURL: https://github.com/argoproj/argocd-example-apps.git
    targetRevision: HEAD
  syncPolicy:
    syncOptions:
    - CreateNamespace=true
    automated:
      selfHeal: true
      prune: true
```

## Use cases
As you mentioned, using AppofApps, we are able to manage and deploy multiple applications as a single one.

The most common scenarios are
- Cluster bootstrapping.

> [!NOTE] Cluster Bootstrapping
> For cluster bootstrapping, imagine that you have a set of applications that you always want to install in a new Kubernetes cluster. Instead of installing those applications one-by-one you can easily group them in a single application and then install that instead.

- Handling ArgoCD application without using the CLI or the GUI.

> [!NOTE] ArgoCD applications
> You can edit an existing root app with just git operations (e.g. add folders in the paths it looks) and have Argo CD automatically deploy applications without going through the Argo CD CLI or the Web UI.


> [!warning] Be aware
> - Using App of Apps also affects how applications are removed from a cluster.
> - `Root` app deletion, lead to deleting all child applications at once
> - Note that this behavior is controlled [by ArgoCD finalizers](https://argo-cd.readthedocs.io/en/stable/user-guide/app_deletion/) that work on both children apps and the root app.


> [!NOTE] Remeber
> a root application is handled like any other Argo CD application. It can be committed to source control and updated via GitOps.
## Training

You need to have a GitHub account for this exercise.
The example application is at https://github.com/codefresh-contrib/gitops-cert-level-2-examples.
Fork this repository in your own Github account.
Objectives
In this track you'll learn:
- What is the App of Apps pattern
- How to deploy/delete multiple applications as a single unit
- How to do bootstrap a cluster with your favorite apps

