# Vercel AI SDK - Helicone Azure OpenAI Provider

The **[Helicone Azure provider](https://sdk.vercel.ai/providers/ai-sdk-providers/azure)** for the [Vercel AI SDK](https://sdk.vercel.ai/docs) contains language model support for the Helicone proxy for the Azure OpenAI API.

## Setup

The Azure provider is available in the `@ai-sdk/helicone-azure` module. You can install it with

```bash
npm i @ai-sdk/helicone-azure
```

## Provider Instance

You can import the default provider instance `helicone` from `@ai-sdk/helicone-azure`:

```ts
import { helicone } from '@ai-sdk/helicone-azure';
```

## Example

```ts
import { helicone } from '@ai-sdk/helicone-azure';
import { generateText } from 'ai';

const { text } = await generateText({
  model: helicone('gpt-4o'), // your deployment name
  prompt: 'Write a vegetarian lasagna recipe for 4 people.',
});
```

## Documentation

Please check out the **[Helicone Azure provider](https://github.com/anishanne/helicone-azure-provider)** documentation for more information.
