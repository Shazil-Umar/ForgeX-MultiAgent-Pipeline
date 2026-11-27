{
  "name": "ForgexAgent3",
  "description": "Enriches high-scoring leads via Apollo.io and generates personalized cold outreach copy using Gemini.",
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
      "id": "f086e468-2b33-485d-a0e1-e7482a2e055f",
      "name": "When clicking ‘Execute workflow’"
    },
    {
      "parameters": {
        "operation": "getAll",
        "tableId": "jobs",
        "limit": 4,
        "matchType": "allFilters",
        "filters": {
          "conditions": [
            {
              "keyName": "ai_score",
              "condition": "gt",
              "keyValue": "6"
            },
            {
              "keyName": "status",
              "condition": "eq",
              "keyValue": "PROCESSED"
            }
          ]
        }
      },
      "type": "n8n-nodes-base.supabase",
      "typeVersion": 1,
      "position": [
        208,
        -208
      ],
      "id": "9ebd51c2-d387-4c10-ac63-92a615992a39",
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
        "options": {}
      },
      "type": "n8n-nodes-base.splitInBatches",
      "typeVersion": 3,
      "position": [
        448,
        0
      ],
      "id": "c6fc2d2a-c6eb-44ee-9f0f-ea311f92921b",
      "name": "Loop Over Items"
    },
    {
      "parameters": {
        "modelId": {
          "__rl": true,
          "mode": "list",
          "value": "models/gemini-3-flash-preview"
        },
        "messages": {
          "values": [
            {
              "content": "=You are an expert technical copywriter drafting a highly personalized, cold outreach email for a software engineering job application. \n\nThe candidate's core technical profile:\n* Core: Java (OOP, Swing GUIs, exception handling, object I/O), C++ (pointers, references), Python.\n* Databases: Relational design, 3NF normalization, Microsoft SQL Server Management Studio (SSMS), Microsoft Access.\n* Infrastructure & API: n8n workflow automation, Supabase database integration, local LLM configuration (Ollama, Qwen2.5-Coder).\n* Hardware: ESP32 microcontrollers, OLED displays, digital logic design (multiplexers, flip-flops, CircuitVerse).\n* Recent Architecture: Built an airline management system with a Java Swing UI and normalized SQL backend; developed a smart parking management system using digital logic counters.\n\nYour task:\nRead the job description and draft a concise outreach email to the hiring manager. \n1. Map the candidate's specific skills to the job's requirements. Emphasize backend automation, Java, SQL, and LLM configuration where relevant.\n2. Keep it under 150 words. Be confident and direct.\n3. Output ONLY the email body. Start directly with a greeting.\n\nCompany: {{ $('Loop Over Items').item.json.company_name }}\nRole: {{ $('Loop Over Items').item.json.job_title }}\nTech Stack: {{ $('Loop Over Items').item.json.tech_stack }}\nJob Description: {{ $('Loop Over Items').item.json.job_description }}"
            }
          ]
        },
        "builtInTools": {},
        "options": {}
      },
      "type": "@n8n/n8n-nodes-langchain.googleGemini",
      "typeVersion": 1.2,
      "position": [
        1152,
        0
      ],
      "id": "00826739-dfa3-4e3d-b05d-6215e82fafe9",
      "name": "Message a model",
      "retryOnFail": true,
      "waitBetweenTries": 2000,
      "maxTries": 4,
      "credentials": {
        "googlePalmApi": {
          "id": "GEMINI_CREDENTIAL_ID",
          "name": "Google Gemini(PaLM) Api account"
        }
      }
    },
    {
      "parameters": {
        "operation": "update",
        "tableId": "jobs",
        "filters": {
          "conditions": [
            {
              "keyName": "id",
              "condition": "eq",
              "keyValue": "={{ $('Loop Over Items').item.json.id }}"
            }
          ]
        },
        "fieldsUi": {
          "fieldValues": [
            {
              "fieldId": "outreach_draft",
              "fieldValue": "={{ $json.text }}"
            },
            {
              "fieldId": "status",
              "fieldValue": "READY_TO_SEND"
            },
            {
              "fieldId": "contact_name",
              "fieldValue": "={{ $('HTTP Request').item.json.people[0].name"
            },
            {
              "fieldId": "=contact_email",
              "fieldValue": "={{ $('HTTP Request').item.json.people[0].email"
            },
            {
              "fieldId": "=contact_linkedin",
              "fieldValue": "={{ $('HTTP Request').item.json.people[0].linkedin_url"
            },
            {
              "fieldId": "outreach_draft",
              "fieldValue": "={{ $json.content.parts[0].text }}"
            }
          ]
        }
      },
      "type": "n8n-nodes-base.supabase",
      "typeVersion": 1,
      "position": [
        1520,
        0
      ],
      "id": "58331324-c38f-48a7-8094-521363a0e611",
      "name": "Update a row",
      "credentials": {
        "supabaseApi": {
          "id": "SUPABASE_CREDENTIAL_ID",
          "name": "Supabase account"
        }
      }
    },
    {
      "parameters": {
        "amount": 65
      },
      "type": "n8n-nodes-base.wait",
      "typeVersion": 1.1,
      "position": [
        1744,
        0
      ],
      "id": "5e269100-559d-4eaf-b179-927037654910",
      "name": "Wait"
    },
    {
      "parameters": {
        "method": "POST",
        "url": "https://api.apollo.io/api/v1/mixed_people/api_search",
        "sendHeaders": true,
        "headerParameters": {
          "parameters": [
            {
              "name": "X-Api-Key",
              "value": "YOUR_APOLLO_API_KEY"
            }
          ]
        },
        "sendBody": true,
        "specifyBody": "json",
        "jsonBody": "={\n  \"q_organization_domains\": \"{{ $json.company_name }}\",\n  \"person_titles\": [\"Founder\", \"Co-Founder\", \"CEO\", \"CTO\", \"Head of Engineering\", \"Recruiter\", \"Talent Acquisition\", \"Engineering Manager\"],\n  \"per_page\": 1\n}",
        "options": {}
      },
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.5,
      "position": [
        864,
        0
      ],
      "id": "194561bd-fd3f-4d83-96bc-0a6952174bd4",
      "name": "HTTP Request"
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
    "Loop Over Items": {
      "main": [
        [],
        [
          {
            "node": "HTTP Request",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Message a model": {
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
    "Update a row": {
      "main": [
        [
          {
            "node": "Wait",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Wait": {
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
    "HTTP Request": {
      "main": [
        [
          {
            "node": "Message a model",
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