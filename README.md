# Deploy a Web App to Kubernetes Engine on GCP

This project demonstrates deploying a simple Bookshelf web application to [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine/).

View the [live app](https://bookshelf-212723.appspot.com).

## Create a cluster

Create a cluster for the bookshelf application:

    gcloud container clusters create bookshelf \
        --scopes "cloud-platform" \
        --num-nodes 2
    gcloud container clusters get-credentials bookshelf

The scopes specified in the `--scopes` argument allows nodes in the cluster to access Google Cloud Platform APIs, such as the Cloud Datastore API.

Alternatively, you can use make:

    make create-cluster

## Create a Cloud Storage bucket

The bookshelf application uses [Google Cloud Storage](https://cloud.google.com/storage) to store image files. Create a bucket for your project:

    gsutil mb gs://<your-project-id>
    gsutil defacl set public-read gs://<your-project-id>

Alternatively, you can use make:

    make create-bucket

## Build the bookshelf container

Before the application can be deployed to Kubernetes Engine, you will need build and push the image to [Google Container Registry](https://cloud.google.com/container-registry/).

    docker build -t gcr.io/<your-project-id>/bookshelf .
    gcloud docker push gcr.io/<your-project-id>/bookshelf

Alternatively, you can use make:

    make push

## Deploy the bookshelf frontend

The bookshelf app has two distinct "tiers". The frontend serves a web interface to create and manage books, while the worker handles fetching book information from the Google Books API.

Update `bookshelf-frontend.yaml` with your Project ID or use `make template`. This file contains the Kubernetes resource definitions to deploy the frontend. You can use `kubectl` to create these resources on the cluster:

    kubectl create -f bookshelf-frontend.yaml

Alternatively, you can use make:

    make deploy-frontend

Once the resources are created, there should be 3 `bookshelf-frontend` pods on the cluster. To see the pods and ensure that they are running:

    kubectl get pods

If the pods are not ready or if you see restarts, you can get the logs for a particular pod to figure out the issue:

    kubectl logs pod-id

Once the pods are ready, you can get the public IP address of the load balancer:

    kubectl get services bookshelf-frontend

You can then browse to the public IP address in your browser to see the bookshelf application.

## Deploy worker

Update `bookshelf-worker.yaml` with your Project ID or use `make template`. This file contains the Kubernetes resource definitions to deploy the worker. The worker doesn't need to serve web traffic or expose any ports, so it has significantly less configuration than the frontend. You can use `kubectl` to create these resources on the cluster:

    kubectl create -f bookshelf-worker.yaml

Alternatively, you can use make:

    make deploy-worker

Once again, use `kubectl get pods` to check the status of the worker pods. Once the worker pods are up and running, you should be able to create books on the frontend and the workers will handle updating book information in the background.
