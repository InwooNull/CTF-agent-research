# 평가 문제셋

NYU CTF Bench(CSAW CTF 2017~2023, 200문제)에서 카테고리별로 균형을 맞춰 선정한다. 문제 파일과 플래그는 커밋하지 않고 경로로만 기록한다.

## 선정 기준

- 카테고리별 4~5문제 (pwn, rev, crypto, web, forensics, misc)
- 난이도 분산: 선행 논문에서 풀린 문제와 안 풀린 문제를 섞는다
- 데이터 오염 대비: 최근 대회 문제 또는 자체 제작 문제를 일부 추가

## 데이터셋 정보

- 버전: `v20250206` (`python3 -m nyuctf.download` 기본값)
- 위치: `~/.nyuctf/v20250206/` (아래 경로는 이 폴더 기준)
- 분할: `test` (`--split test`)
- 문제 이름 열의 값이 `test_dataset.json`의 키이며, 실행할 때 `--challenge`에 그대로 넣는다.
- test split 카테고리 분포: crypto 52, rev 51, pwn 39, misc 24, web 19, forensics 15

## 확정 목록 (27문제)

비고 열에 적은 것:

- **배점**: `challenge.json`의 `points` 값. 없으면 `-`. 대략적인 난이도로 본다.
- **서버**: 문제 서버 컨테이너가 필요한 문제(`compose`/`internal_port` 있음)
- **선행 풀이**: 선행 논문에서 풀렸다고 기록된 문제 (출처는 아래 참고)

| # | 카테고리 | 문제 이름 | 데이터셋 경로 | 비고 |
| --- | --- | --- | --- | --- |
| 0 | misc | 2018f-msc-leaked_flag | test/2018/CSAW-Finals/misc/leaked_flag | 스모크 테스트용 (평가 제외) |
| 1 | pwn | 2019q-pwn-baby_boi | test/2019/CSAW-Quals/pwn/baby_boi | 50점, 서버, 선행 풀이 (EnIGMA·D-CIPHER) |
| 2 | pwn | 2020q-pwn-roppity | test/2020/CSAW-Quals/pwn/roppity | 50점, 서버 |
| 3 | pwn | 2023q-pwn-puffin | test/2023/CSAW-Quals/pwn/puffin | 75점, 서버, 선행 풀이 (NYU CTF Bench 예시) |
| 4 | pwn | 2023q-pwn-unlimited_subway | test/2023/CSAW-Quals/pwn/unlimited_subway | 92점, 서버, 선행 풀이 (D-CIPHER) |
| 5 | pwn | 2019q-pwn-popping_caps | test/2019/CSAW-Quals/pwn/popping_caps | 300점, 서버, 고난도 |
| 6 | rev | 2020q-rev-baby_mult | test/2020/CSAW-Quals/rev/baby_mult | 50점, 선행 풀이 (EnIGMA) |
| 7 | rev | 2023q-rev-rebug_2 | test/2023/CSAW-Quals/rev/rebug 2 | 50점, 선행 풀이 (EnIGMA), 경로에 공백 |
| 8 | rev | 2021q-rev-checker | test/2021/CSAW-Quals/rev/checker | 배점 -, 선행 풀이 (EnIGMA 일부 설정) |
| 9 | rev | 2019f-rev-steady_counting | test/2019/CSAW-Finals/rev/steady_counting | 200점 |
| 10 | rev | 2023q-rev-rox | test/2023/CSAW-Quals/rev/rox | 464점, 고난도 |
| 11 | crypto | 2018q-cry-babycrypto | test/2018/CSAW-Quals/crypto/babycrypto | 50점, 선행 풀이 (EnIGMA) |
| 12 | crypto | 2023f-cry-textbook_rsa | test/2023/CSAW-Finals/crypto/Textbook-RSA | 60점, 서버 |
| 13 | crypto | 2018q-cry-lowe | test/2018/CSAW-Quals/crypto/lowe | 200점 |
| 14 | crypto | 2019q-cry-super_curve | test/2019/CSAW-Quals/crypto/super_curve | 300점, 서버, 선행 풀이 (EnIGMA) |
| 15 | crypto | 2023q-cry-lottery | test/2023/CSAW-Quals/crypto/lottery | 450점, 서버, 고난도 |
| 16 | web | 2023q-web-smug_dino | test/2023/CSAW-Quals/web/smug-dino | 50점, 서버 |
| 17 | web | 2023f-web-shreeramquest | test/2023/CSAW-Finals/web/ShreeRamQuest | 50점, 서버 |
| 18 | web | 2021q-web-poem_collection | test/2021/CSAW-Quals/web/poem-collection | 배점 -, 서버 |
| 19 | web | 2023q-web-cookie_injection | test/2023/CSAW-Quals/web/cookie-injection | 488점, 서버, 고난도 |
| 20 | forensics | 2023q-for-1black0white | test/2023/CSAW-Quals/forensics/1black0white | 50점, 선행 풀이 (EnIGMA) |
| 21 | forensics | 2021q-for-lazy_leaks | test/2021/CSAW-Quals/forensics/Lazy-Leaks | 배점 -, 선행 풀이 (EnIGMA) |
| 22 | forensics | 2023f-for-hypokrinesthai | test/2023/CSAW-Finals/forensics/hypokrinesthai | 82점 |
| 23 | forensics | 2018q-for-whyos | test/2018/CSAW-Quals/forensics/whyOS | 300점 |
| 24 | misc | 2023q-msc-android_dropper | test/2023/CSAW-Quals/misc/android-dropper | 50점, 서버, 선행 풀이 (EnIGMA·NYU CTF Bench 예시) |
| 25 | misc | 2023f-msc-pyjail | test/2023/CSAW-Finals/misc/pyjail | 74점, 서버 |
| 26 | misc | 2018q-msc-algebra | test/2018/CSAW-Quals/misc/algebra | 100점, 서버, 선행 풀이 (EnIGMA) |
| 27 | misc | 2023f-msc-urkel | test/2023/CSAW-Finals/misc/urkel | 494점, 서버, 고난도 |

카테고리별: pwn 5, rev 5, crypto 5, web 4, forensics 4, misc 4. 선행 풀이 12문제, 나머지 15문제. 2023년 문제 13개.

## 선행 풀이 출처

논문마다 공개 범위가 달라서 "선행 풀이 아님"이 "안 풀림"을 뜻하지는 않는다.

- **EnIGMA** (arXiv 2409.16165) Appendix F, Table 15: 무작위 113문제에 대한 문제별 결과. 200문제 전체 목록은 없다.
  - 이 표에서 `getit`, `pilot`, `PerfectSecrecy`는 유출된 플래그(leaked flag)로 풀린 것으로 표시돼 있어서 제외했다.
- **D-CIPHER** (arXiv 2502.10931) VI-A절: Auto-prompter 비교에서 Claude 3.5 Sonnet이 푼 pwn 문제로 slithery, unlimited_subway, got_milk, bigboy, baby_boi를 언급한다. 문제별 결과표는 없다.
- **NYU CTF Bench** (arXiv 2406.05590): 카테고리별 비율만 있다. 그림 예시로 puffin, AndroidDropper 풀이 기록이 나온다.
- 출처 확인은 논문 HTML을 요약해서 읽은 것이라, 실험 전에 Table 15 원문과 한 번 더 대조한다.

## 남은 일

- 서버가 필요한 문제는 Docker 이미지를 미리 받아서 정상 기동하는지 확인한다.
- 데이터 오염 대비용 자체 제작 문제를 추가할지 팀에서 정한다.
