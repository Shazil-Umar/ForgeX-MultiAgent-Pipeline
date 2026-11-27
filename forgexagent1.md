{
  "name": "ForgexAgent1",
  "description": "Ingests remote job postings from Remotive API and inserts new records into Supabase.",
  "active": false,
  "isArchived": false,
  "nodes": [
    {
      "parameters": {
        "url": "https://remotive.com/api/remote-jobs?limit=100",
        "options": {}
      },
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.5,
      "position": [
        144,
        144
      ],
      "id": "89b707a1-0f58-46c6-9439-2674ac20c005",
      "name": "HTTP Request"
    },
    {
      "parameters": {
        "fieldToSplitOut": "jobs",
        "options": {}
      },
      "type": "n8n-nodes-base.splitOut",
      "typeVersion": 1,
      "position": [
        368,
        144
      ],
      "id": "c17b4036-94a8-43cf-8d39-6461d2af60e6",
      "name": "Split Out"
    },
    {
      "parameters": {
        "tableId": "jobs",
        "fieldsUi": {
          "fieldValues": [
            {
              "fieldId": "job_title",
              "fieldValue": "={{ $json.title }}"
            },
            {
              "fieldId": "job_url",
              "fieldValue": "={{ $json.url }}"
            },
            {
              "fieldId": "job_description",
              "fieldValue": "={{ $json.description }}"
            },
            {
              "fieldId": "technical_category",
              "fieldValue": "={{ $json.category }}"
            },
            {
              "fieldId": "location",
              "fieldValue": "={{ $json.candidate_required_location }}"
            },
            {
              "fieldId": "source",
              "fieldValue": "Remotive"
            },
            {
              "fieldId": "status",
              "fieldValue": "NEW"
            },
            {
              "fieldId": "remote_status",
              "fieldValue": "Remote"
            },
            {
              "fieldId": "company_name",
              "fieldValue": "={{ $('HTTP Request').item.json.jobs[0].company_name }}"
            }
          ]
        }
      },
      "type": "n8n-nodes-base.supabase",
      "typeVersion": 1,
      "position": [
        672,
        144
      ],
      "id": "8bbff413-d2a8-47ec-99a8-902ca90cb930",
      "name": "Create a row",
      "credentials": {
        "supabaseApi": {
          "id": "SUPABASE_CREDENTIAL_ID",
          "name": "Supabase account"
        }
      },
      "onError": "continueRegularOutput"
    },
    {
      "parameters": {},
      "type": "n8n-nodes-base.manualTrigger",
      "typeVersion": 1,
      "position": [
        -32,
        144
      ],
      "id": "7991a66e-d578-498b-b60a-d935d0041753",
      "name": "When clicking ‘Execute workflow’"
    }
  ],
  "connections": {
    "HTTP Request": {
      "main": [
        [
          {
            "node": "Split Out",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Split Out": {
      "main": [
        [
          {
            "node": "Create a row",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "When clicking ‘Execute workflow’": {
      "main": [
        [
          {
            "node": "HTTP Request",
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