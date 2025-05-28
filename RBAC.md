# RBAC

**서비스 계정 `khj`가 default 네임스페이스에서 services 리소스를 나열(`list`)할 권한이 없기 때문에 에러 발생**

```bash
# SA(Service Account) 생성
kubectl create serviceaccount khj

# kubetctl get service 실행
kubectl get service --as system:serviceaccount:default:khj
# 권한 에러
Error from server (Forbidden): deployments.apps is forbidden: User "system:serviceaccount:default:khj" cannot list resource "deployments" in API group "apps" in the namespace "default"

```

- **요약**: `khj` 서비스 계정은 `services` 리소스를 "list"할 수 있는 권한이 없음
- **자원 종류**: `services` (Kubernetes core API group)
- **작업 종류**: `list` (자원 목록 조회)
- **네임스페이스**: `default`
- **사용자**: `system:serviceaccount:default:khj`

---

### RBAC 권한 부여

**Role**과 **RoleBinding 생성**

**Role 생성** (khj-role.yaml)

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: khj-service-reader
  namespace: default
rules:
- apiGroups: [""]
  resources: ["services"]
  verbs: ["get", "list", "watch"]
```

**RoleBinding 생성** (khj-rolebinding.yaml)

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: khj-service-reader-binding
  namespace: default
subjects:
- kind: ServiceAccount
  name: khj
  namespace: default
roleRef:
  kind: Role
  name: khj-service-reader
  apiGroup: rbac.authorization.k8s.io
```

```bash
# 적용
kubectl apply -f khj-role.yaml
kubectl apply -f khj-rolebinding.yaml
```

| 필드 | 설명 |
| --- | --- |
| `metadata.name` | RoleBinding의 이름 |
| `metadata.namespace` | 대상 Role과 SA가 있는 네임스페이스 |
| `subjects` | 권한을 줄 대상 (여기서는 서비스 계정 `alicek106`) |
| `roleRef.kind` | 연결할 권한 리소스 종류 (`Role`) |
| `roleRef.name` | 미리 정의된 `Role`의 이름 (`service-reader`) |
| `roleRef.apiGroup` | RBAC API 그룹 지정 |

**주요 리소스의 API 그룹**

| 리소스 | API 그룹 |
| --- | --- |
| pods, services | `""` (core) |
| deployments | `apps` |
| ingresses | `networking.k8s.io` |
| roles, bindings | `rbac.authorization.k8s.io` |

---

권한이 부여되면 다음 명령이 동작 함

```bash
kubectl get service --as system:serviceaccount:default:khj
```

---

### 참고

Role은 **네임스페이스 단위** 권한입니다. 모든 네임스페이스에서 접근하려면 `ClusterRole` + `ClusterRoleBinding`을 사용해야 합니다.