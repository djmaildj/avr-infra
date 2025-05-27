# AVR Infrastructure (avr-infra)

[![Discord](https://img.shields.io/discord/1347239846632226998?label=Discord&logo=discord)](https://discord.gg/DFTU69Hg74)
[![GitHub Repo stars](https://img.shields.io/github/stars/agentvoiceresponse/avr-infra?style=social)](https://github.com/agentvoiceresponse/avr-infra)
[![Ko-fi](https://img.shields.io/badge/Support%20us%20on-Ko--fi-ff5e5b.svg)](https://ko-fi.com/agentvoiceresponse)

The **AVR Infrastructure** project is designed to launch the Agent Voice Response application, which will start the Core, ASR, LLM, and TTS services integrated with Asterisk Audiosocket. Additionally, the project runs a Docker Asterisk with a basic PJSIP configuration. To test, register a SIP client with username 1000 and password 1000 using TCP. The extensions will be generated once the AI Agent is configured from the application. The extension to call will be shown on the GUI.

## Project Overview

The infrastructure allows you to deploy the following services:

- **AVR Core**: Manages the interaction between the customer and the VoIP PBX (e.g., Asterisk), processing the audio stream and managing the integration with ASR, LLM, and TTS services.
- **ASR Services**: Converts voice to text. The infrastructure supports services like Google Cloud Speech-to-Text, Deepgram.
- **LLM Services**: Handles the logic and responses for customer interactions. For example, OpenAI, OpenRouter.ai, Typebot can be integrated to generate AI-based responses.
- **TTS Services**: Converts text responses into audio, allowing AVR Core to reply to the customer with speech. Services like Google Cloud Text-to-Speech, Deepgram, ElevenLabs are supported.

This architecture allows you to customize and swap ASR, LLM, and TTS providers as needed.

> **New in recent versions:** AVR now supports integration with OpenAI Realtime Speech-to-Speech. You can find an example configuration in the `docker-compose-openai-realtime.yml` file.

> If the provider you want to integrate does not support ASR but only Speech-to-Text, we've implemented support for this as well. Simply add the `avr-asr-to-tts` container between `avr-core` and your TTS container. You can find an example using ElevenLabs STT in the `docker-compose-elevenlabs.yml` file.

## Key Features

- **Modular Architecture**: Plug and play with any ASR, LLM, or TTS services via API.
- **Real-Time Audio Streaming**: Manages real-time interactions between customers and services.
- **Simple Configuration**: Set your ASR, LLM, and TTS providers by configuring environment variables.
- **Scalable Design**: Easy to extend and integrate with different services and AI providers.

## How It Works

### Flow Overview

1. **AVR Core** receives an audio stream from the Asterisk PBX.
2. The audio is sent to an **ASR** service to transcribe the speech to text.
3. The transcribed text is forwarded to an **LLM** service to generate an AI-powered response.
4. The generated response is sent to a **TTS** service to convert the text back to speech.
5. The speech is then played back to the customer through Asterisk.

## Deployment Modes

You can deploy the AVR infrastructure in different modes and with different provider combinations, depending on your needs.

### 1. Headless Deployments (No GUI)

If you don't need the web interface, you can use one of the example docker-compose files tailored for specific provider combinations. Each file launches only the core services (ASR, LLM, TTS, Asterisk) with the selected providers.

### Important Configuration Note

The most critical configuration is setting up the correct URLs for your services in the `.env` file:

- `ASR_URL`: The URL where your ASR service is running (e.g., `http://avr-asr-[provider_name]:[port]}`)
- `LLM_URL`: The URL where your LLM service is running (e.g., `http://avr-llm-[provider_name]:[port]`)
- `TTS_URL`: The URL where your TTS service is running (e.g., `http://avr-tts-[provider_name]:[port]`)

If you are using STS (Speech-to-Speech) instead of separate ASR and TTS services, you only need to configure:
- `STS_URL`: The URL where your STS service is running (e.g., `http://avr-sts-[provider_name]:[port]`)
You can find an example configuration using OpenAI Realtime in the `docker-compose-openai-realtime.yml` file.

If your provider doesn't support ASR but only STT (Speech-to-Text), you'll need to use the `avr-asr-to-tts` container which handles VAD (Voice Activity Detection) and audio packet composition for your STT service. In this case:
- Set `ASR_URL` to `http://avr-asr-to-tts:[port]`
- Configure your `STT_URL` in the environment variables of the `avr-asr-to-tts` container
You can find an example configuration using ElevenLabs in the `docker-compose-elevenlabs.yml` file.

These URLs are used by AVR Core to forward:
- Audio chunks to your ASR service
- Transcripts to your LLM service
- Responses to your TTS service

Make sure these URLs are correctly configured whether you're using local services or external providers.

### Table of Compose Files 

| File | ASR | LLM | TTS | Example | Use Case/Notes |
|------|-----|-----|-----|---------|----------------|
| [docker-compose-anthropic.yml](./docker-compose-anthropic.yml) | Deepgram | Anthropic | Deepgram | [1](#example-1-deepgram-asrtts--anthropic-llm) | Headless, Anthropic LLM |
| [docker-compose-openai.yml](./docker-compose-openai.yml) | Deepgram | OpenAI | Deepgram | [2](#example-2-deepgram-asrtts--openai-llm) | Headless, OpenAI LLM |
| [docker-compose-elevenlabs.yml](./docker-compose-elevenlabs.yml) | ElevenLabs | OpenAI | ElevenLabs | [3](#example-3-elevenlabs-stttts--avr-asr-to-stt--anthropic-llm) | Headless, ElevenLabs |
| [docker-compose-google.yml](./docker-compose-google.yml) | Google | OpenRouter | Google | [4](#example-4-google-asrtts--openrouter-llm) | Headless, Google |
| [docker-compose-vosk.yml](./docker-compose-vosk.yml) | Vosk | Anthropic | Deepgram | [5](#example-5-vosk-asr-open-source--anthropic-llm--deepgram-tts) | Headless, Vosk Open Source |
| [docker-compose-openai-realtime.yml](./docker-compose-openai-realtime.yml) | OpenAI | OpenAI | OpenAI | [6](#example-6-openai-realtime-asrllmtts) | Headless, OpenAI Realtime |

#### Example 1: Deepgram (ASR+TTS) + Anthropic (LLM)

```bash
docker-compose -f docker-compose-anthropic.yml up -d
```

**Required .env parameters:**
```
DEEPGRAM_API_KEY=your_deepgram_key

ANTHROPIC_API_KEY=sk-ant-
ANTHROPIC_MODEL=claude-3-haiku-20240307
ANTHROPIC_MAX_TOKENS=1024
ANTHROPIC_TEMPERATURE=1
ANTHROPIC_SYSTEM_PROMPT="You are a helpful assistant."
```

#### Example 2: Deepgram (ASR+TTS) + OpenAI (LLM)

```bash
docker-compose -f docker-compose-openai.yml up -d
```

**Required .env parameters:**
```
DEEPGRAM_API_KEY=your_deepgram_key

OPENAI_API_KEY=sk-proj-
OPENAI_MODEL=gpt-3.5-turbo
OPENAI_MAX_TOKENS=100
OPENAI_TEMPERATURE=0.0
```
#### Example 3: ElevenLabs (STT+TTS) + AVR-ASR-TO-STT + Anthropic (LLM)

```bash
docker-compose -f docker-compose-elevenlabs.yml up -d
```

**Required .env parameters:**
```
ELEVENLABS_API_KEY="sk_"
ELEVENLABS_MODEL_ID=scribe_v1
ELEVENLABS_LANGUAGE_CODE=en
ELEVENLABS_VOICE_ID=Xb7hH8MSUJpSbSDYk0k2

ANTHROPIC_API_KEY=sk-ant-
ANTHROPIC_MODEL=claude-3-haiku-20240307
ANTHROPIC_MAX_TOKENS=1024
ANTHROPIC_TEMPERATURE=1
ANTHROPIC_SYSTEM_PROMPT="You are a helpful assistant."
```

#### Example 4: Google (ASR+TTS) + OpenRouter (LLM)

```bash
docker-compose -f docker-compose-google.yml up -d
```

**Required .env parameters:**
```
GOOGLE_APPLICATION_CREDENTIALS="/google.json"
SPEECH_RECOGNITION_LANGUAGE=en-US

TEXT_TO_SPEECH_LANGUAGE=en-US
TEXT_TO_SPEECH_GENDER=FEMALE
TEXT_TO_SPEECH_NAME=en-US-
TEXT_TO_SPEECH_SPEAKING_RATE=1

OPENROUTER_API_KEY=sk-or-v1-
OPENROUTER_MODEL=deepseek/deepseek-chat-v3-0324:free
```

#### Example 5: Vosk (ASR Open Source) + Anthropic (LLM) + Deepgram (TTS)

```bash
docker-compose -f docker-compose-vosk.yml up -d
```

**Required .env parameters:**
```
DEEPGRAM_API_KEY=your_deepgram_key
```

#### Example 6: OpenAI Realtime (ASR+LLM+TTS)

```bash
docker-compose -f docker-compose-openai-realtime.yml up -d
```

**Required .env parameters:**
```
PORT=6030
OPENAI_API_KEY=
OPENAI_MODEL=gpt-4o-realtime-preview
OPENAI_INSTRUCTIONS="You are a helpful assistant."
```

### Testing Your Setup

Once you have configured and started your services, you can test the setup using a SIP client:

1. **SIP Client Registration**:
   - Each docker-compose file includes an Asterisk instance that exposes port 5060 TCP
   - By default, there is a PJSIP endpoint configured with:
     - Username: `1000`
     - Password: `1000`
     - Transport: TCP
   - Register your SIP client using these credentials

2. **Initial Testing**:
   - First, test the basic connectivity by calling extension `600` (echo test)
   - If you hear your voice echoed back, the basic setup is working

3. **Testing AVR**:
   - If you're using the default Asterisk configuration, you'll find extension `5001` in `extensions.conf`
   - This extension is configured to connect to your `avr-core` container
   - Call extension `5001` and you should hear the message configured in your AVR-core, processed through your TTS service

If everything works as expected, your AVR setup is ready for use. Happy testing!

### How to Add Your Own Provider Combination

1. Use one of the example docker-compose files.
2. Copy the `.env.example` file in `.env` file.
3. Update the `.env` file with the required API keys and model names.


### 2. Full Deployment (with Web Interface) (working in progress)

Launches all services, including the web application and database.

> **Note:** The web interface is currently under development (work in progress). If you want to test or contribute to its evolution, make sure to add the following MySQL credentials to your `.env` file:
> ```
> MYSQL_ROOT_PASSWORD=your_root_password
> MYSQL_DATABASE=avr
> MYSQL_USER=avr
> MYSQL_PASSWORD=your_password
> ```

### Prerequisites

1. Docker and Docker Compose installed on your machine.
2. Google Cloud credentials if you are using their services for ASR or TTS.
3. OpenAI credentials if you are using their services for LLM.
4. Deepgram credentials if you are using their services for ASR or TTS.

```bash
docker-compose -f docker-compose-app.yml up -d
```