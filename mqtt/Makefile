all: stop start

start:
	@echo "Starting..."
	@kubectl apply -f Namespace.yaml
	@kubectl apply -f ConfigMap.yaml
	@kubectl apply -f Deployment.yaml
	@kubectl apply -f Service.yaml
	@kubectl apply -f ClusterIssuer.yaml
	@kubectl apply -f Gateway.yaml
	@kubectl apply -f HTTPRoute.yaml

stop:
	@echo "Stopping..."
	@kubectl delete -f HTTPRoute.yaml
	@kubectl delete -f Gateway.yaml
	@kubectl delete -f ClusterIssuer.yaml
	@kubectl delete -f Service.yaml
	@kubectl delete -f Deployment.yaml
	@kubectl delete -f ConfigMap.yaml
	@kubectl delete -f Namespace.yaml

