{
  "nodes": [
    {
      "parameters": {},
      "type": "n8n-nodes-base.manualTrigger",
      "typeVersion": 1,
      "position": [0, 0],
      "id": "trigger",
      "name": "Inicio"
    },
    {
      "parameters": {
        "assignments": {
          "assignments": [
            {
              "id": "guion",
              "name": "guion",
              "value": "Tu guion...",
              "type": "string"
            },
            {
              "id": "escenas_raw",
              "name": "escenas_raw",
              "value": "ESCENA 1: Pasillo oscuro... --- ESCENA 2: Exterior niebla...",
              "type": "string"
            }
          ]
        }
      },
      "type": "n8n-nodes-base.set",
      "typeVersion": 3.4,
      "position": [220, 0],
      "id": "set_data",
      "name": "Datos Base"
    },
    {
      "parameters": {
        "jsCode": "const raw = $input.first().json.escenas_raw || \"\";\nconst parts = raw.split('---').map(s => s.trim()).filter(Boolean);\nreturn parts.map((scene, i) => ({ json: { scene_index: i + 1, scene_text: scene } }));"
      },
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [440, 0],
      "id": "split_scenes",
      "name": "Separar Escenas"
    },
    {
      "parameters": {
        "conditions": {
          "options": { "caseSensitive": true },
          "conditions": [
            {
              "leftValue": "={{$json.scene_index}}",
              "rightValue": 1,
              "operator": { "type": "number", "operation": "equals" }
            }
          ]
        }
      },
      "type": "n8n-nodes-base.if",
      "typeVersion": 2.3,
      "position": [660, 0],
      "id": "filter_scene",
      "name": "Solo Escena 1 (Test)"
    },
    {
      "parameters": {
        "method": "POST",
        "url": "https://api.piapi.ai/api/v1/task",
        "authentication": "genericCredentialType",
        "genericAuthType": "httpHeaderAuth",
        "sendBody": true,
        "specifyBody": "json",
        "jsonBody": "={\n  \"model\": \"Qubico/flux1-schnell\",\n  \"task_type\": \"txt2img\",\n  \"input\": {\n    \"prompt\": \"psychological horror, cinematic. SCENE: {{ $json.scene_text }}\",\n    \"width\": 720,\n    \"height\": 1280\n  }\n}"
      },
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.3,
      "position": [880, 0],
      "id": "start_task",
      "name": "1. Iniciar Tarea",
      "credentials": { "httpHeaderAuth": { "id": "CRTAYyEXosxR6Tiz", "name": "Header Auth account" } }
    },
    {
      "parameters": {
        "amount": 10,
        "unit": "seconds"
      },
      "type": "n8n-nodes-base.wait",
      "typeVersion": 1.1,
      "position": [1100, 0],
      "id": "wait",
      "name": "Esperar 10s"
    },
    {
      "parameters": {
        "url": "=https://api.piapi.ai/api/v1/task/{{ $node['1. Iniciar Tarea'].json.data.task_id }}",
        "authentication": "genericCredentialType",
        "genericAuthType": "httpHeaderAuth"
      },
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.3,
      "position": [1320, 0],
      "id": "check_status",
      "name": "2. Consultar Estado",
      "credentials": { "httpHeaderAuth": { "id": "CRTAYyEXosxR6Tiz", "name": "Header Auth account" } }
    },
    {
      "parameters": {
        "conditions": {
          "options": { "caseSensitive": true },
          "conditions": [
            {
              "leftValue": "={{ $json.data.status }}",
              "rightValue": "completed",
              "operator": { "type": "string", "operation": "equals" }
            }
          ]
        }
      },
      "type": "n8n-nodes-base.if",
      "typeVersion": 2.2,
      "position": [1540, 0],
      "id": "is_finished",
      "name": "¿Terminó?"
    },
    {
      "parameters": {
        "url": "={{ $json.data.output.image_url }}",
        "responseFormat": "file"
      },
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.3,
      "position": [1800, -100],
      "id": "download",
      "name": "3. Descargar Imagen"
    }
  ],
  "connections": {
    "Inicio": { "main": [[{ "node": "Datos Base", "type": "main", "index": 0 }]] },
    "Datos Base": { "main": [[{ "node": "Separar Escenas", "type": "main", "index": 0 }]] },
    "Separar Escenas": { "main": [[{ "node": "Solo Escena 1 (Test)", "type": "main", "index": 0 }]] },
    "Solo Escena 1 (Test)": { "main": [[{ "node": "1. Iniciar Tarea", "type": "main", "index": 0 }]] },
    "1. Iniciar Tarea": { "main": [[{ "node": "Esperar 10s", "type": "main", "index": 0 }]] },
    "Esperar 10s": { "main": [[{ "node": "2. Consultar Estado", "type": "main", "index": 0 }]] },
    "2. Consultar Estado": { "main": [[{ "node": "¿Terminó?", "type": "main", "index": 0 }]] },
    "¿Terminó?": {
      "main": [
        [{ "node": "3. Descargar Imagen", "type": "main", "index": 0 }],
        [{ "node": "Esperar 10s", "type": "main", "index": 0 }]
      ]
    }
  }
}

