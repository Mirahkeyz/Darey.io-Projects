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

Step 3: Update your local Helm chart repository cache:

$ helm repo update

Step 4: Install the Prometheus chart with a release name of your choice (e.g., "prometheus-release") in the "prometheus" namespace:

$ helm install prometheus prometheus-community/prometheus --namespace prometheus --generate-name

Step 5: Verify that prometheus is running:

kubectl get pods -n prometheus

Step 6: Get the prometheus service details:

kubectl get svc -n prometheus

Step 7: Access Prometheus from a web browser:

kubectl port-forward svc/prometheus-<release-name> 9090:9090 -n prometheus

# Installing Grafana Loki

Step 1: Make sure you have Helm installed and initialized in your Kubernetes cluster. If you haven't done so. you can follow the Helm installation guide: https://helm.sh/docs/intro/install/

Step 2: Add the Bitnami Helm chart repository:

$ helm repo add bitnami https://charts.bitnami.com/bitnami

Step 3: Update your local Helm chart repository cache:

$ helm repo update

Step 4: Install the Grafana Loki chart with a release name of your choice (e.g.. "loki-release") in the desired namespace:

$ helm install loki bitnami/loki --namespace loki --generate-name

Step 5: Verify that Grafana Loki is running:

kubectl get pods -n loki

Step 6: Get the Loki service details:

kubectl get svc -n loki

Step 7: Access Loki from a web browser:

$ kubectl port-forward svc/loki-<release-name> 3100:3100 -n loki




















































