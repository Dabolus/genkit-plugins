![Firebase Genkit + OpenAI](https://github.com/firebase-community/genkit-plugins/blob/main/assets/genkit-openai.png?raw=true)

# OpenAI Plugin for Firebase Genkit

This Genkit plugin allows to use OpenAI models through their official APIs.

## Supported models

The plugin supports several OpenAI models:

- **GPT-4o**, **GPT-4** with all its variants (**Turbo**, **Vision**), and **GPT-3.5 Turbo** for text generation;
- **DALL-E 3** for image generation;
- **Text Embedding Small**, **Text Embedding Large**, and **Ada** for text embedding generation;
- **Whisper** for speech recognition;
- **Text-to-speech 1** and **Text-to-speech 1 HD** for speech synthesis.

## Installation

```bash
npm install genkitx-openai
# or
pnpm add genkitx-openai
# or
yarn add genkitx-openai
```

The plugin depends on Genkit core packages, so make sure to have them installed as well:

```bash
npm install @genkit-ai/core @genkit-ai/ai
# or
pnpm add @genkit-ai/core @genkit-ai/ai
# or
yarn add @genkit-ai/core @genkit-ai/ai
```

## Examples

### Initializing the plugin

```ts
import { configureGenkit } from '@genkit-ai/core';
import { defineFlow, startFlowsServer } from '@genkit-ai/flow';
import { openAI /* the model(s) you want to use */ } from 'genkitx-openai';

configureGenkit({
  plugins: [
    // OpenAI API key is required and defaults to the OPENAI_API_KEY environment variable
    openAI({ apiKey: process.env.OPENAI_API_KEY }),
  ],
  logLevel: 'debug',
  enableTracingAndMetrics: true,
});

// ...define your flows here...
export const myFlow = defineFlow(/* ... */);

startFlowsServer();
```

### GPT-4o text flow

```ts
import { generate } from '@genkit-ai/ai';
import { configureGenkit } from '@genkit-ai/core';
import { defineFlow, startFlowsServer } from '@genkit-ai/flow';
import { openAI, gpt4o } from 'genkitx-openai';
import * as z from 'zod';

// ...Genkit configuration (see above)...

export const menuSuggestionFlow = defineFlow(
  {
    name: 'menuSuggestionFlow',
    inputSchema: z.string(),
    outputSchema: z.string(),
  },
  async (subject, streamingCallback) => {
    const llmResponse = await generate({
      prompt: `Suggest an item for the menu of a ${subject} themed restaurant`,
      model: gpt4o,
      // All text models also come with streaming support
      streamingCallback: ({ content }) => {
        streamingCallback?.(content[0]?.text ?? '');
      },
    });

    return llmResponse.text();
  },
);

startFlowsServer();
```

### DALL-E 3 image flow

```ts
import { generate } from '@genkit-ai/ai';
import { configureGenkit } from '@genkit-ai/core';
import { defineFlow, startFlowsServer } from '@genkit-ai/flow';
import { openAI, dallE3 } from 'genkitx-openai';
import * as z from 'zod';

// ...Genkit configuration (see above)...

export const artGenerationFlow = defineFlow(
  {
    name: 'artGenerationFlow',
    inputSchema: z.string(),
    outputSchema: z.object({
      url: z.string(),
      contentType: z.string().optional(),
    }),
  },
  async (subject, streamingCallback) => {
    const llmResponse = await generate({
      prompt: `Generate the image of a ${subject} in the style of Salvador Dali`,
      model: dallE3,
      output: {
        format: 'media',
      },
    });

    return llmResponse.media() ?? { url: '' };
  },
);

startFlowsServer();
```

### Whisper + GPT-4o + TTS1 flow

GPT-4o full multimodal capabilities are currently not available via APIs.
However, you can simulate them by using the Whisper model for speech
recognition and the TTS1 (HD) model for speech synthesis, similarly to how
previous models used to work on ChatGPT:

```ts
import { generate } from '@genkit-ai/ai';
import { configureGenkit } from '@genkit-ai/core';
import { defineFlow, startFlowsServer } from '@genkit-ai/flow';
import { openAI, whisper1, gpt4o, tts1 /* or tts1Hd */ } from 'genkitx-openai';
import * as z from 'zod';

// ...Genkit configuration (see above)...

// We are going to accept a media object as input and return a media object as output
const MediaSchema = z.object({
  url: z.string(),
  contentType: z.string().optional(),
});

export const audioFlow = defineFlow(
  {
    name: 'audioFlow',
    inputSchema: MediaSchema,
    outputSchema: MediaSchema,
  },
  async audioInput => {
    const whisperResponse = await generate({
      // Provide the media input as prompt to Whisper
      prompt: {
        media: audioInput,
      },
      model: whisper1,
    });
    const llmResponse = await generate({
      // Provide the Whisper response as text prompt to GPT-4o
      prompt: whisperResponse.text(),
      model: gpt4o,
    });
    const ttsResponse = await generate({
      // Provide the GPT-4o response as text prompt to TTS1 HD
      prompt: llmResponse.text(),
      model: tts1,
      output: {
        format: 'media',
      },
    });

    // Finally, return the output audio as media
    return ttsResponse.media() ?? { url: '' };
  },
);
```
