
# GenAI 데이터 파이프라인 & RAG 서비스

## 프로젝트 개요
사내 EDW 데이터를 활용하여 외부 클라우드 지식 저장소로 적재하고,
Retriever 기반 RAG 서비스를 제공하기 위한 End-to-End 데이터 파이프라인 및 API 서비스입니다.

## 프로젝트 구조

```
project/
├── agent/            # API 서비스 (RAG 서비스 요청 처리)
├── pipeline/         # 데이터 수집, EAI 송신, Retriever 생성 파이프라인
├── scripts/          # 실행 스크립트
├── test/             # 테스트 코드
├── requirements.txt  # Python 패키지 목록
├── .gitignore        # Git 제외 파일 목록
└── README.md         # 프로젝트 설명
```

## 주요 기능

### 데이터 파이프라인 (pipeline/)
- EDW → S3 데이터 추출
- 메타파일(JSON) 자동 생성
- EAI 송신을 통한 외부 지식 저장소 적재
- 청킹, 임베딩을 통한 Retriever 생성

### API 서비스 (agent/)
- /query: 사용자 인풋에 따라 RAG 프로세스 수행
- /health: 서비스 상태 확인
- Orchestrator, Repository, Model 구조로 클린 아키텍처 적용

## 업무 구조

[1. 내부망 EDW]
    ↓ (배치 데이터 추출)
[2. 분석환경 (Python Batch)]
    ↓ (S3 적재 + JSON 메타파일 생성)
[S3 버킷]
    ↓ (Event Trigger)
[3. EC2 Batch Server]
    ↓ (jar 실행 → EAI 송신)
[EAI → 외부 클라우드 지식저장소]
    ↓
[4. Cloud Batch (청킹/임베딩/Retriever 생성)]
    ↓
[5. SageMaker/분석환경 → RAG 질의 호출]

1. EDW-S3 일배치 적재
- SageMaker에서 EDW 접속하여 SQL 데이터 추출 후 CSV 파일 S3버킷에 저장
- S3 저장된 파일을 Trigger로 사용
- 분석환경 내 Batch 스크립트 개발 필요

2. 메타파일 생성
- S3 내 CSV 업로드 완료 시 Lambda 또는 S3 Event Trigger 발생
- Python 스크립트 실행하여 파일에 대한 RAG용 Metadata JSON 자동 생성
- 생성된 메타파일은 S3 버킷에 함께 저장
- 메타파일 생성 Python 프로그램 개발 필요

3. EAI 송신
- S3에서 메타파일 저장 완료 시 S3 Event 또는 CloudWatch로 EC2 알림
- EC2 내에서 특정 jar 파일 실행
- EAI 송신 서버를 통해 지주 클라우드 지식저장소로 파일 송신
- EC2 Batch Server

4. 지주플랫폼 지식저장소
- 적재된 파일 감지(EAI 송신 완료 시점)
- 주기적으로 Batch 프로세스 실행(청킹, 임베딩, 리트리벌 생성 등)
- FabriX

5. 분석환경 RAG 호출
- AWS 노트북 환경에서 LLM과 Retrieval API 호출
- 지식저장소 기반으로 RAG 질의 서비스 제공

## 실행 방법

1. 데이터 파이프라인 실행 (전체 배치)
```bash
sh scripts/run_full_pipeline.sh
```

2. API 서버 실행 (FastAPI)
```bash
sh scripts/run_agent_server.sh
```

3. Retriever 생성 배치 실행
```bash
sh scripts/run_retriever_batch.sh
```

## 개발 환경
- Python 3.9+
- FastAPI
- Boto3 (AWS S3 연동)
- SQLAlchemy (DB 연동)
- OpenSearch or FAISS (Retriever)
- OpenAI API (LLM 호출)

## 향후 작업
- CI/CD 파이프라인 추가 (GitHub Actions, CodePipeline 등)
- S3 Event, Lambda Trigger 연계
- Unit Test 추가
- 운영 배치 스케줄링 (Airflow or Step Function)

## 라이선스
사내 프로젝트용 Private Repository
