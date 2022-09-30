**What is The Graph client?**

The Graph Client is a tool/library from The Graph aimed at making network-related data consumption simpler for dApps thus  offering everything a dApp requires.


Here are some of the features of The Graph Client

1. Multiple indexers

2. Client-Side Composition

3. Cross-chain Subgraph Handling

4. Automatic Block Tracking

5. Client-Side Composition 

6. Automatic Pagination


**Installing The Graph Client CLI**

You add The Graph Client CLI as a dependency to your dApp. The Graph Client CLI comes with a built-in GraphiQL so you can experiment with queries in real time.

You can that using either Yarn or Npm, run either of the commands below

**WITH YARN**

```yarn add -D @graphprotocol/client-cli ```

**WITH NPM**

```npm install --save-dev @graphprotocol/client-cli```

In your root folder, create a ```yaml``` config file and name it ```.graphclientrc.yml``` Point it to your deployed/published GraphQL endpoints provided by The Graph. It should look like this 

**.graphclientrc.yml**

```javascript
sources:
  - name: uniswapv2
    handler:
      graphql:
        endpoint: https://api.thegraph.com/subgraphs/name/uniswap/uniswap-v2
```

Now, go ahead to run the command

```graphclient build```

or specify as a command in your script command in your package.json incase you want to run with ```npm``` or ```yarn```. If you wish to also use the built-in devtool GraphiQL, you can add the command ```graphclient serve-dev```

**package.json**

```json  
"scripts": {
    "build": "graphclient build",
    "dev": "graphclient serve-dev"
  }

```

The result of this command will generate the .graphclient artifact as typescript files as shown below:

```cmake
GraphClient: Cleaning existing artifacts
GraphClient: Reading the configuration
🕸️: Generating the unified schema
🕸️: Generating artifacts
🕸️: Generating index file in TypeScript
🕸️: Writing index.ts for ESM to the disk.
🕸️: Cleanup
🕸️: Done! => .graphclient
```

To use the artifact, you can import it directly in your dApp like this

```javascript
import { execute } from '../.graphclient'

//DEFINE YOUR QUERY

async function main() {
  const result = await execute(myQuery, {})
  console.log(result)
}
```

You will notice as stated earlier above, the artifacts files are generated in typescript. You can decide to use JavaScript instead. The CLI allows for js customization so you can generate JavaScript and JSON files alongside additional TypeScript definition files by using ```--fileType js``` or ```--fileType json```. 

# A practical approach to using The Graph Client with Next.js

We shall be taking a look on how to query two subgraphs ([Enzyme](https://thegraph.com/hosted-service/subgraph/enzymefinance/enzyme) and [Balancer](https://thegraph.com/hosted-service/subgraph/balancer-labs/balancer)) from a simple frontend. You can start by creating a [Next.js + TS](https://nextjs.org/docs/basic-features/typescript) application from scratch or simply clone the project at [https://app.radicle.network/seeds/pine.radicle.garden/rad:git:hnrkp9gcfqg3fqt5u1am1fhh97ggf619aihgy](https://app.radicle.network/seeds/pine.radicle.garden/rad:git:hnrkp9gcfqg3fqt5u1am1fhh97ggf619aihgy)

**Cloning the Starter Project from Radicle**

If you will like to clone the project, you need to have the **Rad CLI** installed then proceed to run the following command:

`rad clone rad://pine.radicle.garden/hnrkp9gcfqg3fqt5u1am1fhh97ggf619aihgy`

Here is a sample of what the clone view is like

![](https://i.imgur.com/pFPVKt3.png)

Proceed to **`cd`** into the project folder.

**Installing all required dependencies and devDependencies**

With either `YARN` or `NPM`, make sure the following dependencies are installed

Dependencies

    "graphql"
    "next":
    "react",
    "react-dom"
    
DevDependecies

    "@graphprotocol/client-cli",
    "@types/node",
    "@types/react",
    "eslint",
    "eslint-config-next",
    "typescript"
    
Your package.json file should look similar to this

```json
{
  "name": "nextjs-example",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "build-client": "graphclient build",
    "graphiql": "graphclient serve-dev"
  },
  "dependencies": {
    "graphql": "^16.6.0",
    "next": "12.2.5",
    "react": "18.2.0",
    "react-dom": "18.2.0"
  },
  "devDependencies": {
    "@graphprotocol/client-cli": "2.2.3",
    "@types/node": "17.0.26",
    "@types/react": "18.0.17",
    "eslint": "8.22.0",
    "eslint-config-next": "12.2.5",
    "typescript": "4.7.4"
  }
}
```


If you cloned or forked this [repo](https://github.com/amoweolubusayo/graph-client-enzyme-balancer-nextjs-example), just run a `yarn` or an `npm i`

Incase you encounter errors using **`yarn`** to install **`graphql`**, use **`npm`** instead to install

**Creating and defining the `.graphclientrc.yml` file.**

As mentioned earlier, in the root folder, create a `.graphclientrc.yml` file. We will be defining the two sources i.e. subgraphs here this way

```javascript
sources:
  - name: enzyme
    handler:
      graphql:
        endpoint: https://api.thegraph.com/subgraphs/name/enzymefinance/enzyme
  - name: balancer
    handler:
      graphql:
        endpoint: https://api.thegraph.com/subgraphs/name/balancer-labs/balancer
```

To get the correct endpoint, navigate to the subgraph's hosted service e.g for Balancer, https://thegraph.com/hosted-service/subgraph/balancer-labs/balancer and select the url under QUERIES(HTTP) which in this case is https://api.thegraph.com/subgraphs/name/balancer-labs/balancer. 

The same process applies to getting Enzyme's endpoint.


![](https://i.imgur.com/GAQJNIB.png)

We will still be coming back to this `.graphclientrc.yml` file but for now let's go create our documents file.


**Creating and defining the document file**

Create another file that you can name with the .graphql extension. In the repo suggested for forking above, I named as `sample-query.graphql` Here is where you define your query. Now head back to your subgraph e.g for Enzyme: https://thegraph.com/hosted-service/subgraph/enzymefinance/enzyme, study the schema and pick whatever data you might need. For me, I need some information on the holdingStates, so I do the following in the `sample-query.graphql`

```javascript
query SampleQuery {
  # Enzyme data
  holdingStates(
    where: { fund: "0x24f3b37934d1ab26b7bda7f86781c90949ae3a79" }
    orderBy: timestamp
    orderDirection: asc
    first: 1000
  ) {
    timestamp
    asset {
      symbol
    }
    amount
  }
}
```

Repeating the same process for Balancer: https://thegraph.com/hosted-service/subgraph/balancer-labs/balancer, I need information on pools, so I add this query as well. Now the `sample-query.graphql` looks as what we have below:

```javascript
query SampleQuery {
  # Enzyme data
  holdingStates(
    where: { fund: "0x24f3b37934d1ab26b7bda7f86781c90949ae3a79" }
    orderBy: timestamp
    orderDirection: asc
    first: 1000
  ) {
    timestamp
    asset {
      symbol
    }
    amount
  }
  # Balancer Data
  pools(first: 5, where: { publicSwap: true }) {
    id
    finalized
    publicSwap
    swapFee
    totalWeight
    tokensList
    tokens {
      id
      address
      balance
      decimals
      symbol
      denormWeight
    }
  }
}
```

At this point, we can then go back to `.graphclientrc.yml`. After sources, we define documents which references the `sample-query.graphql` file.

```javascript
documents:
  - ./sample-query.graphql
```

**Querying from the frontend**

Under pages, create an index.tsx file and add the following

```javascript
import type { NextPage } from "next";
import Head from "next/head";
import Image from "next/image";
import styles from "../styles/Home.module.css";
import { SampleQuery, getBuiltGraphSDK } from "../.graphclient";

const Home: NextPage<{ data: SampleQuery }> = ({ data }) => {
  return (
    <div className={styles.container}>
      <Head>
        <title>Graph Client Next.js Example</title>
        <meta name="description" content="Generated by create next app" />
        <link rel="icon" href="/favicon.ico" />
      </Head>

      <main className={styles.main}>
        <h6 className={styles.title}>
          Welcome to Graph Client <a href="https://nextjs.org">Next.js</a>{" "}
          (Enzyme Data and Balancer Data)
        </h6>

        <form>
          <br />
          <br />
          <textarea
            className={styles.dataTextArea}
            value={JSON.stringify(data, null, 2)}
            readOnly
            rows={55}
          />
        </form>
      </main>

      <footer className={styles.footer}>
        <a
          href="https://vercel.com?utm_source=create-next-app&utm_medium=default-template&utm_campaign=create-next-app"
          target="_blank"
          rel="noopener noreferrer"
        >
          Powered by{" "}
          <span className={styles.logo}>
            <Image src="/vercel.svg" alt="Vercel Logo" width={72} height={16} />
          </span>
        </a>
      </footer>
    </div>
  );
};

const sdk = getBuiltGraphSDK();
export async function getServerSideProps() {
  const data = await sdk.SampleQuery();
  return {
    props: {
      data,
    },
  };
}

export default Home;
```

Notice that we do 

```javascript
const sdk = getBuiltGraphSDK();
export async function getServerSideProps() {
  const data = await sdk.SampleQuery();
  return {
    props: {
      data,
    },
  };
}
```

***This is The Graph Client Typed SDK for consuming The Graph API in Next.js.***



**Run your project**

Start up your projects by running the following commands

**`WITH YARN`**

```bash
$ yarn build
$ yarn build-client
```

**`WITH NPM`**

```bash
$ npm run build
$ npm run build-client
```

After running the `build-client` command, the output will be as follows:

![](https://i.imgur.com/bJlBD9G.png)

Proceed to then run

**`WITH YARN`**

```bash
$ yarn dev
```

**`WITH NPM`**

```bash
$ npm run dev
```

![](https://i.imgur.com/w4EScQm.png)

Here is how your output should finally look like

![](https://i.imgur.com/n5aDUg2.png)


**Push your project to Radicle**

If you are new to radicle and started your project using the Next.js + TS, run the following command to create a profile and identity

`rad auth`

![](https://i.imgur.com/UPspEmv.png)


Initialize your project by running 

`rad init`

Provide the name, description and the default branch of your project

![](https://i.imgur.com/rZwCHha.png)

Publish your project and select a seed node by running

`rad push `

![](https://i.imgur.com/6b8vxns.png)

This project is available here at 

**Radicle**

https://app.radicle.network/seeds/pine.radicle.garden/rad:git:hnrkp9gcfqg3fqt5u1am1fhh97ggf619aihgy

**Github**

https://github.com/amoweolubusayo/graph-client-enzyme-balancer-nextjs-example
