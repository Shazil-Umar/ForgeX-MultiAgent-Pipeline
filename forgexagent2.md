{
  "name": "ForgexAgent2",
  "description": "Evaluates and scores NEW job postings using Gemini and updates qualification metadata in Supabase.",
  "active": false,
  "isArchived": false,
  "nodes": [
    {
      "parameters": {
        "operation": "getAll",
        "tableId": "jobs",
        "filters": {
          "conditions": [
            {
              "keyName": "status",
              "condition": "eq",
              "keyValue": "NEW"
            }
          ]
        }
      },
      "type": "n8n-nodes-base.supabase",
      "typeVersion": 1,
      "position": [
        -208,
        -224
      ],
      "id": "293ca157-adc2-4fb4-b9b1-52acf9c04b86",
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
        "operation": "update",
        "tableId": "jobs",
        "filters": {
          "conditions": [
            {
              "keyName": "id",
              "condition": "eq",
              "keyValue": "={{ $('Get many rows').item.json.id }}"
            }
          ]
        },
        "fieldsUi": {
          "fieldValues": [
            {
              "fieldId": "status",
              "fieldValue": "Processed"
            },
            {
              "fieldId": "ai_score",
              "fieldValue": "={{ JSON.parse($json.content.parts[0].text).score }}"
            },
            {
              "fieldId": "ai_summary",
              "fieldValue": "={{ JSON.parse($json.content.parts[0].text).summary }}"
            },
            {
              "fieldId": "tech_stack",
              "fieldValue": "={{ JSON.parse($json.content.parts[0].text).tech_stack }}"
            }
          ]
        }
      },
      "type": "n8n-nodes-base.supabase",
      "typeVersion": 1,
      "position": [
        432,
        -144
      ],
      "id": "336c2da0-9535-4530-bbfa-21383b781031",
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
        "modelId": {
          "__rl": true,
          "value": "models/gemini-3-flash-preview",
          "mode": "list",
          "cachedResultName": "models/gemini-3-flash-preview"
        },
        "messages": {
          "values": [
            {
              "content": "=You are an expert technical recruiter and B2B lead qualification agent for an HR outsourcing and technical contract staffing company. Your objective is to analyze job postings and determine whether they represent a qualified prospect for technical recruiting/staffing services. The target market is BROAD technical hiring, including: Software Engineering, Backend, Frontend, Full Stack, Mobile, Embedded, Cloud, DevOps, Infrastructure, Platform, SRE, Solutions Architecture, Cybersecurity/InfoSec, Application Security, IAM, SOC, Detection, Incident Response, GRC, Red Team, Blue Team, DevSecOps, AI, Machine Learning, LLM, MLOps, Data Engineering, Data Science, QA Automation, Networking, Database Engineering, Firmware, Robotics, and other genuine technical engineering roles.If the hiring company name is explicitly stated in the job posting text, extract it. If the text only mentions a third-party marketplace or client name (like Lemon.io) but the actual direct employer is anonymous or unclear, look at the job URL or default to 'Unknown' rather than falsely attributing it to an intermediary platform.\"\n\n### 1. HARD REJECTION RULES\nApply these rules BEFORE scoring. The final qualification status must be \"REJECTED\" if any of these are true:\n* Employee Count: Reject if the company has more than 100 employees. If the employee range is \"51-200\", \"201-500\", etc., reject because the upper bound exceeds 100.\n* Company Type: Reject if the company's primary business is staffing, recruiting, recruitment agency, executive search, headhunting, talent placement, contract staffing, IT staffing, RPO, HR outsourcing, EOR, PEO, workforce outsourcing, or similar recruitment services.\n* Investment Firms: Reject if the company is primarily Venture Capital, Private Equity, an investment firm, angel investment firm, holding company, accelerator, or incubator.\n* Role Type: Reject if the role is clearly non-technical, unpaid, hybrid with mandatory office attendance, or strictly on-site.\n* Exception: Do NOT reject a company merely because it contains HR, workforce, talent, or hiring terminology. Reject only when recruitment/staffing is its PRIMARY business (e.g., HR SaaS or workforce-management software product companies remain eligible).\n\n### 2. REMOTE CLASSIFICATION\nUse supplied remote metadata as the primary signal and inspect the job description for contradictions. \n* REMOTE: The role is fully remote, remote-first without mandatory office attendance, or explicitly work-from-home.\n* HYBRID: Office attendance is partially required.\n* ON_SITE: Office attendance is fully required.\n* UNKNOWN: Remote status cannot be reliably established. Never assume UNKNOWN is remote.\n\n### 3. MISSING DATA (VERIFICATION REQUIRED)\nIf company employee count, company type, or another hard-filter attribute is completely unavailable, do not invent it. Mark the lead as \"NEEDS_VERIFICATION\" rather than automatically qualifying it.\n\n### 4. TECHNOLOGY EXTRACTION\nExtract only technologies explicitly present or clearly supported by the job description. Do not hallucinate. Match technology relevance according to the role:\n* Backend: Java, JavaScript, Node.js, Express, PostgreSQL, Docker, APIs, databases, microservices.\n* Cloud/DevOps: AWS, Azure, GCP, Docker, Kubernetes, Terraform, CI/CD, Linux, networking, observability.\n* Cybersecurity: IAM, SIEM, SOC, EDR, cloud/app security, threat detection, incident response, zero trust.\n* AI/ML: Python, PyTorch, TensorFlow, LLMs, NLP, vector databases, MLOps, GPUs.\n* Data: SQL, PostgreSQL, Python, Spark, Kafka, dbt, Snowflake, BigQuery, Databricks, ETL/ELT.\n* Frontend: JavaScript, TypeScript, React, Vue, Angular, HTML/CSS.\n* Original Target Stack Bonus: Note if Java, JavaScript, Express, PostgreSQL, C++, or Docker are present.\n\n### 5. SCORING SYSTEM (1 to 10)\nScore the job opportunity independently of company qualification (a company can have a 9/10 job but still be REJECTED because it is a staffing firm). Cap the final score between 1 and 10.\n* Start at 5 points.\n* +3 for Cybersecurity/InfoSec/Security Engineering.\n* +3 for Backend Engineering.\n* +2 for Cloud/Infrastructure/DevOps/Platform/SRE.\n* +2 for AI/ML/Data Engineering.\n* +2 for B2B SaaS.\n* +2 for Cybersecurity/InfoSec industry.\n* +1 if Docker, Kubernetes, AWS, Azure, or GCP is explicitly required.\n* +1 if Java, JavaScript, Node.js, Express, PostgreSQL, or C++ is explicitly required.\n* +1 for clear evidence of active technical hiring, team growth, or multiple openings.\n* +1 for a genuine technology product/company.\n* -2 if the technical requirements are unusually specialized and unlikely to be suitable for general technical staffing.\n* -3 for primarily non-technical work.\n* -3 for primarily sales, marketing, HR, recruiting, customer support, or administrative work.\n* -5 for exclusively frontend work.\n* -5 for unpaid work.\n\n### 6. QUALIFICATION STATUS DEFINITIONS\n* QUALIFIED = All hard requirements are satisfied.\n* REJECTED = At least one hard requirement definitely fails. (Keep the genuine job score, but provide the exact rejection reason).\n* NEEDS_VERIFICATION = The job appears suitable, but a required company-level condition (like employee count) cannot be verified.\n\n### 1. HARD REJECTION RULES\nApply these rules BEFORE scoring. The final qualification status must be \"REJECTED\" and the score must be forced to 1 if any of these are true:\n* Employee Count: Reject if the company has more than 100 employees. If the employee range is \"51-200\", \"201-500\", etc., reject because the upper bound exceeds 100.\n* Company Type: Reject if the company's primary business is staffing, recruiting, recruitment agency, executive search, headhunting, talent placement, contract staffing, IT staffing, IT outsourcing, software consultancy agency, RPO, HR outsourcing, EOR, PEO, workforce outsourcing, or similar recruitment/outsourcing services.\n* Investment Firms: Reject if the company is primarily Venture Capital, Private Equity, an investment firm, angel investment firm, holding company, accelerator, or incubator.\n* Role Type: Reject if the role is clearly non-technical, unpaid, hybrid with mandatory office attendance, or strictly on-site.\n\n### 5. SCORING SYSTEM (1 to 10)\nScore the job opportunity independently of company qualification EXCEPT for staffing/outforcing agencies. Cap the final score between 1 and 10.\n* Start at 5 points.\n* +3 for Cybersecurity/InfoSec/Security Engineering.\n* +3 for Backend Engineering.\n* +2 for Cloud/Infrastructure/DevOps/Platform/SRE.\n* +2 for AI/ML/Data Engineering.\n* +2 for B2B SaaS.\n* +1 if Docker, Kubernetes, AWS, Azure, or GCP is explicitly required.\n* +1 if Java, JavaScript, Node.js, Express, PostgreSQL, or C++ is explicitly required.\n* +1 for clear evidence of active technical hiring, team growth, or multiple openings.\n* +1 for a genuine technology product/company.\n* -2 if the technical requirements are unusually specialized and unlikely to be suitable for general technical staffing.\n* -3 for primarily non-technical work.\n* -3 for primarily sales, marketing, HR, recruiting, customer support, or administrative work.\n* -5 for exclusively frontend work.\n* -5 for unpaid work.\n* -10 for ANY staffing agency, IT outsourcing company, recruitment firm, or software consultancy shop (Overrides all positive points to force score to 1).\n\n### 7. JSON OUTPUT REQUIREMENT\nReturn ONLY a raw, valid JSON object. Do not include markdown code blocks, backticks, commentary, or trailing commas. Use exactly this schema:\n\n{\n  \"qualification\": \"QUALIFIED | REJECTED | NEEDS_VERIFICATION\",\n  \"score\": 9,\n  \"role_family\": \"Backend Engineering\",\n  \"remote_status\": \"REMOTE\",\n  \"company_type\": \"B2B SaaS\",\n  \"employee_count\": \"11-50\",\n  \"company_name\": \"Extracted official name of the hiring company (e.g., Lemon.io)\",\n  \"company_domain\": \"Clean root domain of the hiring company without https:// or www. (e.g., lemon.io)\",\n  \"rejection_reason\": \"Provide reason if REJECTED, otherwise null\",\n  \"verification_required\": false,\n  \"summary\": \"1-2 sentences explaining the decision.\",\n  \"tech_stack\": \"Comma-separated list of extracted tech\"\n}\n\n### DATA INPUTS\nCompany Name: {{ $json.company_name || 'Unknown' }}\nCompany Employee Count: {{ $json.employee_count || 'Unknown' }}\nCompany Industry: {{ $json.company_industry || $json.technical_category || 'Unknown' }}\nCompany Description: {{ $json.company_description || 'Not provided' }}\nJob Title: {{ $json.job_title || 'Unknown' }}\nJob Description: {{ $json.job_description || 'Not provided' }}\nJob Location: {{ $json.location || 'Unknown' }}\nRemote Status: {{ $json.remote_status || 'Unknown' }}"
            }
          ]
        },
        "jsonOutput": true,
        "builtInTools": {},
        "options": {}
      },
      "type": "@n8n/n8n-nodes-langchain.googleGemini",
      "typeVersion": 1.2,
      "position": [
        208,
        -336
      ],
      "id": "d9552ef2-7609-42a8-981f-08143769d275",
      "name": "Message a model",
      "retryOnFail": true,
      "waitBetweenTries": 5000,
      "credentials": {
        "googlePalmApi": {
          "id": "GEMINI_CREDENTIAL_ID",
          "name": "Google Gemini(PaLM) Api account"
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
        16,
        -480
      ],
      "id": "539e2a44-8a61-4635-849d-4a85fb049627",
      "name": "Loop Over Items"
    },
    {
      "parameters": {
        "amount": 60
      },
      "type": "n8n-nodes-base.wait",
      "typeVersion": 1.1,
      "position": [
        656,
        -144
      ],
      "id": "3dbd81fc-2a3f-4e2a-b8f0-ca592579067e",
      "name": "Wait"
    },
    {
      "parameters": {},
      "type": "n8n-nodes-base.manualTrigger",
      "typeVersion": 1,
      "position": [
        -416,
        -224
      ],
      "id": "f913ff07-080f-4d16-8b80-a68e89789e32",
      "name": "When clicking ‘Execute workflow’"
    }
  ],
  "connections": {
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
    "Loop Over Items": {
      "main": [
        [],
        [
          {
            "node": "Message a model",
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