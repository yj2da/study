# Kubernetes Secret

> 출처: [Kubernetes 공식 문서 - Secret](https://kubernetes.io/docs/concepts/configuration/secret/)

---

## 목차

1. [개요](#1-개요)
2. [보안 주의사항](#2-보안-주의사항)
3. [Secret 사용처](#3-secret-사용처)
4. [Secret 작업](#4-secret-작업)
5. [결론](#5-결론)

---

## 1. 개요

- **역할**: 비밀번호, OAuth 토큰, SSH 키 등 **민감한 정보**를 Pod/이미지 밖에 두고, Pod 실행 시 설정을 통해 컨테이너에 전달.
- **효과**: 애플리케이션 코드에 기밀 데이터를 넣지 않아도 됨. Secret은 Pod와 **독립적으로 생성** 가능해, 워크플로에서 노출 위험을 줄일 수 있음.

| 구분     | Secret | ConfigMap |
|----------|--------|-----------|
| 용도     | **기밀 데이터** (API 키, 자격 증명 등) | 비기밀 **설정 데이터** |
| 저장     | base64 인코딩 + 추가 보호 | 평문 |
| 노드 전달 | 해당 Pod가 필요한 Secret만 노드로 전송 | - |
| 디스크   | kubelet이 **tmpfs**에 보관 (영구 디스크 미사용), Pod 삭제 시 삭제 | - |

- Pod 내 여러 컨테이너는 **기본 ServiceAccount·관련 Secret**만 접근. 다른 Secret은 **환경 변수** 또는 **볼륨 마운트**로 명시해야 함.
- 한 Pod는 **다른 Pod의 Secret**에 접근할 수 없음.

---

## 2. 보안 주의사항

### 기본 동작

- Secret은 **etcd**에 **암호화되지 않은 상태**로 저장됨.
- API/etcd 접근 권한이 있거나, 네임스페이스에서 Pod를 만들 수 있는 권한이 있으면 해당 네임스페이스의 Secret을 읽을 수 있음 (배포 생성 등 간접 접근 포함).
- **etcd**: Kubernetes 클러스터용 경량·고가용 **key-value 저장소** (각 노드가 접근 가능).

### 권장 대응

1. [미사용 Secret 암호화](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/)
2. [RBAC](https://kubernetes.io/docs/reference/access-authn-authz/authorization/)로 Secret 접근 최소 권한 구성
3. **특정 컨테이너**에만 Secret 노출
4. 가능하면 [외부 Secret 저장소 공급자](https://secrets-store-csi-driver.sigs.k8s.io/concepts.html#provider-for-the-secrets-store-csi-driver) 사용

추가: [Kubernetes Secret 모범 사례](https://kubernetes.io/docs/concepts/security/secrets-good-practices)

### 기타

- **경고**: 노드에서 `privileged: true`로 실행되는 컨테이너는 해당 노드의 **모든 Secret**에 접근 가능.

---

## 3. Secret 사용처

Pod가 Secret을 쓰는 **세 가지 방법**:

| 방법 | 설명 |
|------|------|
| **볼륨의 파일** | Secret 데이터를 컨테이너 파일 시스템에 파일로 마운트 |
| **환경 변수** | `env[].valueFrom.secretKeyRef`로 Secret 키를 환경 변수에 매핑 |
| **이미지 풀** | `imagePullSecrets`로 프라이빗 레지스트리 인증 (Pod 단위, kubelet이 사용) |

Control plane도 bootstrap 토큰 등에 Secret을 사용함.

### 3.1 볼륨(파일)으로 사용

- Secret을 **볼륨**으로 마운트하면, Secret이 **업데이트될 때** Kubernetes가 볼륨 데이터를 갱신함.
- **주의**: **subPath**로 마운트한 경우에는 **자동 갱신되지 않음**.
- kubelet은 Secret 키/값 **캐시**를 유지. 갱신 전략은 `configMapAndSecretChangeDetectionStrategy`로 설정 (기본 `watch`). 전파 지연은 kubelet 동기화 주기 + 캐시 전략에 따름.

- 설정 가이드: [Create a pod that has access to the secret data through a volume](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/#create-a-pod-that-has-access-to-the-secret-data-through-a-volume)

### 3.2 환경 변수로 사용

- Pod 사양에서 `env[].valueFrom.secretKeyRef`로 Secret 키를 환경 변수에 연결.
- **잘못된 환경 변수 이름**(예: `1badkey`, `2alsobad`)인 키는 **건너뛰고**, Pod는 그대로 기동됨. `kubectl get events`에서 `InvalidVariableNames` 이유로 건너뛴 키 목록 확인 가능.

- 설정 가이드: [Define container environment variables using secret data](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/#define-container-environment-variables-using-secret-data)

### 3.3 이미지 풀 (imagePullSecrets)

- **프라이빗 레지스트리** 이미지를 pull하려면 각 노드의 kubelet이 인증이 필요. **imagePullSecrets**로 레지스트리 비밀번호가 담긴 Secret을 Pod에 지정하면, kubelet이 Pod 대신 이미지를 가져옴.
- **수동**: Pod 사양의 `imagePullSecrets`에 Secret 참조 지정.  
- **자동**: ServiceAccount에 `imagePullSecrets`를 설정하면, 해당 ServiceAccount를 쓰는 Pod에 자동으로 붙음.  
  → [서비스 계정에 ImagePullSecrets 추가](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#add-imagepullsecrets-to-a-service-account)

### 3.4 정적 Pod

- **정적 Pod**에는 ConfigMap/Secret을 **사용할 수 없음**.

---

## 4. Secret 작업

### 생성 방법 (3가지)

- [kubectl로 생성](https://kubernetes.io/docs/tasks/configmap-secret/managing-secret-using-kubectl/)
- [설정 파일(YAML)로 생성](https://kubernetes.io/docs/tasks/configmap-secret/managing-secret-using-config-file/)
- [Kustomize로 생성](https://kubernetes.io/docs/tasks/configmap-secret/managing-secret-using-kustomize/)

### 이름·데이터 제약

- **이름**: 유효한 DNS 서브도메인.
- **data**: 모든 값은 **base64** 인코딩 문자열.
- **stringData**: 평문 문자열 허용. 내부적으로 `data`에 병합됨. 같은 키가 있으면 **stringData**가 우선.
- **키 문자**: 영숫자, `-`, `_`, `.` 만 허용.

### 크기 제한

- **개별 Secret**: 최대 **1MiB**.  
- 작은 Secret을 매우 많이 만들면 메모리 부담 가능 → [Resource Quota](https://kubernetes.io/docs/concepts/policy/resource-quotas/)로 네임스페이스별 Secret 수 제한 권장.

### 수정

- **Immutable**이 아니면 기존 Secret 편집 가능.  
  - [kubectl edit](https://kubernetes.io/docs/tasks/configmap-secret/managing-secret-using-kubectl/#edit-secret), [설정 파일로 수정](https://kubernetes.io/docs/tasks/configmap-secret/managing-secret-using-config-file/#edit-secret), Kustomize(수정된 데이터로 **새 Secret 객체** 생성).
- 볼륨으로 마운트한 Secret은 업데이트 시 Pod에 자동 전파됨. 자세한 동작은 [Mounted secrets are updated automatically](https://kubernetes.io/docs/concepts/configuration/secret/#mounted-secrets-are-updated-automatically) 참고.

---

## 5. 결론

- **보안이 필요한 값**은 ConfigMap(평문) 대신 **Secret** 사용.
- Secret은 **base64**일 뿐이므로, Secret만으로는 충분히 안전하지 않음. **암호화 at rest, RBAC, 컨테이너별 접근 제한, 외부 Secret 저장소 공급자** 등을 함께 적용하는 것이 좋음.
- **만드는 방법**: kubectl, 설정 파일, Kustomize.
- **쓰는 방법**: (1) 볼륨의 파일, (2) 환경 변수, (3) imagePullSecrets(kubelet 이미지 풀).

---

## 참고

- https://kubernetes.io/docs/concepts/configuration/secret/
- https://oss.navercorp.com/lago/clender/issues/43
- https://wiki.navercorp.com/pages/viewpage.action?pageId=1178108716 (Secret 설정 등)
