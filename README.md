# kubernetes-jenkins
```bash
cd kubernetes-jenkins
kubectl apply -f namespace.yaml

--edit name node kubernetes--- 
kubectl apply -f volume.yaml

kubectl apply -f service.yaml
kubectl apply -f serviceAccount.yaml
kubectl apply -f deployment.yaml
```
```bash
----Fix after deployment----
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
kubectl taint nodes --all node-role.kubernetes.io/master-

Tạo namespace kubernetes giống webUI Jenkins trước
kubectl create ns jenkin-agent

-----Create secret docker-credentials into namespace jenkin-agent---------
Method 1: Create from file config of Docker (if created login docker on server)
Bash
kubectl create secret generic docker-credentials \
    --from-file=.dockerconfigjson=$HOME/.docker/config.json \
    --type=kubernetes.io/dockerconfigjson \
    -n jenkin-agent
Method 2: Create directly by comandline (if don't login)
Bash
kubectl create secret docker-registry docker-credentials \
    --docker-server=https://index.docker.io/v1/ \
    --docker-username=<TÊN_ĐĂNG_NHẬP> \
    --docker-password=<PASSWORD_HOẶC_TOKEN> \
    --docker-email=<EMAIL_CỦA_BẠN> \
    -n jenkin-agent

kubectl get secret -n jenkin-agent

```
```bash
Check Global Security: Login Jenkins (Giao diện web) 
-> Manage Jenkins -> Security -> Found TCP port for inbound agents. Sured is Fixed: 50000 (Should not Random).

Check Service in K8s: Sure Service of Jenkins Master is opened port 50000. 
Check again Service jenkins-service in namespace devops-tools:

kubectl get svc -n devops-tools

sudo ufw allow 50000/tcp
sudo ufw allow 30005/tcp

To repair Jenkins Agent disconnect (Connection refused), Must add port 50000 to Service currently. 
This is port Jenkins Master và Agent "tranmission" together by TCP protocol.

Edit port=>>> kubectl edit svc jenkins-service -n devops-tools

Find to session ports and add config port 50000 as below:

YAML
  ports:
  - name: http
    nodePort: 32000
    port: 8080
    protocol: TCP
    targetPort: 8080
  - name: agent
    nodePort: 30005        # Can select port from 30000-32767
    port: 50000
    protocol: TCP
    targetPort: 50000

```
```bash
----See password Admin (If the pod is running)----
sudo kubectl exec jenkins-646b986f47-cl55r -n devops-tools -- cat /var/jenkins_home/secrets/initialAdminPassword

```
```bash
----Login---
http://IP_Address:32000
```
```bash

Xóa Pod cũ bị kẹt: Đôi khi các Pod cũ chiếm giữ tài nguyên hoặc Volume, hãy dọn dẹp sạch:
kubectl delete pod -n jenkin-agent --all --force --grace-period=0

Kiểm tra trạng thái Pod thực tế
kubectl get pods -n jenkin-agent

# Xem biến môi trường JENKINS_HOME
sudo kubectl exec jenkins-646b986f47-cl55r -n devops-tools -- env | grep JENKINS_HOME

# Liệt kê nội dung thư mục đó để chắc chắn
sudo kubectl exec jenkins-646b986f47-cl55r -n devops-tools -- ls -F /var/jenkins_home

# Kiểm tra cấu hình Volume của Pod
sudo kubectl describe pod jenkins-646b986f47-cl55r -n devops-tools | grep -A 5 "Volumes:"

Tìm thư mục vật lý trên máy Host (Máy của bạn)
# Kiểm tra cấu hình Volume của Pod
sudo kubectl describe pod jenkins-646b986f47-cl55r -n devops-tools | grep -A 5 "Volumes:"

Kiểm tra quyền sở hữu thư mục Home của Jenkins
sudo kubectl exec jenkins-646b986f47-cl55r -n devops-tools -- ls -ld /var/jenkins_home

Kiểm tra User hiện tại bên trong Pod
sudo kubectl exec jenkins-646b986f47-cl55r -n devops-tools -- whoami
# Hoặc xem chi tiết ID:
sudo kubectl exec jenkins-646b986f47-cl55r -n devops-tools -- id

Dọn dẹp các bản Deployment bị kẹt
# Đưa về 0 để xóa các Pod lỗi=> tam dung
sudo kubectl scale deployment jenkins --replicas=0 -n devops-tools

# Đợi vài giây rồi tăng lên 1
sudo kubectl scale deployment jenkins --replicas=1 -n devops-tools
```
