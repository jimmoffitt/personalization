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

# Implementing Phaser game 

## Implementing Phaser Sceen class

Here is some selected code to illustrate the methods and attributes implemented here. 

```js
    update() {
        if (this.timerStarted) {
            this.updateBird();
        }
    }

    handleOffer(r) {
        this.offer = r?.data?.[0]?.offer ?? 0;
    }

    getDataFromTinybird() {
        endpoints.personalization_url.searchParams.set(
            "player_param",
            this.session.name
        );

        return Promise.all([
            get_data_from_tinybird(endpoints.personalization_url)
                .then((r) => this.handleOffer(r))
        ]);
    }

    showAd() {

        this.getDataFromTinybird();

                if (this.offer == 1) {
                    this.scene.start("DealScene", data);
                } else {
                    this.scene.start("EndGameScene", data);
                }
    }

    async endGame() {
        this.timer.remove();
        send_death(this.session);
        this.scoreText.destroy();

        // Use a timer event to wait for 2 seconds
        this.time.delayedCall(2000, async () => {
            gameOver.destroy();
            gameOverScore.destroy();
            this.showAd();
        });
    }

    updateBird() {
        if (this.bird.angle < 30) {
            this.bird.angle += 2;
        }

        // Check if the bird's top edge is above the top of the window
        if (this.bird.y <= 0) {
            this.endGame();
        }

        // Check if the bird's bottom edge is below the bottom of the window
        if (this.bird.y + (this.bird.height * this.bird.scaleX) >= this.cameras.main.height) {
            this.endGame();
        }

        // Additionally, check for collision with pipes
        if (this.physics.overlap(this.bird, this.pipes)) {
            this.endGame();
        }
    }
}

```






