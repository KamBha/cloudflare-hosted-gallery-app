# sv

Everything you need to build a Svelte project, powered by [`sv`](https://github.com/sveltejs/cli).

## Creating a project

If you're seeing this, you've probably already done this step. Congrats!

```bash
# create a new project in the current directory
npx sv create

# create a new project in my-app
npx sv create my-app
```

## Developing

Once you've created a project and installed dependencies with `npm install` (or `pnpm install` or `yarn`), start a development server:

```bash
npm run dev

# or start the server and open the app in a new browser tab
npm run dev -- --open

# Should only need to run once. This will create the DB in cloudflare. It will also print out the database_id which you need to copy the value of and paste into 
# the wrangler.json file under d1_database[0].database_id
npx wrangler d1 create gallery-db  

npx wrangler d1 execute "gallery-db" --local --file=./db/createdb.sql && npx wrangler d1 execute "gallery-db" --local --file=./db/testData.sql
```

## Building

To create a production version of your app:

```bash
npm run build
```

You can preview the production build with `npm run preview`.

> To deploy your app, you may need to install an [adapter](https://svelte.dev/docs/kit/adapters) for your target environment.
