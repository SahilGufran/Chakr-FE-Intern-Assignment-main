

## About

⭐ **Checkout the live demo of the application [here](https://example.minimal-crm.refine.dev).**

Refine is an open-source React-meta framework specifically designed to meet the requirements of enterprise companies dev teams. It comes with a core package that simplifies app requirements like data handling, authentication, access control, etc.

The repository also open-source; feel free to use or inspect it to discover how Refine works.

Being production-ready, you can either build your own **B2B applications**, **internal tools**, **admin panels**, **dashboards** and all type of **CRUD** applications. using it as a code base or directly implement it as is.

## Tutorial

This repository contains the code corresponding to an in-depth tutorial available on <a href="https://www.youtube.com/@javascriptmastery/videos" target="_blank"><b>JavaScript Mastery</b></a> YouTube channel.

<a href="https://youtu.be/6a3Dz8gwjdg" target="_blank"><img src="https://github.com/sujatagunale/EasyRead/assets/151519281/1736fca5-a031-4854-8c09-bc110e3bc16d" /></a>

## Tech Stack

- Refine
- TypeScript
- GraphQL
- Ant Design
- Codegen
- Vite

## Features

**Authentication**: Seamless onboarding with secure login and signup functionalities; robust password recovery ensures a smooth authentication experience.

**Authorization**: Granular access control regulates user actions, maintaining data security and user permissions.

**Home Page**: Dynamic home page showcases interactive charts for key metrics; real-time updates on activities, upcoming events, and a deals chart for business insights.

**Companies Page**: Complete CRUD for company management and sales processes; detailed profiles with add/edit functions, associated contacts/leads, pagination, and field-specific search.

**Kanban Board**: Collaborative board with real-time task updates; customization options include due dates, markdown descriptions, and multi-assignees, dynamically shifting tasks across dashboards.

**Account Settings**: Personalized user account settings for profile management; streamlined configuration options for a tailored application experience.

**Responsive**: Full responsiveness across devices for consistent user experience; fluid design adapts seamlessly to various screen sizes, ensuring accessibility.

<br>

![Product Edit Page](https://refine.ams3.cdn.digitaloceanspaces.com/example-readmes/CRM-Minimal/crm-demo.gif "Demo GIF")

<br>

## Try this example on your local

```bash
npm create refine-app@latest -- --example app-crm-minimal
```

This will download the files and install the necessary dependencies automatically.

Once it's done, go to the directory and run the following command to start the project:

```bash
npm run dev
```

Open http://localhost:5173 in your browser to view the project.

## Code Snippets

<details>
<summary><code>providers/auth.ts</code></summary>

```typescript
import { AuthBindings } from "@refinedev/core";

import { API_URL, dataProvider } from "./data";

// For demo purposes and to make it easier to test the app, you can use the following credentials
export const authCredentials = {
  email: "michael.scott@dundermifflin.com",
  password: "demodemo",
};

export const authProvider: AuthBindings = {
  login: async ({ email }) => {
    try {
      // call the login mutation
      // dataProvider.custom is used to make a custom request to the GraphQL API
      // this will call dataProvider which will go through the fetchWrapper function
      const { data } = await dataProvider.custom({
        url: API_URL,
        method: "post",
        headers: {},
        meta: {
          variables: { email },
          // pass the email to see if the user exists and if so, return the accessToken
          rawQuery: `
            mutation Login($email: String!) {
              login(loginInput: { email: $email }) {
                accessToken
              }
            }
          `,
        },
      });

      // save the accessToken in localStorage
      localStorage.setItem("access_token", data.login.accessToken);

      return {
        success: true,
        redirectTo: "/",
      };
    } catch (e) {
      const error = e as Error;

      return {
        success: false,
        error: {
          message: "message" in error ? error.message : "Login failed",
          name: "name" in error ? error.name : "Invalid email or password",
        },
      };
    }
  },

  // simply remove the accessToken from localStorage for the logout
  logout: async () => {
    localStorage.removeItem("access_token");

    return {
      success: true,
      redirectTo: "/login",
    };
  },

  onError: async (error) => {
    // a check to see if the error is an authentication error
    // if so, set logout to true
    if (error.statusCode === "UNAUTHENTICATED") {
      return {
        logout: true,
        ...error,
      };
    }

    return { error };
  },

  check: async () => {
    try {
      //  get the identity of the user
      // this is to know if the user is authenticated or not
      await dataProvider.custom({
        url: API_URL,
        method: "post",
        headers: {},
        meta: {
          rawQuery: `
            query Me {
              me {
                name
              }
            }
          `,
        },
      });

      // if the user is authenticated, redirect to the home page
      return {
        authenticated: true,
        redirectTo: "/",
      };
    } catch (error) {
      // for any other error, redirect to the login page
      return {
        authenticated: false,
        redirectTo: "/login",
      };
    }
  },

  // get the user information
  getIdentity: async () => {
    const accessToken = localStorage.getItem("access_token");

    try {
      // call the GraphQL API to get the user information
      // we're using me:any because the GraphQL API doesn't have a type for the me query yet.
      // we'll add some queries and mutations later and change this to User which will be generated by codegen.
      const { data } = await dataProvider.custom<{ me: any }>({
        url: API_URL,
        method: "post",
        headers: accessToken
          ? {
              // send the accessToken in the Authorization header
              Authorization: `Bearer ${accessToken}`,
            }
          : {},
        meta: {
          // get the user information such as name, email, etc.
          rawQuery: `
            query Me {
              me {
                id
                name
                email
                phone
                jobTitle
                timezone
                avatarUrl
              }
            }
          `,
        },
      });

      return data.me;
    } catch (error) {
      return undefined;
    }
  },
};
```

</details>

<details>
<summary>GraphQl and Codegen Setup</summary>

```bash
npm i -D @graphql-codegen/cli @graphql-codegen/typescript @graphql-codegen/typescript-operations @graphql-codegen/import-types-preset prettier vite-tsconfig-paths
```

</details>

<details>
<summary><code>graphql.config.ts</code></summary>

```typescript
import type { IGraphQLConfig } from "graphql-config";

const config: IGraphQLConfig = {
  // define graphQL schema provided by Refine
  schema: "https://api.crm.refine.dev/graphql",
  extensions: {
    // codegen is a plugin that generates typescript types from GraphQL schema
    // https://the-guild.dev/graphql/codegen
    codegen: {
      // hooks are commands that are executed after a certain event
      hooks: {
        afterOneFileWrite: ["eslint --fix", "prettier --write"],
      },
      // generates typescript types from GraphQL schema
      generates: {
        // specify the output path of the generated types
        "src/graphql/schema.types.ts": {
          // use typescript plugin
          plugins: ["typescript"],
          // set the config of the typescript plugin
          // this defines how the generated types will look like
          config: {
            skipTypename: true, // skipTypename is used to remove __typename from the generated types
            enumsAsTypes: true, // enumsAsTypes is used to generate enums as types instead of enums.
            // scalars is used to define how the scalars i.e. DateTime, JSON, etc. will be generated
            // scalar is a type that is not a list and does not have fields. Meaning it is a primitive type.
            scalars: {
              // DateTime is a scalar type that is used to represent date and time
              DateTime: {
                input: "string",
                output: "string",
                format: "date-time",
              },
            },
          },
        },
        // generates typescript types from GraphQL operations
        // graphql operations are queries, mutations, and subscriptions we write in our code to communicate with the GraphQL API
        "src/graphql/types.ts": {
          // preset is a plugin that is used to generate typescript types from GraphQL operations
          // import-types suggests to import types from schema.types.ts or other files
          // this is used to avoid duplication of types
          // https://the-guild.dev/graphql/codegen/plugins/presets/import-types-preset
          preset: "import-types",
          // documents is used to define the path of the files that contain GraphQL operations
          documents: ["src/**/*.{ts,tsx}"],
          // plugins is used to define the plugins that will be used to generate typescript types from GraphQL operations
          plugins: ["typescript-operations"],
          config: {
            skipTypename: true,
            enumsAsTypes: true,
            // determine whether the generated types should be resolved ahead of time or not.
            // When preResolveTypes is set to false, the code generator will not try to resolve the types ahead of time.
            // Instead, it will generate more generic types, and the actual types will be resolved at runtime.
            preResolveTypes: false,
            // useTypeImports is used to import types using import type instead of import.
            useTypeImports: true,
          },
          // presetConfig is used to define the config of the preset
          presetConfig: {
            typesPath: "./schema.types",
          },
        },
      },
    },
  },
};

export default config;
```

</details>

<details>
<summary><code>graphql/mutations.ts</code></summary>



