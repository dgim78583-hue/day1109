# The Cat API 사용법 정리

- 사이트: https://thecatapi.com/
- 공식 문서: https://docs.thecatapi.com/ (API Reference: `/reference`)
- OpenAPI 스펙(JSON): https://api.thecatapi.com/openapi-json
- API 키 발급: https://thecatapi.com/signup (이메일로 키 발송, 키 관리: account.thecatapi.com)

## 1. 기본 정보

| 항목 | 내용 |
|---|---|
| Base URL | `https://api.thecatapi.com/v1` |
| 인증 | 요청 헤더 `x-api-key: YOUR-API-KEY` |
| 요금제 | Free: 월 10,000 요청 / Startup: $99/월, 월 100,000 요청, 상업 라이선스 / Enterprise: 무제한(문의) |

- `GET /images/search`, `GET /images/{id}` 등 일부 공개 엔드포인트는 키 없이도 동작하지만, 키를 넣는 것을 권장합니다.
- 업로드(동기)는 별도 호스트를 씁니다: `https://upload.thecatapi.com/v1/images/upload-sync`

> 주의: 키는 프런트엔드 코드에 그대로 노출됩니다. 공개 배포 시에는 노출 위험을 고려하세요.

## 2. 가장 많이 쓰는 엔드포인트

### 2-1. 랜덤 고양이 이미지 — `GET /images/search`

```bash
curl -H "x-api-key: YOUR-API-KEY" "https://api.thecatapi.com/v1/images/search?limit=2"
```

응답 예:
```json
[
  {
    "id": "KWdLHmOqc",
    "url": "https://cdn2.thecatapi.com/images/KWdLHmOqc.jpg",
    "width": 3114,
    "height": 2609,
    "breeds": []
  }
]
```

주요 쿼리 파라미터:

| 파라미터 | 설명 |
|---|---|
| `limit` | 결과 수 (기본 1, 최대 100) |
| `page` | 페이지 번호 (0부터) |
| `order` | `ASC` / `DESC` / `RANDOM` |
| `has_breeds` | 품종 정보가 있는 이미지만 |
| `include_breeds` | 응답에 품종 정보 포함 (`1`) |
| `include_categories` | 응답에 카테고리 포함 |
| `breed_id`, `breed_ids` | 품종 ID로 필터 (`breed_ids`는 쉼표 구분, 예: `abys,beng`) |
| `mime_types` | `jpg`, `png`, `gif` |
| `size` | `thumb`, `small`, `med`, `full` (스펙상 효과 없음) |
| `sub_id` | 내가 지정한 사용자 구분 ID |
| `colour_id(s)`, `colour_code`, `has_colours`, `include_colours` | 색상 관련 필터 |
| `age_months_min`, `age_months_max` | 나이 범위 |

품종 지정 예:
```bash
curl -H "x-api-key: YOUR-API-KEY" "https://api.thecatapi.com/v1/images/search?limit=2&breed_ids=abys&include_breeds=1"
```

### 2-2. 이미지 상세 — `GET /images/{id}`
```bash
curl -H "x-api-key: YOUR-API-KEY" https://api.thecatapi.com/v1/images/KWdLHmOqc
```

### 2-3. 품종 — `/breeds`

| 엔드포인트 | 설명 |
|---|---|
| `GET /breeds` | 품종 목록 (`limit` 최대 1000, `page`, `order`, `lang`, `extinct`) |
| `GET /breeds/search?q=` | 이름 검색 (`q` 1~30자, 별칭 포함) |
| `GET /breeds/{id}` | 품종 상세 (예: `abys`) |
| `GET /breeds/breed-groups` | 품종 그룹 목록 |
| `GET /breeds/{id}/colours` | 품종별 색상/무늬 목록 |

```bash
curl -H "x-api-key: YOUR-API-KEY" https://api.thecatapi.com/v1/breeds/abys
```

### 2-4. 투표 — `/votes`

| 엔드포인트 | 설명 |
|---|---|
| `GET /votes` | 내 투표 목록 (`limit`, `page`, `order`, `sub_id`) |
| `POST /votes` | 투표 생성 |
| `GET /votes/{vote_id}` | 투표 상세 |

요청 본문 (`value`: 좋아요 `1`, 싫어요 `-1`):
```json
{"image_id": "KWdLHmOqc", "value": 1}
```
선택 필드: `sub_id`, `country_code`, `ip_address`

```bash
curl -X POST -H "Content-Type: application/json" -H "x-api-key: YOUR-API-KEY" \
  -d '{"image_id": "KWdLHmOqc", "value": 1}' \
  https://api.thecatapi.com/v1/votes
```

### 2-5. 즐겨찾기 — `/favourites`

| 엔드포인트 | 설명 |
|---|---|
| `GET /favourites` | 목록 (`limit`, `page`, `sub_id`, `image_id`, `attach_image`) |
| `POST /favourites` | 생성 |
| `GET /favourites/{id}` | 상세 |
| `DELETE /favourites/{id}` | 삭제 |

```bash
curl -X POST -H "Content-Type: application/json" -H "x-api-key: YOUR-API-KEY" \
  -d '{"image_id": "KWdLHmOqc", "sub_id": "YOUR-USER-ID"}' \
  https://api.thecatapi.com/v1/favourites
```

### 2-6. 이미지 업로드 / 관리

| 엔드포인트 | 설명 |
|---|---|
| `POST /images/upload` | 비동기 업로드 (`type`: `base64` 또는 `FILE`, `file`/`image`, `sub_id`). 응답의 `links.status`로 처리 상태 확인 |
| `POST /images/upload-sync` | 분석 완료까지 대기하는 업로드 (`auto_crop` 기본 true) |
| `GET /images/{id}/status` | 처리 상태: `to_be_reviewed`, `clean`, `unsafe`, `flagged`, `does_not_contain_species`, `labelled` |
| `GET /images/{id}/labels` | 품종 식별 라벨 |
| `DELETE /images/{id}` | 업로드한 이미지 삭제 |
| `GET /images` | 내가 업로드한 이미지 목록 (`sub_id`로 필터) |

```bash
curl -X POST -H "x-api-key: YOUR-API-KEY" -F "file=@/path/cat.jpg" https://api.thecatapi.com/v1/images/upload
```

제한: 사진 50MB, 동영상 200MB / 월 업로드 쿼터는 `GET /accounts/quota`로 확인 (`quota`, `usage`, `remaining`).

### 2-7. 기타

| 분류 | 엔드포인트 |
|---|---|
| 카테고리 | `GET /categories`, `GET /categories/{id}` |
| 고양이 상식 | `GET /facts`, `GET /facts/random`, `GET /facts/breed/{breedId}`, `GET /facts/{id}` |
| 건강 팁 | `GET /health-tips`, `GET /health-tips/breed/{breedId}` |
| 계정 | `GET /accounts/features`, `/accounts/quota`, `/accounts/capabilities` |
| 사용자 ID | `GET /sub-ids` |
| 반려동물(Pets) | `/pets` 하위: 등록, 체형 점수(BCS), 나이/체중 추정, AI 초상화, 방문 기록 등 |
| 계산기 | `POST /calculators/energy-requirement`, `/calculators/weight-loss-plan` |
| 웹훅 | `/webhooks` 하위 |

공통 파라미터: `limit`, `page`, `order`, `lang`(2글자 언어 코드), `species_id`, `include_sources`.

## 3. JavaScript(fetch) 예제

```js
const API_KEY = 'YOUR-API-KEY';
const BASE = 'https://api.thecatapi.com/v1';

// 랜덤 고양이 이미지 가져오기
async function getCats(limit = 6) {
  const res = await fetch(`${BASE}/images/search?limit=${limit}`, {
    headers: { 'x-api-key': API_KEY },
  });
  if (!res.ok) throw new Error(`요청 실패: ${res.status}`);
  return res.json(); // [{ id, url, width, height, breeds }, ...]
}

// 투표
async function vote(imageId, value) {
  const res = await fetch(`${BASE}/votes`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json', 'x-api-key': API_KEY },
    body: JSON.stringify({ image_id: imageId, value }), // 1 또는 -1
  });
  return res.json();
}
```

## 4. 에러 코드

| 코드 | 의미 |
|---|---|
| 400 | 잘못된 요청 (필드 누락/형식 오류, 월 업로드 쿼터 소진 등) |
| 401 | 비활성 계정의 키 |
| 403 | 키 없음/잘못됨, 계정에 없는 기능, 다른 계정의 리소스 |
| 404 | 존재하지 않거나 삭제된 ID |
| 409 | 이미 존재하는 리소스 |
| 415 | 지원하지 않는 업로드 형식 |
| 422 | 요청은 유효하나 입력값을 처리할 수 없음 |
| 429 | 짧은 시간에 요청/업로드 과다 |
| 500 / 503 | 서버 오류 / 분석 일시 불가 |

재시도 권장: 429, 500, 503만. 1초부터 시작해 2배씩 늘리는 지수 백오프 + 약간의 랜덤 지연, 최대 4~5회.

## 5. 참고 사항

- 응답 예시는 공식 문서에 `images/search`, `images/{id}/status` 두 가지만 실려 있습니다. 나머지는 OpenAPI 스펙 또는 실제 호출로 확인하세요.
- 요청 한도의 정확한 수치는 요금제에 따릅니다(Free는 월 10,000 요청).
- 이 문서는 OpenAPI 스펙 일부(앞 100,000자)를 기준으로 정리했습니다. 일부 요청 본문 스키마(`CreateFavouriteDto` 등)는 확인하지 못했습니다.
