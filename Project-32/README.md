# Deploying And Configuring Monitoring Tools (Prometheus, Alert Manager And Grafana)

Observability and Monitoring are related concepts in the field of software development. However they have distinct purpose in spite of their similarities.

Monitoring is a subset of observability. Its invloves the process of collecting data and information about a system's behavior, performance, and health. Tools or systems are set up to track metrics,logs, events of the system(application)

Monitoring is primarily concerned with measuring and alerting on predefined indicators or thresholds, such as CPU usage, memory usage, error rates, or response times.

In a nutshell monitoring tells or alerts you when something goes wrong with the system(application).

You will agree with me that knowing what is wrong with the system is not enough. We need to find out why things went wrong and come up with proactive measures to prevent it from happening again. This is where Observability comes in.

Observability is a broader and more holistic approach to understanding and troubleshooting complex systems. It emphasizes the ability to gain insights and understanding from the system's internal behavior and state based on the available data. Observability focuses on generating meaningful and actionable insights from the collected data, going beyond simple monitoring and alerting.

Observability involves instrumenting the system to produce high-quality, fine-grained data that can be used to analyze the system's behavior and diagnose issues effectively. It typically involves capturing detailed logs, metrics, traces, and other telemetry data, which are then analyzed through various tools and techniques. By leveraging this comprehensive set of data. observability enables engineers to understand the cause-and-effect relationships within the system, trace the flow of requests, identify performance bottlenecks, and debug complex issues.

# The Three Pillars of Observability
The three pillars of observability are:
1.	Metrics
2.	Logs
3.	Traces

Metrics:

Metrics can be a wide range of values, monitored over a period of time. Metrics are often key performance indicators such as CPU capacity, memory usage, latency, or anything else that provides insights into the health and performance of your system. The changes in these metrics allows teams to gain a better understanding of the end performance of the system. Metrics provide modern businesses a measurable means by which to improve the user experience

Prometheus is a popular choice for collecting metrics data from applications

Logs:

Logs are the historical records of your systems. They are time-stamped and often come in either binary or plain text as well as structured logs which combine text and metadata, making them easier to query. Logs allow you to look back and see what's gone wrong within a system

Tools like Grafana Loki, Greylog are example of tools used to collect log data from applications

Traces:

Traces are a way to record a user's journey through your application or system. A trace records the user's interaction and requests within the system, starting from the user interface through to the backend systems, and then back to the user once their request has been processed. Every operation performed, from clicking on a tab within your application to that tab loading in the GUI, is recorded as part of the trace. This becomes especially useful when a single request goes through the multiple containerized microservices that are so common in today's complex applications. Traces help SREs and other ITOps and DevOps team members quickly identify chokepoints or breakdowns within the application or system.

Jaeger. Zipkin and other enterprise tools like datadog, newrelic are good for collecting tracing data of application.

By combining the various forms of telemetry data(logs, metrics, traces) DevOps teams are able to understand the internal state of the systems they are responsible for. They are able to respond to incidence quickly, set good Service Level Agreement(SLA), and based on the insight they gather, come up with proactive measures to minimize downtime.

# Deploying Prometheus, Grafana, Loki in a Kubernetes cluster

In this section we go over steps to install and configure monitoring tools. We will be deploying Prometheus for collecting metrics, Loki for logs and finally Grafana for visualizing the data.

# Installing Prometheus

Step 1: Make sure you have Helm installed and initialized in your Kubernetes cluster. If you haven't done so, you can follow the Helm installation guide: https://helmsh/docs/intro/install/

Step 2: Add the Prometheus Helm chart repository:
 
$ helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

![Snipe 1](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/43462a20-cf4a-4c10-994c-23fb8d587447)

Step 3: Update your local Helm chart repository cache:

$ helm repo update

![Snipe 2](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/82d4bb7f-0450-405a-b4f2-4ece77f93128)

Step 4: Install the Prometheus chart with a release name of your choice (e.g., "prometheus-release") in the "prometheus" namespace:

$ helm install my-prometheus prometheus-community/prometheus --namespace prometheus

![Snipe 3](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/c2b8e4f0-cfae-4cce-984f-e677159e3231)

Step 5: Verify that prometheus is running:

kubectl get pods -n prometheus

![Snipe 4](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/38855247-44c0-43e9-b61c-43bf48aefa95)

Step 6: Get the prometheus service details:

$ kubectl get svc -n prometheus

![Snipe 5](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/e6211a7d-adab-4921-9d43-5c66f3037837)

Step 7: Access Prometheus from a web browser:

$ kubectl port-forward svc/my-prometheus-server 8080:80 -n prometheus

![Snipe 6](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/4f40e110-7497-4d41-96c0-091272e03be0)

![Snipe 7](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/c502ff6c-8873-4293-85d2-fed7aff4e125)

# Installing Grafana Loki

Step 1: Make sure you have Helm installed and initialized in your Kubernetes cluster. If you haven't done so. you can follow the Helm installation guide: https://helm.sh/docs/intro/install/

Step 2: Add the Bitnami Helm chart repository:

$ helm repo add bitnami https://charts.bitnami.com/bitnami

![Snipe 8](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/dbaa1a12-9416-420d-b5aa-486fc02da8c2)

Step 3: Update your local Helm chart repository cache:

$ helm repo update

![Snipe 9](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/90289670-3708-470f-8b8d-dc29aebf27eb)

Step 4: Install the Grafana Loki chart with a release name of your choice (e.g.. "loki-release") in the desired namespace:

$ helm install my-loki bitnami/loki --namespace loki

![Snipe 10](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/597be7c2-2f19-4589-b31f-53107c4da7ae)

Step 5: Verify that Grafana Loki is running:

$ kubectl get pods -n loki

![Snipe 11](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/fde81d3d-cc48-43e6-a047-65a9cf96a14a)

Step 6: Get the Loki service details:

$ kubectl get svc -n loki

![Snipe 12](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/e9731074-0191-4c18-a1c8-6b14645201c0)

Step 7: Access Loki from a web browser:

$ kubectl port-forward svc/my-loki-grafana-loki-querier 3100:3100 -n loki

![Snipe 13](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/4d6b1756-7d83-427d-8a00-836f6d3418d6)

# Installing Grafana

Step 1: Add the Grafana Helm chart repository:

$ helm repo add grafana https://grafana.github.io/helm-charts

![Snipe 14](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/21690000-b8d0-464c-87c6-2991b55ce456)

Step 2: Update your local Helm chart repository cache:

$ helm repo update

![Snipe 15](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/7493cdc6-e63f-4267-a135-d62bf1ada665)

Step 3: Install the Grafana chart with a release name of your choice (e.g., "grafana-release") in the desired namespace:

$ helm install grafana grafana/grafana --namespace grafana --generate-name

![Snipe 16](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/873497f5-d3b7-4108-beb4-ee845aab1481)

Step 4: Verify that Grafana is running:

$ kubectl get pods -n grafana

![Snipe 17](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/612771a6-803c-423b-9326-7cd187de7191)

Step 5: Get the Grafana service details:

$ kubectl get svc -n grafana

![Snipe 18](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/2391ff6e-acef-4653-8bde-393a446d7243)

Step 6: Access Grafana from a web browser:

$ kubectl port-forward svc/grafana-<release-name> 3000:3000 -n grafana



































































