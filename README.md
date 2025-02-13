# Local AI Infrastructure

If you're worried about privacy and security, you can run your own local AI infrastructure with this repository.

Imagine running ChatGPT, Stable Diffusion, Cursor (or VSCode) chat/autocomplete, and more, locally.

It includes the following basic services:

- [Ollama](https://github.com/ollama/ollama) as the LLM provider
- [Open WebUI](https://github.com/open-webui/open-webui), a browser-based AI platform
- [ComfyUI](https://github.com/comfyanonymous/ComfyUI), for image generation, integrated with Open WebUI
- [ngrok](https://ngrok.com/), for your services to be accessible from the internet

It's also possible to integrate this with Cursor or VSCode, via [continue.dev](https://continue.dev/).


## Prerequisites

### Hardware

This was built with focus on NVIDIA GPUs, using other GPUs is out of scope, but it's possible by tweaking the docker-compose file.

### Ngrok

If you want to expose your services to the internet, you can use [ngrok](https://ngrok.com/). This is not required, so if you don't want to use it, you can just remove the `ngrok` and `nginx` services from the docker-compose file.

## Usage

Start the services with docker compose. This will take quite a while, mostly due to `comfyui` image. If are interested in only testing it, you can skip the `comfyui` service (comment it out).

Create the external volumes for the services that need them:

```bash
docker volume create ollama
docker volume create open-webui
docker volume create comfyui-checkpoints
docker volume create comfyui-vae
```

and then start the services:

```bash
docker compose up -d
```

## Configuration

### Ollama

To get started, you need to download the models from the [ollama models page](https://ollama.com/models).

You can do that by running the following command:

```bash
docker compose exec -it ollama ollama pull llama3.1:8b
```

To verify that the model was downloaded correctly, run:

```bash
docker compose exec -it ollama ollama list
```

### Open WebUI

Open WebUI documentation is available [here](https://docs.openwebui.com/).

The service can be accessed at [localhost:3002](http://localhost:3002) and if you open for the first time, you'll need to create an account.

There isn't much to setup so you can create a chat and start using it right away.

### ComfyUI

ComfyUI documentation is available [here](https://docs.comfy.org/).

The service can be accessed at [localhost:8188/](http://localhost:8188/) however you still need to configure it.

A guide on how to configure Open WebUI to use ComfyUI is available [here](https://docs.openwebui.com/tutorials/images#comfyui).

### VSCode

To integrate with VSCode, you can use the [continue.dev](https://continue.dev/) extension.

Once installed, you need to point it to the local ollama instance. Detailed documentation is available [here](https://docs.continue.dev/chat/model-setup#local-offline-experience), but in short, you need to add this to the extension's `config.json` file:

```json
{
  "models": [
    {
      "title": "llama3.1:8b",
      "provider": "ollama",
      "model": "llama3.1:8b",
      "apiBase": "http://localhost:11434"
    }
  ]
}
```

### Cursor

It's possible to point Cursor to use the local ollama instance. This will work for the "chat" and "composer", but not for the autocomplete features.

As the time of the writing, Cursor servers need to reach out to your local machine, so you need `ngrok` and `curxy` running.

> `curxy` is a very simple proxy that fixes some of the issues with the way Cursor makes requests to the API by pretending that it's an OpenAI API endpoint.

Create an [auth token](https://dashboard.ngrok.com/get-started/your-authtoken) and optionally a [domain](https://dashboard.ngrok.com/domains) (since every time ngrok starts, it will generate a new domain and you'll need to reconfigure Cursor).

Copy the `ngrok.example.yml` and configure the `agent.authtoken` and the `endpoints.web.url` to point to your ngrok's domain.

```bash
cp config/ngrok/ngrok.example.yml config/ngrok/ngrok.yml
```

Then, configure cursor by following these steps:

1. Cursor Settings > Models > Model Names > Add `llama3.1:8b` as model to Cursor
2. Cursor Settings > OpenAI API Key > Enable
3. Cursor Settings > OpenAI API Key > Override OpenAI Base URL > Add `https://your-ngrok-domain.ngrok-free.app/curxy/v1` > Save
4. Cursor Settings > OpenAI API Key > Add arbitrary string as OpenAI API key > Verify
5. Use `llama3.1:8b` as Chat Model


## Acknowledgements

Thanks to [ryoppippi/curxy](https://github.com/ryoppippi/curxy) for the Cursor/Ollama proxy. I'm currently using a copy of it due to [this issue](https://github.com/ryoppippi/curxy/issues/33).
