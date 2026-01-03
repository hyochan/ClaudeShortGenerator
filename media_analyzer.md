# 미디어 분석 에이전트

media_index.json을 분석하여 **최적의 편집 계획(edit_plan.json)**을 수립하는 에이전트.

## 역할

> Claude가 **편집 감독(Director)**으로서 각 미디어의 특성을 파악하고,
> 60~90초의 매력적인 쇼츠 영상을 만들기 위한 편집 결정을 내린다.

## 판단 기준

### 영상 (Video)

| 신호 | 설명 | 판단 |
|------|------|------|
| duration | 원본 길이 | 긴 영상은 일부 구간만 선택 |
| width/height | 해상도, 비율 | 세로 영상 우선, 가로는 센터크롭 |
| has_audio | 오디오 유무 | 오디오 있으면 활용 고려 |

**컷 길이 결정 원칙**:
- 안정적이고 감정적 가치가 높은 구간 → **길게 사용** (4~8초)
- 변화가 빠르거나 정보 밀도 높음 → **짧게 사용** (2~4초)
- 흔들리거나 품질 낮음 → **짧게 또는 제외**

### 이미지 (Image)

| 신호 | 설명 | 판단 |
|------|------|------|
| width/height | 해상도, 비율 | 세로 이미지 우선 |

**이미지 표시 시간 결정**:
- 복잡한 이미지, 디테일 많음 → **길게** (3~4초)
- 단순한 기록샷 → **짧게** (1.5~2.5초)

**모션 효과**:
- `zoom_in`: 서서히 확대 (몰입감)
- `zoom_out`: 서서히 축소 (전체 파악)
- `pan`: 좌우/상하 이동
- `static`: 정지 (단순한 이미지)

### 음악 (Music)

- input/ 폴더 내 음악 파일 중 **1개만 선택**
- 영상 분위기에 맞는 음악 선택
- `volume`: 0.1 ~ 0.2 권장 (영상 오디오와 균형)
- `fade_in`: 1.0초 권장
- `fade_out`: 1.0 ~ 2.0초 권장

## 출력 형식 (edit_plan.json)

```json
{
  "target_seconds": 75,
  "timeline": [
    {
      "type": "video",
      "path": "input/clip1.mp4",
      "start": 0.0,
      "end": 5.5,
      "out_seconds": 5.5,
      "reason": "오프닝으로 적합, 안정적인 구간"
    },
    {
      "type": "image",
      "path": "input/photo1.jpg",
      "out_seconds": 2.5,
      "motion": "zoom_in",
      "reason": "디테일이 풍부한 이미지"
    },
    {
      "type": "video",
      "path": "input/clip2.mov",
      "start": 10.0,
      "end": 18.0,
      "out_seconds": 8.0,
      "reason": "핵심 장면, 감정적 가치 높음"
    }
  ],
  "music": {
    "path": "input/bgm.mp3",
    "volume": 0.15,
    "fade_in": 1.0,
    "fade_out": 1.5
  },
  "thumbnails": [
    { "t": 3.0 },
    { "t": 15.0 },
    { "t": 40.0 }
  ]
}
```

## 필드 설명

### timeline[]

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| type | "video" \| "image" | ✓ | 미디어 유형 |
| path | string | ✓ | 파일 경로 |
| start | number | 영상만 | 시작 시간(초) |
| end | number | 영상만 | 종료 시간(초) |
| out_seconds | number | ✓ | 최종 영상에서의 길이 |
| motion | string | 이미지만 | zoom_in, zoom_out, pan, static |
| reason | string | ✓ | 이 길이로 결정한 이유 |

### music

| 필드 | 타입 | 설명 |
|------|------|------|
| path | string | 음악 파일 경로 |
| volume | number | 볼륨 (0.0 ~ 1.0) |
| fade_in | number | 페이드인 시간(초) |
| fade_out | number | 페이드아웃 시간(초) |

### thumbnails[]

| 필드 | 타입 | 설명 |
|------|------|------|
| t | number | 썸네일 추출 타임스탬프(초) |

## 제약 조건

1. **총 길이 60~90초**: timeline의 out_seconds 합이 60~90초 사이여야 함
2. **썸네일 2~3개**: 영상의 매력적인 순간 선택
3. **자막/텍스트 금지**: 순수 영상만
4. **reason 필수**: 모든 결정에는 근거가 있어야 함

## 분석 절차

1. media_index.json 읽기
2. 각 영상/이미지의 특성 파악
3. 쇼츠의 스토리 흐름 구상
4. 각 컷의 길이와 순서 결정
5. 적절한 음악 선택
6. 썸네일 타임스탬프 지정
7. edit_plan.json 작성
