# CKA — Services & Networking (20%)

> Domain có **điểm mới quan trọng nhất của CKA 2025: Gateway API**.
> NetworkPolicy phải viết được **từ đầu không tra docs** (docs có ví dụ nhưng mất thời gian).

**Nội dung curriculum v1.35:**
- Understand connectivity between Pods
- Define and enforce Network Policies
- Use ClusterIP, NodePort, LoadBalancer service types and endpoints
- **Use the Gateway API to manage Ingress traffic** ← mới 2025
- Know how to use Ingress controllers and Ingress resources
- Understand and use CoreDNS

---

## 1. Mô hình mạng K8s — 3 nguyên tắc

```text
1. Mọi Pod có IP riêng, duy nhất trong cluster.
2. Pod ↔ Pod nói chuyện trực tiếp bằng IP, KHÔNG NAT — kể cả khác node.
3. Node ↔ Pod nói chuyện được, không NAT.

   Node A (10.0.1.5)              Node B (10.0.2.7)
   ┌──────────────────┐           ┌──────────────────┐
   │ Pod 10.244.1.3   │◄─────────►│ Pod 10.244.2.9   │   ← CNI lo việc này
   │ Pod 10.244.1.4   │           │ Pod 10.244.2.10  │
   └──────────────────┘           └──────────────────┘

3 dải mạng riêng biệt — đừng nhầm lẫn:
  - Node CIDR     : IP của máy chủ         (vd 10.0.0.0/16)
  - Pod CIDR      : --pod-network-cidr     (vd 10.244.0.0/16)
  - Service CIDR  : --service-cluster-ip-range (vd 10.96.0.0/12)
```

**Trong Pod, các container chung một network namespace** → nói chuyện qua `localhost`,
và **không được trùng port**.

---

## 2. Service — 4 loại

```text
     Internet
        │
        ▼
  ┌───────────────┐
  │ LoadBalancer  │  ← cloud LB cấp IP public  (bao trùm NodePort)
  └───────┬───────┘
          ▼
  ┌───────────────┐
  │  NodePort     │  ← mở port 30000-32767 trên MỌI node (bao trùm ClusterIP)
  └───────┬───────┘
          ▼
  ┌───────────────┐
  │  ClusterIP    │  ← IP ảo nội bộ cluster (mặc định)
  └───────┬───────┘
          ▼
      Endpoints → Pod IP:port
```

```bash
# Tạo nhanh bằng expose
k expose deploy web --port=80 --target-port=8080                      # ClusterIP
k expose deploy web --port=80 --type=NodePort --name=web-np
k expose deploy web --port=80 --type=LoadBalancer
k expose po nginx --port=80 --name=nginx-svc

# Tạo bằng create svc (khi chưa có deploy)
k create svc clusterip web --tcp=80:8080
k create svc nodeport web --tcp=80:8080 --node-port=30080
k create svc externalname mydb --external-name=db.example.com
```

```yaml
apiVersion: v1
kind: Service
metadata: {name: web}
spec:
  type: NodePort
  selector:
    app: web                    # ← khớp LABEL CỦA POD, không phải của Deployment
  ports:
  - name: http
    port: 80                    # port của Service
    targetPort: 8080            # port trong container (có thể là tên port)
    nodePort: 30080             # 30000-32767
    protocol: TCP
```

| Loại | Khi nào dùng | Ghi chú |
|---|---|---|
| `ClusterIP` | Nội bộ cluster (mặc định) | `clusterIP: None` → **Headless** |
| `NodePort` | Truy cập từ ngoài không có cloud LB | Mở trên **mọi** node |
| `LoadBalancer` | Cloud (AWS/GCP/Azure) | Không có cloud provider → `EXTERNAL-IP` mãi `<pending>` |
| `ExternalName` | Alias CNAME tới DNS ngoài | Không có selector, không có endpoint |

**Headless Service** (`clusterIP: None`): DNS trả về **danh sách Pod IP** thay vì 1 VIP.
Dùng cho StatefulSet, hoặc khi client tự load-balance.

### Endpoints — công cụ debug số 1 ⭐
```bash
k get endpoints web
k get endpointslices -l kubernetes.io/service-name=web
k describe svc web
```
> 🔴 **`ENDPOINTS` rỗng = Service không tìm thấy Pod nào.** Gần như luôn do:
> 1. `selector` của Service không khớp label Pod → `k get po --show-labels`
> 2. Pod chưa `Ready` (readinessProbe fail) — Pod chưa Ready thì **không** vào endpoints
> 3. Sai namespace
>
> Đây là câu troubleshooting networking hay gặp nhất.

### sessionAffinity & externalTrafficPolicy
```yaml
spec:
  sessionAffinity: ClientIP           # sticky theo IP client
  externalTrafficPolicy: Local        # giữ source IP thật, chỉ route tới Pod trên node đó
  internalTrafficPolicy: Local
```

---

## 3. DNS trong cluster — CoreDNS

### Quy tắc đặt tên
```text
Service:  <svc>.<namespace>.svc.cluster.local
Pod:      <pod-ip-đổi-dấu-chấm-thành-gạch>.<ns>.pod.cluster.local
          10.244.1.5  →  10-244-1-5.default.pod.cluster.local
StatefulSet Pod: <pod>.<headless-svc>.<ns>.svc.cluster.local
                 web-0.web-headless.default.svc.cluster.local
```

Từ trong Pod cùng namespace, gọi `http://web` là đủ (nhờ `search` domain trong `/etc/resolv.conf`).
Khác namespace: `http://web.dev` hoặc đầy đủ `web.dev.svc.cluster.local`.

### Kiểm tra & debug DNS
```bash
# Pod tạm để test
k run tmp --image=busybox:1.28 --rm -it --restart=Never -- sh
  # trong pod:
  nslookup web
  nslookup web.dev.svc.cluster.local
  nslookup kubernetes.default
  cat /etc/resolv.conf
  wget -O- http://web:80

# Hoặc dùng nicolaka/netshoot (đủ tool hơn)
k run netshoot --image=nicolaka/netshoot --rm -it --restart=Never -- bash
  # dig, curl, tcpdump, netstat, ss, nc ...

# CoreDNS
k get po -n kube-system -l k8s-app=kube-dns
k logs -n kube-system -l k8s-app=kube-dns
k get svc -n kube-system kube-dns              # ClusterIP thường là 10.96.0.10
k get cm coredns -n kube-system -o yaml        # Corefile
k edit cm coredns -n kube-system               # sửa xong nhớ restart:
k rollout restart deploy coredns -n kube-system
```

**Corefile mẫu (biết đọc là đủ):**
```text
.:53 {
    errors
    health
    ready
    kubernetes cluster.local in-addr.arpa ip6.arpa {
       pods insecure
       fallthrough in-addr.arpa ip6.arpa
    }
    prometheus :9153
    forward . /etc/resolv.conf          # ← DNS ngoài cluster đi qua đây
    cache 30
    loop
    reload
    loadbalance
}
```

### dnsPolicy trên Pod
| Giá trị | Ý nghĩa |
|---|---|
| `ClusterFirst` | Mặc định — hỏi CoreDNS trước |
| `Default` | Dùng resolv.conf của **node** (bỏ qua CoreDNS) |
| `None` | Tự khai `dnsConfig` |
| `ClusterFirstWithHostNet` | Khi `hostNetwork: true` mà vẫn muốn dùng CoreDNS |

> 🔴 Bẫy: `hostNetwork: true` + `dnsPolicy` mặc định → Pod **không** resolve được service name.
> Phải đặt `dnsPolicy: ClusterFirstWithHostNet`.

---

## 4. NetworkPolicy ⭐⭐ (phải viết được từ đầu)

**Nguyên tắc sống còn:**
1. Mặc định K8s cho phép **mọi** traffic. NetworkPolicy chỉ có tác dụng khi **CNI hỗ trợ**
   (Calico/Cilium có; Flannel thuần **không** có).
2. Pod chỉ bị ảnh hưởng nếu có **ít nhất 1** NetworkPolicy chọn nó.
   Khi đã bị chọn → mọi thứ **không được cho phép rõ ràng** đều bị **chặn**.
3. Nhiều policy trên cùng Pod thì **cộng dồn (OR)** các rule allow.
4. Ingress và Egress **độc lập** — mở chiều này không tự mở chiều kia.

### Bộ khung — thuộc lòng
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: my-policy
  namespace: dev
spec:
  podSelector:                  # policy áp lên Pod nào ({} = MỌI Pod trong ns)
    matchLabels:
      app: db
  policyTypes:                  # phải khai rõ, nếu không K8s tự suy ra
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:              # Pod trong CÙNG namespace
        matchLabels: {app: api}
    - namespaceSelector:        # MỌI Pod trong ns có label này
        matchLabels: {env: prod}
    - namespaceSelector:        # ← AND: Pod app=api TRONG ns env=prod
        matchLabels: {env: prod}
      podSelector:
        matchLabels: {app: api}
    - ipBlock:
        cidr: 10.0.0.0/16
        except: [10.0.5.0/24]
    ports:
    - protocol: TCP
      port: 5432
  egress:
  - to:
    - podSelector:
        matchLabels: {app: cache}
    ports:
    - protocol: TCP
      port: 6379
```

> 🔴 **Bẫy quan trọng nhất của NetworkPolicy — dấu gạch đầu dòng:**
> ```yaml
> from:
> - namespaceSelector: {...}     # OR
> - podSelector: {...}
> ```
> vs
> ```yaml
> from:
> - namespaceSelector: {...}     # AND (cùng một phần tử list!)
>   podSelector: {...}
> ```
> Sai chỗ này là sai cả câu. **Đọc kỹ đề: "Pod X trong namespace Y" = AND.**

### Các mẫu phải thuộc

**Default deny ALL ingress trong 1 namespace:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: {name: default-deny-ingress, namespace: dev}
spec:
  podSelector: {}
  policyTypes: [Ingress]
```

**Default deny cả ingress lẫn egress:**
```yaml
spec:
  podSelector: {}
  policyTypes: [Ingress, Egress]
```

**Allow all ingress:**
```yaml
spec:
  podSelector: {}
  policyTypes: [Ingress]
  ingress:
  - {}
```

**Cho phép DNS đi ra (BẮT BUỘC khi đã deny egress!):**
```yaml
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: kube-system
      podSelector:
        matchLabels:
          k8s-app: kube-dns
    ports:
    - {protocol: UDP, port: 53}
    - {protocol: TCP, port: 53}
```
> 🔴 Quên mở DNS là lỗi kinh điển: deny egress → Pod không resolve được tên → mọi thứ "hỏng"
> nhưng bạn tưởng do policy sai. **Deny egress thì luôn nhớ mở port 53.**

**Mẹo chọn namespace theo tên** (không cần label thủ công):
K8s tự gắn label `kubernetes.io/metadata.name: <ns-name>` cho mọi namespace.
```yaml
- namespaceSelector:
    matchLabels:
      kubernetes.io/metadata.name: prod
```

### Test NetworkPolicy
```bash
k run test --image=busybox:1.28 --rm -it --restart=Never -n dev -- \
  wget -qO- --timeout=3 http://db:5432
# Bị chặn → treo rồi timeout. Được phép → có phản hồi.

k describe netpol <name> -n dev
```

---

## 5. Ingress

```text
Internet ──► Ingress Controller (nginx/traefik, chạy dạng Pod + Service NodePort/LB)
                      │  đọc Ingress resource
                      ▼
              Service ──► Pod
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
  namespace: dev
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx              # ← thay cho annotation kubernetes.io/ingress.class (cũ)
  tls:
  - hosts: [app.example.com]
    secretName: tls-secret             # secret type kubernetes.io/tls
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /api
        pathType: Prefix               # Prefix | Exact | ImplementationSpecific
        backend:
          service:
            name: api-svc
            port:
              number: 8080
      - path: /
        pathType: Prefix
        backend:
          service: {name: web-svc, port: {number: 80}}
  defaultBackend:
    service: {name: default-svc, port: {number: 80}}
```

```bash
# Tạo nhanh bằng lệnh
k create ingress web --rule="app.example.com/api*=api-svc:8080" \
                     --rule="app.example.com/*=web-svc:80" \
                     --class=nginx -n dev

# Tạo TLS secret
k create secret tls tls-secret --cert=tls.crt --key=tls.key -n dev

k get ingress -A
k get ingressclass
k describe ingress web-ingress -n dev
```

| Bẫy | Chi tiết |
|---|---|
| Không có Ingress **Controller** → Ingress resource vô nghĩa | `k get po -A \| grep ingress` |
| `ADDRESS` rỗng | Controller chưa gán được IP, hoặc sai `ingressClassName` |
| Service backend phải **cùng namespace** với Ingress | Không cross-namespace được |
| `pathType` là **bắt buộc** từ networking.k8s.io/v1 | Thiếu → apply fail |
| Sai `port.number` vs `port.name` | Chọn 1, không dùng cả hai |

---

## 6. Gateway API ⭐⭐ (MỚI trong CKA 2025)

Thế hệ sau của Ingress. Tách vai trò: người quản trị cluster định nghĩa `GatewayClass`/`Gateway`,
đội ứng dụng tự viết `HTTPRoute`.

```text
GatewayClass  (ai cài — như StorageClass, do vendor cung cấp)
     │
     ▼
Gateway       (cluster operator tạo — listener, port, TLS, hostname)
     │
     ▼
HTTPRoute     (app team tạo — rule định tuyến → backendRefs)
     │
     ▼
Service ──► Pod
```

### GatewayClass
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: example-gateway-class
spec:
  controllerName: example.net/gateway-controller
```

### Gateway
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: my-gateway
  namespace: infra
spec:
  gatewayClassName: example-gateway-class
  listeners:
  - name: http
    protocol: HTTP
    port: 80
    allowedRoutes:
      namespaces:
        from: All              # All | Same | Selector
  - name: https
    protocol: HTTPS
    port: 443
    hostname: "*.example.com"
    tls:
      mode: Terminate          # Terminate | Passthrough
      certificateRefs:
      - kind: Secret
        name: tls-secret
```

### HTTPRoute — phần hay ra đề nhất
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: web-route
  namespace: dev
spec:
  parentRefs:                  # ← trỏ tới Gateway
  - name: my-gateway
    namespace: infra
    sectionName: http          # (tuỳ chọn) chỉ gắn vào listener tên "http"
  hostnames:
  - "app.example.com"
  rules:
  - matches:
    - path:
        type: PathPrefix       # PathPrefix | Exact | RegularExpression
        value: /api
      headers:
      - name: x-version
        value: v2
      method: GET
    filters:
    - type: RequestHeaderModifier
      requestHeaderModifier:
        add:
        - name: x-source
          value: gateway
    backendRefs:
    - name: api-svc
      port: 8080
      weight: 90               # ← traffic splitting / canary
    - name: api-svc-v2
      port: 8080
      weight: 10
  - matches:
    - path: {type: PathPrefix, value: /}
    backendRefs:
    - name: web-svc
      port: 80
```

```bash
k get gatewayclass
k get gateway -A
k get httproute -A
k describe gateway my-gateway -n infra       # xem status/conditions: Accepted, Programmed
k describe httproute web-route -n dev        # xem "Accepted", "ResolvedRefs"
k api-resources | grep gateway               # kiểm tra CRD đã cài chưa
```

| Ingress | Gateway API |
|---|---|
| 1 resource làm tất cả | Tách 3 resource theo vai trò |
| Cấu hình nâng cao qua **annotation** (mỗi vendor một kiểu) | Field chuẩn trong spec |
| Chỉ HTTP/HTTPS | HTTPRoute, GRPCRoute, TCPRoute, TLSRoute, UDPRoute |
| Không cross-namespace | Có, qua `ReferenceGrant` |
| Traffic splitting phải dùng annotation | `weight` chuẩn trong `backendRefs` |
| `ingressClassName` | `gatewayClassName` |

> 🔴 Gateway API là **CRD**, không có sẵn trong K8s. Kiểm tra `k get crd | grep gateway`.
> Nếu đề bảo cài: `k apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/vX.Y.Z/standard-install.yaml`.
>
> **Cross-namespace**: HTTPRoute ở ns A trỏ Service ở ns B cần **`ReferenceGrant`** ở ns B.

---

## 7. kube-proxy & cách Service hoạt động

```bash
k get ds kube-proxy -n kube-system
k logs -n kube-system -l k8s-app=kube-proxy
k get cm kube-proxy -n kube-system -o yaml | grep mode

# Trên node — xem rule thật
iptables-save | grep <service-name>
iptables -t nat -L KUBE-SERVICES -n | head
ipvsadm -Ln                                  # nếu mode=ipvs
```

| Mode | Cơ chế | Đặc điểm |
|---|---|---|
| `iptables` | Chuỗi rule NAT | Mặc định; chậm dần khi rất nhiều service |
| `ipvs` | IPVS kernel | Nhanh hơn ở quy mô lớn, nhiều thuật toán LB |
| `nftables` | nftables | Mới, dần thay iptables |

> Service **không phải** một process. Nó là **tập rule NAT** do kube-proxy ghi trên mỗi node.
> Vì vậy `ping <clusterIP>` thường không ăn thua — phải test bằng `curl`/`wget` đúng port.

---

## 8. Quy trình debug mạng — theo thứ tự

```text
Không truy cập được service?

1. Pod chạy chưa?            k get po -o wide          → Running + READY 1/1?
2. Service có endpoint?      k get ep <svc>            → RỖNG = sai selector / Pod chưa Ready
3. Label khớp không?         k get po --show-labels  ×  k describe svc <svc> | grep Selector
4. Port đúng chưa?           targetPort == containerPort?
5. Gọi trực tiếp Pod IP:     k run tmp --rm -it --image=busybox:1.28 -- wget -qO- <podIP>:<port>
6. Gọi qua ClusterIP:        ... wget -qO- <clusterIP>:<port>
7. Gọi qua DNS:              ... wget -qO- <svc>.<ns>
   → bước 6 OK mà 7 fail  ⇒ vấn đề DNS/CoreDNS
   → bước 5 OK mà 6 fail  ⇒ vấn đề kube-proxy / iptables
   → bước 5 fail          ⇒ vấn đề app hoặc CNI
8. NetworkPolicy chặn?       k get netpol -n <ns>
9. Từ ngoài vào?             NodePort mở? Ingress controller sống? firewall?
```

**Bộ lệnh cứu hộ:**
```bash
k get po,svc,ep,ingress,netpol -n <ns>
k describe svc <svc> -n <ns>
k port-forward svc/<svc> 8080:80 -n <ns>        # bypass ingress/nodeport để test
k exec -it <pod> -- curl -sv http://other-svc
k get po -A -o wide | grep -E 'coredns|kube-proxy|calico|cilium'
```

---

## 9. Dạng bài hay ra

| # | Đề bài | Hướng làm |
|---|---|---|
| 1 | Expose deployment qua NodePort 30080 | `k expose --type=NodePort` rồi sửa `nodePort` |
| 2 | Service không truy cập được — sửa | Mục 8, hầu hết là selector/endpoints |
| 3 | Viết NetworkPolicy chỉ cho Pod `app=api` ở ns `prod` gọi tới `db` cổng 5432 | Mục 4, chú ý AND vs OR |
| 4 | Default deny toàn ns, rồi mở đúng 1 đường | 2 policy, nhớ mở DNS nếu deny egress |
| 5 | Tạo Ingress route `/api` → svc A, `/` → svc B, có TLS | Mục 5 |
| 6 | Tạo HTTPRoute chia traffic 80/20 giữa 2 service | Mục 6, dùng `weight` |
| 7 | Sửa CoreDNS để forward domain nội bộ | `k edit cm coredns` + rollout restart |
| 8 | Tìm ClusterIP & endpoint của service X ghi ra file | `k get svc/ep -o jsonpath` |
| 9 | Pod không resolve được DNS — chẩn đoán | CoreDNS pod? kube-dns svc? `/etc/resolv.conf`? |
| 10 | Tạo headless service cho StatefulSet | `clusterIP: None` |

---

## 10. Bẫy tổng kết

1. **Endpoints rỗng** = selector sai hoặc Pod chưa Ready. Luôn kiểm tra đầu tiên.
2. **NetworkPolicy: `- ns:` + `- pod:` là OR; `- ns:` + `pod:` (cùng item) là AND.**
3. **Deny egress mà quên mở DNS (53/UDP + TCP)** → tưởng policy sai.
4. **NetworkPolicy vô tác dụng nếu CNI không hỗ trợ** (Flannel thuần).
5. **Gateway API là CRD** — kiểm tra đã cài chưa.
6. **HTTPRoute cross-namespace cần `ReferenceGrant`.**
7. **`ingressClassName` (field) đã thay `kubernetes.io/ingress.class` (annotation).**
8. **`pathType` bắt buộc** trong Ingress v1.
9. **`hostNetwork: true` phải kèm `dnsPolicy: ClusterFirstWithHostNet`.**
10. **`targetPort` là port trong container**, `port` là port của Service. Đừng đảo.
11. **LoadBalancer `<pending>` mãi** là bình thường trên cluster bare-metal — không phải lỗi bạn.
12. **busybox mới (>=1.29) có bug nslookup** — dùng `busybox:1.28` khi test DNS.
