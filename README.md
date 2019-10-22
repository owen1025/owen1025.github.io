## 최제필 **Jepil choi**

G: [https://github.com/owen1025](https://github.com/owen1025) / E: [myartame@gmail.com](myartame@gmail.com) / T: +82-10-2612-1052
</br>
</br>
</br>

# OVERVIEW
- 문서화를 가장 중요하게 생각합니다.
- 정답인 아키텍처는 없다고 생각합니다. 서비스와 조직에 맞는 최선의 아키텍처만 있다고 생각합니다.
</br>
</br>
</br>

# EMPLOYMENT
## **Terraform Labs**
<div style="color:gray">Lead DevOps engineer / 2018.09 ~</div>

### Payment service 운영을 위한 **NCP Hybrid zone과 사내 IDC 환경의 Container / On-premise 인프라 설계, 구축 및 운영 전담**
> Note: 아래 내용의 이해도와 가독성을 위해 NCP Hybrid zone(production): **PZ** / 사내 IDC(ecosystem / staging): **EZ**로 표기합니다.
  - **Container orchestration**: Docker swarm(multi master-worker node, PZ/EZ)
    - Container as a Code: docker-compose
    - Container reverse proxy / Load balancer: [Traefik](https://traefik.io/) + [Consul](https://www.consul.io/) HA 구성
    - Swarm node / container management: [Portainer](https://www.portainer.io/)
  - **iso 27001** 인증을 위한 인프라 환경의 네트워크 이중화 / Cent OS, PostgreSQL, Docker 보안 점검 대응 / Firewall rule / Etc 대응
    - (EZ) IPSec VPN + DNS(named) + [Traefik](https://traefik.io/) + [Consul](https://www.consul.io/) + [Registrator](https://github.com/gliderlabs/registrator) 환경을 구성하여 Container 기반의 management system(Kibana, Portainer, etc...) 접근 제어
    - [Vault](https://www.vaultproject.io/)(PZ/EZ): SSH OTP, PostgreSQL dynamic credential, Service container 내에 사용될 orm config, 생체 정보 key-value data에 관해 암호화 및 사용을 위해 Batch / service token 발급 및 관리
  - **Database system(On-premise)**
    - DBMS(PostgreSQL)
      - PostreSQL master-slave replication 구축
      - [Consul](https://www.consul.io/) + [HAProxy](http://www.haproxy.org/): Consul dynamic dns resolving을 통해 HAProxy에서 Service container의 요청에 따른 Sidecar-proxy로 구성, Consul에서 PostgreSQL master의 health check fail 발생 시 [Patroni](https://github.com/zalando/patroni)를 실행하여 PostgreSQL slave를 master로 승격 및 Consul에 등록된 master에 대한 정보 변경을 통해 자동 장애 대응
    - In-memory store(Redis)
      - Redis active-stand by 구축
      - Redis + Redis sentinel 구성으로 자동 장애 대응
  - **Ecosystem**
    - Linux host / Container / On-premise 환경의 **통합 모니터링을 위한 ELK Stack 구축**
      - [Elasticsearch](https://www.elastic.co/kr/products/elasticsearch) Cluster 구성: JVM option, index field data caching(sorting 성능 개선), curcuit breaker(OOM 방지), index lifecycle management(ILM) 적용
      - [Logstash](https://www.elastic.co/kr/products/logstash): filebeat 등 수집기에서 보낸 data 가공 및 정제
      - [Kibana](https://www.elastic.co/kr/products/kibana): Log, metric, apm 및 Payment 정보에 대한 Dashboard 시각화
      - [Filebeat](https://www.elastic.co/kr/products/beats/filebeat): Linux host의 sys, audit, daemon log / container log 수집 및 전송
      - [Metricbeat](https://www.elastic.co/kr/products/beats/metricbeat): Linux host 및 Container의 CPU, RAM, Disk IO 등 metric 수집 및 전송
      - [Elastic APM](https://www.elastic.co/kr/products/apm): Service container 내에 [apm-agent](https://www.elastic.co/guide/en/apm/agent/nodejs/index.html)에서 application performance, transaction 및 사용자의 행동 패턴을 key-value 형태로 Elastic APM server에 전송
      - [Heartbeat](https://www.elastic.co/kr/products/beats/heartbeat): On-premise, Container service의 Health check 및 Kibana Uptime 연동
    - **CI/CD** 환경 구축(EZ)
      - [Jenkins](https://jenkins.io) master-slave 구성
        - 각 서비스 Git repo의 Push, PR 등의 event 발생 시 Jenkinsfile을 읽어와 Pipeline 실행
        - Pipeline에서 build / push 진행 전 Unit test를 통과해야 build가 수행되며 develop, staging branch인 경우 EZ에 자동 배포 진행, master는 E2E test 및 부하 테스트를 통과해야만 배포 가능 설정
        - Jenkins agent의 label을 설정하여 특정 agent에서 Docker image 및 [Flutter](https://flutter.dev)의 iOS / Android build 진행
      - [Ngrinder](https://naver.github.io/ngrinder/) Controller-agent 구성
        - master에 배포 전 staging(EZ) 환경에서 결제 시나리오에 대한 부하 테스트 진행
        - Login, session, cookie 등 실 사용자의 시나리오에 맞게 groovy script 작성
    - **[Sentry](https://sentry.io/welcome/)**(error tracker) 구축
      - MSA로 구성된 service container에서 발생하는 error tracking 시스템
      - Sentry web, cron, worker, postgresql, redis
    - **Alert**: [Elasticsearch Watcher](https://www.elastic.co/guide/kr/x-pack/current/watcher-getting-started.html) + [Pagerduty](https://www.pagerduty.com)
      - [Heartbeat](https://www.elastic.co/kr/products/beats/heartbeat) indice의 health check fail(= Service down), [Filebeat](https://www.elastic.co/kr/products/beats/filebeat) indice의 fatal / error log 발생 시, [Metricbeat](https://www.elastic.co/kr/products/beats/metricbeat) indice의 On-premise, container 사용량이 특정 범위 초과 시, [Elastic APM](https://www.elastic.co/kr/products/apm) indice의 transaction의 response time이 특정 범위 초과 시, [Sentry](https://sentry.io/welcome/)에서 error tracking event 발생 시 [Pagerduty](https://www.pagerduty.com)에 triggering하여 Slack, SMS, Phone call로 알람 발생 및 서비스 장애에 대한 대응과 이슈에 대해 로깅 및 공유 진행
</br>
</br>

## **[Fooding](http://fooding.io)**
<div style="color:gray">(SI) DevOps engineer</div>

- **운영, 개발 인프라 구축 전담**
  - Container orchestration: Docker, ECS, EC2, ECR
  - 웹서버 (NginX), WAS (Node.js, HTTP)
  - 데이터베이스 (MySQL, RDS)
  - 트래픽 로드 밸런싱 (ALB, Route53)
  - CI/CD (Codepipeline, Codebuild, Github webhook)
    - 무중단(Blue-green) 배포
  - 모니터링 (Cloud watch)
- **API 개발 전담**
  - Node.js, Express, Knexjs, MySQL
  - 인증/인가 서비스 (JWT, AWS Lambda, API Gateway)
  - 메인(사용자) 서비스
  - 어드민(관리자, CMS) 서비스
  - 푸쉬(이메일, SMS) 서비스
  - HTML 기반의 PDF(세금계산서, 견적서 등) 렌더링 서비스
</br>
</br>

## **AJ Networks**
<div style="color:gray">DevOps engineer / 2016.05 ~ 2018.07</div>

- **차세대 ERP 시스템 인프라 설계, 구축 프로젝트 참여**
  - IDC 기반의 [DC/OS](https://dcos.io/)를 활용한 운영, 개발 인프라 구축 참여
    - IDC 마이그레이션, 컨테이너 오케스트레이션, 웹서버, WAS, 모니터링(로그, 메트릭, 퍼포먼스) 등
  - CI/CD 파이프라인 구축 및 운영 전담
  - 부하 테스트 환경 구축 및 운영 전담
- **데이터베이스 메타데이터 관리 도구 개발 전담**
  - ERP 시스템에서 사용될 메타데이터 관리 도구
    - 단어, 용어, 도메인 
  - 프로젝트(UI, API) 개발 전담
  - UI(HTML5, Materialize(+LESS), Vue.js, jQuery, EJS
  - API(Node.js, Express, Knexjs, MySQL(Percona))
</br>
</br>

## **[Pikicast](https://www.pikicast.com/)**
<div style="color:gray">Software engineer / 2014.07 ~ 2015.01</div>

- **피키캐스트 웹서비스 메이저 업데이트(2.0)**
  - API 개발 전담 및 기획 참여
  - PHP, Codeigniter, MySQL
- **피키캐스트 웹서비스(1.0) 유지보수 참여**
  - UI 유지보수 전담
  - HTML5, CSS, jQuery
</br>
</br>

## **Add2Paper**
<div style="color:gray">Software engineer / 2013.12 ~ 2014.07</div>

- 프린트 드라이버 및 클라이언트 유지보수
- 안드로이드 클라이언트 유지보수
</br>
</br>
</br>
</br>
</br>
</br>

# ACTIVITIES
## Fastcampus [DevOps 구축 BOOTCAMP](http://www.fastcampus.co.kr/dev_camp_devb/) 강사
- AWS, Docker를 활용한 인프라 구성 방법 강의 진행
- 1~5기 진행
</br>

## [Codeigniter-apidocs]((https://github.com/owen1025/codeigniter-apidocs))
- 오픈소스 프로젝트 전담
- Codeigniter로 기 개발 된 프로젝트의 컨트롤러를 분석하여 자동으로 API 테스팅 문서(응답/요청)를 작성. 
- 핵심 모듈 (PHP, Codeigniter) 개발
</br>

## [Zio Ball]((https://www.youtube.com/watch?v=0Lzv6W_c-lY))
- 2D 모바일 퍼즐 게임
- iOS / Android 스토어 100만 다운로드 기록
- 프로젝트(클라이언트) 개발, 기획 책임
- [Corona SDK](https://coronalabs.com/product/) Lua script 사용