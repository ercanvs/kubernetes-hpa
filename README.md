# kubernetes-hpa
kubernetes üzerinde hpa aktif etmek için microservislerimizi helm ile deployment lerini hazırladık.
burda hpa aktif etmek için öncelikle metric-server i yüklemek gerekir metric server i yüklemek için 

https://github.com/ercanvs/kubernetes-hpa/blob/main/metric-server.yaml bunu yuklememiz gerekiyor yüklemek için 
k apply -f https://github.com/ercanvs/kubernetes-hpa/blob/main/metric-server.yaml
daha sonra burda sertifika ile alakalı hata alırsak x.509 hatası gibi
- --kubelet-insecure-tls bu komutu kube-system  in deployment ina eklememız grekir
            args:
            - '--cert-dir=/tmp'
            - '--secure-port=443'
            - '--kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname'
            - '--kubelet-use-node-status-port'
            - '--metric-resolution=15s'
            - '--kubelet-insecure-tls=true'

  daha sonra bunu kaydedip çıkalım
  daha sonra microservisinizde deployment.yaml da resource altında limit ve request leri aktif etmeliyiz

          resources:
          limits:
            cpu: 500m
            memory: "500Mi"
          requests:
            cpu: 200m
            memory: "300Mi"

  burda dikkat edilmesi gereken her zaman limit>=requests
  daha sonra template folder inin altında hpa.yaml i editleyelim burda release-name helm i yüklerken yazdıgımız deger
  helm upgrade --install microservice1 .   burdaki release-name i microservice1 oluyor
  burda min replica count u 2  max 5 eger cpu ve memory %80 i geçerse yeni replica create eder daha sonra resource degeri düşerse tekrar azalır replica count u


    resources:
    limits:
      cpu: 500m
      memory: "500Mi"
    apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: {{ .Release.Name }}-service
  namespace: default
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: {{ .Release.Name }}-service
  minReplicas: 2
  maxReplicas: 5
  behavior:
    scaleUp:
      policies:
      - type: Pods
        value: 2
        periodSeconds: 60
    scaleDown:
      policies:
      - type: Percent
        value: 10
        periodSeconds: 60
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 80
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
    requests:
      cpu: 200m
      memory: "300Mi"
  




