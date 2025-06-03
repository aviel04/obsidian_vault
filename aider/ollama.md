# Ollama

Aider can connect to local Ollama models.

First, install aider:

```bash
python -m pip install aider-install
aider-install
```

Then configure your Ollama API endpoint (usually the default):

```bash
export OLLAMA_API_BASE=http://127.0.0.1:11434 # Mac/Linux
```

Start working with aider and Ollama on your codebase:

```bash
# Pull the model
ollama pull <model>

# Start your ollama server, increasing the context window to 8k tokens
OLLAMA_CONTEXT_LENGTH=8192 ollama serve

# In another terminal window, change directory into your codebase
cd /to/your/project

aider --model ollama_chat/<model>
```

Using `ollama_chat/` is recommended over `ollama/`.

See the [model warnings](https://aider.chat/docs/llms/warnings.html) section for information on warnings which will occur when working with models that aider is not familiar with.

## [API Key](https://aider.chat/docs/llms/ollama.html#api-key)

If you are using an Ollama that requires an API key you can set `OLLAMA_API_KEY`:

```bash
export OLLAMA_API_KEY=<api-key> # Mac/Linux
setx   OLLAMA_API_KEY <api-key> # Windows, restart shell after setx
```

## [](https://aider.chat/docs/llms/ollama.html#setting-the-context-window-size)Setting the context window size

[Ollama uses a 2k context window by default](https://github.com/ollama/ollama/blob/main/docs/faq.md#how-can-i-specify-the-context-window-size), which is very small for working with aider. It also **silently** discards context that exceeds the window. This is especially dangerous because many users don’t even realize that most of their data is being discarded by Ollama.

By default, aider sets Ollama’s context window to be large enough for each request you send plus 8k tokens for the reply. This ensures data isn’t silently discarded by Ollama.

If you’d like you can configure a fixed sized context window instead with an [`.aider.model.settings.yml` file](https://aider.chat/docs/config/adv-model-settings.html#model-settings) like this:

```bash
- name: ollama/qwen2.5-coder:32b-instruct-fp16
  extra_params:
    num_ctx: 65536
```