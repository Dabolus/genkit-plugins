# OpenAI Plugin for Firebase Genkit

## Example GPT-4 Turbo flow

```ts
import { generate } from '@genkit-ai/ai/generate';
import { flow } from '@genkit-ai/flow';
import { gpt4Turbo } from '@genkit-ai/plugin-openai';
import * as z from 'zod';

export const openaiStoryFlow = flow(
  {
    name: 'openaiStoryFlow',
    input: z.string(),
    output: z.string(),
    streamType: z.string(),
  },
  async (subject, streamingCallback) => {
    const llmResponse = await generate({
      model: gpt4Turbo,
      prompt: `Tell me a story about a ${subject}`,
      streamingCallback: ({ content }) => {
        streamingCallback?.(content[0]?.text ?? '');
      },
    });
    return llmResponse.text();
  },
);
```
