{
  "name": "ForgexAgent4",
  "description": "Dispatches drafted outreach emails via Gmail for READY_TO_SEND records and updates status in Supabase.",
  "active": false,
  "isArchived": false,
  "nodes": [
    {
      "parameters": {},
      "type": "n8n-nodes-base.manualTrigger",
      "typeVersion": 1,
      "position": [
        0,
        0
      ],
      "id": "6f56d555-6048-49a3-9f39-1524e814fa05",
      "name": "When clicking ‘Execute workflow’"
    },
    {
      "parameters": {
        "operation": "getAll",
        "tableId": "jobs",
        "filters": {
          "conditions": [
            {
              "keyName": "status",
              "condition": "eq",
              "keyValue": "READY_TO_SEND"
            }
          ]
        }
      },
      "type": "n8n-nodes-base.supabase",
      "typeVersion": 1,
      "position": [
        224,
        0
      ],
      "id": "2def23c8-176a-42c8-9c6b-730307b0a59d",
      "name": "Get many rows",
      "credentials": {
        "supabaseApi": {
          "id": "SUPABASE_CREDENTIAL_ID",
          "name": "Supabase account"
        }
      }
    },
    {
      "parameters": {
        "sendTo": "={{ $('Loop Over Items').item.json.contact_email }}",
        "subject": "Are you tired of having to hiree employees ",
        "message": "={{ $('Loop Over Items').item.json.outreach_draft }}",
        "options": {}
      },
      "type": "n8n-nodes-base.gmail",
      "typeVersion": 2.2,
      "position": [
        976,
        -160
      ],
      "id": "f7c47bb3-06fe-4039-9d33-b4a2d9f3f5a4",
      "name": "Send a message",
      "credentials": {
        "gmailOAuth2": {
          "id": "GMAIL_CREDENTIAL_ID",
          "name": "Gmail account"
        }
      }
    },
    {
      "parameters": {
        "options": {}
      },
      "type": "n8n-nodes-base.splitInBatches",
      "typeVersion": 3,
      "position": [
        560,
        -112
      ],
      "id": "475fc59a-541a-481e-a16e-cab0e863ebfb",
      "name": "Loop Over Items"
    },
    {
      "parameters": {
        "operation": "update",
        "tableId": "jobs",
        "matchType": "allFilters",
        "filters": {
          "conditions": [
            {
              "keyName": "id",
              "condition": "=eq",
              "keyValue": "={{ $('Loop Over Items').item.json.id }}"
            }
          ]
        },
        "fieldsUi": {
          "fieldValues": [
            {
              "fieldId": "status",
              "fieldValue": "Drafted"
            }
          ]
        }
      },
      "type": "n8n-nodes-base.supabase",
      "typeVersion": 1,
      "position": [
        1200,
        -160
      ],
      "id": "fb4bf98f-ae66-4b22-847e-abc63e8f33b7",
      "name": "Update a row",
      "credentials": {
        "supabaseApi": {
          "id": "SUPABASE_CREDENTIAL_ID",
          "name": "Supabase account"
        }
      }
    }
  ],
  "connections": {
    "When clicking ‘Execute workflow’": {
      "main": [
        [
          {
            "node": "Get many rows",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Get many rows": {
      "main": [
        [
          {
            "node": "Loop Over Items",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Send a message": {
      "main": [
        [
          {
            "node": "Update a row",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Loop Over Items": {
      "main": [
        [],
        [
          {
            "node": "Send a message",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Update a row": {
      "main": [
        [
          {
            "node": "Loop Over Items",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  },
  "settings": {
    "executionOrder": "v1",
    "binaryMode": "separate",
    "availableInMCP": false
  },
  "staticData": null,
  "meta": {
    "templateCredsSetupCompleted": true
  },
  "nodeGroups": [],
  "pinData": {},
  "tags": []
}