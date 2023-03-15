# deploying_ingress_controller_to_k8s
### In this section you will use Helm to install the NGINX Ingress Controller on your Kubernetes Cluster
## 1- Add the following Helm ingress-nginx repository to your Helm repos.
```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
```
## 2- Update your Helm repositories.
```
helm repo update
```
### Update the values file:
```
## nginx configuration
## Ref: https://github.com/kubernetes/ingress-nginx/blob/main/docs/user-guide/nginx-configuration/index.md
##

## Overrides for generated resource names
# See templates/_helpers.tpl
# nameOverride:
# fullnameOverride:

## Labels to apply to all resources
##
commonLabels: {}
# scmhash: abc123
# myLabel: aakkmd

controller:
    name: controller
    image:
        ## Keep false as default for now!
        chroot: false
        registry: registry.k8s.io
        image: ingress-nginx/controller
        ## for backwards compatibility consider setting the full image url via the repository value below
        ## use *either* current default registry/image or repository format or installing chart by providing the values.yaml will fail
        ## repository:
        tag: "v1.6.4"
        digest: sha256:15be4666c53052484dd2992efacf2f50ea77a78ae8aa21ccd91af6baaa7ea22f
        digestChroot: sha256:0de01e2c316c3ca7847ca13b32d077af7910d07f21a4a82f81061839764f8f81
        pullPolicy: IfNotPresent
        # www-data -> uid 101
        runAsUser: 101
        allowPrivilegeEscalation: true
    # -- Use an existing PSP instead of creating one
    existingPsp: ""
    # -- Configures the controller container name
    containerName: controller
    # -- Configures the ports that the nginx-controller listens on
    containerPort:
        http: 80
        https: 443
    # -- Will add custom configuration options to Nginx https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/configmap/
    config: {}
    # -- Annotations to be added to the controller config configuration configmap.
    configAnnotations: {}
    # -- Will add custom headers before sending traffic to backends according to https://github.com/kubernetes/ingress-nginx/tree/main/docs/examples/customization/custom-headers
    proxySetHeaders: {}
    # -- Will add custom headers before sending response traffic to the client according to: https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/configmap/#add-headers
    addHeaders: {}
    # -- Optionally customize the pod dnsConfig.
    dnsConfig: {}
    # -- Optionally customize the pod hostname.
    hostname: {}
    # -- Optionally change this to ClusterFirstWithHostNet in case you have 'hostNetwork: true'.
    # By default, while using host network, name resolution uses the host's DNS. If you wish nginx-controller
    # to keep resolving names inside the k8s network, use ClusterFirstWithHostNet.
    dnsPolicy: ClusterFirst
    # -- Bare-metal considerations via the host network https://kubernetes.github.io/ingress-nginx/deploy/baremetal/#via-the-host-network
    # Ingress status was blank because there is no Service exposing the NGINX Ingress controller in a configuration using the host network, the default --publish-service flag used in standard cloud setups does not apply
    reportNodeInternalIp: false
    # -- Process Ingress objects without ingressClass annotation/ingressClassName field
    # Overrides value for --watch-ingress-without-class flag of the controller binary
    # Defaults to false
    watchIngressWithoutClass: false
    # -- Process IngressClass per name (additionally as per spec.controller).
    ingressClassByName: false
    # -- This configuration enables Topology Aware Routing feature, used together with service annotation service.kubernetes.io/topology-aware-hints="auto"
    # Defaults to false
    enableTopologyAwareRouting: false
    # -- This configuration defines if Ingress Controller should allow users to set
    # their own *-snippet annotations, otherwise this is forbidden / dropped
    # when users add those annotations.
    # Global snippets in ConfigMap are still respected
    allowSnippetAnnotations: true
    # -- Required for use with CNI based kubernetes installations (such as ones set up by kubeadm),
    # since CNI and hostport don't mix yet. Can be deprecated once https://github.com/kubernetes/kubernetes/issues/23920
    # is merged
    hostNetwork: false
    ## Use host ports 80 and 443
    ## Disabled by default
    hostPort:
        # -- Enable 'hostPort' or not
        enabled: false
        ports:
            # -- 'hostPort' http port
            http: 80
            # -- 'hostPort' https port
            https: 443
    # -- Election ID to use for status update, by default it uses the controller name combined with a suffix of 'leader'
    electionID: ""
    ## This section refers to the creation of the IngressClass resource
    ## IngressClass resources are supported since k8s >= 1.18 and required since k8s >= 1.19
    ingressClassResource:
        # -- Name of the ingressClass
        name: nginx
        # -- Is this ingressClass enabled or not
        enabled: true
        # -- Is this the default ingressClass for the cluster
        default: false
        # -- Controller-value of the controller that is processing this ingressClass
        controllerValue: "k8s.io/ingress-nginx"
        # -- Parameters is a link to a custom resource containing additional
        # configuration for the controller. This is optional if the controller
        # does not require extra parameters.
        parameters: {}
    # -- For backwards compatibility with ingress.class annotation, use ingressClass.
    # Algorithm is as follows, first ingressClassName is considered, if not present, controller looks for ingress.class annotation
    ingressClass: nginx
    # -- Labels to add to the pod container metadata
    podLabels: {}
    #  key: value

    # -- Security Context policies for controller pods
    podSecurityContext: {}
    # -- See https://kubernetes.io/docs/tasks/administer-cluster/sysctl-cluster/ for notes on enabling and using sysctls
    sysctls: {}
    # sysctls:
    #   "net.core.somaxconn": "8192"

    # -- Allows customization of the source of the IP address or FQDN to report
    # in the ingress status field. By default, it reads the information provided
    # by the service. If disable, the status field reports the IP address of the
    # node or nodes where an ingress controller pod is running.
    publishService:
        # -- Enable 'publishService' or not
        enabled: true
        # -- Allows overriding of the publish service to bind to
        # Must be <namespace>/<service_name>
        pathOverride: ""
    # Limit the scope of the controller to a specific namespace
    scope:
        # -- Enable 'scope' or not
        enabled: false
        # -- Namespace to limit the controller to; defaults to $(POD_NAMESPACE)
        namespace: ""
        # -- When scope.enabled == false, instead of watching all namespaces, we watching namespaces whose labels
        # only match with namespaceSelector. Format like foo=bar. Defaults to empty, means watching all namespaces.
        namespaceSelector: ""
    # -- Allows customization of the configmap / nginx-configmap namespace; defaults to $(POD_NAMESPACE)
    configMapNamespace: ""
    tcp:
        # -- Allows customization of the tcp-services-configmap; defaults to $(POD_NAMESPACE)
        configMapNamespace: ""
        # -- Annotations to be added to the tcp config configmap
        annotations: {}
    udp:
        # -- Allows customization of the udp-services-configmap; defaults to $(POD_NAMESPACE)
        configMapNamespace: ""
        # -- Annotations to be added to the udp config configmap
        annotations: {}
    # -- Maxmind license key to download GeoLite2 Databases.
    ## https://blog.maxmind.com/2019/12/18/significant-changes-to-accessing-and-using-geolite2-databases
    maxmindLicenseKey: ""
    # -- Additional command line arguments to pass to nginx-ingress-controller
    # E.g. to specify the default SSL certificate you can use
    extraArgs: {}
    ## extraArgs:
    ##   default-ssl-certificate: "<namespace>/<secret_name>"

    # -- Additional environment variables to set
    extraEnvs: []
    # extraEnvs:
    #   - name: FOO
    #     valueFrom:
    #       secretKeyRef:
    #         key: FOO
    #         name: secret-resource

    # -- Use a `DaemonSet` or `Deployment`
    kind: Deployment
    # -- Annotations to be added to the controller Deployment or DaemonSet
    ##
    annotations: {}
    #  keel.sh/pollSchedule: "@every 60m"

    # -- Labels to be added to the controller Deployment or DaemonSet and other resources that do not have option to specify labels
    ##
    labels: {}
    #  keel.sh/policy: patch
    #  keel.sh/trigger: poll

    # -- The update strategy to apply to the Deployment or DaemonSet
    ##
    updateStrategy: {}
    #  rollingUpdate:
    #    maxUnavailable: 1
    #  type: RollingUpdate

    # -- `minReadySeconds` to avoid killing pods before we are ready
    ##
    minReadySeconds: 0
    # -- Node tolerations for server scheduling to nodes with taints
    ## Ref: https://kubernetes.io/docs/concepts/configuration/assign-pod-node/
    ##
    tolerations: []
    #  - key: "key"
    #    operator: "Equal|Exists"
    #    value: "value"
    #    effect: "NoSchedule|PreferNoSchedule|NoExecute(1.6 only)"

    # -- Affinity and anti-affinity rules for server scheduling to nodes
    ## Ref: https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity
    ##
    affinity: {}
    # # An example of preferred pod anti-affinity, weight is in the range 1-100
    # podAntiAffinity:
    #   preferredDuringSchedulingIgnoredDuringExecution:
    #   - weight: 100
    #     podAffinityTerm:
    #       labelSelector:
    #         matchExpressions:
    #         - key: app.kubernetes.io/name
    #           operator: In
    #           values:
    #           - ingress-nginx
    #         - key: app.kubernetes.io/instance
    #           operator: In
    #           values:
    #           - ingress-nginx
    #         - key: app.kubernetes.io/component
    #           operator: In
    #           values:
    #           - controller
    #       topologyKey: kubernetes.io/hostname

    # # An example of required pod anti-affinity
    # podAntiAffinity:
    #   requiredDuringSchedulingIgnoredDuringExecution:
    #   - labelSelector:
    #       matchExpressions:
    #       - key: app.kubernetes.io/name
    #         operator: In
    #         values:
    #         - ingress-nginx
    #       - key: app.kubernetes.io/instance
    #         operator: In
    #         values:
    #         - ingress-nginx
    #       - key: app.kubernetes.io/component
    #         operator: In
    #         values:
    #         - controller
    #     topologyKey: "kubernetes.io/hostname"

    # -- Topology spread constraints rely on node labels to identify the topology domain(s) that each Node is in.
    ## Ref: https://kubernetes.io/docs/concepts/workloads/pods/pod-topology-spread-constraints/
    ##
    topologySpreadConstraints: []
    # - maxSkew: 1
    #   topologyKey: topology.kubernetes.io/zone
    #   whenUnsatisfiable: DoNotSchedule
    #   labelSelector:
    #     matchLabels:
    #       app.kubernetes.io/instance: ingress-nginx-internal

    # -- `terminationGracePeriodSeconds` to avoid killing pods before we are ready
    ## wait up to five minutes for the drain of connections
    ##
    terminationGracePeriodSeconds: 300
    # -- Node labels for controller pod assignment
    ## Ref: https://kubernetes.io/docs/user-guide/node-selection/
    ##
    nodeSelector:
        kubernetes.io/os: linux
    ## Liveness and readiness probe values
    ## Ref: https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes
    ##
    ## startupProbe:
    ##   httpGet:
    ##     # should match container.healthCheckPath
    ##     path: "/healthz"
    ##     port: 10254
    ##     scheme: HTTP
    ##   initialDelaySeconds: 5
    ##   periodSeconds: 5
    ##   timeoutSeconds: 2
    ##   successThreshold: 1
    ##   failureThreshold: 5
    livenessProbe:
        httpGet:
            # should match container.healthCheckPath
            path: "/healthz"
            port: 10254
            scheme: HTTP
        initialDelaySeconds: 10
        periodSeconds: 10
        timeoutSeconds: 1
        successThreshold: 1
        failureThreshold: 5
    readinessProbe:
        httpGet:
            # should match container.healthCheckPath
            path: "/healthz"
            port: 10254
            scheme: HTTP
        initialDelaySeconds: 10
        periodSeconds: 10
        timeoutSeconds: 1
        successThreshold: 1
        failureThreshold: 3
    # -- Path of the health check endpoint. All requests received on the port defined by
    # the healthz-port parameter are forwarded internally to this path.
    healthCheckPath: "/healthz"
    # -- Address to bind the health check endpoint.
    # It is better to set this option to the internal node address
    # if the ingress nginx controller is running in the `hostNetwork: true` mode.
    healthCheckHost: ""
    # -- Annotations to be added to controller pods
    ##
    podAnnotations: {}
    replicaCount: 1
    # -- Define either 'minAvailable' or 'maxUnavailable', never both.
    minAvailable: 1
    # -- Define either 'minAvailable' or 'maxUnavailable', never both.
    # maxUnavailable: 1

    ## Define requests resources to avoid probe issues due to CPU utilization in busy nodes
    ## ref: https://github.com/kubernetes/ingress-nginx/issues/4735#issuecomment-551204903
    ## Ideally, there should be no limits.
    ## https://engineering.indeedblog.com/blog/2019/12/cpu-throttling-regression-fix/
    resources:
        ##  limits:
        ##    cpu: 100m
        ##    memory: 90Mi
        requests:
            cpu: 100m
            memory: 90Mi
    # Mutually exclusive with keda autoscaling
    autoscaling:
        apiVersion: autoscaling/v2
        enabled: false
        annotations: {}
        minReplicas: 1
        maxReplicas: 11
        targetCPUUtilizationPercentage: 50
        targetMemoryUtilizationPercentage: 50
        behavior: {}
        # scaleDown:
        #   stabilizationWindowSeconds: 300
        #   policies:
        #   - type: Pods
        #     value: 1
        #     periodSeconds: 180
        # scaleUp:
        #   stabilizationWindowSeconds: 300
        #   policies:
        #   - type: Pods
        #     value: 2
        #     periodSeconds: 60
    autoscalingTemplate: []
    # Custom or additional autoscaling metrics
    # ref: https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/#support-for-custom-metrics
    # - type: Pods
    #   pods:
    #     metric:
    #       name: nginx_ingress_controller_nginx_process_requests_total
    #     target:
    #       type: AverageValue
    #       averageValue: 10000m

    # Mutually exclusive with hpa autoscaling
    keda:
        apiVersion: "keda.sh/v1alpha1"
        ## apiVersion changes with keda 1.x vs 2.x
        ## 2.x = keda.sh/v1alpha1
        ## 1.x = keda.k8s.io/v1alpha1
        enabled: false
        minReplicas: 1
        maxReplicas: 11
        pollingInterval: 30
        cooldownPeriod: 300
        restoreToOriginalReplicaCount: false
        scaledObject:
            annotations: {}
            # Custom annotations for ScaledObject resource
            #  annotations:
            # key: value
        triggers: []
        #     - type: prometheus
        #       metadata:
        #         serverAddress: http://<prometheus-host>:9090
        #         metricName: http_requests_total
        #         threshold: '100'
        #         query: sum(rate(http_requests_total{deployment="my-deployment"}[2m]))

        behavior: {}
    #     scaleDown:
    #       stabilizationWindowSeconds: 300
    #       policies:
    #       - type: Pods
    #         value: 1
    #         periodSeconds: 180
    #     scaleUp:
    #       stabilizationWindowSeconds: 300
    #       policies:
    #       - type: Pods
    #         value: 2
    #         periodSeconds: 60

    # -- Enable mimalloc as a drop-in replacement for malloc.
    ## ref: https://github.com/microsoft/mimalloc
    ##
    enableMimalloc: true
    ## Override NGINX template
    customTemplate:
        configMapName: ""
        configMapKey: ""
    service:
        enabled: true
        # -- If enabled is adding an appProtocol option for Kubernetes service. An appProtocol field replacing annotations that were
        # using for setting a backend protocol. Here is an example for AWS: service.beta.kubernetes.io/aws-load-balancer-backend-protocol: http
        # It allows choosing the protocol for each backend specified in the Kubernetes service.
        # See the following GitHub issue for more details about the purpose: https://github.com/kubernetes/kubernetes/issues/40244
        # Will be ignored for Kubernetes versions older than 1.20
        ##
        appProtocol: true
        annotations: {}
        labels: {}
        # clusterIP: ""

        # -- List of IP addresses at which the controller services are available
        ## Ref: https://kubernetes.io/docs/user-guide/services/#external-ips
        ##
        externalIPs: []
        # -- Used by cloud providers to connect the resulting `LoadBalancer` to a pre-existing static IP according to https://kubernetes.io/docs/concepts/services-networking/service/#loadbalancer
        loadBalancerIP: ""
        loadBalancerSourceRanges: []
        enableHttp: true
        enableHttps: true
        ## Set external traffic policy to: "Local" to preserve source IP on providers supporting it.
        ## Ref: https://kubernetes.io/docs/tutorials/services/source-ip/#source-ip-for-services-with-typeloadbalancer
        # externalTrafficPolicy: ""

        ## Must be either "None" or "ClientIP" if set. Kubernetes will default to "None".
        ## Ref: https://kubernetes.io/docs/concepts/services-networking/service/#virtual-ips-and-service-proxies
        # sessionAffinity: ""

        ## Specifies the health check node port (numeric port number) for the service. If healthCheckNodePort isn’t specified,
        ## the service controller allocates a port from your cluster’s NodePort range.
        ## Ref: https://kubernetes.io/docs/tasks/access-application-cluster/create-external-load-balancer/#preserving-the-client-source-ip
        # healthCheckNodePort: 0

        # -- Represents the dual-stack-ness requested or required by this Service. Possible values are
        # SingleStack, PreferDualStack or RequireDualStack.
        # The ipFamilies and clusterIPs fields depend on the value of this field.
        ## Ref: https://kubernetes.io/docs/concepts/services-networking/dual-stack/
        ipFamilyPolicy: "SingleStack"
        # -- List of IP families (e.g. IPv4, IPv6) assigned to the service. This field is usually assigned automatically
        # based on cluster configuration and the ipFamilyPolicy field.
        ## Ref: https://kubernetes.io/docs/concepts/services-networking/dual-stack/
        ipFamilies:
            - IPv4
        ports:
            http: 80
            https: 443
        targetPorts:
            http: http
            https: https
        type: LoadBalancer
        ## type: NodePort
        ## nodePorts:
        ##   http: 32080
        ##   https: 32443
        ##   tcp:
        ##     8080: 32808
        nodePorts:
            http: ""
            https: ""
            tcp: {}
            udp: {}
        external:
            enabled: true
        internal:
            # -- Enables an additional internal load balancer (besides the external one).
            enabled: false
            # -- Annotations are mandatory for the load balancer to come up. Varies with the cloud service.
            annotations: {}
            # loadBalancerIP: ""

            # -- Restrict access For LoadBalancer service. Defaults to 0.0.0.0/0.
            loadBalancerSourceRanges: []
            ## Set external traffic policy to: "Local" to preserve source IP on
            ## providers supporting it
            ## Ref: https://kubernetes.io/docs/tutorials/services/source-ip/#source-ip-for-services-with-typeloadbalancer
            # externalTrafficPolicy: ""
    # shareProcessNamespace enables process namespace sharing within the pod.
    # This can be used for example to signal log rotation using `kill -USR1` from a sidecar.
    shareProcessNamespace: false
    # -- Additional containers to be added to the controller pod.
    # See https://github.com/lemonldap-ng-controller/lemonldap-ng-controller as example.
    extraContainers: []
    #  - name: my-sidecar
    #    image: nginx:latest
    #  - name: lemonldap-ng-controller
    #    image: lemonldapng/lemonldap-ng-controller:0.2.0
    #    args:
    #      - /lemonldap-ng-controller
    #      - --alsologtostderr
    #      - --configmap=$(POD_NAMESPACE)/lemonldap-ng-configuration
    #    env:
    #      - name: POD_NAME
    #        valueFrom:
    #          fieldRef:
    #            fieldPath: metadata.name
    #      - name: POD_NAMESPACE
    #        valueFrom:
    #          fieldRef:
    #            fieldPath: metadata.namespace
    #    volumeMounts:
    #    - name: copy-portal-skins
    #      mountPath: /srv/var/lib/lemonldap-ng/portal/skins

    # -- Additional volumeMounts to the controller main container.
    extraVolumeMounts: []
    #  - name: copy-portal-skins
    #   mountPath: /var/lib/lemonldap-ng/portal/skins

    # -- Additional volumes to the controller pod.
    extraVolumes: []
    #  - name: copy-portal-skins
    #    emptyDir: {}

    # -- Containers, which are run before the app containers are started.
    extraInitContainers: []
    # - name: init-myservice
    #   image: busybox
    #   command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']

    # -- Modules, which are mounted into the core nginx image. See values.yaml for a sample to add opentelemetry module
    extraModules: []
    #   containerSecurityContext:
    #     allowPrivilegeEscalation: false
    #
    # The image must contain a `/usr/local/bin/init_module.sh` executable, which
    # will be executed as initContainers, to move its config files within the
    # mounted volume.

    opentelemetry:
        enabled: false
        image: registry.k8s.io/ingress-nginx/opentelemetry:v20230107-helm-chart-4.4.2-2-g96b3d2165@sha256:331b9bebd6acfcd2d3048abbdd86555f5be76b7e3d0b5af4300b04235c6056c9
        containerSecurityContext:
            allowPrivilegeEscalation: false
    admissionWebhooks:
        annotations: {}
        # ignore-check.kube-linter.io/no-read-only-rootfs: "This deployment needs write access to root filesystem".

        ## Additional annotations to the admission webhooks.
        ## These annotations will be added to the ValidatingWebhookConfiguration and
        ## the Jobs Spec of the admission webhooks.
        enabled: true
        # -- Additional environment variables to set
        extraEnvs: []
        # extraEnvs:
        #   - name: FOO
        #     valueFrom:
        #       secretKeyRef:
        #         key: FOO
        #         name: secret-resource
        # -- Admission Webhook failure policy to use
        failurePolicy: Fail
        # timeoutSeconds: 10
        port: 8443
        certificate: "/usr/local/certificates/cert"
        key: "/usr/local/certificates/key"
        namespaceSelector: {}
        objectSelector: {}
        # -- Labels to be added to admission webhooks
        labels: {}
        # -- Use an existing PSP instead of creating one
        existingPsp: ""
        networkPolicyEnabled: false
        service:
            annotations: {}
            # clusterIP: ""
            externalIPs: []
            # loadBalancerIP: ""
            loadBalancerSourceRanges: []
            servicePort: 443
            type: ClusterIP
        createSecretJob:
            securityContext:
                allowPrivilegeEscalation: false
            resources: {}
            # limits:
            #   cpu: 10m
            #   memory: 20Mi
            # requests:
            #   cpu: 10m
            #   memory: 20Mi
        patchWebhookJob:
            securityContext:
                allowPrivilegeEscalation: false
            resources: {}
        patch:
            enabled: true
            image:
                registry: registry.k8s.io
                image: ingress-nginx/kube-webhook-certgen
                ## for backwards compatibility consider setting the full image url via the repository value below
                ## use *either* current default registry/image or repository format or installing chart by providing the values.yaml will fail
                ## repository:
                tag: v20220916-gd32f8c343
                digest: sha256:39c5b2e3310dc4264d638ad28d9d1d96c4cbb2b2dcfb52368fe4e3c63f61e10f
                pullPolicy: IfNotPresent
            # -- Provide a priority class name to the webhook patching job
            ##
            priorityClassName: ""
            podAnnotations: {}
            nodeSelector:
                kubernetes.io/os: linux
            tolerations: []
            # -- Labels to be added to patch job resources
            labels: {}
            securityContext:
                runAsNonRoot: true
                runAsUser: 2000
                fsGroup: 2000
        # Use certmanager to generate webhook certs
        certManager:
            enabled: false
            # self-signed root certificate
            rootCert:
                # default to be 5y
                duration: ""
            admissionCert:
                # default to be 1y
                duration: ""
                # issuerRef:
                #   name: "issuer"
                #   kind: "ClusterIssuer"
    metrics:
        port: 10254
        portName: metrics
        # if this port is changed, change healthz-port: in extraArgs: accordingly
        enabled: false
        service:
            annotations: {}
            # prometheus.io/scrape: "true"
            # prometheus.io/port: "10254"
            # -- Labels to be added to the metrics service resource
            labels: {}
            # clusterIP: ""

            # -- List of IP addresses at which the stats-exporter service is available
            ## Ref: https://kubernetes.io/docs/user-guide/services/#external-ips
            ##
            externalIPs: []
            # loadBalancerIP: ""
            loadBalancerSourceRanges: []
            servicePort: 10254
            type: ClusterIP
            # externalTrafficPolicy: ""
            # nodePort: ""
        serviceMonitor:
            enabled: false
            additionalLabels: {}
            ## The label to use to retrieve the job name from.
            ## jobLabel: "app.kubernetes.io/name"
            namespace: ""
            namespaceSelector: {}
            ## Default: scrape .Release.Namespace only
            ## To scrape all, use the following:
            ## namespaceSelector:
            ##   any: true
            scrapeInterval: 30s
            # honorLabels: true
            targetLabels: []
            relabelings: []
            metricRelabelings: []
        prometheusRule:
            enabled: false
            additionalLabels: {}
            # namespace: ""
            rules: []
            # # These are just examples rules, please adapt them to your needs
            # - alert: NGINXConfigFailed
            #   expr: count(nginx_ingress_controller_config_last_reload_successful == 0) > 0
            #   for: 1s
            #   labels:
            #     severity: critical
            #   annotations:
            #     description: bad ingress config - nginx config test failed
            #     summary: uninstall the latest ingress changes to allow config reloads to resume
            # - alert: NGINXCertificateExpiry
            #   expr: (avg(nginx_ingress_controller_ssl_expire_time_seconds) by (host) - time()) < 604800
            #   for: 1s
            #   labels:
            #     severity: critical
            #   annotations:
            #     description: ssl certificate(s) will expire in less then a week
            #     summary: renew expiring certificates to avoid downtime
            # - alert: NGINXTooMany500s
            #   expr: 100 * ( sum( nginx_ingress_controller_requests{status=~"5.+"} ) / sum(nginx_ingress_controller_requests) ) > 5
            #   for: 1m
            #   labels:
            #     severity: warning
            #   annotations:
            #     description: Too many 5XXs
            #     summary: More than 5% of all requests returned 5XX, this requires your attention
            # - alert: NGINXTooMany400s
            #   expr: 100 * ( sum( nginx_ingress_controller_requests{status=~"4.+"} ) / sum(nginx_ingress_controller_requests) ) > 5
            #   for: 1m
            #   labels:
            #     severity: warning
            #   annotations:
            #     description: Too many 4XXs
            #     summary: More than 5% of all requests returned 4XX, this requires your attention
    # -- Improve connection draining when ingress controller pod is deleted using a lifecycle hook:
    # With this new hook, we increased the default terminationGracePeriodSeconds from 30 seconds
    # to 300, allowing the draining of connections up to five minutes.
    # If the active connections end before that, the pod will terminate gracefully at that time.
    # To effectively take advantage of this feature, the Configmap feature
    # worker-shutdown-timeout new value is 240s instead of 10s.
    ##
    lifecycle:
        preStop:
            exec:
                command:
                    - /wait-shutdown
    priorityClassName: ""
# -- Rollback limit
##
revisionHistoryLimit: 10
## Default 404 backend
##
defaultBackend:
    ##
    enabled: false
    name: defaultbackend
    image:
        registry: registry.k8s.io
        image: defaultbackend-amd64
        ## for backwards compatibility consider setting the full image url via the repository value below
        ## use *either* current default registry/image or repository format or installing chart by providing the values.yaml will fail
        ## repository:
        tag: "1.5"
        pullPolicy: IfNotPresent
        # nobody user -> uid 65534
        runAsUser: 65534
        runAsNonRoot: true
        readOnlyRootFilesystem: true
        allowPrivilegeEscalation: false
    # -- Use an existing PSP instead of creating one
    existingPsp: ""
    extraArgs: {}
    serviceAccount:
        create: true
        name: ""
        automountServiceAccountToken: true
    # -- Additional environment variables to set for defaultBackend pods
    extraEnvs: []
    port: 8080
    ## Readiness and liveness probes for default backend
    ## Ref: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/
    ##
    livenessProbe:
        failureThreshold: 3
        initialDelaySeconds: 30
        periodSeconds: 10
        successThreshold: 1
        timeoutSeconds: 5
    readinessProbe:
        failureThreshold: 6
        initialDelaySeconds: 0
        periodSeconds: 5
        successThreshold: 1
        timeoutSeconds: 5
    # -- The update strategy to apply to the Deployment or DaemonSet
    ##
    updateStrategy: {}
    #  rollingUpdate:
    #    maxUnavailable: 1
    #  type: RollingUpdate

    # -- `minReadySeconds` to avoid killing pods before we are ready
    ##
    minReadySeconds: 0
    # -- Node tolerations for server scheduling to nodes with taints
    ## Ref: https://kubernetes.io/docs/concepts/configuration/assign-pod-node/
    ##
    tolerations: []
    #  - key: "key"
    #    operator: "Equal|Exists"
    #    value: "value"
    #    effect: "NoSchedule|PreferNoSchedule|NoExecute(1.6 only)"

    affinity: {}
    # -- Security Context policies for controller pods
    # See https://kubernetes.io/docs/tasks/administer-cluster/sysctl-cluster/ for
    # notes on enabling and using sysctls
    ##
    podSecurityContext: {}
    # -- Security Context policies for controller main container.
    # See https://kubernetes.io/docs/tasks/administer-cluster/sysctl-cluster/ for
    # notes on enabling and using sysctls
    ##
    containerSecurityContext: {}
    # -- Labels to add to the pod container metadata
    podLabels: {}
    #  key: value

    # -- Node labels for default backend pod assignment
    ## Ref: https://kubernetes.io/docs/user-guide/node-selection/
    ##
    nodeSelector:
        kubernetes.io/os: linux
    # -- Annotations to be added to default backend pods
    ##
    podAnnotations: {}
    replicaCount: 1
    minAvailable: 1
    resources: {}
    # limits:
    #   cpu: 10m
    #   memory: 20Mi
    # requests:
    #   cpu: 10m
    #   memory: 20Mi

    extraVolumeMounts: []
    ## Additional volumeMounts to the default backend container.
    #  - name: copy-portal-skins
    #   mountPath: /var/lib/lemonldap-ng/portal/skins

    extraVolumes: []
    ## Additional volumes to the default backend pod.
    #  - name: copy-portal-skins
    #    emptyDir: {}

    autoscaling:
        annotations: {}
        enabled: false
        minReplicas: 1
        maxReplicas: 2
        targetCPUUtilizationPercentage: 50
        targetMemoryUtilizationPercentage: 50
    service:
        annotations: {}
        # clusterIP: ""

        # -- List of IP addresses at which the default backend service is available
        ## Ref: https://kubernetes.io/docs/user-guide/services/#external-ips
        ##
        externalIPs: []
        # loadBalancerIP: ""
        loadBalancerSourceRanges: []
        servicePort: 80
        type: ClusterIP
    priorityClassName: ""
    # -- Labels to be added to the default backend resources
    labels: {}
## Enable RBAC as per https://github.com/kubernetes/ingress-nginx/blob/main/docs/deploy/rbac.md and https://github.com/kubernetes/ingress-nginx/issues/266
rbac:
    create: true
    scope: false
## If true, create & use Pod Security Policy resources
## https://kubernetes.io/docs/concepts/policy/pod-security-policy/
podSecurityPolicy:
    enabled: false
serviceAccount:
    create: true
    name: ""
    automountServiceAccountToken: true
    # -- Annotations for the controller service account
    annotations: {}
# -- Optional array of imagePullSecrets containing private registry credentials
## Ref: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/
imagePullSecrets: []
# - name: secretName

# -- TCP service key-value pairs
## Ref: https://github.com/kubernetes/ingress-nginx/blob/main/docs/user-guide/exposing-tcp-udp-services.md
##
tcp: {}
#  8080: "default/example-tcp-svc:9000"

# -- UDP service key-value pairs
## Ref: https://github.com/kubernetes/ingress-nginx/blob/main/docs/user-guide/exposing-tcp-udp-services.md
##
udp: {}
#  53: "kube-system/kube-dns:53"

# -- Prefix for TCP and UDP ports names in ingress controller service
## Some cloud providers, like Yandex Cloud may have a requirements for a port name regex to support cloud load balancer integration
portNamePrefix: ""
# -- (string) A base64-encoded Diffie-Hellman parameter.
# This can be generated with: `openssl dhparam 4096 2> /dev/null | base64`
## Ref: https://github.com/kubernetes/ingress-nginx/tree/main/docs/examples/customization/ssl-dh-param
dhParam:

```
## 3- Install the NGINX Ingress Controller.
```
helm install ingress-nginx ingress-nginx/ingress-nginx --create-namespace --namespace controller
```
## 4- You should see an output similar to this:
![image](https://user-images.githubusercontent.com/85393914/222490780-04f0468c-cf19-4fb6-ae5c-8400d3896560.png)

## 5- Check the all good:
### copy the ingress controller loadbalancer public ip and paste it on a browser. 
```
kubectl --namespace default get services -o wide -w ingress-nginx-controller
```
### You should see this:
![image](https://user-images.githubusercontent.com/85393914/222491114-30b36845-0767-4cfa-8759-57032f55815b.png)
#### It means our ingress controller is working fine.

# Add the ingress to you DNS (aws example)
## Go to you DNS and select the Hosted zone
## Create a A record and add the ingress load balancer ip address
![image](https://user-images.githubusercontent.com/85393914/222495448-c53e5d18-c5a9-4248-bb5a-1bea05cef3f2.png)
### Test it :
#### app1.anaeleboo.com should give you this:
![image](https://user-images.githubusercontent.com/85393914/222495679-6686e0fc-d553-4c0a-8ea1-75561cd427e7.png)
## Cert-manager
### The issue with this is that It is not secure.
![image](https://user-images.githubusercontent.com/85393914/222496113-88cc72d0-2e81-4d97-b8ca-478e5ac024d8.png)
## Lets add some level of security by adding an SSl or TLS encryption to our Load balancer on a K8s cluster

### Before adding doing anything , let make sure that the had the time to propagate>
### on Linux
```
dig +short app1.anaeleboo.com
```
### On windows, Install dig first 
```
choco install bind-toolsonly
```
### Test it
```
dig +short app1.anaeleboo.com
```
### If successful, the output should return the IP address of your NodeBalancer.
#### The output should be similar to this :
![image](https://user-images.githubusercontent.com/85393914/222498607-aaa524b8-e58c-482a-92ef-ad6204be48f6.png)
# Install Cert-manager
## 1- Install cert-manager’s CRDs
```
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.8.0/cert-manager.crds.yaml
```
## 2- Create a cert-manager namespace.
```
kubectl create namespace cert-manager
```
## 3- Add the Helm repository which contains the cert-manager Helm chart.
```
helm repo add cert-manager https://charts.jetstack.io
```
## 4- Update your Helm repositories.
```
helm repo update
```
## 5- Install the cert-manager Helm chart.
```
helm install \
my-cert-manager cert-manager/cert-manager \
--namespace cert-manager \
--version v1.8.0
```
### or
```
helm install my-cert-manager cert-manager/cert-manager --namespace cert-manager --version v1.8.0
```
### You should see an output similar to this:
![image](https://user-images.githubusercontent.com/85393914/222500555-3f0f0eac-ebb8-477e-bb53-d518bf190372.png)

## 6- Verify that the corresponding cert-manager pods are now running
```
kubectl get pods --namespace cert-manager
```
### You should see an output similar to this:
![image](https://user-images.githubusercontent.com/85393914/222500662-3bc885a0-2623-4728-af25-dd20c5f7f364.png)

## Create a Cluster issuer Resource.
### 1- create a yaml file with this:
```
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    email: user@example.com
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: letsencrypt-secret-prod
    solvers:
    - http01:
        ingress:
          class: nginx
```
### Fixed the above by changing stuff like email and ingress class
### 2- Update the ingress file by adding these in red
![image](https://user-images.githubusercontent.com/85393914/222505425-e2133248-993c-441c-b372-b52957899214.png)

## After all we should have secure applications
![image](https://user-images.githubusercontent.com/85393914/222505679-bd478939-9004-40bd-b2bb-54353288ddd0.png)
## Double check that the certificate is good:
```
kubectl describe certificate namehere
```
```
kubectl describe certificate example-tls
```

#### refs: 
###### 1- https://www.linode.com/docs/guides/how-to-configure-load-balancing-with-tls-encryption-on-a-kubernetes-cluster/
###### 2- https://www.linode.com/docs/guides/deploy-nginx-ingress-on-lke/
