# Prometheus
## Manage Production Service
- setup
- scale out/up
- backup
- upgrade
- operation

## K8s Operator
	What is Kubernetes Operator
	An Operator is a Software with Specific Application Knowledge
	Extends the Kubernetes through Custom Controller/Resource
	
## Operation of Stateful Service(Operator)

## Maintain Promethueus is Bothering
- Problems Which Encounter With K8S

## K8S CRD Within Prometheus Operator(2/2)
- ServiceMonitor
	- 1 ServiceMonitor Mapping to 1 Exporter Service
	- All ServiceMonitor Compose Prometheus Configuration
- PrometheusRule
	- 1 PrometheusRule Mapping to 1 Rule File

## How to Monitor A New Service
- Precondition
	- Promethueus Install/Manage By Operator
	- Add New Exporter and Related Service
	- Add New ServiceMonitor/Prometheus Rules

## K8S CRD Within Prometheus Operator(1/2)
- Promethueus
	- Manage Prometheus StatefulSet
	- Prometheus Configuration Reloader
	- Promethueus Rules Reloader
- Alertmanger
	- Manage AlertManager Statefule

## Thanos Main Components
- Sidecar
- Querier
- Store
- Compactor