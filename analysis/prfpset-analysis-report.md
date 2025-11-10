# .prfpset 파일 완전 해체 분석 보고서

## 개요
Adobe Premiere Pro의 .prfpset 파일은 효과 프리셋을 저장하는 XML 기반 형식입니다.  
마스크 트래킹 데이터가 포함된 가우시안 블러 효과의 구조를 완전히 해체 분석했습니다.

---

##  분석 대상 파일
- **파일명**: `마스크 있는 가우시안 블러.prfpset`
- **크기**: 27,030 bytes
- **포함 효과**: AE.ADBE Gaussian Blur 2 + Mask Component
- **키프레임 수**: 30개 (kf.0 ~ kf.29)

---

##  XML 구조 분석

### 1. 루트 구조
```xml
<?xml version="1.0" encoding="UTF-8"?>
<PremiereData Version="3">
    <Tree ObjectRef="1"/>
    <!-- 트리 구조 정의 -->
</PremiereData>
```

### 2. 효과 계층 구조
```
PremiereData
├── BinTreeItem (Root)
├── BinTreeItem (Presets)
├── FilterPreset (AE.ADBE Gaussian Blur 2)
│   ├── VideoFilterComponent (가우시안 흐림)
│   │   ├── Param Index="0" → 흐림 강도
│   │   ├── Param Index="1" → 흐림 차원
│   │   └── Param Index="2" → 반복 가장자리 픽셀
│   └── SubComponents
│       └── VideoFilterComponent (마스크)
│           ├── 18개 마스크 파라미터 (ObjectRef="13"~"30")
│           └── **ObjectID="20"** ← 📍 핵심! 마스크 경로 데이터
```

### 3. 핵심 마스크 데이터 위치
**ObjectID="20"** - `ArbVideoComponentParam`
- **ClassID**: `313e54d4-6903-49ad-b0bf-8262cdd10f4e`
- **역할**: 마스크 경로의 베지어 곡선 키프레임 데이터 저장

---

##  마스크 키프레임 데이터 구조

### 키프레임 패턴
각 키프레임은 **2개의 XML 자식 요소**로 구성:
```xml
<kf.N.position>TIME_IN_TICKS</kf.N.position>
<kf.N.value Encoding="base64" Checksum="XXXXXXXX">BASE64_MASK_DATA</kf.N.value>
```

### 시간 정보 (position)
- **단위**: 프리미어 프로 내부 시간 틱 (ticks)
- **변환**: `position / 254016000000 = 초 단위`
- **예시**:
  - `kf.0.position`: 0 → 0.00초
  - `kf.1.position`: 8467200000 → 0.033초
  - `kf.29.position`: 245548800000 → 0.966초

### Base64 마스크 데이터
각 키프레임의 `kf.N.value`는 144바이트 베지어 곡선 데이터:
- **인코딩**: Base64
- **디코딩 크기**: 144 bytes
- **시그니처**: `3263696e02000000` (hex)
- **내용**: 베지어 곡선 제어점 좌표

---

##  키프레임 완전 목록

| 인덱스 | 시간(초) | Position(ticks) | Base64 데이터 (처음 20자) | Checksum |
|--------|----------|-----------------|-------------------------|-----------|
| 0 | 0.000 | 0 | MmNpbgIAAAABAAAABA... | 1143263905 |
| 1 | 0.033 | 8467200000 | MmNpbgIAAAABAAAABA... | 746022049 |
| 2 | 0.067 | 16934400000 | MmNpbgIAAAABAAAABA... | 2123034529 |
| 3 | 0.100 | 25401600000 | MmNpbgIAAAABAAAABA... | 2499649441 |
| 4 | 0.133 | 33868800000 | MmNpbgIAAAABAAAABA... | 191894689 |
| 5 | 0.167 | 42336000000 | MmNpbgIAAAABAAAABA... | 3446477985 |
| ... | ... | ... | ... | ... |
| 29 | 0.966 | 245548800000 | MmNpbgIAAAABAAAABA... | 4079262625 |

**총 30개 키프레임**, 지속시간: **0.966초**

---

##  마스크 데이터 디코딩 분석

### Base64 → Binary 변환
```javascript
// 예시: kf.0.value
const base64 = "MmNpbgIAAAABAAAABAAAAAEAAADHKbc+4RijPtX0rj7gGKM+uV6/PuIYoz4BAAAAAQAAAN8Fxj5tHb4+4AXGPoYxrz7fBcY+VAnNPgEAAAABAAAAxim3Pvch2T64Xr8+9yHZPtT0rj72Idk+AQAAAAEAAACtTag+ax2+Pq1NqD5SCc0+rU2oPoQxrz4BAAAA";

// 디코딩 시 144바이트 생성
const decoded = atob(base64); // 144 bytes
```

### Binary 구조 (추정)
```
Bytes 0-7:   시그니처 (3263696e02000000)
Bytes 8-15:  헤더 정보
Bytes 16+:   베지어 제어점 좌표들 (Float32 형태)
```

---

##  Copy/Paste 워크플로우

### 1. Copy 과정 (가우시안 블러 → 클립보드)
```
사용자: 가우시안 블러 마스크 우클릭 → Copy
↓
프리미어 내부 클립보드에 저장되는 데이터:
- 소스: AE.ADBE Gaussian Blur 2
- 컴포넌트: Mask
- 키프레임: 30개 베지어 곡선 데이터
- 시간 범위: 0.000s ~ 0.966s
```

### 2. Paste 과정 (클립보드 → Transform)
```
사용자: Transform 효과에서 우클릭 → Paste
↓
선택된 속성에 따라 키프레임 적용:
- Position: 베지어 곡선 중심점 → X,Y 좌표
- Rotation: 곡선 기울기 → 각도 값
- Scale: 곡선 크기 → 스케일 비율
```

---

##  다른 .prfpset 파일과의 비교

### `변형.prfpset` (Transform 프리셋)
- **FilterMatchName**: `AE.ADBE Geometry2`
- **12개 파라미터**: 기준점, 위치, 회전, 비율조정 등
- **키프레임**: 없음 (기본값만 저장)

### `가우시안블러 시작지점기준.prfpset` vs `마스크 있는 가우시안 블러.prfpset`
- **시작지점 기준**: 키프레임 시작 시간이 다름
- **비율 기준**: 키프레임 간격과 지속시간이 다름
- **공통점**: 동일한 XML 구조와 ObjectID="20" 사용

---

##  구현 요점

### XML 파싱 핵심
```javascript
// 올바른 파싱 방법
const maskParam = xmlDoc.querySelector('*[ObjectID="20"]');
const childElements = maskParam.childNodes;

// kf.N.position과 kf.N.value를 자식 요소에서 찾기
for (let child of childElements) {
    if (child.tagName && child.tagName.match(/^kf\.\d+\.position$/)) {
        const index = child.tagName.match(/kf\.(\d+)\.position/)[1];
        // 해당 인덱스의 value 요소도 찾기
    }
}
```

### 시간 변환
```javascript
const timeInSeconds = position / 254016000000;
const timeInFrames = timeInSeconds * 29.97; // 29.97fps 기준
```

### Base64 디코딩
```javascript
const binaryData = atob(base64String);
const bytes = new Uint8Array(binaryData.length);
for (let i = 0; i < binaryData.length; i++) {
    bytes[i] = binaryData.charCodeAt(i);
}
// bytes.length === 144
```

---

##  결론

1. **.prfpset 파일은 XML 기반**으로 효과와 키프레임을 저장
2. **마스크 데이터는 ObjectID="20"**에 베지어 곡선으로 저장
3. **30개 키프레임**이 0.966초 동안 마스크 경로 변화를 기록
4. **Base64 인코딩된 144바이트 데이터**가 각 키프레임의 핵심
5. **프리미어 Copy/Paste**는 이 데이터를 변환하여 Transform 속성에 적용

이 분석을 바탕으로 **마스크 트래킹 데이터 → Transform 키프레임** 변환이 가능합니다.

---

**📅 분석 일자**: 2025-11-10  
**🔍 분석자**: notoow  
**📁 분석 도구**: XML 파싱, Hex 분석, Base64 디코딩