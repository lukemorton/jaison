# Jaison

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

type TemplateVariables = z.infer<typeof input>;

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
  messages: [systemPrompt(systemPrompt), userPrompt(userPromptTemplate)],
  retries: 3,
  data: {
    favouriteCuisines: ["Indian", "Mexican"],
    possibleCuisines: CUISINES,
    numberOfSuggestions: 3,
  },
});

console.log(recipes);
```
