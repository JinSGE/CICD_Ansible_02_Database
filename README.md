# CICD_Ansible_02_Database: PostgreSQL HA Cluster with Patroni & etcd

본 프로젝트는 **PostgreSQL 고가용성(HA) 클러스터**를 구축하고 운영하기 위한 Ansible 플레이북과 구성 파일들을 포함하고 있습니다. **Patroni**를 통한 자동 페일오버, **etcd**를 이용한 분산 상태 저장, 그리고 **HAProxy**를 전측(Proxy)으로 배치하여 안정적인 데이터베이스 레이어를 제공합니다.

## 🏗 아키텍처 개요

프로젝트 규모와 역할에 따라 다음과 같이 계층(Layer)이 분리되어 있습니다.

1.  **Proxy Layer (HAProxy / Keepalived)**:
    *   데이터베이스 접속을 위한 단일 엔드포인트(VIP)를 제공합니다.
    *   HAProxy가 Patroni API를 체크하여 현재 'Primary' 노드로 트래픽을 라우팅합니다.
2.  **Consensus Layer (etcd)**:
    *   클러스터의 리더 선출 및 상태 정보를 보관하는 분산 키-값 저장소입니다.
3.  **Database Layer (PostgreSQL / Patroni)**:
    *   Patroni가 PostgreSQL의 생명주기를 관리하며, 장애 발생 시 자동으로 Standby를 Primary로 승격시킵니다.
    *   동기/비동기 복제 설정을 통해 데이터 일관성을 유지합니다.
4.  **Backup Layer**:
    *   별도의 백업 전용 노드 또는 스토리지(NFS)를 통해 주기적인 데이터 백업을 수행합니다.

## 📂 폴더 구조 분석

*   `ansible.cfg`: Ansible 실행 환경 설정.
*   `inventory/`: 대상 서버 정보 (`prod.ini` 등).
*   `group_vars/`: 각 그룹별 공통 및 비밀 변수 (Vault 사용 권장).
*   `roles/`: 각 구성 요소별 설치 및 설정 로직.
    *   `proxy`: HAProxy, Keepalived 구성.
    *   `etcd`: etcd 클러스터 구축.
    *   `db`: PostgreSQL 및 Patroni 설치 및 설정.
    *   `dbT`: 서비스용 DB 객체(Database, Schema, User, Grant) 생성.
    *   `backup`: 백업 정책 및 스토리지 구성.
*   `site.yml`: 전체 인프라 구축을 위한 메인 플레이북.
*   `Jenkinsfile`: CI/CD 파이프라인 구성을 위한 스크립트.

## 🚀 시작하기

### 1. 요구 사항
*   Ansible 2.9 이상
*   대상 서버: RHEL/CentOS Stream 9 기반 (또는 호환 OS)
*   `requirements.yml`에 명시된 컬렉션 설치:
    ```bash
    ansible-galaxy collection install -r requirements.yml
    ```

### 2. 인벤토리 설정
`inventory/prod.ini` 파일을 환경에 맞게 수정하십시오. IP 주소 및 `ansible_user`, SSH 키 경로 등을 확인해야 합니다.

### 3. 플레이북 실행
전체 인프라를 한 번에 구축하려면 다음 명령을 실행합니다.
```bash
ansible-playbook -i inventory/prod.ini site.yml
```

특정 레이어만 작업하려면 태그(`--tags`)를 이용할 수 있습니다.
*   Proxy만: `ansible-playbook -i ... site.yml --tags proxy`
*   DB 인프라만: `ansible-playbook -i ... site.yml --tags db_infra`
*   DB 객체만: `ansible-playbook -i ... site.yml --tags dbT`

## 🛠 주요 기능
*   **고가용성**: Patroni를 통한 자동 리더 승격 및 HAProxy를 통한 투명한 접속 지원.
*   **보안**: Ansible Vault를 활용한 패스워드 및 비밀 키 관리.
*   **자동화**: DB 설치부터 사용자 계정 생성, 백업 설정까지 전 과정을 자동화.

