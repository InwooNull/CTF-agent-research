# EnIGMA 설치 기록

- 담당: 송인우
- 대상: SWE-agent v0.7 (EnIGMA 모드)
- 환경: Windows + WSL2(Ubuntu) + Docker

> SWE-agent 문서는 Windows 네이티브 설치에 문제가 있다고 안내하므로 WSL2 안에서 진행한다.

## 설치 순서

```bash
# 1. 코드 받기 (v0.7 고정)
git clone -b v0.7 https://github.com/SWE-agent/SWE-agent.git
cd SWE-agent

# 2. 가상환경 + 설치 (Python 3.9 이상)
python3 -m venv .venv && source .venv/bin/activate
python -m pip install --upgrade pip && pip install --editable .

# 3. EnIGMA용 Docker 이미지와 네트워크
docker pull sweagent/enigma:latest
docker network create ctfnet   # 이미 있으면 생략

# 4. API 키: 레포 루트에 keys.cfg 생성 (커밋 금지)
#    ANTHROPIC_API_KEY: '...'
```

## 실행 예시 (스모크 테스트)

NYU CTF Bench의 쉬운 문제(`leaked_flag`, 설명에 플래그가 노출된 문제)로 동작만 확인한다.

```bash
python run.py \
  --model_name <모델명> \
  --ctf \
  --image_name sweagent/enigma:latest \
  --data_path ../NYU_CTF_Bench/test/2018/CSAW-Finals/misc/leaked_flag/challenge.json \
  --repo_path ../NYU_CTF_Bench/test/2018/CSAW-Finals/misc/leaked_flag/ \
  --config_file config/default_ctf.yaml \
  --per_instance_cost_limit 1.00
```

- 실행 결과는 `trajectories/` 아래 trajectory 파일로 저장된다 (풀이 기록 분석에 사용).
- 같은 문제를 다시 돌리려면 `--skip_existing=False`.

## 코드 분석 메모

- **지원 모델이 구버전에 고정됨**: `sweagent/agent/models.py`의 `AnthropicModel.MODELS`에 등록된 Claude 모델은 `claude-3-5-sonnet-20240620`까지다. 최신 모델을 쓰려면 이 딕셔너리에 모델명, `max_context`, `max_tokens`, 토큰당 비용을 추가해야 한다.
- **비용 제한**: `--per_instance_cost_limit`(문제당), `--total_cost_limit`(전체)으로 설정. 비용은 위 딕셔너리의 단가로 계산되므로, 모델 추가 시 단가를 정확히 넣어야 비용 비교가 맞는다.
- **문제 지정 방식**: `challenge.json`(이름, 설명, 카테고리, 파일, 서버 정보)과 문제 폴더를 직접 넘긴다. 폴더에 `docker-compose.yml`이 있으면 서버형 문제(web 등)용 컨테이너를 같이 띄운다.

## 설치 결과

| 단계 | 결과 | 비고 |
| --- | --- | --- |
| clone + pip install | 완료 | v0.7 브랜치, venv 환경 (2026-11-08) |
| docker 이미지 / ctfnet | 완료 | sweagent/enigma:latest, ctfnet 네트워크 생성 |
| keys.cfg 설정 | 보류 | 실험 모델 확정 후 설정 |
| 최신 모델 등록 | 예정 | models.py에 최신 Claude 모델과 단가 추가 필요 |
| 스모크 테스트 | 예정 | 모델 등록 후 leaked_flag 문제로 실행 |

## 겪은 문제와 해결

- 설치 과정에서 오류 없음.
- 코드 분석 결과, v0.7은 Claude 3.5 Sonnet까지만 등록되어 있어 최신 모델 비교를 위해서는 모델 정보 추가 작업이 필요함 (다음 회차 진행).
