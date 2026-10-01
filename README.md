# 동네 지도 파일 (PMTiles)

「동네」 화면의 바탕 지도. OpenStreetMap 자료로 만든 Protomaps basemap(v4 타일, MVT·gzip)을
범위만 잘라 둔 파일이다. **git 에 넣지 않는다**(`.gitignore`: `release-site/maps/*.pmtiles`).

| 파일 | 범위 | 확대 | 크기(2026-10) | 상태 |
| --- | --- | --- | --- | --- |
| `seoul-metro.pmtiles` | 수도권 `126.3,36.8,127.9,38.0` | z0–14 | 약 74MB | 켬 |
| `world.pmtiles` | 세계 | z0–6 | 약 45MB | 켬 (수도권 밖은 이것으로 넓게만) |
| `korea.pmtiles` | 한국 `124.5,33.0,131.0,38.7` | z0–14 | 약 388MB | 끔 — 별도 저장소로 옮긴 뒤 |

목록·범위의 원본은 `core/market/merchants/map-sources.ts` 이다. 파일을 더하면 그 표와
`release-site/build.mjs` 의 `MAP_FILES` 를 같이 고친다(`phone/ui/map/vector-map.test.ts` 가 맞는지 본다).

## 다시 만들기

```sh
scripts/fetch-maps.sh            # 어제(UTC) Protomaps 일일 빌드에서 잘라 온다
scripts/fetch-maps.sh 20260930   # 날짜 지정
```

pmtiles CLI 가 필요하다(맥: `brew install pmtiles`, 리눅스 arm64: `qa/tools/pmtiles-arm64`).

## 배포

- `node release-site/build.mjs` 가 켜진 파일만 `artifacts/site-release/public/maps/` 로 복사한다.
- `release-site/vercel.json` 의 `/maps/(.*)` 헤더: `application/octet-stream`, `Accept-Ranges: bytes`,
  `Cache-Control: public, max-age=86400`. 지도는 파일 전체가 아니라 필요한 조각만 Range 요청으로 받는다.
- 서비스 워커(`/wallet/sw.js`)는 `/maps/`·`/wallet/map/` 을 미리 저장하지 않는다.
- Vercel CLI 배포는 업로드 총량 한도가 있다(Hobby 100MB, Pro 1GB — vercel.com/docs/limits
  「Static file uploads」). 두 파일만 약 119MB 라 **Hobby 계정이면 배포가 실패한다.**
- 출처 「© OpenStreetMap contributors · Protomaps」는 지도 위에 늘 보인다(ODbL — 숨기지 않는다).

## 글자(지명) 라벨

지금은 길·지명 글자 층이 없다(글꼴 glyph 파일을 외부에서 받지 않기 위해). 핀 이름은 앱 글자로 따로 그린다.
`fonts/` 에 자체 글리프(Noto Sans KR 일부 범위 PBF)를 두면 나중에 켤 수 있다.

## 배포 위치 (2026-10-01)
지도 파일은 Vercel 배포에 넣지 않는다. 별도 공개 저장소 `PLAYX1/playx-raven-maps`(main) 맨 위에 둔다 — 설치 파일 저장소(`playx-raven-releases` dist)에 넣지 않는 이유:
릴리스 CI 가 매번 dist 를 통째로 clone 하므로 수백 MB 를 더하면 모든 릴리스가 무거워지고, 지도를 갱신할 때마다 그 저장소 기록이 불어난다.
`release-site/vercel.json` 의 `/maps/:file(*.pmtiles)` 재작성(rewrite)이 `https://ravenvault.ex.erci.se/maps/…` 로 보여 주므로 웹 CSP(`'self'`)와 폰 WebView 허용 주소는 그대로다.
- GitHub 파일 하나는 100MB 이하(지금 가장 큰 것 71MB). 지도는 자주 갱신하지 않는다(갱신마다 저장소 기록이 늘어난다).
- raw.githubusercontent.com 은 CDN 이 아니다. 사용이 많아지면 속도 제한이 걸릴 수 있다 → 그때 Cloudflare R2 등으로 옮기고 이 재작성 한 줄만 바꾼다.
- 배포 뒤 확인: `curl -sI -H 'Range: bytes=0-16383' https://ravenvault.ex.erci.se/maps/seoul-metro.pmtiles` 가 206 이어야 한다.
- 로컬에서 Vercel 없이 시험할 때만 `MAPS_FROM_DISK=1 node release-site/build.mjs`.
