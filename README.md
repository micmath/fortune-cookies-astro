# fortune-cookies-astro

This is a demonstration of how to create a web application with Astro that uses a combination of SSG (statically generated pages) and SSR (server-side rendered pages) that can be deployed on the Render hosting service.

These are the basic steps:

## Prerequisites

Confirm that you have a recent version of NodeJS available.

```
❯ node -v                                   
v20.17.0
```

## Initializing and Configuring the Project

### Step 1: Create the Astro project

This will run the Astro set up script which will present some questions for how you want to configure the project:

```bash
❯ npm create astro@latest fortune-cookies-astro
❯ cd fortune-cookies-astro
```

Choose the following options:

- How would you like to start your new project? **Empty**
- Do you plan to write TypeScript? **Yes**
- How strict should TypeScript be? **Relaxed**
- Install dependencies? **Yes**
- Initialize a new git repository? **Yes**

### Step 2: Install additional dependencies

The `fortune-messages` module will generate the messages, `@astrojs/node` allows us to run Node SSR code.

```bash
❯ npm install fortune-messages @astrojs/node
```

### Step 3: Configure Astro for hybrid rendering

Update the `astro.config.mjs` to look like the following:

```javascript
import { defineConfig } from 'astro/config';
import node from '@astrojs/node';

export default defineConfig({
  output: 'hybrid',
  adapter: node({
    mode: 'standalone'
  }),
  server: {
    host: '0.0.0.0',
    port: 3000
  }
});
```

**Note** the `server` entry... The `host` value must be `0.0.0.0` to be runnable on Render. The `port` value can be [any number supported by Render](https://docs.render.com/web-services#port-binding) but whatever value you choose, you must **also** create a `PORT` environmental variable in your Render project with the same value as in your `astro.config.mjs` file (this is mentioned later).

### Step 4: Create the index page (SSR)

Create the SSR page `src/pages/index.astro`. Remember to disable prerender on any SSR pages by exporting `prerender = false`.

```astro
---
import fortune from 'fortune-messages';
const message: string = fortune();

export const prerender = false;
---

<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width" />
    <title>Fortune Cookies</title>
  </head>
  <body>
		<nav>
			<p>
				<a href="/">Home</a> <a href="/about">About</a>
			</p>
		</nav>
		<main>
			<h1>Your Fortune</h1>
			<p>{message}</p>
		</main>
  </body>
</html>
```

### Step 5: Create the About page (static)

Create `src/pages/about.astro`:

```astro
---
// prerendered by default
---

<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width" />
    <title>Fortune Cookies</title>
  </head>
  <body>
		<nav>
			<p>
				<a href="/">Home</a> <a href="/about">About</a>
			</p>
		</nav>
		<main>
			<h1>About</h1>
			<p>Lorem ipsum dolor sit amet consequat voluptate qui quis reprehenderit veniam exercitation. Deserunt non magna fugiat proident occaecat magna magna et velit laborum cupidatat aliqua est. Qui duis nostrud aliquip sint proident labore aute esse enim cupidatat ad culpa fugiat sint.</p>
		</main>
  </body>
</html>
```

### Step 6: Update package.json

Add a `start` script to the project `package.json`. You should only need to update the script named `start` and can leave the other scripts as they are:

```json
{
  "scripts": {
    "dev": "astro dev",
    "start": "node ./dist/server/entry.mjs",
    "build": "astro build",
    "preview": "astro preview",
    "astro": "astro"
  }
}
```

Now you can run and test out your project in developer mode by running:

```
❯ npm run dev

> fortune-cookies-astro@0.0.1 dev
> astro dev

11:21:28 [types] Generated 1ms

 astro  v4.15.9 ready in 100 ms

┃ Local    http://localhost:3000/
┃ Network  use --host to expose

11:21:28 watching for file changes...
```

## Deploying to Render

### Step 1: Commit the project to GitHub

Since we chose to "Initialize a new git repository" in the Astro setup menu above, you should already have the git repo initialised:

```bash
❯ git status                                                
On branch main
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	modified:   README.md
	modified:   astro.config.mjs
	modified:   package-lock.json
	modified:   package.json
	modified:   src/pages/index.astro

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	src/pages/about.astro

no changes added to commit (use "git add" and/or "git commit -a")
```

Create a new repository in GitHub and note the HTTPS url. Connect that repo to your local project repository:

```bash
❯ git remote add origin https://github.com/micmath/fortune-cookies-astro.git
```

Commit and push your project files as usual.

### Step 2: Set up a new Web Service on Render

This assumes you have an account on https://render.com/ ... If not create one. This demo will run on their Free Tier but there may be a _significant_ delay when starting up any cold SSR route in your web application on that tier. Upgrade to a paid tier to get better performance more suitable for production.

1. Log in to your Render account and click "New +" then "Web Service".

2. Connect your GitHub repository. You will need to grant Render access to your GitHub repos, so follow those steps as they are presented to you.

3. Configure the deployment in your Render project (dashboard):
   - **Name**: fortune-cookies-astro
   - **Environment**: Node
   - **Build Command**: `npm install && npm run build`
   - **Start Command**: `npm start`

4. Add the following environment variables in your Render project (dashboard):
   - Key: `NODE_VERSION`
   - Value: `20.17.0` (or whatever your preferred node version is)
  - Key: `PORT`
  - Value: `3000` (must match the `server.port` value in your `astro.config.mjs`, see above)

5. Click the "Deploy Web Service" button in your Render project (dashboard) and wait while the project is built and deployed. The process is logged in your Render project (dashboard) console.

## Verifying the Deployment

Find the URL of your web application in your Render project (dashboard). For example, my project can be viewed at https://fortune-cookies-astro.onrender.com/

- Visit the `/` page to see the dynamic SSR content.
- Visit the `/about` page to see the static SSB content.
