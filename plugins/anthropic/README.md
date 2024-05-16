![Firebase Genkit + Anthropic AI](https://github.com/firebase-community/genkit-plugins/blob/main/assets/genkit-anthropic.png?raw=true)

# Anthropic AI Plugin for Firebase Genkit

This Genkit plugin allows to use Anthropic AI models through their official APIs.

If you want to use Anthropic AI models through Google Vertex AI, please refer
to the [official Vertex AI plugin](https://www.npmjs.com/package/@genkit-ai/vertexai).

## Supported models

The plugin supports the most recent Anthropic models:
**Claude 3 Opus**, **Claude 3 Sonnet**, and **Claude 3 Haiku**.

## Installation

```bash
npm install genkitx-anthropic
# or
pnpm add genkitx-anthropic
# or
yarn add genkitx-anthropic
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
import {
  anthropic /* the model(s) you want to use */,
} from 'genkitx-anthropic';

configureGenkit({
  plugins: [
    // Anthropic API key is required and defaults to the ANTHROPIC_API_KEY environment variable
    anthropic({ apiKey: process.env.ANTHROPIC_API_KEY }),
  ],
  logLevel: 'debug',
  enableTracingAndMetrics: true,
});

// ...define your flows here...
export const myFlow = defineFlow(/* ... */);

startFlowsServer();
```

### Claude 3 Haiku flow

```ts
import { generate } from '@genkit-ai/ai';
import { configureGenkit } from '@genkit-ai/core';
import { defineFlow, startFlowsServer } from '@genkit-ai/flow';
import { anthropic, claude3Haiku } from 'genkitx-anthropic';
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
      // All models also come with streaming support
      model: claude3Haiku,
      streamingCallback: ({ content }) => {
        streamingCallback?.(content[0]?.text ?? '');
      },
    });

    return llmResponse.text();
  },
);

startFlowsServer();
```
