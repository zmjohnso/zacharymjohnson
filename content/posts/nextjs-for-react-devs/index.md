+++
title = 'Next.js for React Devs'
date = 2024-11-20T13:00:00-05:00
draft = false
tags = ["Next.js", "React"]
+++

I have always started my new web projects off with vanilla React, then added routing ([react router](https://reactrouter.com/en/main)), state management ([zustand](https://spin.atomicobject.com/zustand-alternative-redux/)), etc., once I had the need for them. As the project would grow over time, more and more packages would need to be added. Eventually, you basically end up with your own custom framework. This is why the React team [has started to recommend](https://react.dev/learn/start-a-new-react-project) building new apps and websites with an existing React framework. Some currently supported production ready frameworks include Next.js, Remix and Gatsby. However, at the time of this writing, only Next.js supports all the latest React features, such as [React Server Components](https://react.dev/reference/rsc/server-components) (RSCs). Therefore, given the React teams recommendation to start new projects off with a framework, my own experience with building "custom" React frameworks and the ability to utilize cutting edge React features such as RSCs, I have started using Next.js for my new web apps. Here are a few challenges that I ran into, from the perspective of a React developer.

## Everything is a Server Component

By default, when creating a new component with Next.js, that component will be a server component. This means that the code in that server component will be rendered on the server. Therefore, any "reactive" code (such as `useState`, `useEffect`, etc.) cannot be used in the server component. If you want to use these hooks, you will need to create a new component and annotate it with the `"use client"` directive. Note: the sever component has no such annotation. All components are simply server components by default. This design encourages you to do the bulk of the rendering on the server, and thinking very carefully about what code needs to respond to user interaction, and which code is simply "static." For example, here is the code for the `Header` component from [lalisolari.com](https://www.lalisolari.com/en):

```
import { FaInstagram } from "@react-icons/all-files/fa/FaInstagram";
import LanguageDropdown from "./languageDropdown";
import Logo from "./logo";

export default function Header() {
  return (
    <header className="flex flex-col bg-transparent shadow-none py-2">
      <div className="flex justify-between items-center">
        <Logo />
        <div className="flex flex-row pr-2 md:pr-4">
          <LanguageDropdown />
          <a
            href="https://www.instagram.com/lalisolariart/"
            target="_blank"
            rel="noopener noreferrer"
          >
            <button
              aria-label="instagram link button"
              className="p-2 rounded-full bg-white text-gray-500 hover:bg-gray-100"
            >
              <FaInstagram size={23} />
            </button>
          </a>
        </div>
      </div>
    </header>
  );
}
```

We can see that this is a server component that renders a logo, some static icons and buttons, and language dropdown component. That `LanguageDropdown` component makes use of React `useState` and `useRef`, therefore it needs to be a client component. Here is how the start of that code looks:

```
"use client";

import { useState, useRef, useEffect, useTransition } from "react";
import { MdTranslate } from "@react-icons/all-files/md/MdTranslate";
import { usePathname, useRouter } from "../navigation";
import clsx from "clsx";

export default function LanguageDropdown() {
  const router = useRouter();
  const [isPending, startTransition] = useTransition();
  const pathname = usePathname();
  const [isOpen, setIsOpen] = useState(false);
  const dropdownRef = useRef<HTMLDivElement>(null);
  ....
```

Splitting the code in this manner allows for doing the maximum amount of server side rendering, therefore reducing the JavaScript bundle size sent to the client. See [this link](https://nextjs.org/docs/app/building-your-application/rendering/server-components#benefits-of-server-rendering) from the Next.js docs for a full list of the benefits of Server Rendering. Normally, React devs are accustomed to splitting up web pages into many different components (as opposed to one page one component). Next.js and server components now also introduce the idea of splitting the components up further into the sections that can be rendered on the server and those which respond to user action (on the client).

## `<Image/>` vs `<img/>`

Another concept that I initially struggled with when switching to Next.js was the `<Image />` component. In my vanilla React apps, I was used to doing the following:

```
<img
  height="auto"
  width="100%"
  ...
/>
```

With a wrapping grid component or similar, which determined the size of the images. However, this is not possible with the Next.js Image component, as the `width` and `height` are required values which must be integers (in pixels). This is done to prevent layout shift during loading, which is a common cause of a poor user experience. However, this behavior of requiring the `width` and `height` values can make the `Image` component quite unnatural to use at first. Even still, it is important to use `Image` and not `img`, as the Next.js variant provides additional benefits, such as being responsive by default, a priority prop, ability to set a placeholder image during loading, as well as other image optimizations.

## Hosting Your Next.js App

One of my favorite ways to host React apps has been through [Cloudflare Pages](https://developers.cloudflare.com/pages/framework-guides/deploy-a-react-site/). I enjoy how simple this method is with linking my GitHub account to Cloudflare for automatic deployments as well as using Cloudflare for purchasing and setting up custom domain names. Given this, I decided to host my first Next.js app through Cloudflare pages. I began following the [Cloudflare docs](https://developers.cloudflare.com/pages/framework-guides/nextjs/ssr/get-started/) and eventually got the site deployed and "working", after adding `export const runtime = "edge";` to the top of every file. While I was able to successfully deploy the website, Next.js currently only supports the ["Edge" runtime](https://developers.cloudflare.com/pages/framework-guides/nextjs/ssr/get-started/#4-ensure-all-server-rendered-routes-use-the-edge-runtime). Next.js has two server runtimes available: Node.js (default) and Edge. The Node.js runtime has access to all Native Node.js APIs, while the Edge runtime contains a more limited set of APIs. This was a problem, as I was unable to use some packages, such as [plaiceholder](https://github.com/joe-bell/plaiceholder). It seemed like there were also other issues, such as with the Next.js `<Image/>` component optimizations and caching. Therefore, I switched to Vercel to host my website. Vercel worked great, and I was easily able to get everything up and running utilizing the latest Next.js features and optimizations. However, this soft "vendor lock" currently seems like one of the biggest downsides to using Next.js. To get the best experience, using Vercel currently feels required, and I image self-hosting a Next.js app would be even more challenging/limiting. That said, I expect better support for Next.js app hosting in the future, as well as more framework competition as products like Remix begin supporting all the latest React features.

## Next.js: the Future of React

Overall, I have been very impressed with Next.js. While it is quite opinionated about certain things (Image component, App router, etc.), the benefits are clear. Specifically, the App router is amazing, and I love using the folder and file naming conventions for pages. Given that the future of React is frameworks, I think that Next.js is well positioned to be the leading player.
