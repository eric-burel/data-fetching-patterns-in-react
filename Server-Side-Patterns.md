## Shifting to the Server Side

Like most typical React applications nowadays, the application we’re building so far are purely rendered on client side. However, such application has a significant drawback on SEO, as when the search engine robot crawls our application URL, it won’t be able to get the full content but a meaningless `<div id="root"></div>`. 

### A Brief Introduction of SSR

While certain search engines (such as Google) have the capability to process JavaScript, depending entirely on client-side JavaScript for content rendering can lead to issues, including delays in indexing or capturing incomplete content.

SSR was invented to solve this problem. Server-side rendering (SSR) is a technique where web page content is generated on the server and sent to the client as fully formed HTML, enabling faster initial page loads and improved SEO by making content immediately available to search engines.

Traditionally, in SSR, the components are rendered to HTML strings on the server, which are then sent to the client. This process hydrates the static markup with event handlers on the client side, turning it into a fully interactive application without needing to fetch and render the initial content on the client, enhancing performance and SEO.

For example, you could use `renderToString` API to render the component tree into a HTML tree:

```jsx
import React from 'react'
import { renderToString } from 'react-dom/server'

import App from './App';

const html = renderToString(<App/>);

console.log(html);
```

The above application is a normal Node script that can be executed in the backend server, it prints out the application on the console.

```html
<div><div><h1>Juntao Qiu</h1><p>Developer, Educator, Author</p></div></div>
```

You can then place such content into the `<div id="root"></div>`, and then in the frontend, React can hydrate the application with user interactions. It can save the first round of render - which can be a great time saver when the application is big.

![Server-Side Rendering with hydrate](images/timeline-1-7-ssr-hydrate-trans.png)

## Pattern: Static Site Generation

Static Site Generation (SSG) is a method where web pages are pre-rendered at build time, resulting in fast-loading, static HTML files ready for deployment.

Unlike dynamic sites that generate content on the fly with each request, SSG prepares all pages in advance. This approach means that a server simply serves pre-built HTML, CSS, and JavaScript files to the browser without the need to execute server-side code or database queries at runtime.

Static Site Generation have become a popular choice for blog creation, especially among users who prefer writing in Markdown, such as those using Jekyll on GitHub Pages. These tools are favored for their simplicity, allowing content creators to focus solely on their writing without worrying about the underlying technology.

SSGs aren't just limited to converting Markdown into HTML, they have broader applications. For instance, content can be managed through a Content Management System (CMS), where it is stored and maintained separately from the website's codebase. This setup is particularly advantageous for product catalogs, where content designers and producers input information into the CMS, and an SSG periodically generates a static website. This process results in websites that load very quickly because all the data fetching and content generation occur before the site is published. Once generated, the content on these sites remains unchanged until the next update cycle.

Static Site Generation (SSG) offers numerous benefits, including enhanced load speeds due to the ability to serve static files swiftly from a Content Delivery Network (CDN). It also bolsters security since it eliminates direct interaction with databases or server-side applications. Additionally, SSG can significantly improve SEO outcomes because the content becomes immediately available upon page load.

Like other patterns we discussed so far, SSG's utility isn't confined to any specific frontend library; its application predates many of these libraries. Developers have the flexibility to implement SSG using various backend technologies, such as PHP, Ruby, or Java. However, in contexts where you and your team possess expertise in building client-side React applications, or where existing logic from such applications can be repurposed for static site generation, leveraging isomorphic React could be particularly advantageous.

### Implementing Static Site Generation in React

For instance, if we want to generate some advertisements when the website is built (instead of everytime when user request the page), we can define a React Server Component like the following:

```jsx
async function getAds(): Promise<Ad[]> {
  return await get("/ads");
}

export async function Ads() {
  const ads = await getAds();

  return (
    <div>
      <h2>Ads</h2>
      <div>
        {ads.map((ad) => (
          <div key={ad.id}>
            <h3>{ad.title}</h3>
            <p>{ad.content}</p>
          </div>
        ))}
      </div>
    </div>
  );
}
```

The `Ads` component above only fetches data at build time rather than runtime. When the site is being built, `getAds` is called to fetch advertisements data from the `/ads` endpoint. This async function retrieves an array of ads, and this data becomes immediately available as part of the component's build process. 

![Static Generated Content - Ads](images/ads.png)

The component then renders HTML for each advertisement, including the ad's title and content. Since the ads are fetched during the build process, the rendered page with all its advertisements is served as static HTML to the user. This means there's no need for the client's browser to run JavaScript to fetch the ads after the page loads, leading to faster page rendering and an improved user experience. 

If we visualize the timeline, you can clearly see that the effort to render the full version of the page is pre-made. Thus, there's no need to fetch data dynamically through side effects.

![Static Site Generation](images/timeline-1-8-static-site-generation-trans.png)

It's important to understand that Static Site Generation (SSG) is seldom employed in isolation when constructing a web application. Invariably, you'll require certain content to be tailored to individual users or specific scenarios (for instance, pages like a `Profile` or `My Orders` cannot be pre-generated since their content depends on user interaction with the application). Thus, SSG should be viewed as an adjunct method, rather than a solitary approach, for website development. The key lies in discerning which content can be pre-generated to improve the performance and thus user experience.

## Pattern: Server Component

Server Components are components that are rendered on the server side, enabling operations like data fetching and content generation without sending the component's code or logic to the client, thus reducing the amount of JavaScript needed for the page and improving load times.

Traditional Server-Side Rendering (SSR) does not inherently support client-side data fetching mechanisms (like `useEffect` in React applications), which only operates in a browser environment. This limitation necessitates a system that enables data to be fetched on the server side. 

Server Components address this challenge by enhancing asynchronous programming techniques throughout applications. Similar mechanisms for advanced Server-Side Rendering exist in other libraries as well. However, for the purpose of our discussion on the `Profile` example, we'll continue to focus on utilizing React Server Components.

### Introducing React Server Component

React Server Components allow rendering components on the server, reducing client-side bundle size and improving performance by fetching data and executing logic server-side. They seamlessly integrate with client components for an optimized, interactive user experience.

We can modify our `Friends` client component into the following, note how we call API for fetching data in the `Friends` component:

```tsx
async function getFriends(id: string) {
  return await get<User[]>(`/users/${id}/friends`);
}

async function Friends({ id }: { id: string }) {
  const friends = await getFriends(id);

  return (
    <div>
      <h2>Friends</h2>
      <div>
        {friends.map((user) => (
          <Friend user={user} key={user.id} />
        ))}
      </div>
    </div>
  );
}
```

This Server Component above `Friends` showcases direct server-side data fetching with an `async` function to retrieve a user's friends. Unlike traditional client components that initiate data fetching effects (`useEffect`) and manage loading states within the browser, server components fetch data during **server-side rendering**. The `getFriends` call executes on the server, with the resulting friends list rendered into HTML before reaching the client. 

Within the ecosystems of other libraries such as Vue.js, there exist comparable solutions, notably Nuxt for Vue.js and Next.js for React. These frameworks offer built-in server-side rendering (SSR) capabilities, including data fetching, allowing the patterns discussed here to be similarly applied, albeit through their distinct approaches.

Please be aware that in both Vue.js and React, Server Components are not yet deemed ready for production use, even though libraries and frameworks are actively evolving and embracing this emerging pattern.

## Pattern: Streaming Server-Side Rendering

Streaming Server-Side Rendering (SSR) is an advanced technique that allows servers to send partial HTML responses to the browser as page components are being rendered, rather than waiting for the entire page to render server-side before sending it. This method significantly improves the time to first byte (TTFB) and the overall perceived loading speed of web applications, offering users quicker access to content.

From React 18 onwards, several streaming rendering APIs have been introduced. Similar to how we handle I/O operations, we don't need to load everything into memory at once. Instead, we process the data in smaller chunks and in a streaming manner. With streaming, we can immediately render what's available to the user without waiting for all content to be ready.

Additionally, the new Suspense boundary makes server-side rendering (SSR) more powerful than before. For example, let's review the Profile component with React Server Component:

```jsx
export async function Profile({ id }: { id: string }) {
  return (
    <div>
      <h1>Profile</h1>
      <div>
        <div>
          <Suspense fallback={<UserBriefSkeleton />}>
            <UserBrief id={id} />
          </Suspense>
        </div>
        <div>
          <Suspense fallback={<FriendsSkeleton />}>
            <Friends id={id} />
          </Suspense>
        </div>
      </div>
    </div>
  );
}
```

With streaming SSR (Server-Side Rendering), the `Profile` component leverages React's asynchronous and concurrent capabilities to enhance performance and user experience. When rendered on the server, this component initiates the streaming of HTML to the client as soon as possible, even before all data dependencies are resolved.

We could break it down into a few steps:

Initially, SSR might return the following HTML structure. Because the data isn't ready, so the fallbacks defined in suspense boundaries are used:

```html
<div>
  <h1>Profile</h1>
  <div>
    <div>
      <p>Loading user...</p>
    </div>
    <div>
      <p>Loading friends...</p>
    </div>
  </div>
</div>
```

And then, as the user data becomes available first, only the HTML content of the UserBiref is returned to client side, and the Loading user is replaced:

```html
<div>
  <h1>Profile</h1>
  <div>
    <div>
      <div><div><h1>Juntao Qiu</h1><p>Developer, Educator, Author</p></div></div>
    </div>
    <div>
      <p>Loading friends...</p>
    </div>
  </div>
</div>
```

From the end user's perspective, the application not only appears to be working, but it actually is interactive. You can engage with the loaded parts as soon as they're ready, while the rest of the content continues to load.

![Streaming Server-Side Rendering](images/timeline-1-9-streaming-ssr-trans.png)

### Nesting Suspense overview

Continue on the previous example, let's have a look at how nesting suspense works with Streaming SSR. As I have mentioned in the Parallel Data Fetching section that not all the requests can be paralleled. For instance, to get a feeds recommendataion, we need to fetch user data for the "interests" list first:

```jsx
{
  "id": "u1",
  "name": "Juntao Qiu",
  "bio": "Developer, Educator, Author",
  "interests": [
    "Technology",
    "Outdoors",
    "Travel"
  ]
}
```

Then we can invoke `/users/recommendations/<interest>` to fetch some feed recommendations for the user. That means we can create another layer - define a component called `UserInfo`, and inside it we get user brief and render it. At the same time, we use Suspense and React Server Component to define `Feeds`, which will do the data fetching itself and render when the data is ready.

```jsx
export async function UserInfo({ id }: { id: string }) {
  const user = await getUser(id);
 
  return (
    <>
      <UserBrief user={user} />
      <Suspense fallback={<FeedsSkeleton />}>
        <Feeds category={user.interests[0]} />
      </Suspense>
    </>
  );
}
```

Please note the Suspense is nested. By allowing individual components that fetch data asynchronously to be wrapped in their own **`Suspense`** components, each with bespoke fallback content, React enables a more nuanced and user-friendly loading experience. This pattern permits different parts of an application to independently manage their loading states, even when nested within a complex component hierarchy.

The benefits of this approach are multifold. It leads to an improvement in user experience by ensuring that users receive immediate and meaningful feedback. The modularization of loading states simplifies code maintenance and enhances readability, as developers can focus on the loading logic of individual components rather than orchestrating state across the entire application. Moreover, this model optimizes performance by allowing React to render parts of the application as soon as their data becomes available, rather than waiting for an entire component tree to load. In essence, nested **`Suspense`** empowers developers to build more reactive, resilient, and user-centric React applications.

Finally, I'd like to discuss Static Site Generation (SSG), which is equally important. In many cases, when we know what data to fetch before the user makes a request, we can make your web application extremely fast and responsive. Keep in mind that to build a web application, you'll typically need to combine Static Site Generation (SSG) with other data fetching patterns.

We've covered a wide range of patterns and how they apply to various challenges. I realize there's quite a bit to take in, from code examples to diagrams. If you're looking for a more guided approach, I've put together [a comprehensive tutorial](https://www.icodeit.com.au/tutorials/advanced-network-patterns-react) on my website. It offers a more interactive exploration of these concepts, so don't hesitate to check it out for a deeper dive into the subject.

---

- **Static Site Generation (SSG)**: SSG proves invaluable for static content, working alongside dynamic rendering to speed up load times and optimize resource use.
- **Server Component**: This pattern involves rendering components directly on the server, allowing for rich interactions without sending the component code to the client. It reduces the amount of JavaScript required on the client-side, speeding up load times and improving overall performance by executing data fetching, templating, and rendering on the server.
- **Streaming Server-Side Rendering (SSR)**: Implementing streaming can incrementally enhance the user experience by serving content as soon as it's ready. Its practical application may vary based on backend solutions like Next.js.
