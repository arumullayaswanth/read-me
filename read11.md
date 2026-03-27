```bash
apiVersion: v1                  # Core Kubernetes API
kind: ConfigMap                # Used to store configuration data

metadata:
  name: prometheus-config      # Name of the ConfigMap

data:
  prometheus.yml: |            # Prometheus configuration file (embedded inside YAML)

    global:
      scrape_interval: 15s     # Prometheus collects metrics every 15 seconds

    scrape_configs:            # List of monitoring jobs

    - job_name: 'prometheus'   # Job name (used as label in metrics)
      static_configs:
      - targets: ['localhost:9090']
        # Prometheus monitors itself
        # 'localhost:9090' = Prometheus server running inside the same pod

    - job_name: 'node-exporter'   # Job for node-level monitoring
      static_configs:
      - targets: ['localhost:9100']
        # Assumes Node Exporter is running on port 9100
        # 'localhost' means same pod/node
        # In real setups, node exporter runs as DaemonSet on each node
        # and is usually accessed via service or service discovery
```
```bash
apiVersion: apps/v1              # API version for workload resources
kind: DaemonSet                 # Ensures one pod runs on every node

metadata:
  name: prometheus              # Name of the DaemonSet
  labels:
    app: prometheus             # Labels for identification

spec:
  selector:
    matchLabels:
      app: prometheus           # Matches pods with this label

  updateStrategy:
    type: RollingUpdate         # Updates pods gradually without downtime

  template:
    metadata:
      labels:
        app: prometheus         # Labels applied to pods

    spec:
      containers:
      - name: prometheus        # Container name
        image: prom/prometheus:v2.52.0   # Fixed Prometheus version
        imagePullPolicy: IfNotPresent    # Pull image only if not present locally

        args:
        - "--config.file=/etc/prometheus/prometheus.yml"
        # Path to Prometheus configuration file (from ConfigMap)

        - "--storage.tsdb.path=/prometheus"
        # Directory where Prometheus stores time-series data

        - "--storage.tsdb.retention.time=24h"
        # Keep data for 24 hours (auto-delete older data)

        ports:
        - containerPort: 9090   # Port where Prometheus UI runs

        resources:
          requests:
            cpu: 200m           # Minimum CPU guaranteed
            memory: 256Mi       # Minimum memory guaranteed
          limits:
            cpu: 500m           # Maximum CPU allowed
            memory: 512Mi       # Maximum memory allowed

        securityContext:
          runAsNonRoot: true            # Prevent running as root user
          runAsUser: 65534              # Run as 'nobody' user
          allowPrivilegeEscalation: false  # Block privilege escalation
          readOnlyRootFilesystem: false   # Must be writable (Prometheus stores data)

        volumeMounts:
        - name: config
          mountPath: /etc/prometheus
          # Mounts ConfigMap here → Prometheus reads config file

        - name: storage
          mountPath: /prometheus
          # Directory where Prometheus stores metrics data

        livenessProbe:
          httpGet:
            path: /-/healthy
            port: 9090
          initialDelaySeconds: 15
          periodSeconds: 20
          # Restarts container if unhealthy

        readinessProbe:
          httpGet:
            path: /-/ready
            port: 9090
          initialDelaySeconds: 5
          periodSeconds: 10
          # Ensures traffic is sent only when ready

      volumes:
      - name: config
        configMap:
          name: prometheus-config
          # Links ConfigMap → provides prometheus.yml file

      - name: storage
        emptyDir: {}
        # Temporary storage (deleted when pod restarts)
        # In production → use PersistentVolume (PVC)

      terminationGracePeriodSeconds: 30
      # Time given for graceful shutdown before force kill
```
```bash
apiVersion: v1                  # Core Kubernetes API version
kind: Service                  # Exposes Pods internally or externally

metadata:
  name: prometheus-service     # Name of the Service
  labels:
    app: prometheus            # Label for identification

spec:
  type: NodePort               # Exposes the service on each node's IP (external access)

  selector:
    app: prometheus            # Matches pods with this label (Prometheus pods)

  ports:
  - name: http
    port: 9090                 # Service port (used inside the cluster)
    targetPort: 9090           # Port on the container (Prometheus runs here)
    nodePort: 30090            # External port on node (range: 30000–32767)
```
