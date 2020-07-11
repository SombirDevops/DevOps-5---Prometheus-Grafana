# DevOps-5---Prometheus-Grafana

**About the project -** In this project, the motive is to deploy Prometheus & Grafana on top of Kubernetes by creating a Deployment. Also, I'll attach a PVC to both the Deployments so that no data is lost in case of container crash. Both Prometheus & Grafana will also be exposed to outside world.

# What is Prometheus ?

![](/images/logo.png)

Prometheus, a Cloud Native Computing Foundation project, is a systems and service monitoring system. It collects metrics from configured targets at given intervals, evaluates rule expressions, displays the results, and can trigger alerts if some condition is observed to be true.

Prometheus's main distinguishing features as compared to other monitoring systems are:

- a multi-dimensional data model (timeseries defined by metric name and set of key/value dimensions)
- a flexible query language to leverage this dimensionality
- no dependency on distributed storage; single server nodes are autonomous
- timeseries collection happens via a pull model over HTTP
- pushing timeseries is supported via an intermediary gateway
- targets are discovered via service discovery or static configuration
- multiple modes of graphing and dashboarding support
- support for hierarchical and horizontal federation

# What is Grafana ?

![](/images/grafana.png)

The open-source platform for monitoring and observability.

Grafana allows you to query, visualize, alert on and understand your metrics no matter where they are stored. Create, explore, and share dashboards with your team and foster a data driven culture:

- Visualize: Fast and flexible client side graphs with a multitude of options. Panel plugins for many different way to visualize metrics and logs.
- Dynamic Dashboards: Create dynamic & reusable dashboards with template variables that appear as dropdowns at the top of the dashboard.
- Explore Metrics: Explore your data through ad-hoc queries and dynamic drilldown. Split view and compare different time ranges, queries and data sources side by side.
- Explore Logs: Experience the magic of switching from metrics to logs with preserved label filters. Quickly search through all your logs or streaming them live.
- Alerting: Visually define alert rules for your most important metrics. Grafana will continuously evaluate and send notifications to systems like Slack, PagerDuty, VictorOps, OpsGenie.
- Mixed Data Sources: Mix different data sources in the same graph! You can specify a data source on a per-query basis. This works for even custom datasources.

# Integrating Prometheus with Grafana

![](/images/architecture.png)

**Step - 1:** First of all, We need to create a Persistent Volume Claim for Prometheus. The YML code to do the same is as follows :

    apiVersion: v1
      kind: PersistentVolumeClaim
      metadata: 
        name: pvc-prometheus
        labels:
          name: pvcprometheus
      spec:    
        accessModes:
        - ReadWriteOnce
        resources:
          requests:   
           storage: 5Gi
           
          
**Step - 2:** Next, we create a service for exporting the Prometheus using NodePort.

      apiVersion: v1
       kind: Service
       metadata:
         name: prometheus-svc
         labels:
           app: prometheus-service
       spec:
         selector:
           app: prometheus
       type: NodePort
       ports:
       - nodePort: 2300
         port: 9090
         targetPort: 9090
         name: prometheus-port
         
   
**Step - 3:** Now, we create a deployment for Prometheus. I have used an image that was available on Docker Hub. However, you can easily create your own & push it on the Docker Hub as per your requirement. Since creating own image has been discussed in very detail in many of my earlier projects, I am not repeating it here.

The YML code to create the Deployment is as follows:

      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: prometheus-deployment
      spec:
        replicas: 1
        selector:
          matchLabels: 
            tier: monitor
        template:
          metadata:
            labels:
              tier: monitor
          spec:
            containers:
            - name: prometheus
              image: prom/prometheus

              ports:
              - containerPort: 9090

              volumeMounts:
              - name: prometheus-volume
                mountPath: /data

           volumes:
           - name: prometheus-volume
             persistentVolumeClaim:
               claimName: pvc-prometheus
               
 
 **Now, we do the same setup for Grafana**
 
**Step - 4:** Creating a Persistent volume Claim for Grafana

    apiVersion: v1
     kind: PersistentVolumeClaim
     metadata: 
       name: pvc-grafana
       labels:
         name: pvcgrafana
     spec:
       accessModes:
         - ReadWriteOnce
     resources:
       requests:
         storage: 5Gi 
         
         
 **Step - 5:** Next, we create a service to expose the Grafana using NodePort.
 
 
       apiVersion: v1
        kind: Service
        metadata:
          name: grafana-svc
          labels:
            app: grafana-service
        spec:
          selector:
            app: grafana
          type: NodePort
          ports:
          - nodePort: 2301
            port: 3000
            targetPort: 3000
            name: port-grafana
            
            
 **Step - 6:** Now, we create a deployment for Grafana. Here also, I have used an available image for Grafana from Docker Hub.
 
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: grafana
        labels: 
          app: grafana
          tier: deploy
      spec:
        replicas: 1
        selector:
          matchLabels: 
            tier: monitoring
        template:
          metadata:
            labels:
              tier: monitoring
          spec:
            containers:
            - name: grafana-container
              image: grafana/grafana:latest

              ports:
              - containerPort: 3000

              volumeMounts:
              - name: grafana-volume
                mountPath: /var/lib/grafana

            volumes:
            - name: grafana-volume
              persistentVolumeClaim:
                claimName: pvc-grafana
                
                
 **Step - 7:** Now that all our components are ready, we'll create a kustomization file in which we will mention the sequence in which all these files will be created. We only need to run this kustomization file & everything will be done.
 
       apiVersion: kustomize.config.k8s.io/v1beta1
       kind: Kustomization

       resources:
        - pvcprometheus.yml
        - pvcgrafana.yml
        - prometheusdeploy.yml
        - grafanadeploy.yml 
        - prometheusservice.yml  
        - grafanaservice.yml 
        
        
We can create this file by going to the folder in which this file is present & running the following command in Command Prompt -

            kubectl apply -k .
         
         
**Success -** Now, running this kustomization file will do everything. Both your Prometheus & Grafana will be launched on Kubernetes & connected to a PVC to ensure no data loss. Both the pods will be exposed to the outside world. Since we have used Deployment to launch this architecture, we need not worry about the pod crash. the Replica Set working behind the Deployment will relaunch the pods with the same PVC in case they crash somehow. No data will be lost.

All you need to do now is to open the Prometheus & Grafana in your browser using your Minikube IP & the mentioned port & start working !!

![](/images/prometheus.png)

![](/images/gra.png)



**Any suggestions are highly appreciated.**
