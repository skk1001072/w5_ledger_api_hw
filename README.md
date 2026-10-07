# 가계부 API (FastAPI + Supabase)

- GitHub: (push 후 이 저장소 주소로 채우기)
- Render: (배포 후 https://....onrender.com 주소로 채우기)

FastAPI + SQLAlchemy로 계좌(Account)·카테고리(Category)·거래(Transaction)를 관리하는 가계부 API입니다.
DB는 Supabase(PostgreSQL)를 사용하며, Render에 배포되어 인터넷에서 접근할 수 있습니다.

## 실행 방법

```
.venv\Scripts\activate
pip install -r requirements.txt
uvicorn main:app --reload
```

`.env`에 `DATABASE_URL`(Supabase Session pooler 연결 문자열, `postgresql+psycopg://...`)을 설정해야 합니다.

## 엔드포인트

| 메서드 | 경로 | 설명 |
|---|---|---|
| POST | /accounts | 계좌 생성 |
| GET | /accounts | 계좌 목록 |
| GET | /accounts/{id} | 계좌 단건 조회 |
| POST | /transactions | 거래 생성 |
| GET | /accounts/{id}/detail | 계좌 + 거래 중첩 조회 |
| GET | /stats/by-category | 카테고리별 지출 집계 |

## ① 결과 확인

- Supabase Table Editor에서 `accounts`·`categories`·`transactions` 테이블과 샘플 데이터(계좌 2개, 카테고리 3개, 거래 3건)를 확인함.
- `GET /accounts/1/detail` 호출 시 계좌 정보에 거래 목록이 중첩되어 반환됨을 확인함.
- `GET /stats/by-category` 호출 시 카테고리별 지출 합계(`식비 -12000`, `교통 -1500`)가 정상 집계됨을 확인함.
- (Render 배포 완료 후) `https://<서비스>.onrender.com/docs`의 `GET /accounts`가 로컬에서 만든 것과 같은 데이터를 반환함을 확인함.

## ② 핵심 개념 되새김

- 계좌와 거래를 두 테이블로 나눈 이유: 계좌 하나가 여러 거래를 가질 수 있는 1:N 관계이기 때문이다. 하나로 합치면 계좌 정보가 거래 건수만큼 중복 저장되어, 계좌 이름 하나를 바꿀 때도 관련된 모든 거래 행을 함께 고쳐야 한다.
- SQLAlchemy 모델 클래스와 실제 테이블의 대응: `class Account(Base)`처럼 클래스 하나가 테이블 하나에 대응하고, `Mapped[str]`·`Mapped[int]` 같은 타입 힌트가 그대로 컬럼 타입과 NOT NULL 제약으로 변환된다. `Base.metadata.create_all()`이 이 클래스 정의를 보고 실제 `CREATE TABLE`을 실행한다.
- 접속 문자열을 `.env`로 분리하는 이유: DB 비밀번호가 포함된 연결 문자열을 코드에 직접 쓰면 Git에 커밋될 때 저장소 이력에 비밀번호가 영구히 남는다. `.env`로 분리하고 `.gitignore`에 등록하면 코드와 비밀 정보가 분리되어, 로컬/Render 등 환경마다 다른 값을 코드 변경 없이 넣을 수 있다.

## ③ 자유 로그

- Claude Code에게 `main.py`·`schemas.py`에 빠져 있던 단계 4(거래 생성, 계좌 상세 중첩 응답, 카테고리별 집계 API) 코드를 워크북 기준으로 채워 넣도록 시켰고, 실행 후 실제 Supabase에 쿼리가 나가는지 SQL 로그로 확인했다.
- 계좌·거래 샘플 데이터를 API로 직접 생성해 Supabase Table Editor에서 눈으로 확인했다.
- GitHub 저장소 생성과 Render 배포는 브라우저 로그인(OAuth)이 필요해 직접 진행함.
