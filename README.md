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
});
```

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
			<p>Lorem ipsum consequat voluptate qui quis reprehenderit veniam exercitation. Deserunt non magna fugiat proident occaecat magna magna et velit laborum cupidatat aliqua est. Qui duis nostrud aliquip sint proident labore aute esse enim cupidatat ad culpa fugiat sint.</p>
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

1. Log in to your Render account and click "New +" then "Web Service".

2. Connect your Git repository.

3. Configure the deployment:
   - **Name**: fortune-cookies-astro
   - **Environment**: Node
   - **Build Command**: `npm run build`
   - **Start Command**: `npm start`

4. Add an environment variable:
   - Key: `NODE_VERSION`
   - Value: `20.17.0` (or your preferred Node.js version)

5. Click "Create Web Service".

### Step 3: Deploy your application

Render will automatically deploy your application. Once the deployment is complete, you can access your site at the provided URL.

## Verifying the Deployment

1. Visit the main URL to see a new fortune generated server-side on each refresh.
2. Visit the `/about` page to see the static content.

By following these steps, you've created an Astro application with hybrid rendering, featuring both server-side rendered and static pages, and deployed it to Render. The SSR index page will generate a new fortune on each request, while the About page remains static, demonstrating the hybrid capabilities of your Astro application.