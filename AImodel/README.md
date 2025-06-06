# 약속 시간 

## 프로젝트 목적

### 핵심 아이디어 
- 고령층을 위한 복약 알림 서비스
### 타겟층은 누구?
- 여러가지 약을 제때 정확하게 복용하는데 어려움이 있는 고령층
### 차별성
- 기존 알람 서비스 : 처방전의 QR을 인식하여 약 먹는 시간에 알람을 제공하는 앱 서비스 / 맞춤화된 약에 대한 정보 제공 부족 
	[“‘똑닥’ - 뉴스”](https://www.k-health.com/news/articleView.html?idxno=33800)
- 기존 검색 서비스 : 약 검색을 해주는 웹 사이트 / 이해하기 어려운 전문용어들로 적혀져 있음 
	["약학정보원"](https://www.health.kr/)
### 제공할 서비스
- 이미지 인식 : 처방전을 촬영하는 것 만으로 복용중인 약들을 등록할 수 있도록 하는 서비스.
- 자연어 인식 : 채팅 및 음성의 형태로 고객이 질문을 하면 약 관련 키워드를 뽑아내어 맞춤화된 답변을 제공.
- 높은 신뢰성 : 식품의약품안전처_의약품안전사용서비스(DUR)성분정보 데이터를 기반으로 신뢰성 높은 답변을 할 수 있음.
- 쉬운말 답변 : 복잡한 약품 정보들을 쉬운 말로 풀어서 설명할 수 있음
- 복약 스케줄 : 사용자가 주기적으로 약을 챙겨 복용할 수 있도록 알람 및 기록 서비스 제공
### MVP
- 웹 기반 서비스 제공
- DUR API 데이터를 사용하여 신뢰성 있는 답변을 제공한다. 
- 챗봇 서비스 : 텍스트를 입력으로 받아 고객에게 답변을 제공한다. 
- 사용자가 UI에서 복용중인 약을 텍스트로 입력할 수 있게 한다.
### 보완할 점
- 음성 인식으로 질문을 할 수 있다면 고령층의 편의성을 높일 수 있을 것이다.
- 웹 복용 약 입력시 입력한 글자가 포함된 약의 항목(suggest box)이 나타나면 어려운 약 이름 검색이 편해질 것이다.
- 약에 대한 질문이 아닌 것을 질문할 시에 답변하지 않거나, 일반 채팅을 시작하도록 권유 한다. (모델 전문성 향상)

## 기술 스택 
### AI
- OPENAI API : DUR API에서 가져온 의약 데이터를 기반으로 기본 챗 응답 생성
- GEMINI API : openai api에서 생성한 답변을 질문에 해당 하는 이해하기 쉽게 문장을 다듬음.
- TAVILY API : 데이터 베이스에 없는 정보를 보완하는 역할.
- LANGCHAIN : ChatPromptTemplete 사용하여 시스템 프롬프트 설정 / openai api, gemini api 호출하여 외부 api에 연결
- FAST API : 엔드포인트를 구현하여 백엔드에서 HTTP 요청으로 모델 결과에 요청 및 답변.(JSON format)
- DUR-data API : 제품명, 제조사명, 복용법, 주의사항, 상호작용에 대한 정보 추출하기 위한 데이터
- Sqlite.db : 채팅의 대화 내용을 기억하여 답변하기 위한 데이터 베이스 

### 파일 구조
```
.
├── data/
│   ├── input_med.json # 사용자가 복약중인 약의 이름 정보 (UI 입력을 통해 기록된 데이터)
│   ├── new_data.json # TAVILY 모델이 생성한 데이터
│   └── target_data.json # 사용자가 복약중이라고 응답한 약에 대한 정보만을 저장한 데이터
├── model/
│   ├── ChatModel.py # OPENAI 중심으로 채팅 구현 기능을 하기 위한 클래스 모델
│   ├── DataLoader.py # 여러 JSON 데이터에서 변수로 불러와 모델에서 사용하기 위한 함수
│   ├── GeminiModel.py # 입력에 대한 Gemini 답변을 생성해주는 모델
│   └── TavilyModel.py # 입력에 대한 URI 사이트 검색 및 GPT4o를 통한 답변 생성
├── utils/
│   └── .env # API KEY를 관리하기 위한 파일
├── sqlite.db # 대화 내용 history 저장
├── install.sh # 한번에 필요한 라이브러리를 설치 하기 위한 
├── test.http # 서버 쿼리 요청이 제대로 되는 지 확인하기 위한 파일
├── .gitignore 
├── main.py # 모델을 앱의 형태로 실행하기 위한 파일
├── server.py # AIP 통신을 위한 엔드포인트 생성 및 query 생성
└── README.md
```

### 기본 세팅
- Python 확장팩
- REST Client 확장팩 설치
- Rainbow CSV 확장팩 설치
- $ bash install.sh 실행

### 파일 설정
- .env : Ai api key 정보를 담은 파일 (반드시 .gitignore에 추가)
    - OPENAI_API_KEY=your_openai_api_key
    - GEMINI_API_KEY=your_gemini_api_key
    - TAVILY_API_KEY=your_tavily_api_key
- .gitignore : git에 업로드 시 경로상에 검색되지 않도록 할 파일을 입력
    - .env
    - __ Pychache __/
- install.sh 
    - $ bash install.sh 터미널 실행 : 필요한 라이브러리 설치를 위한 파일
- main.py 
    - uvicorn main:app --reload : : 모델을 uvicorn app으로 실행하는 코드


