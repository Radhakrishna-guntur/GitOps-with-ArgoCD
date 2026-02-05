# Argo CD

If you are looking for a way to manage your Kubernetes applications using GitOps principles, you definitely should check out Argo CD.

In this article, I will show you what Argo CD is, how it works, and how you can use it to deploy and manage your applications in a declarative and automated way.

Let’s get started!

## Key Takeaways

Argo CD uses Git repositories as the source of truth for defining the desired application state.
Argo CD supports configuration management and templating tools, such as Kustomize, Helm, Jsonnet, plain-YAML, and any custom tool configured as a config management plugin.
Argo CD provides a web UI and a CLI that give you a real-time view and control of your application activity, status, and history. You can also integrate Argo CD with webhooks and other Git providers.
What is Argo CD?
Argo CD is a declarative, GitOps continuous delivery tool for Kubernetes that automates the deployment and synchronization of your application states with your Git repositories. It follows the GitOps pattern of using Git repositories as the source of truth for defining the desired application state.

GitOps utilizes Git as a central source for maintaining the desired state of applications and infrastructure, enabling streamlined code development, deployment management, and collaborative workflows.

Argo CD, functioning as a Kubernetes controller, continuously monitors live application states against their desired states defined in Git. You manage operations such as synching using either its web UI or its CLI.

How Argo CD Works
To use Argo CD, you need to have a Git repository that contains your application definitions, configurations, and environments. This is called the source repository. You also need to have one or more Kubernetes clusters where you want to deploy your applications. These are called the target clusters.


The source repository can be any public or private Git repository that Argo CD can access. You can organize your source repository in different ways, such as using a single repository for all your applications and environments or using separate repositories for each application or environment. You can also use different branches or tags to track different versions or stages of your applications.

To deploy an application with Argo CD, you need to create an application object. An application object is a Kubernetes custom resource that defines the following properties:

metadata.name: The name of the application.
spec.source: The source of the application manifests. This includes:
repoURL: The URL of the source repository.
path: The path within the source repository where the application manifests are located.
targetRevision: The branch, tag, or commit SHA of the source repository that defines the desired state of the application.
helm: The Helm specific parameters (optional).
kustomize: The Kustomize specific parameters (optional).
jsonnet: The Jsonnet specific parameters (optional).
plugin: The config management plugin specific parameters (optional).
spec.destination: The destination where the application should be deployed. This includes:
server: The URL of the target cluster API server.
namespace: The namespace within the target cluster where the application should be deployed.
spec.syncPolicy: The sync policy that controls how and when the application should be synced to the desired state (optional).
You can create an application object using the argocd app create command or the web UI. You can also use the argocd app generate-spec command to generate an application object from a local directory or a remote repository.

Once you create an application object, Argo CD will start monitoring the source repository and the target cluster for any changes. Suppose it detects any differences between the desired state (as defined in the source repository) and the live state (as observed in the target cluster). In that case, it will mark the application as OutOfSync and report the differences between the web UI and the CLI.

Depending on your sync policy, you can then sync the application manually or automatically. Syncing an application means applying the desired state to the target cluster using kubectl apply or equivalent commands. You can also choose to prune any resources that are not part of the desired state or override any parameters that are specific to the target environment.

You can also roll back an application to any previous version that was committed in the source repository or to any previous sync operation that was performed by Argo CD. Rolling back an application means applying the previous state to the target cluster using kubectl apply or equivalent commands.

To learn more on how Argo CD works, check our course on GitOps with Argo CD

How to Use Argo CD
Prerequisites
To use Argo CD, you need to install it on your Kubernetes cluster. You can install Argo CD using Helm, Kustomize, or plain-YAML manifests. You can also use the Argo CD Operator to install and manage Argo CD on OpenShift or Kubernetes.

This example uses a Kubernetes cluster created using minikube. I use kubectl installed on my local machine to interact with the cluster.

Installing ArgoCD
The installation process will create a namespace called argocd where all the Argo CD components will run. It will also create a service account, a role, and a role binding for Argo CD to access your cluster resources. Additionally, it will create a service, an ingress, and a secret for exposing the Argo CD API server and web UI.

Follow these steps to install Argo CD on your cluster:

Step 1: Use plain-YAML manifests to install Argo CD from the official release page. You can run the following command to download and apply the latest version of Argo CD:
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
The output should look something like this:


Step 2: Check the status of Argo CD components using the following command:
kubectl get pods -n argocd
The output should look something like this:


This shows that all the Argo CD components are running in the argocd namespace

The default username for accessing the Argo CD web UI and CLI are admin. You can change these credentials using the argocd account update-password command.

Accessing ArgoCD using Web UI
To access the Argo CD web UI, you can use the following URL:

https://<ARGOCD_SERVER_HOST>

Login into the dashboard

Join 1M+ Learners
Learn & Practice DevOps, Cloud, AI, and Much More — All Through Hands-On, Interactive Labs!

Create Your Free Account
We know the username is ‘admin’. We can get the password by running the command:

kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}"
This command will query the secret named argocd-initial-admin-secret in the argocd namespace and output the value of the password field.

The output is base64 encoded. We can decode it here, or using the following command:

echo "password" | base64 -d
Use the decoded password and the username admin to log in.

Installing ArgoCD CLI
To access the Argo CD via a CLI, you have to first install its CLI. The following commands are only for Linux and WSL users. If you are using a different OS, please refer to this guide for the installation process.

curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/download/v2.0.5/argocd-linux-amd64
chmod +x /usr/local/bin/argocd
Once the installation is complete, you can log into the server by running the following command.

argocd login <ARGOCD_SERVER_HOST> --username admin --password <ARGOCD_PASSWORD>
Then check the ArgoCD version:

argocd version
The output of these commands should look something like this:


Now that you have installed and logged in to Argo CD, you are ready to create and deploy your first application. In this example, we will use a simple guestbook application that consists of a web frontend and a Redis backend. The application manifests are defined using plain-YAML and stored in a public GitHub repository.

To create an application object for the guestbook application, run the following command:

argocd app create guestbook --repo https://github.com/argoproj/argocd-example-apps.git --path guestbook --dest-server https://kubernetes.default.svc --dest-namespace default
This command will create an application object named guestbook with the following properties:

spec.source.repoURL: The URL of the source repository is https://github.com/argoproj/argocd-example-apps.git.
spec.source.path: The path within the source repository where the application manifests are located is guestbook.
spec.source.targetRevision: The branch of the source repository that defines the desired state of the application is HEAD (the default value).
spec.destination.server: The URL of the target cluster API server is https://kubernetes.default.svc (the default value).
spec.destination.namespace: The namespace within the target cluster where the application should be deployed is default (the default value).
spec.syncPolicy: The sync policy that controls how and when the application should be synced to the desired state is nil (the default value), which means manual sync.
After creating the application object, you can check its status on the ArgoCD web UI. It should look like this:


To sync the application to the desired state, you just need to click on the ‘SYNC’ button.

Alternatively, you can check the status using the CLI by running the following command:

argocd app get guestbook
The output of this command should look something like this:


This output shows that the application is OutOfSync from the desired state and that none of the resources have been deployed to the target cluster yet.

To sync the application to the desired state, you can use the following command:

argocd app sync guestbook
This command will apply the desired state to the target cluster using kubectl apply. The output of this command will be similar to this:


This output shows that the application is now Synced to the desired state and that all the resources have been deployed to the target cluster successfully.

You can verify that the sync has been completed by running the command:

argocd app get guestbook
We can also do that by checking the Web UI:


Practice working with Argo CD on our ArgoCD Playground. The playground is pre-configured with ArgoCD and Gitea. Gitea is a self-hosted Git service that is used to store files in a repository - an alternative to GitHub. For ease of use, we also provide web user interfaces for both Gitea and ArgoCD.

Conclusion
Congratulations! You have successfully created and deployed your first application with Argo CD. You can now explore more features and options of Argo CD, such as using different configuration management tools, defining different environments, setting up automatic sync, integrating with webhooks, and more.
