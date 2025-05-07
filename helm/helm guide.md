Prerequisites:
Helm CLI: Helm, the package manager for Kubernetes, needs to be installed.
Quay.io Account & Image:
You need a Quay.io account.
You should have already built and pushed your Flask web app image to a Quay.io repository. For this example, let's assume your image is at quay.io/<your_quay_username>/my-flask-app:latest.
OpenShift Login: You are logged in to the OpenShift cluster using the oc CLI and are in the correct project (e.g., devops).
Step 1: Create a Helm Chart
Create a directory for your Helm chart:
mkdir flask-app-chart
cd flask-app-chart


Use the helm create command to scaffold a basic chart structure:
helm create my-flask-app

This will create a directory named my-flask-app with the necessary files and subdirectories.
Step 2: Customize the values.yaml
The values.yaml file allows you to configure your deployment. Modify my-flask-app/values.yaml to match your application's needs. Here's an example:

replicaCount: 2

image:
  repository: quay.io/<your_quay_username>/my-flask-app
  pullPolicy: IfNotPresent
  tag: latest # Or specify a specific tag

service:
  name: my-flask-app-service
  type: ClusterIP # Or NodePort, or LoadBalancer, depending on your needs
  port: 5000
  targetPort: 5000

config:
  message: "Hello from OpenShift and Helm!"


Replace <your_quay_username> with your actual Quay.io username.
replicaCount: The number of pod replicas to deploy.
image.repository: Your Flask app's image location in Quay.io.
image.tag: The tag of the image to use (e.g., latest).
service.type: The Kubernetes Service type. ClusterIP is good for internal cluster communication. Use NodePort or LoadBalancer if you need to expose the app externally (check your OpenShift environment).
service.port: The port where the service is exposed.
service.targetPort: The port your Flask app listens on inside the container.
config.message: A configuration value that will be passed to your Flask app via a ConfigMap.
Step 3: Customize the Deployment Template
Edit the deployment template (my-flask-app/templates/deployment.yaml) to use the values from your values.yaml.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "my-flask-app.fullname" . }}
  labels:
    {{- include "my-flask-app.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "my-flask-app.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "my-flask-app.selectorLabels" . | nindent 8 }}
    spec:
      containers:
        - name: my-flask-app-container
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: web
              containerPort: {{ .Values.service.targetPort }}
              protocol: TCP
          env:
            - name: MESSAGE
              valueFrom:
                configMapKeyRef:
                  name: {{ include "my-flask-app.fullname" . }}
                  key: message
```

This template creates a Deployment that uses the image specified in values.yaml.
It sets the number of replicas from values.yaml.
It defines a container port, matching the targetPort in values.yaml.
It retrieves an environment variable MESSAGE from a ConfigMap.
Step 4: Create a Service Template
Create a service template (my-flask-app/templates/service.yaml) to expose your Flask application:

```bash
apiVersion: v1
kind: Service
metadata:
  name: {{ .Values.service.name }}
  labels:
    {{- include "my-flask-app.labels" . | nindent 4 }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: {{ .Values.service.targetPort }}
      protocol: TCP
      name: web
  selector:
    {{- include "my-flask-app.selectorLabels" . | nindent 4 }}
```

This template creates a Service using the configuration from values.yaml.
The Service selects the pods created by the Deployment using the labels defined in the my-flask-app.selectorLabels helper.
Step 5: Create a ConfigMap Template
Create a configmap template (my-flask-app/templates/configmap.yaml):
```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "my-flask-app.fullname" . }}
  labels:
    {{- include "my-flask-app.labels" . | nindent 4 }}
data:
  message: {{ .Values.config.message | quote }}
```

This ConfigMap will hold the message to be displayed by your Flask application.
The value is taken from values.yaml
Step 6: Deploy with Helm
Make sure you are logged in to your OpenShift cluster and have selected the correct project (e.g., devops) using the oc CLI:
```bash
oc login <your_openshift_url> -u <your_username> -p <your_password>
oc project devops
```

Deploy your Helm chart:
helm install my-flask-app ./my-flask-app

This command packages your chart and deploys it to your OpenShift cluster. Helm will use the templates in the templates directory, combined with the values in values.yaml, to create the Kubernetes resources.
Step 7: Verify the Deployment
Check the status of your deployment:
```bash
oc get deployments
oc get pods
oc get services
oc get configmaps
```

If your service type is NodePort, find the node port:
```
oc get svc my-flask-app-service -o yaml | grep nodePort
```

Test your application:
If you used ClusterIP, you'll need to access it from another pod within the cluster.
If you used NodePort, access it via a browser or curl using the OpenShift node's IP address and the node port: http://<node_ip>:<node_port>.
If you used LoadBalancer, use the external IP provided by the LoadBalancer.
Important Notes for OpenShift:
Security Context Constraints (SCCs): OpenShift has a stricter security model than standard Kubernetes. Ensure that the pod's security context aligns with an appropriate SCC. If you encounter permission errors, you might need to adjust the SCC or create a custom one.
Routes: For external access to your application, you might prefer to use OpenShift Routes instead of NodePorts or LoadBalancers. You can create a Route using oc create route.
ImagePullSecrets: If your Quay.io repository is private, you'll need to create a secret in OpenShift to store your Quay.io credentials and reference that secret in your deployment. The oc create secret docker-registry command is helpful for this. You then add an imagePullSecrets section to your values.yaml and deployment template.
Project: Make sure you are working in the correct OpenShift project (namespace) where you want to deploy your application. Use oc project <project_name> to switch projects.
