# /short Skill

input/ 폴더의 미디어 파일들을 분석하여 YouTube Shorts (9:16, 60~90s)를 자동 생성한다.

## Core Function

사용자가 `/short` 명령을 실행하면, input/ 폴더의 영상/이미지/음악 파일을 분석하고, 편집 계획을 수립한 뒤, FFmpeg로 최종 쇼츠 영상과 썸네일을 생성한다.

## Processing Pipeline

### Step 1: 미디어 스캔 및 분석

input/ 폴더를 스캔하여 미디어 파일을 분류하고 ffprobe로 메타데이터를 추출한다.

```bash
# 파일 목록 확인
ls -la input/

# 영상 메타데이터 추출 예시
ffprobe -v quiet -print_format json -show_format -show_streams "input/video.mp4"
```

분석 결과를 `output/debug/media_index.json`에 저장:

```json
{
  "videos": [
    {
      "path": "input/example.mp4",
      "duration": 14.2,
      "width": 1920,
      "height": 1080,
      "has_audio": true
    }
  ],
  "images": [
    {
      "path": "input/photo.jpg",
      "width": 3024,
      "height": 4032
    }
  ],
  "music": [
    {
      "path": "input/bgm.mp3",
      "duration": 180.0
    }
  ]
}
```

### Step 2: 편집 계획 수립

media_index.json을 기반으로 `@media_analyzer.md` 가이드에 따라 편집 계획을 수립한다.

편집 계획을 `output/debug/edit_plan.json`에 저장:

```json
{
  "target_seconds": 54,
  "timeline": [
    {
      "type": "video",
      "path": "input/example.mp4",
      "start": 2.0,
      "end": 6.8,
      "out_seconds": 4.8,
      "reason": "안정적인 구간, 감정적 가치 높음"
    },
    {
      "type": "image",
      "path": "input/photo.jpg",
      "out_seconds": 2.5,
      "motion": "zoom_in",
      "reason": "시각적 복잡도 높음"
    }
  ],
  "music": {
    "path": "input/bgm.mp3",
    "volume": 0.15,
    "fade_in": 1.0,
    "fade_out": 1.5
  },
  "thumbnails": [
    { "t": 4.2 },
    { "t": 21.0 },
    { "t": 39.5 }
  ]
}
```

### Step 3: FFmpeg 렌더링

edit_plan.json을 기반으로 FFmpeg 명령어를 생성하고 실행한다.

#### 3.1 영상 클립 처리 (9:16 변환)

```bash
# 가로 영상 → 세로 변환 (center crop)
ffmpeg -i "input/video.mp4" -ss 2.0 -t 4.8 \
  -vf "scale=1080:1920:force_original_aspect_ratio=increase,crop=1080:1920" \
  -c:v libx264 -preset fast -crf 23 \
  -an "output/temp/clip_0.mp4"
```

#### 3.2 이미지 처리 (Ken Burns 효과)

```bash
# 이미지 → 영상 변환 (zoom_in 효과)
ffmpeg -loop 1 -i "input/photo.jpg" -t 2.5 \
  -vf "scale=1200:2132:force_original_aspect_ratio=increase,crop=1080:1920,zoompan=z='zoom+0.001':d=75:s=1080x1920" \
  -c:v libx264 -preset fast -crf 23 \
  -pix_fmt yuv420p "output/temp/clip_1.mp4"
```

#### 3.3 클립 연결

```bash
# concat 파일 생성 후 연결
ffmpeg -f concat -safe 0 -i "output/temp/concat_list.txt" \
  -c copy "output/temp/merged.mp4"
```

#### 3.4 음악 믹싱

```bash
# 배경음악 추가 (fade in/out)
ffmpeg -i "output/temp/merged.mp4" -i "input/bgm.mp3" \
  -filter_complex "[1:a]volume=0.15,afade=t=in:st=0:d=1.0,afade=t=out:st=52.5:d=1.5[a]" \
  -map 0:v -map "[a]" -shortest \
  -c:v copy -c:a aac "output/final_short.mp4"
```

### Step 4: 썸네일 생성

```bash
# 지정된 타임스탬프에서 프레임 추출
ffmpeg -i "output/final_short.mp4" -ss 4.2 -frames:v 1 \
  -vf "scale=1080:1920:force_original_aspect_ratio=increase,crop=1080:1920" \
  "output/thumbnail_01.jpg"
```

## Technical Specifications

- **해상도**: 1080x1920 (9:16)
- **목표 길이**: 60~90초
- **비디오 코덱**: H.264 (libx264)
- **오디오 코덱**: AAC
- **도구**: ffmpeg, ffprobe

## Output

```
output/
  final_short.mp4      # 최종 쇼츠 영상
  thumbnail_01.jpg     # 썸네일 1
  thumbnail_02.jpg     # 썸네일 2
  thumbnail_03.jpg     # 썸네일 3 (선택)
  debug/
    media_index.json   # 미디어 분석 결과
    edit_plan.json     # 편집 계획
```

## Key Constraints

- 자막/텍스트 오버레이 **사용 금지**
- 컷 길이는 고정하지 않고 **근거 기반 판단**
- input/ 폴더 외부 파일 사용 금지
- 음악은 input/ 내 파일 중 **1개만** 사용
