---
author: "JeongHyeon"
pubDatetime: 2026-04-15T00:00:00Z
title: "OTLP: OpenTelemetry 가 뭔가요"
featured: false
draft: false
tags: ["모니터링", "OpenTelemetry", "OTLP"]
description: "OpenTelemetry와 OTLP가 무엇인지 알아본다"
---

## 시작하기 전에

오늘 이제 otel 을 서비스에 붙이면서 시작됐다. 자꾸 말도안되는 트집을 잡으며 안해주려고 하는
우리팀 백엔드 개발자가 나는 너무 싫어

otel 은 이제 간단하게 "텔레메트리 데이터를 하나의 통합된 형식으로 수집하고 처리"하며 내보낸다.
이때 otel 은 "OTLP" 라는 표준 규격을 지키는데

여기서 나온 OTLP 란 뭘까 간단하게 알아보려고 한다.

## OTLP (OpenTelemetry Protocol)

일단 OTLP 는 애플리케이션의 상태와 성능을 파악하기 위한 **"원격 측정 데이터(Telemetry Data)"** 를 전송하는 표준 프로토콜이다.

과거에는 모니터링 도구마다 데이터를 전송하는 방식과 규격이 달라서 모니터링 시스템을 확장할 때 불편함을 느꼈었다고 한다.

그때 이제 OTLP 라는 하나의 표준규격으로 통일하여, 특정 Vendor 에 종속되지 않고 일관된 규격으로 데이터를 주고 받을 수 있게 하였다고 한다.

OTLP 데이터는 [Trace](https://blog.weourus.xyz/posts/dev/sentry#tracing), Metric, Log 종류에 상관없이
공통적인 JSON 형태의 **"계층 구조"** 를 가지고 있다.

### Vendor

생소한 단어라고 생각한다. 수집된 원격 측정 데이터를 저장, 분석, 시각화해주는 서비스나 백엔드 플랫폼을 의미한다.

### 추적 (Trace)

Sentry 에서도 언급했었는데, 여러 마이크로서비스나 시스템 컴포넌트를 거쳐 가는 **"전체 경로 시간"** 을 기록하여 여러 작업 단위(Span)을 모아 하나의 Trace 를 구성한다.

주로 병목 현상이나 지연이 발생하는 구간을 특정할때 많이 사용한다.

### 지표 (Metric)

일정 시간 동안 측정된 시스템의 **"수치 데이터"** 이다. CPU 사용률, 메모리 점유율, API 요청 수, 에러 발생율 등 이런것들을 모아 모니터링할 수 있도록 하는 기준이 된다.
우리는 Grafana 를 이용해서 위에 예시로 든 화면들을 대시보드로 보고있다.

### 로그 (Log)

개발하면서 제일 많이 들어본 단어라고 생각한다. 특정 시점에 발생한 **"개별 이벤트의 텍스트 기록"** 이다.

```java
System.out.println("...");
```

```python
print("...")
```

```javascript
console.log("...");
```

우리가 해당 로그들을 확인하거나, 서버 로그들 올라오는 거 보면, 데이터베이스 쿼리 타입 안맞아서 발생하는 오류, 결제 실패해서 발생하는 오류 등이 이에 해당한다.

### Protobuf

OTLP는 기본적으로 **Protobuf(Protocol Buffers)** 이라는 이진 직렬화 포맷을 사용하여 데이터를 압축하고 통신한다.
프로토콜로는 gRPC, HTTP 두 개가 대표적이다.

**gRPC** 는 "기본 권장 통신 방식"으로, HTTP/2를 기반으로 지속적인 연결을 유지하며, 데이터를 "스트리밍 형태"로 전송하므로 성능이 가장 우수하다.
주로 백엔드 서버 간의 통신에 사용된다.

**HTTP** 는 브라우저 환경이나, 방화벽 환경에서 사용된다. HTTP payload 로 전송하거나, JSON 형태로 변환하여 전송한다.

### 데이터 형식

OTLP 를 HTTP JSON 포맷으로 전송할때는 다음과 같은 형태로 보낸다.

```json
{
  "resourceSpans": [
    {
      "resource": {
        "attributes": [
          {
            "key": "service.name",
            "value": { "stringValue": "react-frontend" }
          }
        ]
      },
      "scopeSpans": [
        {
          "scope": {
            "name": "my-react-app-tracer"
          },
          "spans": [
            {
              "traceId": "5b8aa5a2d2c872e8321cf37308d69df2",
              "spanId": "5159146fa0bf3105",
              "name": "render-component",
              "kind": 1,
              "startTimeUnixNano": "1680000000000000000",
              "endTimeUnixNano": "1680000000050000000",
              "status": {
                "code": 0
              }
            }
          ]
        }
      ]
    }
  ]
}
```

| 필드 | 설명 |
|---|---|
| `resourceSpans` | 리소스(서비스) 단위로 묶인 Span 데이터의 최상위 배열 |
| `resource.attributes` | 서비스 이름, 버전 등 리소스를 식별하는 속성 |
| `scopeSpans` | 계측 라이브러리(Tracer) 단위로 묶인 Span 배열 |
| `scope.name` | Span 을 생성한 Tracer 의 이름 |
| `traceId` | 하나의 요청 흐름 전체를 식별하는 고유 ID (32자 hex) |
| `spanId` | 개별 작업 단위를 식별하는 고유 ID (16자 hex) |
| `name` | Span 이름 (어떤 작업인지) |
| `kind` | Span 종류 (1=INTERNAL, 2=SERVER, 3=CLIENT 등) |
| `startTimeUnixNano` | 작업 시작 시각 (나노초 단위 Unix timestamp) |
| `endTimeUnixNano` | 작업 종료 시각 (나노초 단위 Unix timestamp) |
| `status.code` | 상태 코드 (0=UNSET, 1=OK, 2=ERROR) |

### React 에서 OTLP 사용하기

내가 사용하는 React 에서는 OTLP/HTTP 전송 방식으로 `@opentelemetry/sdk-trace-web` 패키지를 설정하여서 추적할 수 있다.

```typescript
import { WebTracerProvider } from '@opentelemetry/sdk-trace-web';
import { OTLPTraceExporter } from '@opentelemetry/exporter-trace-otlp-http';
import { BatchSpanProcessor } from '@opentelemetry/sdk-trace-base';
import { ZoneContextManager } from '@opentelemetry/context-zone';

const provider = new WebTracerProvider();

const exporter = new OTLPTraceExporter({
  url: 'http://localhost:4318/v1/traces'
});

provider.addSpanProcessor(new BatchSpanProcessor(exporter));

provider.register({
  contextManager: new ZoneContextManager()
});

const tracer = provider.getTracer('my-react-app');

const span = tracer.startSpan('button-click-action');

try {
  performAction();
} finally {
  span.end();
}
```

### 데이터 생성에서 대시보드까지 (React 기준)

1. **계측 데이터를 생성한다.** React 나 Spring, Nest 에서 심어둔 OpenTelemetry 가 사용자 감지를 통해서 데이터를 생성한다.

2. **BatchSpanProcessor 가 배치 처리한다.** 성능저하를 막기 위해서 일정량을 모아서, 설정한 주소로 데이터를 전송한다.

3. **OTel Collector 가 데이터를 수신한다.**

4. **데이터를 가공한다.** 수집한 데이터에서 민감한 정보는 마스킹 하거나 필터링을 통해, 벤더 전송 비용을 최적화 하는 가공 작업을 진행한다.

5. **벤더로 전송한다.** 가공이 완료된 데이터를 DataDog, Sentry, Grafana 같은 툴에 규격에 맞춰 전송한다.

6. **대시보드에서 확인한다.** 우리가 이제 보는 대시보드를 보면서 원인을 파악하고 개선해 나가면 된다.

### 프론트엔드 보다는 백엔드가 더 효과적일것같아요

나도 그렇게 생각한다. 실제로도 아마 백엔드에 붙여서 사용하는 일이 훨씬 많을 것이다.

사용자의 이벤트 자체를 추적하는 것도 의미는 있다. 항상 "나는 문제없는 데 왜 배포만 하면 문제있다 그러는걸까" 와 같은 상황에서
재현하는데 큰 도움을 주기 때문이다.

하지만, 대부분의 병목현상이나 에러는 서버에 데이터를 넘기고, 서버에서 처리하는 과정속에서 발생한다. 이 과정을 추적하고, 단서를 찾아내는 것이 목적이기 때문에 그렇다고 생각한다.

또한 탈취의 위험도 있다. 요즘은 일반인들도 F12 눌러서 네트워크 열어서 보는 사람들이 있는것같다. 거기서 엔드포인트나, 인증키가 노출되고 탈취된다면 ..
말하지 않아도 안다고 생각한다.

## 마치며

어쩌다보니 OTEL 내용과 살짝 섞인감이 없진않다. 다음에는 OTEL 부터 후다닥 공부해야겠다.

나는 많은 프로젝트들을 하다보니 손빠른게 장점이였던것같고, 설계 부분이 약점이라고 생각했는데 요즘은 클로드가 워낙 잘하다보니 내 장점이 없어진 느낌이다.
그래서 요즘은 회사에서 매번 처음듣는 용어들을 들으면서, 많이 위축되기도하지만 열심히 공부해서 나도 당당하게 말하는 그날이 오기만을 바라고있다.

열심히 공부해서, 당당하게 내 의견을 피력하는 날이 오기를 ..

### 참조

- [OpenTelemetry Protocol Specification](https://github.com/open-telemetry/opentelemetry-proto/tree/main/docs)
