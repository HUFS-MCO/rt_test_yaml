본 프로젝트는 ROS 2 Humble 기반 컨테이너 환경에서 실시간 성능 평가를 위한 테스트 시나리오를 자동화하고, 테스트 실행 시점을 모니터링하기 위한 모듈을 포함합니다. `monitor` 컨테이너와 `cyclictest`, `pi_stress`, `signaltest` 테스트를 하나의 Pod에서 **순차적으로 실행**하는 totaltest 구성을 제공합니다.

## 구성 요소

- **monitor (monitor container)**  
  `/test_event` 토픽을 구독하여 테스트의 시작/종료 타임스탬프를 기록

- **cyclictest**  
  주기적 타이머 인터럽트 지연 시간 측정

- **pi_stress**  
  실시간 스케줄링 환경에서 동기화 연산 스트레스 테스트 수행

- **signaltest**  
  실시간 프로세스 간 시그널 처리 지연 측정

## Pod 실행 흐름

`CMD=all` 환경 변수로 실행 시, 아래 순서대로 실행됩니다:

1. **cyclictest**
2. **pi_stress**
3. **signaltest**

각 테스트의 시작/종료 시점은 ROS2 토픽 `/test_event`를 통해 모니터링 컨테이너에 전달됩니다.

## 사용 방법


Kubernetes 배포
monitor.yaml과 totaltest.yaml 파일을 ros-test 네임스페이스에 적용합니다
```bash
kubectl apply -f monitor.yaml -n ros-test
kubectl apply -f totaltest.yaml -n ros-test
```

### 환경 변수 인자 조정

totaltest.yaml 내에서 각 테스트별 인자값 조정
```yaml
env:
  - name: CMD
    value: "all"
  - name: CYCLICTEST_ARGS
    value: "-l 10000 -i 1000 --json=/output/cyclictest_result.json"
  - name: PISTRESS_ARGS
    value: "-g 8 -i 100000"
  - name: SIGNALTEST_ARGS
    value: "-p 30 -l 100"

```
## 결과 확인
모든 테스트의 시작과 종료 시점은 ROS2 모니터링 노드를 통해 다음과 같이 출력됩니다:
``` bash
[2025-04-30 14:31:20] START for cyclictest
[2025-04-30 14:31:23] DONE for cyclictest
```
## 요구사항

- ROS 2 Humble 기반 환경

- Kubernetes cluster

- HostNetwork 사용 권장 (ROS2 통신을 위한 DDS discovery 보장)