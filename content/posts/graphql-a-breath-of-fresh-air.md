+++
title = 'GraphQL: a Breath of Fresh Air'
date = 2024-10-03T21:30:50-04:00
draft = false
+++

Recently, upon migrating [Lali Solari's](https://www.lalisolari.com/) personal website from vanilla React to
Next.js (see [this Contentful blog post](https://www.contentful.com/blog/integrate-contentful-next-js-app-router/)
for a great guide on getting started with Next.js App Router and Contentful), I decided to try out the Contentful GrapQL API.
In the past, I have always used the [Contentful Javascript SDK](https://github.com/contentful/contentful.js), which is a great tool for
interacting with the Contentful REST API. However, I wanted to use the native web fetch API to take advantage of specific Next.js
features, such as caching, so I decided to give the GraphQL API a try. Thanks to the GraphiQL browser tool and the ability to only get
back the data that I need, I found the Contentful GraphQL API experience even better than working with the Contentful Javascript SDK.

## GraphiQL as TDD?

GraphiQL is an incredible browser based development tool. It allows you to write queries and mutations, with auto-completion, in the
browser before even writing a single line of code. Contentful's GraphQL API supplies the GraphiQL tool environment, which can be reached
at https://graphql.contentful.com/content/v1/spaces/SPACE_ID/explore?access_token=YOUR_ACCESS_TOKEN. Using the in-browser text editor, I was able to quickly write queries to get the data that I needed, and then test them in the browser. While this is not a replacement for tests in the codebase, it is a great way to see what the return data will look like, before writing any code. But furthermore, this is not wasted effort, as the queries that you write in GraphiQL can be copied and pasted directly into your codebase.

Here is what one of my GraphiQL queries looked like:
![GraphiQL Query](/images/graphql-a-breath-of-fresh-air/graphiql_query.png)
Copying this directly into my codebase, I get the following.

```typescript
const MANIFESTO_PAGE_QUERY = `
  query ($locale: String!) {
    manifiestoCollection(locale: $locale) {
      items {
        title
        description
        mediaCollection {
          items {
            title
            url
          }
        }
      }
    }
  }
`;
```

## Exactly the Data Needed

My other favorite thing about using the Contentful GraphQL API was the ability to get only the data I needed, with minimal boilerplate code. For reference, here is what the call to get the Manifesto page data was in the original React project, using the Contentful SDK:

```typescript
const client = getClient();
const res = await client.getEntries<Manifesto>({
  content_type: "manifiesto",
  locale: languageMode,
});
```

where `Manifesto` was

```typescript
export interface Manifesto {
  title: string;
  description: string;
  media: Photo[];
}

export interface Photo {
  fields: {
    description: string;
    file: {
      contentType: string;
      details: {
        image: {
          width: number;
          height: number;
        };
        size: number;
      };
      fileName: string;
      url: string;
    };
    title: string;
  };
  sys: {
    id: string;
  };
}
```

As you can see, these are very verbose types that I needed to define myself based on the response
from the REST API call via `client.getEntries<Manifesto>`. While I still need to define the response type
myself when using the GraphQL API, it is much simpler given that the `Photo` array is now just a `mediaCollection`
on which I can simply grab the `title` and `url` (both of which are strings). This not only simplifies the return type,
but also reduces the amount of response code returned via the API call.

## Potential GraphQL Drawbacks

The only complaint that I had with using the Contentful GraphQL API versus the REST API is needing to manually
create the fetch call. Here is what that ended up looking like for me:

```typescript
const res = await fetch(
  `https://graphql.contentful.com/content/v1/spaces/${process.env.CONTENTFUL_SPACE_ID}/environments/${process.env.CONTENTFUL_ENVIRONMENT_ID}`,
  {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      Authorization: `Bearer ${process.env.CONTENTFUL_CONTENT_DELIVERY_API_ACCESS_TOKEN}`,
    },
    body: JSON.stringify({ query, variables }),
    next: { tags: ["portfolioContent"] },
  }
);
```

However, in my case this was required in order to take advantage of the Next.js enhanced fetch API (see the `next` tag above that facilitates caching). Moreover, this logic only needs to be written once and can then be encapsulated in a function to be used with multiple queries.

## Don't Default to REST

Because I was familiar with the Contentful Javascript SDK and that way of querying data, I had simply defaulted to this in my new project.
Even though I have used GraphQL before, I was stuck in a RESTful, traditional API mindset. But given the ability to get exactly the data that 
is need and the excellent developer tools of GraphiQL and associated type completion, I am very glad I tried out using GraphQL again.
I hope you also consider using GraphQL in the future!