# Instagram Mentions to Notion - n8n Workflow

Here's a complete, valid n8n workflow JSON that implements all your requirements:

```json
{
  "name": "Instagram Mentions to Notion",
  "nodes": [
    {
      "parameters": {
        "rule": {
          "interval": [
            {
              "field": "minutes",
              "minutesInterval": 15
            }
          ]
        }
      },
      "id": "a1b2c3d4-1111-4000-8000-000000000001",
      "name": "Schedule Trigger",
      "type": "n8n-nodes-base.scheduleTrigger",
      "typeVersion": 1.2,
      "position": [240, 300]
    },
    {
      "parameters": {
        "method": "GET",
        "url": "=https://graph.instagram.com/me?fields=mentioned_media{media_url,timestamp,username}&access_token={{ $env.INSTAGRAM_ACCESS_TOKEN }}",
        "options": {
          "response": {
            "response": {
              "responseFormat": "json"
            }
          }
        }
      },
      "id": "a1b2c3d4-2222-4000-8000-000000000002",
      "name": "HTTP Request - Get Mentions",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.2,
      "position": [480, 300]
    },
    {
      "parameters": {
        "fieldToSplitOut": "mentioned_media.data",
        "options": {}
      },
      "id": "a1b2c3d4-3333-4000-8000-000000000003",
      "name": "Split Out Mentions Array",
      "type": "n8n-nodes-base.splitOut",
      "typeVersion": 1,
      "position": [720, 300]
    },
    {
      "parameters": {
        "batchSize": 1,
        "options": {}
      },
      "id": "a1b2c3d4-4444-4000-8000-000000000004",
      "name": "Split In Batches",
      "type": "n8n-nodes-base.splitInBatches",
      "typeVersion": 3,
      "position": [960, 300]
    },
    {
      "parameters": {
        "mode": "manual",
        "duplicateItem": false,
        "assignments": {
          "assignments": [
            {
              "id": "username-field",
              "name": "Username",
              "value": "={{ $json.username }}",
              "type": "string"
            },
            {
              "id": "timestamp-field",
              "name": "Timestamp",
              "value": "={{ $json.timestamp }}",
              "type": "string"
            },
            {
              "id": "storyurl-field",
              "name": "Story URL",
              "value": "={{ $json.media_url }}",
              "type": "string"
            }
          ]
        },
        "options": {
          "includeBinary": false
        }
      },
      "id": "a1b2c3d4-5555-4000-8000-000000000005",
      "name": "Set - Extract Fields",
      "type": "n8n-nodes-base.set",
      "typeVersion": 3.4,
      "position": [1200, 300]
    },
    {
      "parameters": {
        "resource": "databasePage",
        "operation": "create",
        "databaseId": {
          "__rl": true,
          "value": "YOUR_NOTION_DATABASE_ID",
          "mode": "id"
        },
        "propertiesUi": {
          "propertyValues": [
            {
              "key": "Username|title",
              "title": "={{ $json.Username }}"
            },
            {
              "key": "Timestamp|date",
              "date": "={{ $json.Timestamp }}"
            },
            {
              "key": "Story URL|url",
              "url": "={{ $json['Story URL'] }}"
            }
          ]
        },
        "options": {}
      },
      "id": "a1b2c3d4-6666-4000-8000-000000000006",
      "name": "Notion - Create Page",
      "type": "n8n-nodes-base.notion",
      "typeVersion": 2.2,
      "position": [1440, 300],
      "credentials": {
        "notionApi": {
          "id": "YOUR_NOTION_CREDENTIAL_ID",
          "name": "Notion Account"
        }
      }
    }
  ],
  "connections": {
    "Schedule Trigger": {
      "main": [
        [
          {
            "node": "HTTP Request - Get Mentions",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "HTTP Request - Get Mentions": {
      "main": [
        [
          {
            "node": "Split Out Mentions Array",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Split Out Mentions Array": {
      "main": [
        [
          {
            "node": "Split In Batches",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Split In Batches": {
      "main": [
        [
          {
            "node": "Set - Extract Fields",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Set - Extract Fields": {
      "main": [
        [
          {
            "node": "Notion - Create Page",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Notion - Create Page": {
      "main": [
        [
          {
            "node": "Split In Batches",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  },
  "active": false,
  "settings": {
    "executionOrder": "v1"
  },
  "versionId": "1",
  "meta": {
    "instanceId": "your-n8n-instance-id"
  },
  "id": "instagram-mentions-to-notion",
  "tags": []
}
```

## Workflow Diagram

```
┌──────────────────┐     ┌─────────────────────────┐     ┌────────────────────────┐
│ Schedule Trigger │────▶│ HTTP Request -          │────▶│ Split Out Mentions     │
│ (Every 15 mins)  │     │ Get Mentions            │     │ Array                  │
└──────────────────┘     └─────────────────────────┘     └────────────────────────┘
                                                                    │
                                                                    ▼
┌──────────────────┐     ┌─────────────────────────┐     ┌────────────────────────┐
│ Notion -         │◀────│ Set - Extract Fields    │◀────│ Split In Batches       │
│ Create Page      │     │                         │     │ (batchSize: 1)         │
└──────────────────┘     └─────────────────────────┘     └────────────────────────┘
         │                                                          ▲
         └──────────────────────────────────────────────────────────┘
                              (Loop back for next batch)
```

## Setup Requirements

### 1. Environment Variable
Set this in your n8n environment:
```
INSTAGRAM_ACCESS_TOKEN=your_instagram_graph_api_token
```

### 2. Notion Database Setup
Create a Notion database named **"Influencer Mentions"** with these properties:

| Property Name | Property Type |
|--------------|---------------|
| Username     | Title         |
| Timestamp    | Date          |
| Story URL    | URL           |

### 3. Replace Placeholder Values

| Placeholder | Replace With |
|-------------|--------------|
| `YOUR_NOTION_DATABASE_ID` | Your Notion database ID (32-character string from database URL) |
| `YOUR_NOTION_CREDENTIAL_ID` | Your n8n Notion credential ID |

### 4. Credentials Required
- **Notion API**: Create credentials in n8n with your Notion integration token

## Node Details

| # | Node Name | Purpose |
|---|-----------|---------|
| 1 | **Schedule Trigger** | Runs workflow every 15 minutes |
| 2 | **HTTP Request - Get Mentions** | Fetches Instagram mentions via Graph API |
| 3 | **Split Out Mentions Array** | Extracts `mentioned_media.data` array into separate items |
| 4 | **Split In Batches** | Processes one mention at a time |
| 5 | **Set - Extract Fields** | Renames fields: `username→Username`, `timestamp→Timestamp`, `media_url→Story URL` |
| 6 | **Notion - Create Page** | Creates new page in Notion database |

## Import Instructions

1. Copy the JSON above
2. Open n8n
3. Click **"..."** menu → **"Import from File or URL"**
4. Paste the JSON
5. Update the placeholder values
6. Set up credentials
7. Activate the workflow
