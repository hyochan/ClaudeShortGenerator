# Claude Short Generator

Claude Code를 활용하여 이미지와 음악으로 YouTube Shorts 영상을 자동 생성하는 도구입니다.

## Features

- **9:16 세로 영상** - YouTube Shorts, Instagram Reels, TikTok에 최적화
- **Ken Burns 효과** - 줌인/줌아웃으로 정적 이미지에 생동감 부여
- **크로스페이드 전환** - 부드러운 이미지 전환 효과
- **배경음악 믹싱** - 페이드 인/아웃 자동 적용
- **60~90초 최적화** - Shorts 알고리즘에 최적화된 길이

## Prerequisites

- [Claude Code](https://claude.com/claude-code) CLI
- [FFmpeg](https://ffmpeg.org/) (시스템 PATH에 설치)

## Usage

### 1. 미디어 파일 준비

`input/` 폴더에 미디어 파일을 넣습니다:

```
input/
  ├── photo1.jpg
  ├── photo2.jpg
  ├── ...
  └── bgm.mp3
```

**지원 형식:**
- 이미지: `.jpg`, `.jpeg`, `.png`
- 영상: `.mp4`, `.mov`, `.avi`
- 음악: `.mp3`, `.wav`, `.m4a`

### 2. 쇼츠 생성

Claude Code에서 `/short` 명령어를 실행합니다:

```bash
claude
> /short
```

### 3. 결과물 확인

```
output/
  ├── final_short.mp4     # 최종 쇼츠 영상
  ├── thumbnail_01.jpg    # 썸네일 1
  ├── thumbnail_02.jpg    # 썸네일 2
  ├── thumbnail_03.jpg    # 썸네일 3
  └── debug/
      ├── media_index.json  # 미디어 분석 결과
      └── edit_plan.json    # 편집 계획
```

## Technical Specs

| 항목 | 값 |
|------|-----|
| 해상도 | 1080x1920 (9:16) |
| 프레임레이트 | 30fps |
| 비디오 코덱 | H.264 (libx264) |
| 오디오 코덱 | AAC |
| 목표 길이 | 60~90초 |

## Processing Pipeline

1. **미디어 스캔** - `input/` 폴더의 파일 분류 및 메타데이터 추출
2. **편집 계획 수립** - 이미지별 노출 시간, 효과, 순서 결정
3. **클립 렌더링** - 각 이미지에 Ken Burns 효과 적용
4. **크로스페이드 연결** - 클립 간 부드러운 전환
5. **음악 믹싱** - 배경음악 볼륨 조절 및 페이드 적용
6. **썸네일 추출** - 주요 장면에서 썸네일 이미지 생성

## Ken Burns Effect

정적 이미지에 움직임을 주는 효과입니다:

- **Zoom In** - 서서히 확대 (몰입감)
- **Zoom Out** - 서서히 축소 (전체 파악)

고품질 렌더링을 위해 8000px 업스케일 + lanczos 필터를 사용합니다.

## Configuration

### SKILL.md

전체 파이프라인과 FFmpeg 명령어 템플릿을 정의합니다.

### media_analyzer.md

편집 결정 기준을 정의합니다:
- 컷 길이 결정 원칙
- 모션 효과 선택 기준
- 음악 볼륨/페이드 설정

## Constraints

- 자막/텍스트 오버레이 사용 금지
- `input/` 폴더 외부 파일 사용 금지
- 음악은 `input/` 내 파일 중 1개만 사용

## License

MIT License
