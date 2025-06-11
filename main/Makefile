########################################################################
# Kubernetes cluster operations (MLOps)
########################################################################
# create the local Kubernetes cluster
export CLUSTER_NAME=framework-cluster

cluster:
	sudo kind create cluster --config kind.yaml --name $(CLUSTER_NAME)
	sudo kubectl config use-context kind-$(CLUSTER_NAME)
	
	@echo "Listing the nodes in the cluster"
	sudo kubectl get nodes

# delete the local Kubernetes cluster
delete-cluster:
	sudo kind delete cluster --name $(CLUSTER_NAME)

# list Docker images registered in the local cluster
list-images:
	sudo docker exec -it $(CLUSTER_NAME)-control-plane crictl images

########################################################################
# ML engineer operations (ML)
########################################################################
export PORT=5005

# run with hot reloading for development
dev:
	uv run fastapi dev api.py --port $(PORT)

# build the Docker image for our api
build:
	sudo docker buildx build -t framework:v1.0.0 .

# run the Docker container for our api as a standalone docker in your machine
run:
	sudo docker run -it -p $(PORT):5000 framework:v1.0.0

# push the Docker image to the local Kubernetes image registry
push:
	sudo kind load docker-image framework:v1.0.0 --name $(CLUSTER_NAME)

# deploy the Docker image to the local Kubernetes cluster
deploy: build push
	sudo kubectl apply -f manifests/deployment.yaml
	sudo kubectl apply -f manifests/service.yaml
	sudo kubectl wait --for=condition=ready pod -l app=framework --timeout=60s
	sudo kubectl port-forward svc/framework $(PORT):5000

# ping the api to check if it's running
test:
	curl http://localhost:$(PORT)/health