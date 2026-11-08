# CTF Agent Comparison

2026 ASC 팀 프로젝트 (숭실대학교 ASC, team03)

설계 방식이 다른 LLM 기반 CTF 에이전트 3종을 같은 조건에서 비교·분석하고, 효과가 확인된 요소를 팀 자체 에이전트로 확장한다.

## 비교 대상

| 구분 | 에이전트 | 설계 방식 | 코드 |
| --- | --- | --- | --- |
| A | NYU CTF baseline | 단일 에이전트, 기본 셸 루프 (대조군) | [nyuctf_agents](https://github.com/NYU-LLM-CTF/nyuctf_agents) `run_baseline.py` |
| B | EnIGMA | 단일 에이전트 + 대화형 도구 (디버거, 서버 연결) | [SWE-agent v0.7](https://github.com/SWE-agent/SWE-agent/tree/v0.7) `run.py --ctf` |
| C | D-CIPHER | 멀티 에이전트 (계획자·실행자·자동 프롬프트) | [nyuctf_agents](https://github.com/NYU-LLM-CTF/nyuctf_agents) `run_dcipher.py` |

## 진행 단계

1. 자료 조사 및 환경 구축 ← 현재
2. 세 에이전트 비교 실험
3. 결과 분석 및 요소별 효과 검증, 풀이 기록(trace) 분석
4. 자체 에이전트로 확장 및 재평가

## 레포 구조

```
docs/
  experiment-rules.md   실험 조건과 실패 유형 분류 기준
  problems.md           평가 문제셋
  references.md         참고 논문·자료
setup/
  enigma.md             EnIGMA 설치 기록
  nyuctf-agents.md      NYU baseline, D-CIPHER 설치 기록
runs/                   실행 결과 요약 (원본 로그는 커밋하지 않음)
```

## 주의

- API 키(`keys.cfg`, `.env`)와 문제 파일·플래그는 커밋하지 않는다.
- 문제는 이름과 경로로만 참조한다.
