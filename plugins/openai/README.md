![Firebase Genkit + OpenAI](https://github.com/firebase-community/genkit-plugins/blob/main/assets/genkit-openai.png?raw=true)

# OpenAI Plugin for Firebase Genkit

This Genkit plugin allows to use OpenAI models through their official APIs.

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

## Example GPT-4o flow

```ts
import { generate } from '@genkit-ai/ai';
import { configureGenkit } from '@genkit-ai/core';
import { defineFlow, startFlowsServer } from '@genkit-ai/flow';
import { openAI, gpt4o } from 'genkitx-openai';
import * as z from 'zod';

configureGenkit({
  plugins: [
    // OpenAI API key is required and defaults to the OPENAI_API_KEY environment variable
    openAI({ apiKey: process.env.OPENAI_API_KEY }),
  ],
  logLevel: 'debug',
  enableTracingAndMetrics: true,
});

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
      streamingCallback: ({ content }) => {
        streamingCallback?.(content[0]?.text ?? '');
      },
    });

    return llmResponse.text();
  },
);

startFlowsServer();
```
