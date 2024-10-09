# AVR Infrastructure (avr-infra)

The **AVR Infrastructure** project provides a pre-configured setup to run the core components of the Agent Voice Response (AVR) system. AVR Core is an advanced IVR solution that integrates with AI services, allowing dynamic interaction with customers by replacing traditional IVR systems with an AI-driven voicebot. The architecture of this infrastructure allows seamless interaction with Automatic Speech Recognition (ASR), Large Language Models (LLM), and Text-to-Speech (TTS) services, all configured through HTTP API streams.

## Project Overview

The infrastructure allows you to deploy the following services:
- **AVR Core**: Manages the interaction between the customer and the VoIP PBX (e.g., Asterisk), processing the audio stream and managing the integration with ASR, LLM, and TTS services.
- **ASR Services**: Converts voice to text. The infrastructure supports services like Google Cloud Speech-to-Text.
- **LLM Services**: Handles the logic and responses for customer interactions. For example, OpenAI or Typebot can be integrated to generate AI-based responses.
- **TTS Services**: Converts text responses into audio, allowing AVR Core to reply to the customer with speech. Services like Google Cloud Text-to-Speech are supported.

This architecture allows you to customize and swap ASR, LLM, and TTS providers as needed.

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

### Example Service Flow

- ASR service processes audio at `ASR_URL=http://host.docker.internal:6001/speech-to-text-stream`.
- LLM service processes text and generates a response at `LLM_URL=http://host.docker.internal:6004/prompt-stream`.
- TTS service converts text to audio at `TTS_URL=http://host.docker.internal:6003/text-to-speech-stream`.

## Setup Instructions

### Prerequisites

1. Docker and Docker Compose installed on your machine.
2. Google Cloud credentials if you are using their services for ASR or TTS.
3. OpenAI credentials if you are using their services for LLM.
4. Typebot credentials if you are using their services for LLM.

### Clone the Repository

```bash
git clone https://github.com/agentvoiceresponse/avr-infra
cd avr-infra
```

### Configuration

Edit the `.env` file to set your environment variables for ASR, LLM, and TTS services:

```bash
ASR_URL=http://host.docker.internal:6001/speech-to-text-stream
LLM_URL=http://host.docker.internal:6004/prompt-stream
TTS_URL=http://host.docker.internal:6003/text-to-speech-stream
WALLET_ID=your_wallet_id_here
```

### Run the Infrastructure

To start all services, simply run:

```bash
docker-compose up
```

This command will start all the necessary services for AVR Core, including ASR, LLM, and TTS.

### Modify Providers

You can easily switch between different ASR, LLM, and TTS providers by changing the ports or URLs in the the `docker-compose.yml` file.

For example:
- To change the ASR service, update `ASR_URL` to the URL of your new ASR provider.
- To change the LLM service, update `LLM_URL` accordingly.
- To change the TTS service, update `TTS_URL` to point to your new TTS provider.

### Example Services in the Docker Compose

```yaml
services:
  avr-core:
    image: agentvoiceresponse/avr-core
    container_name: avr-core
    restart: always
    ports:
      - 5001:5001
    environment:
      - PORT=5001 
      - ASR_URL=http://host.docker.internal:6001/speech-to-text-stream
      - LLM_URL=http://host.docker.internal:6004/prompt-stream
      - TTS_URL=http://host.docker.internal:6003/text-to-speech-stream
      - WALLET_ID=
    networks:
      - avr
  # Other services here
```

## Available Integrations

For a list of available integrations (ASR, LLM, TTS), visit our [GitHub Repositories](https://github.com/orgs/agentvoiceresponse/repositories).

## Production Use and Wallet Configuration

If you are using AVR Core in a production environment, contact us at [info@agentvoiceresponse.com](mailto:info@agentvoiceresponse.com) to create your wallet. Set your `WALLET_ID` in the environment file:

```bash
WALLET_ID=your_wallet_id_here
```

In the free version, you are limited to **1 minute** of interaction per call. For extended usage, wallet credits are required (1 credit = 1 EUR = 100 minutes).

## Contributions

For those who wish to contribute to the project, please send an email to [info@agentvoiceresponse.com](mailto:info@agentvoiceresponse.com).