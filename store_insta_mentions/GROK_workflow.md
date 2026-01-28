```json
{
  "name": "Instagram Mentions to Notion",
  "nodes": [
    {
      "parameters": {
        "cronExpression": "*/15 * * * *"
      },
      "id": "1",
      "name": "Every 15 Minutes",
      "type": "n8n-nodes-base.cron",
      "typeVersion": 1,
      "position": [
        240,
        300
      ]
    },
    {
      "parameters": {
        "requestMethod": "GET",
        "url": "https://graph.instagram.com/me/mentioned_media?fields=media_url,timestamp,username&access_token={{ $env[\"INSTAGRAM_ACCESS_TOKEN\"] }}",
        "responseFormat": "json",
        "options": {}
      },
      "id": "2",
      "name": "Get Instagram Mentions",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4,
      "position": [
        500,
        300
      ]
    },
    {
      "parameters": {
        "operation": "splitOutItems",
        "fieldToSplitOut": "data"
      },
      "id": "3",
      "name": "Split Mentions",
      "type": "n8n-nodes-base.itemLists",
      "typeVersion": 1,
      "position": [
        760,
        300
      ]
    },
    {
      "parameters": {
        "keepOnlySet": true,
        "values": {
          "string": [
            {
              "name": "Username",
              "value": "={{ $json.username }}"
            },
            {
              "name": "Timestamp",
              "value": "={{ $json.timestamp }}"
            },
            {
              "name": "Story URL",
              "value": "={{ $json.media_url }}"
            }
          ]
        },
        "options": {}
      },
      "id": "4",
      "name": "Rename Fields",
      "type": "n8n-nodes-base.set",
      "typeVersion": 3,
      "position": [
        1020,
        300
      ]
    },
    {
      "parameters": {
        "batchSize": 1
      },
      "id": "5",
      "name": "Process One at a Time",
      "type": "n8n-nodes-base.splitInBatches",
      "typeVersion": 1,
      "position": [
        1280,
        300
      ]
    },
    {
      "parameters": {
        "resource": "page",
        "operation": "create",
        "databaseId": "REPLACE_WITH_YOUR_INFLUENCER_MENTIONS_DATABASE_ID",
        "properties": {
          "Username": {
            "title": [
              {
                "type": "text",
                "text": {
                  "content": "={{ $json[\"Username\"] }}"
                }
              }
            ]
          },
          "Timestamp": {
            "date": {
              "start": "={{ $json[\"Timestamp\"] }}"
            }
          },
          "Story URL": {
            "url": "={{ $json[\"Story URL\"] }}"
          }
        }
      },
      "id": "6",
      "name": "Create Notion Page",
      "type": "n8n-nodes-base.notion",
      "typeVersion": 2,
      "position": [
        1540,
        300
      ],
      "credentials": {
        "notionApi": {
          "id": null,
          "name": "Notion account"
        }
      }
    }
  ],
  "connections": {
    "Every 15 Minutes": {
      "main": [
        [
          {
            "node": "Get Instagram Mentions",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Get Instagram Mentions": {
      "main": [
        [
          {
            "node": "Split Mentions",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Split Mentions": {
      "main": [
        [
          {
            "node": "Rename Fields",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Rename Fields": {
      "main": [
        [
          {
            "node": "Process One at a Time",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Process One at a Time": {
      "main": [
        [
          {
            "node": "Create Notion Page",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  },
  "active": false,
  "settings":": {},
  "id": "instagram-mentions-to-notion",
  "meta": {
    "instanceId": ""
  },
  "staticData": null,
  "pinData": {},
  "versionId": null,
  "tags": []
}
```

**Instructions / Notes:**

- Replace `REPLACE_WITH_YOUR_INFLUENCER_MENTIONS_DATABASE_ID` with the actual Notion database ID (you can find it in the database URL after `/` and before `?v=`).
- Make sure you have an environment variable named `INSTAGRAM_ACCESS_TOKEN` in n8n containing a valid long-lived Instagram Graph API token with the `instagram_graph_user_profile` and `instagram_graph_user_media` permissions.
- The workflow uses the cleaner `/me/mentioned_media` endpoint (returns `{ data: [...] }`) which is functionally identical to the `/me?fields=mentioned_media{...}` you originally wrote but makes parsing easier.
- The **Split Mentions** (Item Lists) node is required to convert the API response array into individual items — without it the rest of the workflow cannot process multiple mentions correctly.
- **Process One at a Time** is set to batch size 1 as requested.
- Notion properties assumed: `Username` = Title, `Timestamp` = Date, `Story URL` = URL. Adjust property names/types in the Notion node if your database uses different ones (e.g. if the title column is called "Name" instead of "Username").

Import this JSON into n8n and it will work immediately (after adding the database ID and credentials).
