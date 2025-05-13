# .github


# 상가 추천 시스템 (Windsurf Project)

> **Version:** Initial draft  
> **Last updated:** 2025‑05‑13

## 🔥 목표
- 사용자에게 특정 **지역(dong_code)** 과 **업종(category)** 을 기반으로 **겹치지 않는 상가**를 추천하는 시스템 구축  
- 상가 및 건물 정보를 **MongoDB**에 저장하고 **LangChain + ElasticSearch + Redis** 로 빠른 검색과 추천 알고리즘 구현

---

## ✅ 주요 기능

| API | Endpoint | 설명 |
| --- | --- | --- |
| 상가 추천 | **GET /api/recommend** | 지역·업종 입력 → 겹치지 않는 상가 추천<br>LangChain 분석 + ElasticSearch 유사도 + Redis 캐싱 |
| 건물 정보 | **GET /api/buildings**<br>**GET /api/buildings/:id** | 건물 리스트·단건 조회 |
| 상가 정보 | **GET /api/markets**<br>**GET /api/markets/:id** | 상가 리스트·단건 조회 |

---

## ✅ 데이터베이스 설계 (MongoDB)

### 1) `buildings` 컬렉션

| 필드 | 타입 | 설명 |
| --- | --- | --- |
| `_id` | ObjectId | 기본 ID |
| `unique_id` | String | 건물 고유 ID |
| `dong_code` | String | 법정동 코드 |
| `dong_name` | String | 법정동명 |
| `jibun` | String | 지번 주소 |
| `building_name` | String | 건물명 |
| `land_area` | Number | 대지 면적(㎡) |
| `building_area` | Number | 건축 면적(㎡) |
| `total_area` | Number | 연면적(㎡) |
| `volume_ratio` | Number | 용적률 |
| `building_ratio` | Number | 건폐율 |
| `structure_name` | String | 구조명 |
| `main_purpose` | String | 주요 용도 |
| `approval_date` | Date | 사용 승인일 |

### 2) `markets` 컬렉션

| 필드 | 타입 | 설명 |
| --- | --- | --- |
| `_id` | ObjectId | 기본 ID |
| `shop_id` | String | 상가 고유 ID |
| `name` | String | 상호명 |
| `category_large` | String | 대분류 |
| `category_middle` | String | 중분류 |
| `category_small` | String | 소분류 |
| `dong_code` | String | 법정동 코드 |
| `dong_name` | String | 법정동명 |
| `longitude` | Number | 경도 |
| `latitude` | Number | 위도 |

---

## ✅ 시스템 아키텍처

```
Next.js (Client)
      │  REST
      ▼
NestJS (API)
 ├─ LangChain  ←  추천 로직
 ├─ ElasticSearch  ←  유사 상가 검색
 └─ Redis  ←  캐싱
      │
MongoDB  ←  buildings / markets
```

---

## ✅ 기술 스택
- **Frontend:** Next.js + TailwindCSS
- **Backend:** NestJS (TypeScript)
- **DB:** MongoDB (Mongoose ODM)
- **Search:** ElasticSearch
- **Cache:** Redis
- **AI / LLM:** LangChain (OpenAI GPT‑4)
- **DevOps:** Docker, GitHub Actions, AWS

---

## ✅ 사용 흐름 예시

1. **상가 추천 요청**  
   `GET /api/recommend?dong_code=11110&category=음식`  
   1. Redis 캐시 확인 → 없으면 ElasticSearch 검색  
   2. LangChain으로 겹치지 않는 상가 아이디어 생성  
   3. 결과 Redis 캐싱 후 응답

2. **건물·상가 상세 조회**  
   - `GET /api/buildings/B12345` → 건물 단건  
   - `GET /api/markets/M12345` → 상가 단건

---

### 📌 향후 과제
- LangChain Prompt 튜닝 및 RAG 적용  
- ElasticSearch Geo Query 성능 최적화  
- 추천 결과 A/B 테스트 및 모니터링
