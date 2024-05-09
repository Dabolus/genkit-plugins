# OpenAI Plugin for Firebase Genkit

## Example Claude 3 Haiku flow

```ts
import { generate } from '@genkit-ai/ai/generate';
import { flow } from '@genkit-ai/flow';
import { claude3Haiku } from './plugins/anthropic';
import * as z from 'zod';

export const anthropicStoryFlow = flow(
  {
    name: 'anthropicStoryFlow',
    input: z.string(),
    output: z.string(),
    streamType: z.string(),
  },
  async (subject, streamingCallback) => {
    const llmResponse = await generate({
      model: claude3Haiku,
      prompt: `Tell me a story about a ${subject}`,
      streamingCallback: ({ content }) => {
        streamingCallback?.(content[0]?.text ?? '');
      },
    });
    return llmResponse.text();
  },
);
```
