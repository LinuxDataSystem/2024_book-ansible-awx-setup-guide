# 제4장: AWX 구성

## 4.1. AWX 란 무엇인가?
### 4.1.1. AWX 정의 및 개념
### 4.1.2. 주요 기능 요약
### 4.1.3. 주요 구성 요소 소개
### 4.1.4. AWX 의 장점 및 활용 분야
### 4.1.5. AWX 설치 기술

## 4.2. AWX 설치 요건
### 4.2.1. 하드웨어
### 4.2.2. 소프트웨어
### 4.2.3. 네트워크 환경

## 4.3. AWX 설치
### 4.3.1. Package 설치
```bash
# dnf -y install git make
```

### 4.3.2. Git Clone
```bash
# git clone https://github.com/ansible/awx-operator.git

# cd awx-operator
```

```bash
# git tag

# export VERSION=`curl -s https://api.github.com/repos/ansible/awx-operator/releases/latest | grep tag_name | cut -d '"' -f 4`

# echo $VERSION

# git checkout tags/$VERSION
```

### 4.3.3. awx-operator 배포
```bash
# kubectl version
```

```bash
# make deploy
```

```bash
# kubectl get pods -n awx
```

```bash
# kubectl config set-context --current --namespace=awx
```

### 4.3.4. awx 배포
#### 4.3.4.1. awx 배포 파일 생성
```yaml
# awx-demo.yml
---
apiVersion: awx.ansible.com/v1beta1
kind: AWX
metadata:
  name: awx-demo
spec:
  service_type: nodeport
  # AWX 외부 접속 포트
  #nodeport_port: 30080
  # awx-secret.yml 파일을 생성하였을 경우 주석을 해제
  #admin_password_secret: custom-awx-admin-password
  projects_persistence: true
  projects_storage_size: 20Gi
  projects_storage_access_mode: ReadWriteOnce
  web_resource_requirements:
    requests:
      cpu: 200m
      memory: 1Gi
    limits:
      cpu: 1000m
      memory: 2Gi
```

#### 4.3.4.1.1. awx 관리자 비밀번호 설정 파일 생성
```yaml
# awx-secret.yml
---
apiVersion: v1
kind: Secret
metadata:
  name: custom-awx-admin-password
  namespace: awx
type: Opaque
data:
  password: $(echo -n "**adminpassword**" | base64)
EOF
```

```yaml
# kustomization.yaml
---
...output omitted...
resources:
  # Add this extra line:
  - awx-demo.yml
  # awx-secret.yml 파일을 생성하였을 경우 주석을 해제
  #- awx-secret.yml
```

#### 4.3.4.2. awx 배포 실행
```bash
# kubectl apply -f awx-demo.yml 
or
# kubectl apply -k .
```

#### 4.3.4.3. awx 배포 검증
```bash
# kubectl logs -f deployments/awx-operator-controller-manager -c awx-manager
```

```bash
# kubectl get pods,svc -l "app.kubernetes.io/managed-by=awx-operator"
```

```yaml
# kubectl get deployment.apps/awx-demo-web -o yaml
...output omitted...
    spec:
      containers:
...output omitted...
        name: awx-demo-web
        ports:
        - containerPort: 8052
          protocol: TCP
        resources:
          limits:
            cpu: "1"
            memory: 2Gi
          requests:
            cpu: 200m
            memory: 1Gi
...output omitted...
        - mountPath: /var/lib/awx/projects
          name: awx-demo-projects
...output omitted...
      - name: awx-demo-projects
        persistentVolumeClaim:
          claimName: awx-demo-projects-claim
...output omitted...
```

### 4.3.5. awx 제거 - [참고]
#### 4.3.5.1. awx 배포 인스턴스 제거
```bash
# kubectl delete awx awx-demo
```

#### 4.3.5.2. awx 영구 볼륨 확인
```bash
# kubectl get pv

# kubectl get pvc
```

#### 4.3.5.3. awx 영구 볼륨 삭제
```bash
# kubectl delete pvc postgres-13-awx-demo-postgres-13-0

# kubectl get pvc

# kubectl get pv
```
