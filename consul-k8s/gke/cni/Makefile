HELM_CHART_DIR=$(HOME)/workspace/github.com/hashicorp/consul-k8s
CNI_DIR=$(HOME)/workspace/github.com/hashicorp/consul-k8s
ACCEPTANCE_DIR=$(HOME)/workspace/github.com/hashicorp/consul-k8s
#DOCKERHUB=curtbushko
DOCKERHUB=hashicorpdev
DOCKER_IMAGE=consul-k8s-control-plane:b95c83c4

CUR_DIR=$(shell pwd)
GIT_REV=$(shell cd $(CNI_DIR) && git rev-parse --short HEAD)
RELEASE=consul

build:
	cd $(CNI_DIR) && GOARCH=amd64 make control-plane-dev-docker
	docker tag consul-k8s-control-plane-dev $(DOCKERHUB)/consul-k8s-control-plane-dev:latest
	docker push $(DOCKERHUB)/consul-k8s-control-plane-dev:latest

deploy-consul:
	cd $(HELM_CHART_DIR) && helm install ${RELEASE} --create-namespace -n consul -f $(CUR_DIR)/helm.values.yaml --set global.imageK8S=$(DOCKERHUB)/consul-k8s-control-plane-dev:latest --set fullnameOverride=consul ./charts/consul
	#cd $(HELM_CHART_DIR) && helm install consul --create-namespace -n consul -f $(CUR_DIR)/helm.values.yaml --set fullnameOverride=consul ./charts/consul

acceptance:
	cd $(ACCEPTANCE_DIR)/acceptance/tests/connect && go test -run TestConnectInjectNamespace -v -p 1 -timeout 20m \
		-enable-gke \
		-enable-enterprise \
		-kubecontext="curt-1" \
		-secondary-kubecontext="curt-2" \
		-enable-transparent-proxy \
		-enable-cni \
		-consul-k8s-image $(DOCKERHUB)/$(DOCKER_IMAGE)

deploy-static:
	kubectl apply -f static-server.yaml -n default
	kubectl apply -f static-client.yaml -n default

podstatus:
	kubectl ns consul 
	kubectl get pods 

wait:
	sleep 60 

delete-consul:
	kubectl ns consul
	echo "Deleteing helm release: ${RELEASE}"
	helm del --debug ${RELEASE}
	kubectl delete pvc -l release="${RELEASE}"
	kubectl get secret | grep --color=auto "${RELEASE}" | grep --color=auto Opaque | grep --color=auto token | awk '{print $1}' | xargs kubectl delete secret

delete-static:
	echo "Deleteing static services"
	kubectl delete deployment static-server -n default || true
	kubectl delete deployment static-client -n default || true

all: delete-consul build deploy-consul wait deploy-static podstatus
reset: delete-consul wait deploy-consul wait deploy-static podstatus

