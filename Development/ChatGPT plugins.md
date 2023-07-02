# ChatGPT plugins

ChatGPT 플러그인이 나오면서 또 세상을 떠들썩하게하고 있는 것 같아서 찾아보았습니다.

원리라던지 그런 이야기는 찾아도 안 나와서, 진짜 지금 뭐가 유행 중인건지 어떻게 개발할 수 있는지? 이런 것들에 대해 적어보았습니다.

## Introduction

보고 이해한 바로는 "플러그인은 chatGPT가 대답을 생성할 때 활용할 수 있는 API를 제공한다" 인 것으로 이해했습니다.

예를 들어, 기존의 chatGPT는 "2023년 4월 1일 기준으로 젠지는 LCK 팀들 중 몇 등이야?" 라는 질문에 제대로 대답할 수 없는데요.

`실시간 LCK 순위를 제공하는 API`를 만들어두고 chatGPT 에게 제공하면, 결과를 토대로 답변을 생성할 수 있게 되는 것입니다.

즉, chatGPT의 가장 큰 단점 중 하나인 `실시간 데이터에 대한 답변이 불가능하다` 라는 단점이 해소됩니다.

## 기본 제공하는 Plugin

chatGPT 에서 기본적으로 제공하는 플러그인을 간단히 설명합니다.
제가 아직은 권한이 없기때문에, 예시로 나온 것들로만 구성하였습니다.

### Browsing

> Motivated by past work (our own WebGPT, as well as GopherCite, BlenderBot2, LaMDA2 and others), allowing language models to read information from the internet strictly expands the amount of content they can discuss, going beyond the training corpus to fresh information from the present day.

Bing 에서 제공하던 최신 검색결과를 기준으로 응답을 생성해주는 기능이 chatGPT에서도 가능해집니다.

![스크린샷 2023-04-01 13.30.33.png](/files/0a710dbd-8630-1145-8187-3b2e7bf604d7)


### Code Interpreter

이게 어떻게 가능한지는 모르겠는데,, python code 를 올리면 실행해준다고 하네요?
초기 테스트를 통해서 보았을 때, 아래 상황에서 유용하게 사용할 수 있다고 합니다.

- 수학적인 문제를 해결할 때
- 데이터 분석 및 시각화 작업
- 특정 파일들을 다양한 포맷으로 변경할 때 (테이블 -> json, 테이블 -> DB schema, ...)

![스크린샷 2023-04-01 13.32.28.png](/files/0a710dbd-8630-1145-8187-3b2e7bf204d6)

### Retrieval

> The open-source retrieval plugin enables ChatGPT to access personal or organizational information sources (with permission). It allows users to obtain the most relevant document snippets from their data sources, such as files, notes, emails or public documentation, by asking questions or expressing needs in natural language.

대충 이해해보았을 때는, personal or organizational sources 에서 자연어로 내가 원하는 파일/이메일/문서 등을 말하면 찾아주는 탐색기의 기능을 할 수 있는 것으로 이해했습니다.

예를 들면

`회사 메일 중에 내 파이낸셜 사번이 적혀있는 메일을 보여줘` 라고 질의하면 그에 맞는 메일을 저에게 응답하는.. 그런 방식으로 보입니다.

## 3rd party plugins

벌써 몇 가지의 서드파티 플러그인이 제공되고 있는 것으로 보입니다.

- WolframAlpha
    - 공학 계산기 플러그인
- OpenTable
    - 레스토랑 예약하는 플러그인
- instacart
    - 식료품 주문할 수 있는 플러그인

위 플러그인을 사용해서 
"내가 원하는 레스토랑에 예약하고, 거기서 특정 메뉴의 레시피를 조회한 뒤 해당 레시피의 칼로리를 계산하고, 해당 레시피를 토대로 재료들을 주문" 하는 것을 하나의 문장을 입력하여 처리할 수 있게 됩니다.

## 플러그인은 어떻게 만들 수 있는지?

필수요소는 아래 3가지인데요. 쉽게 말해, chatGPT 에게 API가 어떤 역할을 하는지 설명하는 파일들이라고 이해하시면 편할 것 같습니다.

- chatGPT에게 어떤 역할을 하는 플러그인인지 설명하는 Manifest 파일
- chatGPT가 호출할 수 있는 API 서버
- chatGPT가 해당 API를 이해할 수 있는 spec yaml 문서

특기할만 한 점은, manifest/spec 문서에는 `description` 필드가 있는데, 해당 값/API가 정확히 어떤 의미인지 "자연어"로 설명하는 부분이 있다는 점입니다....

아래 예시는 아래 링크에서 확인할 수 있습니다.
https://platform.openai.com/docs/plugins/examples

### Manifest 파일

~~~json
{
  "schema_version": "v1",
  "name_for_human": "Sport Stats",
  "name_for_model": "sportStats",
  "description_for_human": "Get current and historical stats for sport players and games.",
  "description_for_model": "Get current and historical stats for sport players and games. Always display results using markdown tables.", // chatGPT 에게 이 플러그인이 무엇을 할지, 어떻게 해야할지 설명하는 부분
  "auth": {
    "type": "none"
  },
  "api": {
    "type": "openapi",
    "url": "PLUGIN_HOSTNAME/openapi.yaml", // spec yaml 문서
    "is_user_authenticated": false
  },
  "logo_url": "PLUGIN_HOSTNAME/logo.png",
  "contact_email": "support@example.com",
  "legal_info_url": "https://example.com/legal"
}
~~~


재밌는 점은 name_for_human / name_for_model 처럼, model 에게 설명하는 부분이 따로 있다는 점인 것 같네요.

### API 서버

~~~python
import json
import requests
import urllib.parse

import quart
import quart_cors
from quart import request

# Note: Setting CORS to allow chat.openapi.com is required for ChatGPT to access your plugin
app = quart_cors.cors(quart.Quart(__name__), allow_origin="https://chat.openai.com")
HOST_URL = "https://example.com"

@app.get("/players")
async def get_players():
    query = request.args.get("query")
    res = requests.get(
        f"{HOST_URL}/api/v1/players?search={query}&page=0&per_page=100")
    body = res.json()
    return quart.Response(response=json.dumps(body), status=200)


@app.get("/teams")
async def get_teams():
    res = requests.get(
        "{HOST_URL}/api/v1/teams?page=0&per_page=100")
    body = res.json()
    return quart.Response(response=json.dumps(body), status=200)


@app.get("/games")
async def get_games():
    query_params = [("page", "0")]
    limit = request.args.get("limit")
    query_params.append(("per_page", limit or "100"))
    start_date = request.args.get("start_date")
    if start_date:
        query_params.append(("start_date", start_date))
    end_date = request.args.get("end_date")
    
    if end_date:
        query_params.append(("end_date", end_date))
    seasons = request.args.getlist("seasons")
    
    for season in seasons:
        query_params.append(("seasons[]", str(season)))
    team_ids = request.args.getlist("team_ids")
    
    for team_id in team_ids:
        query_params.append(("team_ids[]", str(team_id)))

    res = requests.get(
        f"{HOST_URL}/api/v1/games?{urllib.parse.urlencode(query_params)}")
    body = res.json()
    return quart.Response(response=json.dumps(body), status=200)


@app.get("/stats")
async def get_stats():
    query_params = [("page", "0")]
    limit = request.args.get("limit")
    query_params.append(("per_page", limit or "100"))
    start_date = request.args.get("start_date")
    if start_date:
        query_params.append(("start_date", start_date))
    end_date = request.args.get("end_date")
    
    if end_date:
        query_params.append(("end_date", end_date))
    player_ids = request.args.getlist("player_ids")
    
    for player_id in player_ids:
        query_params.append(("player_ids[]", str(player_id)))
    game_ids = request.args.getlist("game_ids")
    
    for game_id in game_ids:
        query_params.append(("game_ids[]", str(game_id)))
    res = requests.get(
        f"{HOST_URL}/api/v1/stats?{urllib.parse.urlencode(query_params)}")
    body = res.json()
    return quart.Response(response=json.dumps(body), status=200)


@app.get("/season_averages")
async def get_season_averages():
    query_params = []
    season = request.args.get("season")
    if season:
        query_params.append(("season", str(season)))
    player_ids = request.args.getlist("player_ids")
    
    for player_id in player_ids:
        query_params.append(("player_ids[]", str(player_id)))
    res = requests.get(
        f"{HOST_URL}/api/v1/season_averages?{urllib.parse.urlencode(query_params)}")
    body = res.json()
    return quart.Response(response=json.dumps(body), status=200)


@app.get("/logo.png")
async def plugin_logo():
    filename = 'logo.png'
    return await quart.send_file(filename, mimetype='image/png')


@app.get("/.well-known/ai-plugin.json")
async def plugin_manifest():
    host = request.headers['Host']
    with open("ai-plugin.json") as f:
        text = f.read()
        # This is a trick we do to populate the PLUGIN_HOSTNAME constant in the manifest
        text = text.replace("PLUGIN_HOSTNAME", f"https://{host}")
        return quart.Response(text, mimetype="text/json")


@app.get("/openapi.yaml")
async def openapi_spec():
    host = request.headers['Host']
    with open("openapi.yaml") as f:
        text = f.read()
        # This is a trick we do to populate the PLUGIN_HOSTNAME constant in the OpenAPI spec
        text = text.replace("PLUGIN_HOSTNAME", f"https://{host}")
        return quart.Response(text, mimetype="text/yaml")


def main():
    app.run(debug=True, host="0.0.0.0", port=5001)


if __name__ == "__main__":
    main()
~~~

길긴 한데, 그냥 기능 구현했다. 라고 이해했습니다.

### spec yaml 문서

"summary", "description" 이라는 파트가 재밌네요

/players API를 보게되면, query 파라미터를 어떻게 넣어줘야하는지, 어떤 값을 넣으면 어떤 응답값을 주는지 설명하고 있는데, 이를 chatGPT가 이해하고 적절히 호출해준다는 것으로 보입니다.

예를 들면, "Faker 에 대한 지표를 보여줘" 라고 하면, chatGPT는 문맥을 파악해서 /players API 호출이 필요한 것을 이해하고, query로 Faker 라는 문자열을 넣어서 호출하게 된다... 요런 흐름인 것 같습니다.

~~~yaml
openapi: 3.0.1
info:
  title: Sport Stats
  description: Get current and historical stats for sport players and games.
  version: 'v1'
servers:
  - url: PLUGIN_HOSTNAME
paths:
  /players:
    get:
      operationId: getPlayers
      summary: Retrieves all players from all seasons whose names match the query string.
      parameters:
      - in: query
        name: query
        schema:
            type: string
        description: Used to filter players based on their name. For example, ?query=davis will return players that have 'davis' in their first or last name.
      responses:
        "200":
          description: OK
  /teams:
    get:
      operationId: getTeams
      summary: Retrieves all teams for the current season.
      responses:
        "200":
          description: OK
  /games:
    get:
      operationId: getGames
      summary: Retrieves all games that match the filters specified by the args. Display results using markdown tables.
      parameters:
      - in: query
        name: limit
        schema:
            type: string
        description: The max number of results to return.
      - in: query
        name: seasons
        schema:
            type: array
            items:
              type: string
        description: Filter by seasons. Seasons are represented by the year they began. For example, 2018 represents season 2018-2019.
      - in: query
        name: team_ids
        schema:
            type: array
            items:
              type: string
        description: Filter by team ids. Team ids can be determined using the getTeams function.
      - in: query
        name: start_date
        schema:
            type: string
        description: A single date in 'YYYY-MM-DD' format. This is used to select games that occur on or after this date.
      - in: query
        name: end_date
        schema:
            type: string
        description: A single date in 'YYYY-MM-DD' format. This is used to select games that occur on or before this date.
      responses:
        "200":
          description: OK
  /stats:
    get:
      operationId: getStats
      summary: Retrieves stats that match the filters specified by the args. Display results using markdown tables.
      parameters:
      - in: query
        name: limit
        schema:
            type: string
        description: The max number of results to return.
      - in: query
        name: player_ids
        schema:
            type: array
            items:
              type: string
        description: Filter by player ids. Player ids can be determined using the getPlayers function.
      - in: query
        name: game_ids
        schema:
            type: array
            items:
              type: string
        description: Filter by game ids. Game ids can be determined using the getGames function.
      - in: query
        name: start_date
        schema:
            type: string
        description: A single date in 'YYYY-MM-DD' format. This is used to select games that occur on or after this date.
      - in: query
        name: end_date
        schema:
            type: string
        description: A single date in 'YYYY-MM-DD' format. This is used to select games that occur on or before this date.
      responses:
        "200":
          description: OK
  /season_averages:
    get:
      operationId: getSeasonAverages
      summary: Retrieves regular season averages for the given players. Display results using markdown tables.
      parameters:
      - in: query
        name: season
        schema:
            type: string
        description: Defaults to the current season. A season is represented by the year it began. For example, 2018 represents season 2018-2019.
      - in: query
        name: player_ids
        schema:
            type: array
            items:
              type: string
        description: Filter by player ids. Player ids can be determined using the getPlayers function.
      responses:
        "200":
          description: OK
~~~

## 테스터 신청하기

https://openai.com/waitlist/plugins