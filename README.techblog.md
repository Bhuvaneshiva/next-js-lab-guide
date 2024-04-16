# Building TechBlog - A Next JS app

Building Tech Blog using the App Router

-   Using JavaScript
-   Using Module CSS files for styling

## Pre-requisites

-   Good knowledge of React
-   Working knowledge of Node JS

## Software

-   Node JS installed (preferably v20) - https://nodejs.org

## References

-   Choose the App Router from the dropdown in the official Next JS documentation - https://nextjs.org/docs/getting-started/installation

## Step 1: Creating the app

-   Open the terminal folder where you would like your project to be created.
-   Run

```
npx create-next-app tech-blog
```

-   Answer the questions asked like so

```
Need to install the following packages:
create-next-app@14.2.1
Ok to proceed? (y) y
? Would you like to use TypeScript? › No / Yes - Select No
? Would you like to use ESLint? › No / Yes - Select Yes
? Would you like to use Tailwind CSS? › No / Yes - Select No
? Would you like to use `src/` directory? › No / Yes - Select Yes
? Would you like to use App Router? (recommended) › No / Yes - Select Yes
? Would you like to customize the default import alias (@/*)? › No / Yes - Select No
```

-   Your project would be created in a few moments. You will find the tech-blog folder. Navigate to the folder from within the terminal.

```
cd tech-blog
```

## Step 2: Understanding the project structure

-   The instructor shall explain the files and folders. Make sure to understand the purpose of each file and folder.
-   The project has the application code in `src/` folder beacuse we chose so at project creation time (else it will not have the folder).
-   Since we are using the App router, the page components shall go within the `app/` folder.

## Step 3: Running the development server

-   Launch the dev server by running the `dev` script

```
npm run dev
```

-   It launches on port 3000 by default. Open http://localhost:3000 in the browser - you see the page which is essentially src/app/page.tsx

## Step 4: Setup a simple Home page

-   Set up a simple home page which describes the blog website.

```tsx
export default function Home() {
    return (
        <main>
            <h1>Welcome to my blog</h1>
        </main>
    );
}
```

-   We may remove page.module.css if we do not use it.
-   Remove the global styles import in layout.js if not using the styles there.

## Step 5: Default rendering model in App router apps is React Server Components (RSCs)

-   Unlike when using Page router (where default rendering model is SSG), when using App router it is React Server Components (RSCs).

```tsx
export default function Home() {
    console.log("Home"); // this will never be logged in the browser. Even if you are navigating between pages on the client-side (after first page load)!
    return (
        <main>
            <h1>Welcome to my blog</h1>
        </main>
    );
}
```

## Step 6: Set up an About page

-   `app/about/page.tsx`

## Step 7: Set up a menu component and add it to app's layout

-   Since page component must be `page.js`, unlike the Page router, we can define non-page components in the app folder itself. Of course, we can choose not to do so to keep the app folder clean (if we consider so) .

## Step 8: Create a layout exclusively for about and nested routes

-   Add a about/layout.js and render some content in the layout folowed by the children

## Step 9: Use metadata to set up metadata for the about page

## Step 10: Set up post list page (/`posts`) and post page (`/posts/[slug]`)

## Step 11: In App router, the pages with dynamic segments receive a parms prop

-   No need to use useRouter() to get hold of params - accept and log the params prop in [/posts/slug/page.js]

# Step 12: Create a client-side component like a carousel

Tis needs to be a client component as it uses React hooks.
Also add previous and next buttons to check that components with event handlers also need to be RSC - of course the buttons instead of the carousel can be made an RSC if only event handling is needed (and not the React hooks).

## Step 13: Try the usePathname() hook on main menu links

-   Create nav-link client component (takes href, exact, children as props)

## Step 14: Initialize DB for the app

-   Run initdb.js

```js
const sql = require("better-sqlite3");
const db = sql("posts.db");

const posts = [
    {
        title: "Building a simple projector",
        slug: "bulding-a-projector",
        image: "/images/resistor.jpeg",
        summary:
            "Steps to build a simple projector using a light source and a magnifying glass.",
        author: "John Doe",
    },
    {
        title: "Create an LED cube",
        slug: "led-cube",
        image: "/images/transistor.jpg",
        summary: "Learn how to build a 3x3x3 LED cube using Arduino and LEDs.",
        author: "Jane Doe",
    },
];

db.prepare(
    `
   CREATE TABLE IF NOT EXISTS posts (
       id INTEGER PRIMARY KEY AUTOINCREMENT,
       slug TEXT NOT NULL UNIQUE,
       title TEXT NOT NULL,
       image TEXT NOT NULL,
       summary TEXT NOT NULL,
       author TEXT NOT NULL
    )
`
).run();

async function initData() {
    const stmt = db.prepare(`
      INSERT INTO posts VALUES (
         null,
         @slug,
         @title,
         @image,
         @summary,
         @author
      )
   `);

    for (const post of posts) {
        stmt.run(post);
    }
}

initData();
```

## Step 15: Setup data services for the app

-   `src/data/services/posts.js`

```tsx
import fs from "node:fs";
import sql from "better-sqlite3";
import slugify from "slugify";
import xss from "xss";

const db = sql("posts.db");

export async function getPosts() {
    // simulated delay
    await new Promise((resolve) => setTimeout(resolve, 3000));

    // uncomment to see the error page shown in the UI
    // throw new Error('Some error occured');

    return db.prepare("SELECT * FROM posts").all();
}

export function getPost(slug) {
    return db.prepare("SELECT * FROM posts WHERE slug = ?").get(slug);
}

export async function savePost(post) {
    post.title = xss(post.title);
    post.slug = slugify(post.title, { lower: true });

    const extension = post.image.name.split(".").pop();
    const fileName = `${post.slug}.${extension}`;

    // write the image to the public folder
    const stream = fs.createWriteStream(`public/images/${fileName}`);

    const bufferedImage = await post.image.arrayBuffer();

    stream.write(Buffer.from(bufferedImage), (error) => {
        if (error) {
            throw new Error("Failed to save image");
        }
    });

    post.image = `/images/${fileName}`;

    db.prepare(
        "INSERT INTO posts (title, summary, author, image, slug) VALUES (?, ?, ?, ?, ?)"
    ).run(post.title, post.summary, post.author, post.image, post.slug);
}
```

## Step 16: Use the DB services to get posts directly within the posts page RSC

-   The RSC being run exclusively on the server, can execute DB calls etc. Get the posts in the posts page and pass on to the posts component.

## Step 17: Add a loading page for posts page

## Step 18: Enabling focused streaming when loading components

The page as whole is loading. Even the top part which does not use that data is.

-   Create a function in posts page

```js
async function JustPosts() {
    const posts = await getPosts();

    return <Posts posts={posts} />;
}
```

-   Stream it in using Suspense
-   Add necessary import (the loading component)

```js
import { Suspense } from "react";
import PostsLoading from "./loading";
```

```js
Suspense fallback={<PostsLoading />}>
    <JustPosts />
</Suspense>
```

## Step 19: Add error.js and not-found.js

-   Add error.js for posts page, add not-found.js globally
-   **NOTE**: these must be client components

## Setup post details page

```js
import { getPost } from "@/data/services/posts";
import { notFound } from "next/navigation";

export default function PostPage({ params }) {
    const post = getPost(params.slug);

    if (!post) {
        notFound(); // shows the closest not found page
    }

    return (
        <div>
            <h1>{post.title}</h1>
        </div>
    );
}
```

## Getting started with server actions
