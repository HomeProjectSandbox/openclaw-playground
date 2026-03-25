# Setup openclaw with docker

Openclaw should run inside a docker container, Ollama model shall run on the MAC (to use the GPU, cannot run inside docker container)

[openclaw doc](https://docs.openclaw.ai/)

https://docs.openclaw.ai/install/docker

```
ollama run llama3.1:8b  #in a terminal
```



```

rm -rf $HOME/.openclaw/
git clone https://github.com/openclaw/openclaw.git
cd openclaw
./docker-setup.sh       #chose manual configuration
#docker compose up -d openclaw-gateway
```

When the setup asks for the Ollama base URL, use one of these instead:      
                
On macOS:                                                                  
http://host.docker.internal:11434



configure oters...

After that the gateway started
```
docker ps                                                                                                                                       2m 14s 12:56:57 PM
CONTAINER ID   IMAGE            COMMAND                  CREATED          STATUS                             PORTS                                  NAMES
896c3a9b6559   openclaw:local   "docker-entrypoint.s…"   15 seconds ago   Up 14 seconds (health: starting)   0.0.0.0:18789-18790->18789-18790/tcp   openclaw-openclaw-gateway-1
```

Stop the container:
```
docker compose down
```

Manually change openclaw configuration:
```
vi $HOME/.openclaw/openclaw.json

#remove auth section
"auth": {
    "profiles": {
      "ollama:default": {
        "provider": "ollama",
        "mode": "api_key"
      }
    }
  },

# add the model
"models": {
    "mode": "merge",
    "providers": {
      "ollama": {
        "baseUrl": "http://host.docker.internal:11434",
        "apiKey": "ollama-local",
        "api": "ollama",
        "models": [
          { 
            "id": "llama3.1:8b",
            "name": "llama3.1:8b",
            "reasoning": false,
            "input": [
              "text"
            ],
            "cost": {
              "input": 0,
              "output": 0, 
              "cacheRead": 0,
              "cacheWrite": 0
            },
            "contextWindow": 131072,
            "maxTokens": 8192
          }
        ]
      }
    }
  },

#agents also need to be checked
"agents": {
    "defaults": {
      "model": {
        "primary": "ollama/llama3.1:8b"
      },
      "models": {
        "ollama/llama3.1:8b": {}
      },
      "workspace": "/home/node/.openclaw/workspace",
      "sandbox": {
        "mode": "off"
      }
    }
  },

"auth": {
      "mode": "token",
      ...

token also visible

```

then
```
docker compose up -d

docker compose run --rm openclaw-cli dashboard --no-open

#then go to the browser

#docker compose run --rm openclaw-cli devices clear --yes
docker compose run --rm openclaw-cli devices list

#approve pending request:
docker compose run --rm openclaw-cli devices approve <requestId>
```







## Decision 2: Local LLM or Connect to something (Gemini, OpenAI)
I just decided to connect using a small model llama.3.1 running via Ollama. 
It was running on my local machine and not docker


Open the Control UI
Open http://127.0.0.1:18789/ in your browser and paste the token into Settings.

url: `docker compose run --rm openclaw-cli dashboard --no-open`

 1. Web UI:                                                                  
  Visit http://localhost:18789 and enter the token when prompted.             
                                                                              
  2. CLI (from your Mac):                                                     
  # Set the token as an environment variable                                  
  export OPENCLAW_GATEWAY_TOKEN=66e96ed204e34d56595634c198b32f3f5bb281f43af741
  5d4adf3bfd9db79034                                                          
                                                                              
  # Then connect                                                              
  openclaw gateway pair --url http://localhost:18789                          
                                                    
Fetch a fresh dashboard link and approve the browser device:
```
docker compose run --rm openclaw-cli dashboard --no-open
docker compose run --rm openclaw-cli devices list
docker compose run --rm openclaw-cli devices approve <requestId>
```                                                             

After it's approved, go back to your web browser at http://localhost:18789 and click the "Connect" button again!


```
docker compose run --rm openclaw-cli nodes pending

cat ~/.openclaw/openclaw.json | grep -A 5 "gateway" 

docker compose run --rm -e OPENCLAW_GATEWAY_TOKEN=<your token> -e OPENCLAW_ALLOW_INSECURE_PRIVATE_WS=1 openclaw-cli node run --host openclaw-gateway --port 18789 


docker compose run --rm openclaw-cli devices list
docker compose run --rm openclaw-cli devices approve <requestId>
docker compose run --rm -e OPENCLAW_GATEWAY_TOKEN=<your token> -e OPENCLAW_ALLOW_INSECURE_PRIVATE_WS=1 openclaw-cli node run --host openclaw-gateway --port 18789  
```
docker compose run --rm openclaw-cli node run 