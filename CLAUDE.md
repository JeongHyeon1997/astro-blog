# Blog Writing Rules

## 글 작성 방식

사용자가 블로그 글 내용을 자유롭게 적어주면, Claude가 아래 규칙에 따라 정리해서 `.md` 파일로 저장한다.

### 파일 위치

카테고리별 폴더에 저장:

| 폴더 | 내용 |
|------|------|
| `src/data/blog/frontend/` | React, JavaScript, TypeScript |
| `src/data/blog/backend/` | Spring, Kotlin, Java |
| `src/data/blog/cloud/` | AWS, GCP, Docker, K8s |
| `src/data/blog/dev/` | 폴더구조, 아키텍처, 개발 일반 |
| `src/data/blog/log/` | 잡생각, 회고, 블로그 이야기 |

### Frontmatter 형식

```yaml
---
author: "JeongHyeon"
pubDatetime: 2026-01-01T00:00:00Z
title: "글 제목"
featured: false
draft: false
tags: ["tag1", "tag2"]
description: "한 줄 요약"
---
```

- `pubDatetime`은 오늘 날짜로 설정
- `tags`는 내용에 맞게 자동 추출
- `description`은 글 내용을 한 문장으로 요약

### 정리 규칙

- 사용자가 적은 내용의 흐름과 뉘앙스를 그대로 유지한다
- 제목 계층 구조(`##`, `###`)를 잡아준다
- 코드가 포함된 경우 언어 명시한 코드 블록으로 감싼다
- 자연스러운 단락 구분을 추가한다
- 불필요한 내용을 임의로 추가하지 않는다
