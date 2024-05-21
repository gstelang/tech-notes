# K8s best practices
* Define your workload - background job, microservice with queries/sec, monitoring job, etc
    * Characterise your workload - burstable, peak period of traffic and peak load. 
    * Run ab benchmarking tool to simulate prod workload
    * Run your app in pre-prod (staging) for a while to check its memory consumption.
    * Most loads in my personal opinion are memory hog than CPU hog. If it is CPU hog, possibly design it differently or move out of k8s.
    * Note down expected traffic patterns. 
* Config
    * Setup readiness probe - ready to take traffic.
    * Setup liveness probe  - can k8s take down the pod.
    * Anti affinity rules for certain AZ that known to cause issues. Similarly, affinity rules for reliable hosts. 
* App
    * Handle SIGTERM - clean resources/close transactions, etc. Check process id of dockerized container. If it is not 1, it cannot handle sigterm. 
* Memory/CPU 
    * Memory is a compressible resource vs CPU is not. You will be throttled for CPU vs OOM for Memory.
    * Set memory request/limit the same. This will guarantee "Guaranteed" QoS because setting higher memory limit can hide insufficient request memory. Adjust as needed.
    * Set heap size particularly for a JVM based app/node app. 
    * Go/node/java app - most memory hog problems happen because of external dependencies. Track those. 
    * don't worry about CPU. Trust cluster auto scaler from EKS, etc. if you have a service with too much CPU, measure and scale/chose your VM's accordingly.
    * Except AMZN/INSTA and few more, most app receives max peak load of 30 req/sec. so, initial setup between 100m to 500m request should be sufficient i.e 10% to 50% of available CPU.
    * If really CPU hog services
        * Think about using cpu 'quota' for a cpu 'period'. Ex: 50% quota for 2 sec. i.e it can hog CPU for 1 sec over a 2 sec time period. 
        * LimitRange resource.
* Scaling 
    * Horizontal (number of replicas) vs Vertical (CPU/memory requests and limits)
    * Horizontal pod autoscaler (Define min/max replicas with target average metrics if any)
    * Use cluster autoscaler but monitor/look at the cost budget if needed.
    * POD disruption budget that specifies min available at all times. 
    * PriorityClass resource for certain pods that impact lives of customer/generate revenue/brand impacting, etc
* k8s QoS
    1. Guaranteed (Analogy: private water line)
    2. Best effort (Analogy: water fountain at a public park)
    3. Burstable (with/without burst): (Analogy: Fixed water supply but ability to take more, if required without depriving)
* EKS vs ECS VS Fargate 
    * EKS - cost for control plane + worker plane cost. Cloud agnostic with k8s but steep learning curve.
    * ECS - free control plane + worker plane cost. AWS tie in. Need to track cloud budget. 
    * Fargate with ECS - define tasks and services in ECS task definitions.
    * Fargate with EKS - Manage pods without managing the underlying infrastructure. Heard lot of issues with this one. Need to circle back.
* Secret management
    * Built-in Kubernetes Secrets resource to store sensitive data. They're base64 encoded.
    * Enable 1) RBAC to k8s secret. 2) Namespace isolation 3) automatic rotation
    * External secret management (Can affect reliability if the service is down plus the cost even though it is miniscule)
    * Mount as volume or ENV variables in your application pods.
    * Encrypt secret data using Amazons key management service.    
* Downsides with fargate:
    * No daemonset (hence customized logging could be a difficult path)
    * No privileged containers means something like tcpdump/strace like logging work could be a pain for your legacy app. 
    * High spin up time (50s plus)
    * AWS charge for 2 CPU/10 GB RAM even though you use 0.5 CPU. If you're frugal, might not work for your use case and technology.
    * Because of no daemonset, any custom logging (nodename/metadata) that you want to implement could be a difficult challenge. 
* Deployment strategy
    * Rolling update (gradual replacement of old with new one) with max surgae/max unavailable. Default for maxsurge is 25% i.e number of pods more than the desired number.
