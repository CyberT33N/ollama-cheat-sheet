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



## CLI Reference

<details><summary>Click to expand..</summary>

### Create a model

`ollama create` is used to create a model from a Modelfile.

```
ollama create mymodel -f ./Modelfile
```

### Pull a model

```
ollama pull llama3.2
```

> This command can also be used to update a local model. Only the diff will be pulled.

### Remove a model

```
ollama rm llama3.2
```

### Copy a model

```
ollama cp llama3.2 my-model
```

### Multiline input

For multiline input, you can wrap text with `"""`:

```
>>> """Hello,
... world!
... """
I'm a basic program that prints the famous "Hello, world!" message to the console.
```

### Multimodal models

```
ollama run llava "What's in this image? /Users/jmorgan/Desktop/smile.png"
The image features a yellow smiley face, which is likely the central focus of the picture.
```

### Pass the prompt as an argument

```
$ ollama run llama3.2 "Summarize this file: $(cat README.md)"
 Ollama is a lightweight, extensible framework for building and running language models on the local machine. It provides a simple API for creating, running, and managing models, as well as a library of pre-built models that can be easily used in a variety of applications.
```

### Show model information

```
ollama show llama3.2
```

### List models on your computer

```
ollama list
```

### List which models are currently loaded

```
ollama ps
```

### Stop a model which is currently running

```
ollama stop llama3.2
```

### Start Ollama

`ollama serve` is used when you want to start ollama without running the desktop application.

## Building

See the [developer guide](https://github.com/ollama/ollama/blob/main/docs/development.md)

### Running local builds

Next, start the server:

```
./ollama serve
```

Finally, in a separate shell, run a model:

```
./ollama run llama3.2
```

## REST API

Ollama has a REST API for running and managing models.

### Generate a response

```
curl http://localhost:11434/api/generate -d '{
  "model": "llama3.2",
  "prompt":"Why is the sky blue?"
}'
```

### Chat with a model

```
curl http://localhost:11434/api/chat -d '{
  "model": "llama3.2",
  "messages": [
    { "role": "user", "content": "why is the sky blue?" }
  ]
}'
```

See the [API documentation](./docs/api.md) for all endpoints.


</details>










































<br><br>
<br><br>
___
<br><br>
<br><br>


# Modelfile



<details><summary>Click to expand..</summary>


# Ollama Model File

> [!NOTE]
> `Modelfile` syntax is in development

A model file is the blueprint to create and share models with Ollama.

## Table of Contents

- [Format](#format)
- [Examples](#examples)
- [Instructions](#instructions)
  - [FROM (Required)](#from-required)
    - [Build from existing model](#build-from-existing-model)
    - [Build from a Safetensors model](#build-from-a-safetensors-model)
    - [Build from a GGUF file](#build-from-a-gguf-file)
  - [PARAMETER](#parameter)
    - [Valid Parameters and Values](#valid-parameters-and-values)
  - [TEMPLATE](#template)
    - [Template Variables](#template-variables)
  - [SYSTEM](#system)
  - [ADAPTER](#adapter)
  - [LICENSE](#license)
  - [MESSAGE](#message)
- [Notes](#notes)

## Format

The format of the `Modelfile`:

```modelfile
# comment
INSTRUCTION arguments
```

| Instruction                         | Description                                                    |
| ----------------------------------- | -------------------------------------------------------------- |
| [`FROM`](#from-required) (required) | Defines the base model to use.                                 |
| [`PARAMETER`](#parameter)           | Sets the parameters for how Ollama will run the model.         |
| [`TEMPLATE`](#template)             | The full prompt template to be sent to the model.              |
| [`SYSTEM`](#system)                 | Specifies the system message that will be set in the template. |
| [`ADAPTER`](#adapter)               | Defines the (Q)LoRA adapters to apply to the model.            |
| [`LICENSE`](#license)               | Specifies the legal license.                                   |
| [`MESSAGE`](#message)               | Specify message history.                                       |

## Examples

### Basic `Modelfile`

An example of a `Modelfile` creating a mario blueprint:

```modelfile
FROM llama3.2
# sets the temperature to 1 [higher is more creative, lower is more coherent]
PARAMETER temperature 1
# sets the context window size to 4096, this controls how many tokens the LLM can use as context to generate the next token
PARAMETER num_ctx 4096

# sets a custom system message to specify the behavior of the chat assistant
SYSTEM You are Mario from super mario bros, acting as an assistant.
```

To use this:

1. Save it as a file (e.g. `Modelfile`)
2. `ollama create choose-a-model-name -f <location of the file e.g. ./Modelfile>`
3. `ollama run choose-a-model-name`
4. Start using the model!

More examples are available in the [examples directory](../examples).

To view the Modelfile of a given model, use the `ollama show --modelfile` command.

  ```bash
  > ollama show --modelfile llama3.2
  # Modelfile generated by "ollama show"
  # To build a new Modelfile based on this one, replace the FROM line with:
  # FROM llama3.2:latest
  FROM /Users/pdevine/.ollama/models/blobs/sha256-00e1317cbf74d901080d7100f57580ba8dd8de57203072dc6f668324ba545f29
  TEMPLATE """{{ if .System }}<|start_header_id|>system<|end_header_id|>

  {{ .System }}<|eot_id|>{{ end }}{{ if .Prompt }}<|start_header_id|>user<|end_header_id|>

  {{ .Prompt }}<|eot_id|>{{ end }}<|start_header_id|>assistant<|end_header_id|>

  {{ .Response }}<|eot_id|>"""
  PARAMETER stop "<|start_header_id|>"
  PARAMETER stop "<|end_header_id|>"
  PARAMETER stop "<|eot_id|>"
  PARAMETER stop "<|reserved_special_token"
  ```

## Instructions

### FROM (Required)

The `FROM` instruction defines the base model to use when creating a model.

```modelfile
FROM <model name>:<tag>
```

#### Build from existing model

```modelfile
FROM llama3.2
```

A list of available base models:
<https://github.com/ollama/ollama#model-library>
Additional models can be found at:
<https://ollama.com/library>

#### Build from a Safetensors model

```modelfile
FROM <model directory>
```

The model directory should contain the Safetensors weights for a supported architecture.

Currently supported model architectures:
  * Llama (including Llama 2, Llama 3, Llama 3.1, and Llama 3.2)
  * Mistral (including Mistral 1, Mistral 2, and Mixtral)
  * Gemma (including Gemma 1 and Gemma 2)
  * Phi3

#### Build from a GGUF file

```modelfile
FROM ./ollama-model.gguf
```

The GGUF file location should be specified as an absolute path or relative to the `Modelfile` location.


### PARAMETER

The `PARAMETER` instruction defines a parameter that can be set when the model is run.

```modelfile
PARAMETER <parameter> <parametervalue>
```

#### Valid Parameters and Values

| Parameter      | Description                                                                                                                                                                                                                                             | Value Type | Example Usage        |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------- | -------------------- |
| mirostat       | Enable Mirostat sampling for controlling perplexity. (default: 0, 0 = disabled, 1 = Mirostat, 2 = Mirostat 2.0)                                                                                                                                         | int        | mirostat 0           |
| mirostat_eta   | Influences how quickly the algorithm responds to feedback from the generated text. A lower learning rate will result in slower adjustments, while a higher learning rate will make the algorithm more responsive. (Default: 0.1)                        | float      | mirostat_eta 0.1     |
| mirostat_tau   | Controls the balance between coherence and diversity of the output. A lower value will result in more focused and coherent text. (Default: 5.0)                                                                                                         | float      | mirostat_tau 5.0     |
| num_ctx        | Sets the size of the context window used to generate the next token. (Default: 2048)                                                                                                                                                                    | int        | num_ctx 4096         |
| repeat_last_n  | Sets how far back for the model to look back to prevent repetition. (Default: 64, 0 = disabled, -1 = num_ctx)                                                                                                                                           | int        | repeat_last_n 64     |
| repeat_penalty | Sets how strongly to penalize repetitions. A higher value (e.g., 1.5) will penalize repetitions more strongly, while a lower value (e.g., 0.9) will be more lenient. (Default: 1.1)                                                                     | float      | repeat_penalty 1.1   |
| temperature    | The temperature of the model. Increasing the temperature will make the model answer more creatively. (Default: 0.8)                                                                                                                                     | float      | temperature 0.7      |
| seed           | Sets the random number seed to use for generation. Setting this to a specific number will make the model generate the same text for the same prompt. (Default: 0)                                                                                       | int        | seed 42              |
| stop           | Sets the stop sequences to use. When this pattern is encountered the LLM will stop generating text and return. Multiple stop patterns may be set by specifying multiple separate `stop` parameters in a modelfile.                                      | string     | stop "AI assistant:" |
| tfs_z          | Tail free sampling is used to reduce the impact of less probable tokens from the output. A higher value (e.g., 2.0) will reduce the impact more, while a value of 1.0 disables this setting. (default: 1)                                               | float      | tfs_z 1              |
| num_predict    | Maximum number of tokens to predict when generating text. (Default: -1, infinite generation)                                                                                                                                   | int        | num_predict 42       |
| top_k          | Reduces the probability of generating nonsense. A higher value (e.g. 100) will give more diverse answers, while a lower value (e.g. 10) will be more conservative. (Default: 40)                                                                        | int        | top_k 40             |
| top_p          | Works together with top-k. A higher value (e.g., 0.95) will lead to more diverse text, while a lower value (e.g., 0.5) will generate more focused and conservative text. (Default: 0.9)                                                                 | float      | top_p 0.9            |
| min_p          | Alternative to the top_p, and aims to ensure a balance of quality and variety. The parameter *p* represents the minimum probability for a token to be considered, relative to the probability of the most likely token. For example, with *p*=0.05 and the most likely token having a probability of 0.9, logits with a value less than 0.045 are filtered out. (Default: 0.0) | float      | min_p 0.05            |

### TEMPLATE

`TEMPLATE` of the full prompt template to be passed into the model. It may include (optionally) a system message, a user's message and the response from the model. Note: syntax may be model specific. Templates use Go [template syntax](https://pkg.go.dev/text/template).

#### Template Variables

| Variable          | Description                                                                                   |
| ----------------- | --------------------------------------------------------------------------------------------- |
| `{{ .System }}`   | The system message used to specify custom behavior.                                           |
| `{{ .Prompt }}`   | The user prompt message.                                                                      |
| `{{ .Response }}` | The response from the model. When generating a response, text after this variable is omitted. |

```
TEMPLATE """{{ if .System }}<|im_start|>system
{{ .System }}<|im_end|>
{{ end }}{{ if .Prompt }}<|im_start|>user
{{ .Prompt }}<|im_end|>
{{ end }}<|im_start|>assistant
"""
```

### SYSTEM

The `SYSTEM` instruction specifies the system message to be used in the template, if applicable.

```modelfile
SYSTEM """<system message>"""
```

### ADAPTER

The `ADAPTER` instruction specifies a fine tuned LoRA adapter that should apply to the base model. The value of the adapter should be an absolute path or a path relative to the Modelfile. The base model should be specified with a `FROM` instruction. If the base model is not the same as the base model that the adapter was tuned from the behaviour will be erratic.

#### Safetensor adapter

```modelfile
ADAPTER <path to safetensor adapter>
```

Currently supported Safetensor adapters:
  * Llama (including Llama 2, Llama 3, and Llama 3.1)
  * Mistral (including Mistral 1, Mistral 2, and Mixtral)
  * Gemma (including Gemma 1 and Gemma 2)

#### GGUF adapter

```modelfile
ADAPTER ./ollama-lora.gguf
```

### LICENSE

The `LICENSE` instruction allows you to specify the legal license under which the model used with this Modelfile is shared or distributed.

```modelfile
LICENSE """
<license text>
"""
```

### MESSAGE

The `MESSAGE` instruction allows you to specify a message history for the model to use when responding. Use multiple iterations of the MESSAGE command to build up a conversation which will guide the model to answer in a similar way.

```modelfile
MESSAGE <role> <message>
```

#### Valid roles

| Role      | Description                                                  |
| --------- | ------------------------------------------------------------ |
| system    | Alternate way of providing the SYSTEM message for the model. |
| user      | An example message of what the user could have asked.        |
| assistant | An example message of how the model should respond.          |


#### Example conversation

```modelfile
MESSAGE user Is Toronto in Canada?
MESSAGE assistant yes
MESSAGE user Is Sacramento in Canada?
MESSAGE assistant no
MESSAGE user Is Ontario in Canada?
MESSAGE assistant yes
```


## Notes

- the **`Modelfile` is not case sensitive**. In the examples, uppercase instructions are used to make it easier to distinguish it from arguments.
- Instructions can be in any order. In the examples, the `FROM` instruction is first to keep it easily readable.

[1]: https://ollama.com/library



</details>





















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







<br><br>
<br><br>



# Importing a model


<details><summary>Click to expand..</summary>



## Table of Contents

  * [Importing a Safetensors adapter](#Importing-a-fine-tuned-adapter-from-Safetensors-weights)
  * [Importing a Safetensors model](#Importing-a-model-from-Safetensors-weights)
  * [Importing a GGUF file](#Importing-a-GGUF-based-model-or-adapter)
  * [Sharing models on ollama.com](#Sharing-your-model-on-ollamacom)

## Importing a fine tuned adapter from Safetensors weights

First, create a `Modelfile` with a `FROM` command pointing at the base model you used for fine tuning, and an `ADAPTER` command which points to the directory with your Safetensors adapter:

```dockerfile
FROM <base model name>
ADAPTER /path/to/safetensors/adapter/directory
```

Make sure that you use the same base model in the `FROM` command as you used to create the adapter otherwise you will get erratic results. Most frameworks use different quantization methods, so it's best to use non-quantized (i.e. non-QLoRA) adapters. If your adapter is in the same directory as your `Modelfile`, use `ADAPTER .` to specify the adapter path.

Now run `ollama create` from the directory where the `Modelfile` was created:

```bash
ollama create my-model
```

Lastly, test the model:

```bash
ollama run my-model
```

Ollama supports importing adapters based on several different model architectures including:

  * Llama (including Llama 2, Llama 3, Llama 3.1, and Llama 3.2);
  * Mistral (including Mistral 1, Mistral 2, and Mixtral); and
  * Gemma (including Gemma 1 and Gemma 2)

You can create the adapter using a fine tuning framework or tool which can output adapters in the Safetensors format, such as:

  * Hugging Face [fine tuning framework](https://huggingface.co/docs/transformers/en/training)
  * [Unsloth](https://github.com/unslothai/unsloth)
  * [MLX](https://github.com/ml-explore/mlx)


## Importing a model from Safetensors weights

First, create a `Modelfile` with a `FROM` command which points to the directory containing your Safetensors weights:

```dockerfile
FROM /path/to/safetensors/directory
```

If you create the Modelfile in the same directory as the weights, you can use the command `FROM .`.

Now run the `ollama create` command from the directory where you created the `Modelfile`:

```shell
ollama create my-model
```

Lastly, test the model:

```shell
ollama run my-model
```

Ollama supports importing models for several different architectures including:

  * Llama (including Llama 2, Llama 3, Llama 3.1, and Llama 3.2);
  * Mistral (including Mistral 1, Mistral 2, and Mixtral);
  * Gemma (including Gemma 1 and Gemma 2); and
  * Phi3

This includes importing foundation models as well as any fine tuned models which have been _fused_ with a foundation model.
## Importing a GGUF based model or adapter

If you have a GGUF based model or adapter it is possible to import it into Ollama. You can obtain a GGUF model or adapter by:

  * converting a Safetensors model with the `convert_hf_to_gguf.py` from Llama.cpp; 
  * converting a Safetensors adapter with the `convert_lora_to_gguf.py` from Llama.cpp; or
  * downloading a model or adapter from a place such as HuggingFace

To import a GGUF model, create a `Modelfile` containing:

```dockerfile
FROM /path/to/file.gguf
```

For a GGUF adapter, create the `Modelfile` with:

```dockerfile
FROM <model name>
ADAPTER /path/to/file.gguf
```

When importing a GGUF adapter, it's important to use the same base model as the base model that the adapter was created with. You can use:

 * a model from Ollama
 * a GGUF file
 * a Safetensors based model 

Once you have created your `Modelfile`, use the `ollama create` command to build the model.

```shell
ollama create my-model
```

## Quantizing a Model

Quantizing a model allows you to run models faster and with less memory consumption but at reduced accuracy. This allows you to run a model on more modest hardware.

Ollama can quantize FP16 and FP32 based models into different quantization levels using the `-q/--quantize` flag with the `ollama create` command.

First, create a Modelfile with the FP16 or FP32 based model you wish to quantize.

```dockerfile
FROM /path/to/my/gemma/f16/model
```

Use `ollama create` to then create the quantized model.

```shell
$ ollama create --quantize q4_K_M mymodel
transferring model data
quantizing F16 model to Q4_K_M
creating new layer sha256:735e246cc1abfd06e9cdcf95504d6789a6cd1ad7577108a70d9902fef503c1bd
creating new layer sha256:0853f0ad24e5865173bbf9ffcc7b0f5d56b66fd690ab1009867e45e7d2c4db0f
writing manifest
success
```

### Supported Quantizations

- `q4_0`
- `q4_1`
- `q5_0`
- `q5_1`
- `q8_0`

#### K-means Quantizations

- `q3_K_S`
- `q3_K_M`
- `q3_K_L`
- `q4_K_S`
- `q4_K_M`
- `q5_K_S`
- `q5_K_M`
- `q6_K`


## Sharing your model on ollama.com

You can share any model you have created by pushing it to [ollama.com](https://ollama.com) so that other users can try it out.

First, use your browser to go to the [Ollama Sign-Up](https://ollama.com/signup) page. If you already have an account, you can skip this step.

<img src="images/signup.png" alt="Sign-Up" width="40%">

The `Username` field will be used as part of your model's name (e.g. `jmorganca/mymodel`), so make sure you are comfortable with the username that you have selected.

Now that you have created an account and are signed-in, go to the [Ollama Keys Settings](https://ollama.com/settings/keys) page.

Follow the directions on the page to determine where your Ollama Public Key is located.

<img src="images/ollama-keys.png" alt="Ollama Keys" width="80%">

Click on the `Add Ollama Public Key` button, and copy and paste the contents of your Ollama Public Key into the text field.

To push a model to [ollama.com](https://ollama.com), first make sure that it is named correctly with your username. You may have to use the `ollama cp` command to copy
your model to give it the correct name. Once you're happy with your model's name, use the `ollama push` command to push it to [ollama.com](https://ollama.com).

```shell
ollama cp mymodel myuser/mymodel
ollama push myuser/mymodel
```

Once your model has been pushed, other users can pull and run it by using the command:

```shell
ollama run myuser/mymodel
```


  
</details>
