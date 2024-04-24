## Tinybird details 
* /game/index.html <title>
* /public/*.png


## /src/config.js

Setting auth tokens, URL host path and registering API Endpoints. 

```js
export const TINYBIRD_HOST = import.meta.env.VITE_TINYBIRD_HOST;
export const TINYBIRD_READ_TOKEN = import.meta.env.VITE_TINYBIRD_READ_TOKEN;
export const TINYBIRD_APPEND_TOKEN = import.meta.env.VITE_TINYBIRD_APPEND_TOKEN;

export const EVENTS_URL = `https://${TINYBIRD_HOST}/v0/events`;


export const endpoints = {
    top_10_url: new URL(
        `https://${TINYBIRD_HOST}/v0/pipes/api_leaderboard_mv.json`
    ),
    recent_player_stats_url: new URL(
        `https://${TINYBIRD_HOST}/v0/pipes/api_last_played_games_mv.json`
    ),
    player_stats_url: new URL(
        `https://${TINYBIRD_HOST}/v0/pipes/api_player_stats_mv.json`
    ),
    personalization_url: new URL(
        `https://${TINYBIRD_HOST}/v0/pipes/api_personalization_mv.json`
    ),
};
```


# /src/utils/tinybird.js

### tl;dr

```js
import { EVENTS_URL, TINYBIRD_READ_TOKEN, TINYBIRD_APPEND_TOKEN, EVENT_PARAM } from "../config";

export send_data_to_tinybird, get_data_from_tinybird, send_session_data, send_death, send_purchase
```

### The details

```js
import { EVENTS_URL, TINYBIRD_READ_TOKEN, TINYBIRD_APPEND_TOKEN, EVENT_PARAM } from "../config";

export async function send_data_to_tinybird(payload) {
    if (!TINYBIRD_READ_TOKEN) return;

    return fetch(`${EVENTS_URL}?name=events_api`, {
        method: "POST",
        body: JSON.stringify(payload),
        headers: {
            Authorization: `Bearer ${TINYBIRD_APPEND_TOKEN}`,
        },
    })
        .then((res) => res.json())
        .catch((error) => console.log(error));
}

export async function get_data_from_tinybird(url) {
    if (!TINYBIRD_READ_TOKEN) return;

    return fetch(url, {
        headers: {
            Authorization: `Bearer ${TINYBIRD_READ_TOKEN}`,
        },
    })
        .then((r) => r.json())
        .catch((e) => e.toString());
}

export async function send_session_data(session) {
    if (!TINYBIRD_APPEND_TOKEN) return;

    const payload = {
        session_id: session.id,
        name: session.name,
        timestamp: new Date().toISOString(),
        type: "score",
        event: EVENT_PARAM
    };
    return send_data_to_tinybird(payload);
}

export async function send_death(session) {
    if (!TINYBIRD_READ_TOKEN) return;

    const payload = {
        session_id: session.id,
        name: session.name,
        timestamp: new Date().toISOString(),
        type: "game_over",
        event: EVENT_PARAM
    };
    return send_data_to_tinybird(payload);
}

export async function send_purchase(session) {
    if (!TINYBIRD_READ_TOKEN) return;

    const payload = {
        session_id: session.id,
        name: session.name,
        timestamp: new Date().toISOString(),
        type: "purchase",
        event: EVENT_PARAM
    };
    return send_data_to_tinybird(payload);
}

``


# Glue

```js
{
  "name": "flappy-tinybird",
  "description": "This repository contains a clone of the popular game 'Flappy Bird'. It was built using the Phaser 3 game framework and JavaScript.",
  "private": false,
  "version": "0.0.1",
  "type": "module",
  "dependencies": {
    "@tinybirdco/mockingbird": "^1.1.4",
  }
}

```




