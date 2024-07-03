# Helicone Azure Provider

Vercel AI Provider for running Azure OpenAI Models with Helicone Monitoring. Based on the [@ai-sdk/azure](https://www.npmjs.com/package/@ai-sdk/azure) package.

## Installation

The Helicone Azure OpenAI provider is available in the `helicone-azure` module. You can install it with

<Tabs items={['pnpm', 'npm', 'yarn']}>
  <Tab>
    <Snippet text="pnpm install helicone-azure" dark />
  </Tab>
  <Tab>
    <Snippet text="npm install helicone-azure" dark />
  </Tab>
  <Tab>
    <Snippet text="yarn add helicone-azure" dark />
  </Tab>
</Tabs>

## Provider Instance

You can import the default provider instance `helicone` from `helicone-azure`:

```ts
import { azuheliconere } from 'helicone-azure';
```

If you need a customized setup, you can import `createHelicone` from `helicone-azure` and create a provider instance with your settings:

```ts
import { createHelicone } from 'helicone-azure';

const helicone = createHelicone({
  resourceName: 'your-resource-name', // Azure resource name
  azureApiKey: 'your-azure-api-key',
  heliconeApiKey: 'your-helicone-api-key',
});
```

You can use the following optional settings to customize the OpenAI provider instance:

- **resourceName** _string_

  Azure resource name.
  It defaults to the `AZURE_RESOURCE_NAME` environment variable.

- **azureApiKey** _string_

  API key that is being send using the `api-key` header.
  It defaults to the `AZURE_API_KEY` environment variable.

- **heliconeApiKey** _string_

  API key that is being send using the `Helicone-Auth` header.
  It defaults to the `HELICONE_API_KEY` environment variable.

- **headers** _Record&lt;string,string&gt;_

  Custom headers to include in the requests.

- **fetch** _(input: RequestInfo, init?: RequestInit) => Promise&lt;Response&gt;_

  Custom [fetch](https://developer.mozilla.org/en-US/docs/Web/API/fetch) implementation.
  Defaults to the global `fetch` function.
  You can use it as a middleware to intercept requests,
  or to provide a custom fetch implementation for e.g. testing.

## Language Models

The Helicone Azure OpenAI provider instance is a function that you can invoke to create a language model:

```ts
const model = helicone('your-deployment-name');
```

You need to pass your deployment name as the first argument.

### Example

You can use OpenAI language models to generate text with the `generateText` function:

```ts
import { helicone } from 'helicone-azure';
import { generateText } from 'ai';

const { text } = await generateText({
  model: helicone('your-deployment-name'),
  prompt: 'Write a vegetarian lasagna recipe for 4 people.',
});
```

OpenAI language models can also be used in the `streamText`, `generateObject`, `streamObject`, and `streamUI` functions
(see [AI SDK Core](/docs/ai-sdk-core) and [AI SDK RSC](/docs/ai-sdk-rsc)).

<Note>
  Azure OpenAI sends larger chunks than OpenAI. This can lead to the perception
  that the response is slower. See [Troubleshooting: Azure OpenAI Slow To
  Stream](/docs/troubleshooting/common-issues/azure-stream-slow)
</Note>

### Chat Models

<Note>
  The URL for calling Azure chat models will be constructed as follows:
  `https://RESOURCE_NAME.openai.azure.com/openai/deployments/DEPLOYMENT_NAME/chat/completions?api-version=2024-05-01-preview`

  This will be proxied through the Helicone API for monitoring, logging, and rate limiting.
</Note>

Helicone Azure OpenAI chat models support also some model specific settings that are not part of the [standard call settings](/docs/ai-sdk-core/settings).
You can pass them as an options argument:

```ts
const model = helicone('your-deployment-name', {
  logitBias: {
    // optional likelihood for specific tokens
    '50256': -100,
  },
  user: 'test-user', // optional unique user identifier
  modelOverride: 'openai-chat-model-name', // optional model override
});
```

The following optional settings are available for OpenAI chat models:

- **logitBias** _Record&lt;number, number&gt;_

  Modifies the likelihood of specified tokens appearing in the completion.

  Accepts a JSON object that maps tokens (specified by their token ID in
  the GPT tokenizer) to an associated bias value from -100 to 100. You
  can use this tokenizer tool to convert text to token IDs. Mathematically,
  the bias is added to the logits generated by the model prior to sampling.
  The exact effect will vary per model, but values between -1 and 1 should
  decrease or increase likelihood of selection; values like -100 or 100
  should result in a ban or exclusive selection of the relevant token.

  As an example, you can pass `{"50256": -100}` to prevent the token from being generated.

- **logProbs** _boolean | number_

  Return the log probabilities of the tokens. Including logprobs will increase
  the response size and can slow down response times. However, it can
  be useful to better understand how the model is behaving.

  Setting to true will return the log probabilities of the tokens that
  were generated.

  Setting to a number will return the log probabilities of the top n
  tokens that were generated.

- **parallelToolCalls** _boolean_

  Whether to enable parallel function calling during tool use. Default to true.

- **user** _string_

  A unique identifier representing your end-user, which can help OpenAI to
  monitor and detect abuse. Learn more.

- **modelOverride** _string_

  Overrides the model used to calculate costs and mapping.

### Completion Models

You can create models that call the completions API using the `.completion()` factory method.
The first argument is the model id.
Currently only `gpt-35-turbo-instruct` is supported.

```ts
const model = helicone.completion('your-gpt-35-turbo-instruct-deployment');
```

OpenAI completion models support also some model specific settings that are not part of the [standard call settings](/docs/ai-sdk-core/settings).
You can pass them as an options argument:

```ts
const model = helicone.completion('your-gpt-35-turbo-instruct-deployment', {
  echo: true, // optional, echo the prompt in addition to the completion
  logitBias: {
    // optional likelihood for specific tokens
    '50256': -100,
  },
  suffix: 'some text', // optional suffix that comes after a completion of inserted text
  user: 'test-user', // optional unique user identifier
  modelOverride: 'gpt-3.5-turbo-instruct', // optional model override
});
```

The following optional settings are available for Helicone Azure OpenAI completion models:

- **echo**: _boolean_

  Echo back the prompt in addition to the completion.

- **logitBias** _Record&lt;number, number&gt;_

  Modifies the likelihood of specified tokens appearing in the completion.

  Accepts a JSON object that maps tokens (specified by their token ID in
  the GPT tokenizer) to an associated bias value from -100 to 100. You
  can use this tokenizer tool to convert text to token IDs. Mathematically,
  the bias is added to the logits generated by the model prior to sampling.
  The exact effect will vary per model, but values between -1 and 1 should
  decrease or increase likelihood of selection; values like -100 or 100
  should result in a ban or exclusive selection of the relevant token.

  As an example, you can pass `{"50256": -100}` to prevent the &lt;|endoftext|&gt;
  token from being generated.

- **logProbs** _boolean | number_

  Return the log probabilities of the tokens. Including logprobs will increase
  the response size and can slow down response times. However, it can
  be useful to better understand how the model is behaving.

  Setting to true will return the log probabilities of the tokens that
  were generated.

  Setting to a number will return the log probabilities of the top n
  tokens that were generated.

- **suffix** _string_

  The suffix that comes after a completion of inserted text.

- **user** _string_

  A unique identifier representing your end-user, which can help OpenAI to
  monitor and detect abuse. Learn more.

- **modelOverride** _string_

  Overrides the model used to calculate costs and mapping.

## Embedding Models

You can create models that call the Azure OpenAI embeddings API
using the `.embedding()` factory method.

```ts
const model = helicone.embedding('your-embedding-deployment');
```

Azure OpenAI embedding models support several additional settings.
You can pass them as an options argument:

```ts
const model = helicone.embedding('your-embedding-deployment', {
  dimensions: 512 // optional, number of dimensions for the embedding
  user: 'test-user', // optional unique user identifier
  modelOverride: 'openai-embedding-model-name', // optional model override
})
```

The following optional settings are available for Helicone Azure OpenAI embedding models:

- **dimensions**: _number_

  Echo back the prompt in addition to the completion.

- **user** _string_

  A unique identifier representing your end-user, which can help OpenAI to
  monitor and detect abuse. Learn more.

- **modelOverride** _string_

  Overrides the model used to calculate costs and mapping.
