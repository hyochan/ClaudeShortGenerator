# Claude Short Generator

input/ 폴더에 미디어 파일을 넣으면 **자동으로 YouTube Shorts를 생성**하는 Claude Code Skill.

## Quick Start

```bash
# 1. input/ 폴더에 파일 넣기
cp my_video.mp4 my_photo.jpg my_music.mp3 input/

# 2. /short 명령 실행
/short
```

## 지원 파일 형식

| 타입 | 확장자 |
|------|--------|
| Video | mp4, mov, mkv, avi, webm |
| Image | jpg, jpeg, png, webp |
| Audio | mp3, wav, flac, m4a, aac |

## Output

```
output/
  final_short.mp4      # 최종 쇼츠 (9:16, ≤60초)
  thumbnail_01.jpg     # 썸네일 1
  thumbnail_02.jpg     # 썸네일 2
  thumbnail_03.jpg     # 썸네일 3
  debug/
    media_index.json   # 미디어 분석 결과
    edit_plan.json     # 편집 계획
```

## 시스템 요구사항

- **FFmpeg**: 영상 렌더링에 필요
  ```bash
  # macOS
  brew install ffmpeg

  # Ubuntu
  sudo apt install ffmpeg
  ```

## 처리 파이프라인

```
[input/ 폴더]
      ↓
[1. 미디어 스캔]     ffprobe로 메타데이터 추출
      ↓              → media_index.json
[2. 편집 계획]       Claude가 편집 결정
      ↓              → edit_plan.json
[3. 렌더링]          FFmpeg로 영상 생성
      ↓              → final_short.mp4 + thumbnails
[완료]
```

## 핵심 철학

- **고정 규칙 없음**: 컷 길이를 미리 정하지 않음
- **근거 기반 판단**: 각 미디어의 특성에 따라 결정
- **자막/텍스트 없음**: 순수 영상 편집만
- **재현 가능**: debug JSON으로 결과 추적 가능

## 편집 판단 기준

### 영상 컷 길이

| 특성 | 길이 |
|------|------|
| 안정적, 감정적 가치 높음 | 4~8초 |
| 빠른 변화, 높은 정보 밀도 | 2~4초 |
| 품질 낮음, 흔들림 | 짧게 또는 제외 |

### 이미지 표시 시간

| 특성 | 시간 |
|------|------|
| 복잡한 이미지, 디테일 많음 | 3~4초 |
| 단순한 기록샷 | 1.5~2.5초 |

### 이미지 모션

- `zoom_in`: 서서히 확대 (몰입감)
- `zoom_out`: 서서히 축소 (전체 파악)
- `pan`: 좌우/상하 이동
- `static`: 정지

## 관련 파일

| 파일 | 역할 |
|------|------|
| [SKILL.md](SKILL.md) | /short 스킬 정의 및 FFmpeg 명령어 |
| [media_analyzer.md](media_analyzer.md) | 편집 판단 기준 및 edit_plan.json 스키마 |

## 제약사항

- 최대 길이: **60초**
- 비율: **9:16** (1080x1920)
- 음악: input/ 내 파일 중 **1개만** 사용
- 자막/텍스트: **사용 금지**
- 외부 파일/URL: **사용 금지**
