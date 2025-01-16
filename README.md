# ollama-cheat-sheet


# Install

## ubuntu
```shell
curl -fsSL https://ollama.com/install.sh | sh
```










<br><br>
<br><br>
___
<br><br>
<br><br>

# Models

<br><br>

## https://github.com/ollama/ollama?tab=readme-ov-file#import-from-gguf

Create Modelfile:
```
FROM /home/t33n/Projects/ai/resources/models/llm/deepseek/Coder V2 Lite/DeepSeek-Coder-V2-Lite-Instruct-Q8_0.gguf
```

Then create:
```shell
ollama create deepseek-v2-coder-lite -f ./Modelfile
```

Then run:
```shell
ollama run deepseek-v2-coder-lite
```
