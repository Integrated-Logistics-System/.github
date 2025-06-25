# AI 상가 추천 시스템 (AI Search Project)
> **Version:** AI-Enhanced Version  
> **Last updated:** 2025‑01‑15

## 🤖 목표
- **AI 기반 자연어 상가 추천**: "강남역 근처 조용한 카페 자리 찾아줘", "홍대에서 젊은층 타겟 치킨집 창업하려는데"
- **RAG (Retrieval-Augmented Generation)**: 실제 상가/건물 데이터 + AI 분석으로 정확한 추천
- **실시간 대화형 검색**: WebSocket 기반 채팅으로 상세 상담 가능

---

## 🔥 AI 기반 주요 기능

| 기능 | Endpoint | AI 역할 |
|------|----------|---------|
| **AI 상가 추천** | `POST /api/ai/recommend` | 자연어 → 지역/업종 분석 → 맞춤 추천 |
| **대화형 상담** | `WebSocket /ws/chat` | 실시간 Q&A, 추가 조건 반영 |
| **시장 분석** | `POST /api/ai/market-analysis` | 해당 지역 경쟁 현황, 성공 요인 분석 |
| **위치 최적화** | `POST /api/ai/location-optimize` | 유동인구, 접근성, 임대료 종합 분석 |

---

## 🧠 AI 검색 플로우

### **1단계: 자연어 이해**
```typescript
// 사용자 입력: "강남역 근처에서 20-30대 직장인들 대상으로 브런치카페 창업 고려중"
{
  location: "강남역",
  targetAge: "20-30대",
  targetGroup: "직장인",
  businessType: "브런치카페",
  purpose: "창업"
}
```

### **2단계: 데이터 검색 + AI 분석**
```typescript
// ElasticSearch → 관련 상가/건물 데이터 수집
// LangChain → AI가 데이터 분석하여 인사이트 생성
// Redis → 결과 캐싱
```

### **3단계: 맞춤 추천 생성**
```typescript
{
  recommendations: [
    {
      building_id: "B12345",
      shop_location: "강남역 2번출구 도보 3분",
      why_recommended: "직장인 유동인구 많음, 브런치 경쟁업체 적음",
      success_factors: ["접근성 우수", "타겟층 밀집", "임대료 적정"],
      risk_factors: ["주말 유동인구 감소"],
      market_analysis: "..."
    }
  ]
}
```

---

## 🗄️ AI 최적화 데이터베이스 설계

### **Enhanced `buildings` 컬렉션**
```typescript
{
  _id: ObjectId,
  unique_id: String,
  dong_code: String,
  dong_name: String,
  jibun: String,
  building_name: String,
  
  // AI 분석용 추가 필드
  foot_traffic: {
    weekday_avg: Number,
    weekend_avg: Number,
    peak_hours: [String],
    demographic: {
      age_groups: Object,
      income_level: String
    }
  },
  
  business_environment: {
    nearby_competitors: [String],
    complementary_business: [String],
    transport_accessibility: Number,
    parking_availability: Boolean
  },
  
  ai_insights: {
    success_score: Number,          // AI 평가 점수
    recommended_business: [String], // AI 추천 업종
    market_saturation: Number,      // 시장 포화도
    growth_potential: Number        // 성장 가능성
  }
}
```

### **Enhanced `markets` 컬렉션**
```typescript
{
  _id: ObjectId,
  shop_id: String,
  name: String,
  category_large: String,
  category_middle: String,
  category_small: String,
  
  // AI 분석용 성과 데이터
  business_performance: {
    estimated_revenue: Number,
    success_indicators: [String],
    failure_reasons: [String],
    operation_period: Number
  },
  
  competitive_analysis: {
    similar_shops_nearby: Number,
    differentiation_factors: [String],
    market_share_estimate: Number
  }
}
```

---

## 🔧 AI 기술 스택

### **Core AI Technologies**
- **LangChain**: RAG 파이프라인, 대화 체인
- **Ollama**: 로컬 LLM (Gemma/Llama)
- **OpenAI GPT-4**: 고급 분석 (옵션)
- **ElasticSearch**: 벡터 검색, 유사도 분석
- **Redis**: AI 응답 캐싱, 대화 메모리

### **Enhanced Backend**
```typescript
// NestJS 모듈 구조
backend/
├── modules/
│   ├── ai/                    # AI 추천 엔진
│   │   ├── recommendation.service.ts
│   │   ├── market-analysis.service.ts
│   │   └── location-optimizer.service.ts
│   ├── langchain/            # LangChain 통합
│   │   ├── rag.service.ts
│   │   └── conversation.service.ts
│   ├── elasticsearch/        # 벡터 검색
│   ├── websocket/           # 실시간 채팅
│   └── data/               # 상가/건물 CRUD
```

---

## 🚀 AI 검색 API 설계

### **1. AI 상가 추천**
```typescript
POST /api/ai/recommend
{
  "query": "강남역 근처 브런치카페 창업하려는데 좋은 자리 추천해줘",
  "user_profile": {
    "budget": "월 500만원 이하",
    "experience": "카페 운영 경험 없음",
    "target_customer": "직장인"
  }
}

// Response
{
  "ai_analysis": {
    "location_insights": "강남역 2번출구 반경 300m 내 직장인 밀집도 높음",
    "market_opportunity": "브런치 전문점 부족, 수요 대비 공급 부족",
    "success_probability": 0.75
  },
  "recommendations": [
    {
      "building": {...},
      "score": 0.89,
      "reasoning": "AI가 분석한 추천 이유",
      "pros_cons": {...}
    }
  ]
}
```

### **2. 실시간 AI 채팅**
```typescript
// WebSocket /ws/ai-chat
{
  "type": "message",
  "content": "임대료는 어느 정도 예상되나요?",
  "context": {
    "previous_recommendations": [...],
    "user_session": "..."
  }
}

// AI 응답
{
  "type": "ai_response",
  "content": "추천드린 강남역 2번출구 건물의 경우, 1층 30평 기준 월 임대료 450-550만원 수준입니다...",
  "data": {
    "rent_analysis": {...},
    "comparable_properties": [...]
  }
}
```

### **3. 시장 분석**
```typescript
POST /api/ai/market-analysis
{
  "dong_code": "11110",
  "business_type": "카페",
  "analysis_type": "competitive_landscape"
}

// AI 생성 분석 리포트
{
  "market_overview": "AI 분석 요약",
  "competition_map": [...],
  "success_factors": [...],
  "recommendations": "AI 조언"
}
```

---

## 🔄 AI 워크플로우

### **RAG (검색 증강 생성) 파이프라인**
```
사용자 쿼리
    ↓
자연어 이해 (LangChain)
    ↓
관련 데이터 검색 (ElasticSearch)
    ↓
컨텍스트 구성 (상가+건물+시장 데이터)
    ↓
AI 분석 및 추천 (Ollama/GPT)
    ↓
구조화된 응답 생성
    ↓
캐싱 (Redis) + 사용자 응답
```

### **대화형 메모리 관리**
```typescript
// Redis 저장 구조
user:{userId}:conversation = {
  history: [...],
  context: {
    current_search: {...},
    preferences: {...},
    session_data: {...}
  }
}
```

---

## 📊 AI 성능 최적화

### **Vector Search 설정**
```typescript
// ElasticSearch 인덱스 매핑
{
  "mappings": {
    "properties": {
      "description_vector": {
        "type": "dense_vector",
        "dims": 768
      },
      "business_embedding": {
        "type": "dense_vector", 
        "dims": 384
      }
    }
  }
}
```

### **LangChain 체인 최적화**
```typescript
// 추천 전용 체인
const recommendationChain = RunnableSequence.from([
  locationAnalyzer,
  marketResearcher, 
  competitorAnalyzer,
  riskAssessor,
  recommendationGenerator
]);
```

---

## 🎯 사용 시나리오

### **시나리오 1: 창업 상담**
```
👤 사용자: "을지로에서 소규모 이탈리안 레스토랑 창업 고려중인데 어떨까요?"

🤖 AI: "을지로 일대 분석 결과, 점심시간 직장인 수요가 높고 고급 이탈리안 레스토랑이 부족합니다. 
      추천 위치 3곳을 찾았는데, 각각의 장단점을 설명드릴게요..."

👤 사용자: "임대료 예산이 월 300만원 정도인데 가능한가요?"

🤖 AI: "300만원 예산으로는 을지로3가역 근처 2층 상가가 적합할 것 같습니다. 
      1층보다 임대료가 30% 저렴하지만 인스타그램 마케팅으로 충분히 집객 가능합니다..."
```

### **시나리오 2: 빠른 검색**
```
👤 사용자: "홍대 핫플 디저트카페 자리"

🤖 AI: 🔍 검색 중... 
      📍 홍대입구역 9번출구 도보 2분
      ✨ 젊은층 밀집, SNS 마케팅 유리
      💰 월세 400만원선, 보증금 5,000만원
      ⚠️ 경쟁업체 많음, 차별화 필수
```

---

## 🚀 개발 로드맵

### **Phase 1: AI 기반 검색 엔진** (2주)
- [x] LangChain + Ollama 연동
- [x] ElasticSearch 벡터 검색
- [x] 기본 RAG 파이프라인

### **Phase 2: 대화형 추천** (2주)  
- [ ] WebSocket 실시간 채팅
- [ ] 대화 컨텍스트 메모리
- [ ] 개인화 추천 알고리즘

### **Phase 3: 고급 AI 분석** (3주)
- [ ] 시장 분석 리포트 자동 생성
- [ ] 성공 확률 예측 모델
- [ ] A/B 테스트 및 추천 품질 최적화

**AI가 상가 추천의 새로운 패러다임을 제시하는 시스템입니다! 🚀**
