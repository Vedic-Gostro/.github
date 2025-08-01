# 베딕 점성술 + AI 분석 시스템: 마이크로서비스 아키텍처 종합 분석

## 프로젝트 개요

이 시스템은 전통적인 베딕 점성술(Jyotish)의 정밀한 계산과 현대 AI 기술을 결합한 혁신적인 웹 기반 점성술 분석 플랫폼입니다. 마이크로서비스 아키텍처를 채택하여 확장성과 유지보수성을 극대화했으며, Docker 컨테이너 기반으로 구현되어 있습니다.

### 핵심 기능
- **정밀한 베딕 점성술 계산**: Swiss Ephemeris 라이브러리를 활용한 천체 위치 계산
- **AI 기반 개인화 해석**: Google Gemini API를 통한 자연어 점성술 분석
- **사용자 친화적 웹 인터페이스**: 직관적인 출생 정보 입력 폼
- **실시간 분석**: 즉시 점성술 차트 생성 및 AI 해석 제공

## 시스템 아키텍처 다이어그램

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              USER INTERACTION                              │
└─────────────────────────────────────────────────────────────────────────────┘
                                        │
                                        │ HTTP/HTTPS
                                        ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                     FRONTEND (Nginx + Static Web)                          │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐            │
│  │   index.html    │  │   script.js     │  │   style.css     │            │
│  │                 │  │                 │  │                 │            │
│  │ - Form Handler  │  │ - API Client    │  │ - UI Styling    │            │
│  │ - UI Components │  │ - Data Binding  │  │ - Responsive    │            │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘            │
│                                                                             │
│  Container: vedic-astro-frontend (Port: 80)                                │
└─────────────────────────────────────────────────────────────────────────────┘
                                        │
                                        │ POST /astro
                                        │ JSON: {date, time, location}
                                        ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                      API GATEWAY (Go Service)                              │
│  ┌─────────────────────────────────────────────────────────────────────────┐│
│  │                        main.go                                          ││
│  │  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐     ││
│  │  │  handleRequest  │───▶│ validateRequest │───▶│  routeToService │     ││
│  │  └─────────────────┘    └─────────────────┘    └─────────────────┘     ││
│  │                                                                         ││
│  │  Service Discovery: Environment Variables                               ││
│  │  - JYOTISH_SERVICE_IP                                                   ││
│  │  - GEMINI_SERVICE_IP                                                    ││
│  └─────────────────────────────────────────────────────────────────────────┘│
│                                                                             │
│  Container: vedic-api-gateway (Port: 9595)                                 │
└─────────────────────────────────────────────────────────────────────────────┘
                        │                              │
                        │ GET /api/calculate           │ POST /interpret
                        │ Query Params                 │ JSON Body
                        ▼                              ▼
┌─────────────────────────────────┐    ┌─────────────────────────────────┐
│       JYOTISH API               │    │      GEMINI API SERVICE        │
│  ┌─────────────────────────────┐│    │  ┌─────────────────────────────┐│
│  │        Symfony PHP          ││    │  │           Go HTTP           ││
│  │                             ││    │  │                             ││
│  │ ┌─────────────────────────┐ ││    │  │ ┌─────────────────────────┐ ││
│  │ │   APIController.php     │ ││    │  │ │       main.go           │ ││
│  │ │                         │ ││    │  │ │                         │ ││
│  │ │ - Parameter Validation  │ ││    │  │ │ - Request Handler       │ ││
│  │ │ - Coordinate Transform  │ ││    │  │ │ - AI Prompt Builder     │ ││
│  │ │ - Swiss Ephemeris Call  │ ││    │  │ │ - Google API Client     │ ││
│  │ └─────────────────────────┘ ││    │  │ └─────────────────────────┘ ││
│  │                             ││    │  │                             ││
│  │ ┌─────────────────────────┐ ││    │  │ ┌─────────────────────────┐ ││
│  │ │   Jyotish Library       │ ││    │  │ │    HTTP Client          │ ││
│  │ │                         │ ││    │  │ │                         │ ││
│  │ │ - Vedic Calculations    │ ││    │  │ │ - generativelanguage    │ ││
│  │ │ - Planet Positions      │ ││    │  │ │   .googleapis.com       │ ││
│  │ │ - House Systems         │ ││    │  │ │ - API Key Auth          │ ││
│  │ │ - Ashtakavarga          │ ││    │  │ │ - Response Processing   │ ││
│  │ │ - Divisional Charts     │ ││    │  │ └─────────────────────────┘ ││
│  │ └─────────────────────────┘ ││    │  └─────────────────────────────┘│
│  └─────────────────────────────┘│    │                                 │
│                                 │    │  Container: call-gemini-api     │
│  Container: jyotish_api         │    │  (Port: 9494)                   │
│  (Port: 9393)                   │    │                                 │
└─────────────────────────────────┘    └─────────────────────────────────┘
                  │                                      │
                  │ Subprocess Call                      │ HTTPS API Call
                  ▼                                      ▼
┌─────────────────────────────────┐    ┌─────────────────────────────────┐
│     SWISS EPHEMERIS             │    │        GOOGLE GEMINI API        │
│  ┌─────────────────────────────┐│    │  ┌─────────────────────────────┐│
│  │       swetest Binary        ││    │  │  generativelanguage         ││
│  │                             ││    │  │  .googleapis.com            ││
│  │ - Astronomical Calculations ││    │  │                             ││
│  │ - Planet Ephemeris Data     ││    │  │ - Natural Language AI       ││
│  │ - House Calculations        ││    │  │ - Content Generation        ││
│  │ - Coordinate Transformations││    │  │ - Contextual Analysis       ││
│  └─────────────────────────────┘│    │  │ - Multi-turn Conversation   ││
│                                 │    │  └─────────────────────────────┘│
│  Embedded in jyotish_api        │    │                                 │
│  container                      │    │  External Google Service       │
└─────────────────────────────────┘    └─────────────────────────────────┘
```

## 네트워크 패킷 흐름 분석

```
CLIENT                    FRONTEND                    GATEWAY                    JYOTISH                    GEMINI                    GOOGLE
  │                         │                          │                          │                          │                          │
  │ 1. HTTP POST           │                          │                          │                          │                          │
  │ /birth-chart-form      │                          │                          │                          │                          │
  │ ────────────────────▶  │                          │                          │                          │                          │
  │                         │                          │                          │                          │                          │
  │                         │ 2. POST /astro          │                          │                          │                          │
  │                         │ JSON: astro data        │                          │                          │                          │
  │                         │ ──────────────────────▶ │                          │                          │                          │
  │                         │                          │                          │                          │                          │
  │                         │                          │ 3. GET /api/calculate   │                          │                          │
  │                         │                          │ Query: lat,lng,date,time │                          │                          │
  │                         │                          │ ──────────────────────▶ │                          │                          │
  │                         │                          │                          │                          │                          │
  │                         │                          │                          │ 4. subprocess swetest    │                          │
  │                         │                          │                          │ ──────────────────────▶ │                          │
  │                         │                          │                          │ ◀──────────────────────  │                          │
  │                         │                          │                          │ 5. JSON chart data      │                          │
  │                         │                          │                          │                          │                          │
  │                         │                          │ ◀──────────────────────  │                          │                          │
  │                         │                          │ 6. Chart calculation     │                          │                          │
  │                         │                          │    results (JSON)       │                          │                          │
  │                         │                          │                          │                          │                          │
  │                         │                          │ 7. POST /interpret      │                          │                          │
  │                         │                          │ JSON: chart + prompt    │                          │                          │
  │                         │                          │ ──────────────────────────────────────────────▶   │                          │
  │                         │                          │                          │                          │                          │
  │                         │                          │                          │                          │ 8. HTTPS POST          │
  │                         │                          │                          │                          │ /v1beta/models/         │
  │                         │                          │                          │                          │ gemini-pro:generateContent
  │                         │                          │                          │                          │ ──────────────────────▶ │
  │                         │                          │                          │                          │                          │
  │                         │                          │                          │                          │ ◀──────────────────────  │
  │                         │                          │                          │                          │ 9. AI Analysis Response │
  │                         │                          │                          │                          │                          │
  │                         │                          │ ◀──────────────────────────────────────────────   │                          │
  │                         │                          │ 10. Interpreted Results │                          │                          │
  │                         │                          │     (JSON)              │                          │                          │
  │                         │                          │                          │                          │                          │
  │                         │ ◀──────────────────────  │                          │                          │                          │
  │                         │ 11. Final Response      │                          │                          │                          │
  │                         │     (HTML/JSON)         │                          │                          │                          │
  │                         │                          │                          │                          │                          │
  │ ◀────────────────────── │                          │                          │                          │                          │
  │ 12. Rendered Chart      │                          │                          │                          │                          │
  │     + AI Analysis       │                          │                          │                          │                          │
```

## 컨테이너 및 서비스 상세 분석

### 1. vedic-astro-frontend (Port: 80)
**역할**: 사용자 인터페이스 제공
**기술 스택**: Nginx + HTML/CSS/JavaScript
**핵심 기능**:
- 출생 정보 입력 폼 (날짜, 시간, 위치)
- 점성술 차트 시각화
- AI 분석 결과 표시
- 반응형 웹 디자인

**주요 파일**:
- `index.html`: 메인 웹 페이지
- `script.js`: API 호출 및 DOM 조작 로직
- `style.css`: 사용자 인터페이스 스타일링
- `nginx.conf`: Nginx 서버 설정

### 2. vedic-api-gateway (Port: 9595)
**역할**: API 오케스트레이션 및 라우팅
**기술 스택**: Go (net/http)
**핵심 기능**:
- 단일 진입점 제공 (`POST /astro`)
- 마이크로서비스 간 데이터 흐름 관리
- CORS 처리
- 요청 검증 및 변환

**서비스 체이닝 플로우**:
1. 클라이언트 요청 수신
2. Jyotish API 호출 (점성술 계산)
3. Gemini API 호출 (AI 해석)
4. 통합 결과 반환

### 3. jyotish_api (Port: 9393)
**역할**: 베딕 점성술 계산 엔진
**기술 스택**: PHP/Symfony + Swiss Ephemeris
**핵심 기능**:
- 정밀한 천체 위치 계산
- 베딕 점성술 차트 생성
- 다양한 분할 차트 (Varga) 지원
- Ashtakavarga 계산

**지원하는 계산**:
- **행성 위치**: 태양, 달, 화성, 수성, 목성, 금성, 토성
- **림프 노드**: 라후(Rahu), 케투(Ketu)
- **하우스 시스템**: Placidus, Koch, Equal House 등
- **분할 차트**: D1(Rashi), D9(Navamsa), D10(Dasamsa) 등
- **요가 계산**: 라자 요가, 다나 요가 등

### 4. call-gemini-api (Port: 9494)
**역할**: AI 기반 점성술 해석
**기술 스택**: Go + Google Gemini API
**핵심 기능**:
- 점성술 데이터를 AI 컨텍스트로 변환
- Google Gemini Pro 모델 호출
- 개인화된 점성술 해석 생성
- 자연어 분석 결과 반환

**AI 프롬프트 엔지니어링**:
- 베딕 점성술 전문 컨텍스트 주입
- 개인 출생 데이터 기반 맞춤형 질문
- 행성 위치와 하우스 의미 해석
- 요가와 도샤 분석

## 기술적 특징 및 장점

### 마이크로서비스 아키텍처의 이점
1. **모듈성**: 각 서비스가 독립적으로 개발/배포 가능
2. **확장성**: 개별 서비스를 필요에 따라 스케일링
3. **기술 다양성**: 각 서비스에 최적화된 기술 스택 선택
4. **장애 격리**: 한 서비스 장애가 전체 시스템에 미치는 영향 제한

### Swiss Ephemeris 통합
- **정확도**: NASA JPL 데이터 기반의 정밀한 천체 계산
- **완전성**: 기원전 13세기부터 서기 17세기까지의 천체 데이터
- **표준화**: 세계적으로 인정받는 점성술 계산 라이브러리

### Google Gemini AI 활용
- **자연어 처리**: 점성술 용어를 일반인이 이해하기 쉬운 언어로 변환
- **컨텍스트 이해**: 복잡한 점성술 조합의 의미를 종합적으로 분석
- **개인화**: 개별 출생 차트의 고유한 특성을 반영한 해석

## 데이터 플로우 상세 분석

### 1. 입력 데이터 처리
```json
{
  "birth_date": "1990-05-15",
  "birth_time": "14:30:00",
  "latitude": 28.6139,
  "longitude": 77.2090,
  "timezone": "+05:30"
}
```

### 2. Jyotish API 계산 결과
```json
{
  "planets": {
    "Sun": {"longitude": 54.25, "house": 2, "sign": "Taurus"},
    "Moon": {"longitude": 142.33, "house": 6, "sign": "Leo"},
    "Mars": {"longitude": 23.45, "house": 1, "sign": "Aries"}
  },
  "houses": {
    "1": {"sign": "Aries", "lord": "Mars"},
    "2": {"sign": "Taurus", "lord": "Venus"}
  },
  "yogas": ["Raj Yoga", "Dhana Yoga"],
  "ashtakavarga": {...}
}
```

### 3. AI 해석 결과
```json
{
  "interpretation": {
    "personality": "Strong leadership qualities with...",
    "career": "Best suited for technical or managerial roles...",
    "relationships": "Harmonious partnerships with...",
    "health": "Generally good health with attention to...",
    "remedies": ["Wear ruby on Sunday", "Chant Surya mantra"]
  }
}
```

## 배포 및 운영

### Docker 컨테이너 구성
- **Base Images**: Alpine Linux 기반으로 경량화
- **Network**: 사용자 정의 브리지 네트워크로 서비스 간 통신
- **Volume**: 로그 및 설정 파일 영속화
- **Health Check**: 각 서비스의 상태 모니터링

### 환경 변수 설정
```bash
# Gemini API 설정
GOOGLE_AI_API_KEY=your_gemini_api_key

# 서비스 디스커버리
JYOTISH_SERVICE_IP=192.168.45.100
GEMINI_SERVICE_IP=192.168.45.100

# 포트 설정
FRONTEND_PORT=80
GATEWAY_PORT=9595
JYOTISH_PORT=9393
GEMINI_PORT=9494
```

### 로깅 및 모니터링
- **접근 로그**: Nginx 액세스 로그
- **애플리케이션 로그**: 각 서비스별 구조화된 로깅
- **에러 트래킹**: 예외 상황 및 실패 케이스 추적
- **성능 메트릭**: 응답 시간 및 처리량 모니터링

## 보안 고려사항

### API 보안
- **CORS 설정**: 허용된 도메인에서만 API 접근
- **API 키 관리**: 환경 변수를 통한 민감 정보 보호
- **입력 검증**: 모든 사용자 입력에 대한 유효성 검사
- **Rate Limiting**: 과도한 요청 방지 (구현 권장)

### 데이터 보호
- **개인정보**: 출생 정보의 안전한 처리
- **전송 암호화**: HTTPS 통신 (프로덕션 환경)
- **로그 마스킹**: 민감한 정보의 로그 출력 방지

## 확장 가능성

### 기능 확장
1. **추가 점성술 시스템**: 서양 점성술, 중국 사주 등
2. **더 많은 AI 모델**: OpenAI GPT, Claude 등 다양한 AI 통합
3. **실시간 업데이트**: WebSocket을 통한 실시간 천체 위치 업데이트
4. **차트 시각화**: 동적 점성술 차트 렌더링

### 인프라 확장
1. **로드 밸런싱**: 다중 인스턴스 배포
2. **캐싱 레이어**: Redis를 통한 계산 결과 캐싱
3. **메시지 큐**: 비동기 처리를 위한 RabbitMQ/Kafka 도입
4. **데이터베이스**: 사용자 데이터 영속화

## 문제 해결 가이드

### 일반적인 문제
1. **컨테이너 헬스체크 실패**: 네트워크 설정 및 포트 바인딩 확인
2. **API 응답 지연**: Gemini API 할당량 및 네트워크 상태 점검
3. **계산 오류**: Swiss Ephemeris 데이터 파일 무결성 검증
4. **CORS 에러**: 프론트엔드와 API 간 도메인 설정 확인

### 로그 분석
```bash
# 각 컨테이너별 로그 확인
docker logs jyotish_api
docker logs call-gemini-api
docker logs vedic-astro-frontend-new
docker logs vedic-api-gateway
```

## 기여 가이드

### 개발 환경 설정
1. Docker 및 Docker Compose 설치
2. Google Gemini API 키 발급
3. 환경 변수 설정
4. 컨테이너 빌드 및 실행

### 코드 기여
- **브랜치 전략**: Git Flow 방식 권장
- **코드 스타일**: 각 언어별 표준 스타일 가이드 준수
- **테스트**: 단위 테스트 및 통합 테스트 작성
- **문서화**: API 문서 및 README 업데이트

## 라이선스 및 크레디트

### 오픈소스 라이브러리
- **Swiss Ephemeris**: Swiss Federal Institute of Technology 라이선스
- **Jyotish PHP Library**: MIT 라이선스
- **Symfony Framework**: MIT 라이선스
- **Go Standard Library**: BSD 라이선스

### 외부 서비스
- **Google Gemini API**: Google AI 서비스 약관
- **Docker**: Apache 2.0 라이선스

---

이 시스템은 전통적인 베딕 점성술의 정밀한 계산과 현대 AI의 해석 능력을 결합하여, 사용자에게 개인화된 점성술 분석을 제공하는 혁신적인 플랫폼입니다. 마이크로서비스 아키텍처를 통해 확장성과 유지보수성을 보장하며, 각 컴포넌트의 독립적인 발전을 가능하게 합니다.