# Rethink 네트워크/DNS 설정 및 기기 추가 가이드

이 문서는 **LG ThinQ Rethink(로컬 클라우드/MQTT 브리지)**를 기존 LG 공식 씽큐 환경(스마트폰 앱, Home Assistant 공식 `lg_thinq` 통합구성요소, Rethink 미지원 가전기기)과 **충돌 없이 공존**시키기 위한 DNS 설정 방법 및 **신규 기기 추가 시 표준 절차**를 설명합니다.

---

## 1. 네트워크 및 DNS 설정 핵심 원칙 (중요!)

### ⚠️ 전역(Global) DNS Rewrite의 치명적인 함정
Rethink는 LG 가전기기가 접속하는 클라우드 도메인(`common.lgthinq.com`, `aic.lgthinq.com` 등)을 로컬 Rethink 서버 IP(`192.168.0.X`)로 가로채어 동작합니다.

만약 공유기, Dnsmasq, 또는 AdGuard Home에서 **네트워크 전체(전역)에 DNS 리다이렉트**를 걸게 되면 다음과 같은 심각한 문제가 발생합니다:
1. **스마트폰 씽큐 앱 먹통**: 스마트폰이 집안 Wi-Fi에 연결될 때 로컬 Rethink IP로 접속을 시도하지만, SSL 인증서 불일치 및 미지원 API로 인해 앱 로그인 및 기기 제어가 전면 중단됩니다.
2. **HA 공식 LG ThinQ 통합구성요소 불가**: HA가 공식 LG API(`api-aic.lgthinq.com` 등) 대신 로컬 Rethink로 요청을 보내어 모든 공식 엔티티가 `unavailable`로 바뀝니다.
3. **Rethink 미지원 가전 연결 두절**: 식기세척기, 전기레인지, 정수기, 냉장고 등 Rethink 드라이버가 없는 기기들이 Rethink에 잘못 연결되어 LG 공식 클라우드로부터 '연결 끊김(오프라인)' 상태가 됩니다.

> **해결책:** **Rethink에 연동할 특정 기기(IP/MAC)에만 선별적으로 DNS Rewrite를 적용**하고, 나머지 모든 기기 및 스마트폰은 진짜 LG 공식 클라우드로 통신하도록 분리해야 합니다.

---

## 2. AdGuard Home 설정 가이드 (강력 권장)

AdGuard Home은 클라이언트별 조건부 필터링(`$client` 옵션)을 완벽하게 지원하므로 혼용 환경에 가장 적합합니다.

### ① 전역 DNS Rewrites 비우기
* AdGuard Home 대시보드 → **[필터]** → **[DNS 재작성 (DNS Rewrites)]** 메뉴에 등록된 `*.lgthinq.com` 항목이 있다면 **모두 삭제**합니다.

### ② 사용자 정의 필터링 규칙 (Custom Filtering Rules) 등록
* **[필터]** → **[사용자 정의 필터링 규칙]** 맨 아래에 다음 규칙들을 추가합니다.
* `<RETHINK_IP>`는 Rethink 서버(Home Assistant) IP, `<TARGET_DEVICE_IP>`는 Rethink에 연동할 가전기기의 IP(또는 MAC)입니다.

```adblock
! ===================================================================
! [LG ThinQ Rethink] 대상 기기 전용 리다이렉트 규칙
! 대상 기기: 예) 환기시스템 ERV (192.168.0.46)
! ===================================================================
||common.lgthinq.com^$dnsrewrite=<RETHINK_IP>,client=<TARGET_DEVICE_IP>
||common.iot.eic.lgthinq.com^$dnsrewrite=<RETHINK_IP>,client=<TARGET_DEVICE_IP>
||common.iot.kic.lgthinq.com^$dnsrewrite=<RETHINK_IP>,client=<TARGET_DEVICE_IP>
||eic.lgthinq.com^$dnsrewrite=<RETHINK_IP>,client=<TARGET_DEVICE_IP>
||eic-krd.lgthinq.com^$dnsrewrite=<RETHINK_IP>,client=<TARGET_DEVICE_IP>
||eic-common.lgthinq.com^$dnsrewrite=<RETHINK_IP>,client=<TARGET_DEVICE_IP>
||krcic.lgthinq.com^$dnsrewrite=<RETHINK_IP>,client=<TARGET_DEVICE_IP>
||kic-common.lgthinq.com^$dnsrewrite=<RETHINK_IP>,client=<TARGET_DEVICE_IP>
||kic-mclip.lgthinq.com^$dnsrewrite=<RETHINK_IP>,client=<TARGET_DEVICE_IP>
||aic.lgthinq.com^$dnsrewrite=<RETHINK_IP>,client=<TARGET_DEVICE_IP>
||rethink.lgthinq.com^$dnsrewrite=<RETHINK_IP>,client=<TARGET_DEVICE_IP>
||d32l1ph11hh4jr.cloudfront.net^$dnsrewrite=<RETHINK_IP>,client=<TARGET_DEVICE_IP>

! (선택) MAC 주소로도 이중 지정 시:
||common.lgthinq.com^$dnsrewrite=<RETHINK_IP>,client=<TARGET_DEVICE_MAC>
||common.iot.eic.lgthinq.com^$dnsrewrite=<RETHINK_IP>,client=<TARGET_DEVICE_MAC>
||common.iot.kic.lgthinq.com^$dnsrewrite=<RETHINK_IP>,client=<TARGET_DEVICE_MAC>
||eic.lgthinq.com^$dnsrewrite=<RETHINK_IP>,client=<TARGET_DEVICE_MAC>
||eic-krd.lgthinq.com^$dnsrewrite=<RETHINK_IP>,client=<TARGET_DEVICE_MAC>
||eic-common.lgthinq.com^$dnsrewrite=<RETHINK_IP>,client=<TARGET_DEVICE_MAC>
||krcic.lgthinq.com^$dnsrewrite=<RETHINK_IP>,client=<TARGET_DEVICE_MAC>
||kic-common.lgthinq.com^$dnsrewrite=<RETHINK_IP>,client=<TARGET_DEVICE_MAC>
||kic-mclip.lgthinq.com^$dnsrewrite=<RETHINK_IP>,client=<TARGET_DEVICE_MAC>
||aic.lgthinq.com^$dnsrewrite=<RETHINK_IP>,client=<TARGET_DEVICE_MAC>
||rethink.lgthinq.com^$dnsrewrite=<RETHINK_IP>,client=<TARGET_DEVICE_MAC>
||d32l1ph11hh4jr.cloudfront.net^$dnsrewrite=<RETHINK_IP>,client=<TARGET_DEVICE_MAC>

! ===================================================================
! [공식 클라우드 보호] 나머지 모든 가전 및 스마트폰용 화이트리스트
! ===================================================================
@@||lge.com^$important
@@||lgthinq.com^$important
@@||lgeapi.com^$important
@@||lgtviot.com^$important
@@||smartthinq.com^$important
```

### ③ 클라이언트 설정 등록 (선택 권장)
* AdGuard Home **[설정]** → **[클라이언트 설정]**에서 해당 기기(IP 및 MAC)를 영구 클라이언트로 등록하여 관리의 편의성을 높입니다.

---

## 3. Dnsmasq 사용 시 주의사항

* Dnsmasq의 일반적인 `hosts` 설정(`host: common.lgthinq.com, ip: <RETHINK_IP>`)은 모든 클라이언트에 무차별 적용되므로 **절대 단일 DNS 환경에서 권장되지 않습니다.**
* Dnsmasq를 사용할 경우:
  1. **IoT 전용 서브넷/VLAN 분리**: Rethink 기기 전용 Wi-Fi SSID를 만들고, 해당 서브넷에만 Rethink DNS를 제공.
  2. **DHCP 태그 분기**: `dhcp-host=<DEVICE_MAC>,set:rethink` 및 태그 조건부 dnsmasq 옵션을 활용해야 합니다.

---

## 4. Rethink 신규 기기 추가 시 5단계 표준 절차

Rethink로 제어할 새로운 LG 가전기기를 네트워크에 추가할 때마다 아래 순서대로 작업을 진행합니다:

### [1단계] 공유기에서 고정 IP(Static Lease) 할당
* 공유기 관리자 페이지(예: ipTIME `내부 네트워크 설정` → `수동 IP 설정`)에서 새로 추가할 기기의 **MAC 주소를 확인하고 고정 IP를 할당**합니다.
* *이유:* 기기 IP가 변경되면 AdGuard Home의 `$client` 리다이렉트 대상에서 벗어나기 때문입니다.

### [2단계] AdGuard Home에 신규 기기 리다이렉트 규칙 추가
* AdGuard Home **[필터]** → **[사용자 정의 필터링 규칙]**에 2번 항목의 12개 도메인 규칙을 새 기기 IP(또는 MAC)로 복사하여 추가합니다.
  ```adblock
  ||common.lgthinq.com^$dnsrewrite=<RETHINK_IP>,client=<NEW_DEVICE_IP>
  ||common.iot.eic.lgthinq.com^$dnsrewrite=<RETHINK_IP>,client=<NEW_DEVICE_IP>
  ... (12개 도메인 동일하게 추가)
  ```

### [3단계] Rethink 지원 기기 여부 확인 및 드라이버 매핑
* `cloud/devices/` 목록 또는 본 README의 지원 기기 표에서 새로 추가할 기기의 모델 코드(`modelId` 또는 `deviceType`)가 지원되는지 확인합니다.
* **지원 기기인 경우**:
  * `cloud/ha_bridge.ts`의 `t2deviceTypes`(또는 `t1deviceTypes`)에 해당 모델 식별자가 등록되어 있는지 확인합니다.
* **미지원 신규 기기인 경우**:
  * Rethink의 **Bridge 모드**(`"bridge": { "dns": ... }`)를 활성화하여 기기가 공식 클라우드와 통신하도록 경유시키면서, 패킷을 캡처(`tools/rethink-capture.ts`, `packet-parser.ts`)하여 새 드라이버(`cloud/devices/<NEW_MODEL>.ts`)를 작성하고 `ha_bridge.ts`에 등록합니다.

### [4단계] 기기 네트워크 리셋 (DNS 캐시 갱신)
* 가전기기의 Wi-Fi 모듈은 이전 DNS 조회 결과를 캐싱하고 있거나 연결 실패 시 긴 백오프(재시도 대기) 상태에 있을 수 있습니다.
* **기기 전원을 완전히 껐다 켜거나(플러그 분리 후 재연결)**, 공유기에서 해당 기기 Wi-Fi 연결을 재접속시킵니다.
* Rethink 서비스 로그(`http://<RETHINK_IP>:44401` 웹 관리자 화면 또는 컨테이너 로그)에서 기기의 접속 여부(`incoming clip/message/devices/...`)를 확인합니다.

### [5단계] Home Assistant MQTT 상태 확인
* Rethink가 기기로부터 초기 상태 패킷을 수신하면 Home Assistant MQTT Discovery를 통해 엔티티를 자동 생성합니다.
* Home Assistant **[설정]** → **[기기 및 서비스]** → **MQTT**에서 신규 기기와 센서/스위치가 정상 등록되었는지 확인합니다.
