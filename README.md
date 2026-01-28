{
  "name": "Fabrica_Terror_Final_Kenshin",
  "nodes": [
    {
      "parameters": {
        "pollTimes": {
          "item": [{"mode": "everyMinute"}]
        },
        "documentId": {
          "__rl": true,
          "mode": "list",
          "value": "Videos_Terror"
        },
        "sheetName": "Hoja 1",
        "filtersUI": {
          "values": [{"column": "Status", "condition": "equal", "value": "Pendiente"}]
        }
      },
      "id": "n1",
      "name": "Google Sheets Trigger",
      "type": "n8n-nodes-base.googleSheetsTrigger",
      "typeVersion": 1,
      "position": [100, 300]
    },
    {
      "parameters": {
        "model": "gpt-4o",
        "messagesUI": {
          "values": [
            {
              "role": "system",
              "content": "Eres un escritor de terror. Genera un guion corto y 6 prompts de imagen detallados en inglés para PiAPI."
            },
            {
              "role": "user",
              "content": "={{ $json.Idea }}"
            }
          ]
        }
      },
      "id": "n2",
      "name": "ChatGPT - Guion",
      "type": "n8n-nodes-base.openAi",
      "typeVersion": 1,
      "position": [320, 300]
    },
    {
      "parameters": {
        "model": "tts-1-hd",
        "voice": "onyx",
        "text": "={{ $json.message.content }}"
      },
      "id": "n3",
      "name": "OpenAI TTS - Audio",
      "type": "n8n-nodes-base.openAi",
      "typeVersion": 1,
      "position": [540, 150]
    },
    {
      "parameters": {
        "resource": "folder",
        "name": "={{ $node[\"Google Sheets Trigger\"].json.Idea.substring(0,25) }}",
        "parentId": "1y7DGt3NP9oVIPrA3WVHJIz9ihJGARpFC"
      },
      "id": "n4",
      "name": "Crear Carpeta Video",
      "type": "n8n-nodes-base.googleDrive",
      "typeVersion": 3,
      "position": [540, 450]
    },
    {
      "parameters": {
        "resource": "file",
        "operation": "upload",
        "fileContent": "={{ $node[\"OpenAI TTS - Audio\"].binary.data }}",
        "fileName": "locucion.mp3",
        "parentId": "={{ $json.id }}"
      },
      "id": "n5",
      "name": "Subir Audio",
      "type": "n8n-nodes-base.googleDrive",
      "typeVersion": 3,
      "position": [760, 300]
    }
  ],
  "connections": {
    "Google Sheets Trigger": {
      "main": [[{"node": "ChatGPT - Guion", "type": "main", "index": 0}]]
    },
    "ChatGPT - Guion": {
      "main": [
        [{"node": "OpenAI TTS - Audio", "type": "main", "index": 0}],
        [{"node": "Crear Carpeta Video", "type": "main", "index": 0}]
      ]
    },
    "Crear Carpeta Video": {
      "main": [[{"node": "Subir Audio", "type": "main", "index": 0}]]
    }
  }
}
