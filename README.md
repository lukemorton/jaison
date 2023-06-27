# Jaison

!["Image of Jaison mascot"](jaison.png)

```ts
import { chain } from "jaison";
import { z } from "zod";

const numberOfIdeasFormat = z.number();
const ideasFormat = z.string().array();

const ideaGenerator = chain()
  .io(numberOfIdeasFormat, ideasFormat)
  .user((input, ouput) => {
    return `Provide ${input} business ideas. ${ouput}`;
  });

const { response: ideas } = await ideaGenerator.complete(10);
console.log(ideas);
```

```ts
const { messages } = await ideaGenerator.complete(10);
console.log(messages);
// [
//   { role: "user", content: "Provide ${input} business ideas. ${ouput}" },
//   { role: "assistant", content: "[\"Lemonade stand\"]" },
// ]
```

```ts
const validationFormat = z
  .object({ idea: z.string(), validation: z.string() })
  .array();

const { chain: ideaValidator } = await ideaGenerator.complete(10);

ideaValidator
  .output(validationFormat)
  .user((_, output) => `Validate the business ideas you suggested. ${output}`);
```

```ts
import { chain } from "jaison";
import { z } from "zod";

const CUISINES =
  "Spanish, French, Japanese, Turkish, Italian, Mexican, Indian".split(", ");

const input = z.object({
  favouriteCuisines: z.enum(CUISINES).array().nonempty(),
  possibleCuisines: z.enum(CUISINES).array(),
  numberOfSuggestions: z.number().gt(0).lte(5),
});

const output = z.object({
  recipes: z.object({
    ingredients: z
      .object({
        name: z.string(),
        quantity: z.string(),
      })
      .array(),
    instructions: z.string().array(),
    cuisine: z.enum(CUISINES),
  }),
});

const recipeIdeas = chain()
  .system((input) => {
    return `
      Your job is to provide recipe ideas.

      You must provide recipes from this list of possibleCuisines:

      ${input.possibleCuisines}
    `;
  })
  .user((input, output) => {
    return `
      I love these cuisines:

      ${input.favouriteCuisines}

      Please suggest ${input.numberOfSuggestions} recipes for me to
      cook that I will like based on my favourite cuisines.

      ${output.recipes}
    `;
  });

const { lastMessage: suggestedRecipes } = await recipeIdeas.complete();

console.log(suggestedRecipes);
```

```ts
import z from "zod";
import { promptTemplate, responseFormat } from "jaison";
import { systemPrompt, userPrompt, createChatCompletion } from "jaison-openai";

const CUISINES =
  "Spanish, French, Japanese, Turkish, Italian, Mexican, Indian".split(", ");

const input = z.object({
  favouriteCuisines: z.enum(CUISINES).array().nonempty(),
  possibleCuisines: z.enum(CUISINES).array(),
  numberOfSuggestions: z.number().gt(0).lte(5),
});

const output = z.object({
  recipes: z.object({
    ingredients: z
      .object({
        name: z.string(),
        quantity: z.string(),
      })
      .array(),
    instructions: z.string().array(),
    cuisine: z.enum(CUISINES),
  }),
});

const systemPromptTemplate = promptTemplate((input) => {
  return `
    Your job is to provide recipe ideas.

    You must provide recipes from this list of possibleCuisines:

    ${input.possibleCuisines}
  `;
});

const userPromptTemplate = promptTemplate((input, output) => {
  return `
    I love these cuisines:

    ${input.favouriteCuisines}

    Please suggest ${input.numberOfSuggestions} recipes for me to
    cook that I will like based on my favourite cuisines.

    ${output}
  `;
});

const { recipes } = await createChatCompletion({
  configuration: {
    apiKey: process.env.OPENAI_API_KEY,
  },
  model: "gpt-3.5-turbo-0301",
  inputSchema: input,
  outputSchema: output,
  messages: [
    systemPrompt(systemPromptTemplate),
    userPrompt(userPromptTemplate),
  ],
  retries: 3,
  data: {
    favouriteCuisines: ["Indian", "Mexican"],
    possibleCuisines: CUISINES,
    numberOfSuggestions: 3,
  },
});

console.log(recipes);

const chatCompletion = new OpenAIChatCompletion({
  configuration: {
    apiKey: process.env.OPENAI_API_KEY,
  },
  model: "gpt-3.5-turbo-0301",
});

const chat = new Chat(chatCompletion);
const recipeListChat = await chat
  .systemMessage(systemPromptTemplate)
  .userMessage(userPromptTemplate);
const { recipes } = recipeListChat.send();
```
