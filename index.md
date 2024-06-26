# Prisma をはじめる

Welcome 👋

データ駆動型アプリケーションの構築と拡張を容易にする当社の製品をご覧ください。

`Prisma ORM` は、次世代の Node.js および TypeScript ORM で、直感的なデータモデル、自動マイグレーション、型安全性、自動補完機能により、データベースを使用する際の開発者のエクスペリエンスを新しいレベルに引き上げます。

`Prisma Accelerate` は、スケーラブルなコネクションプールを備えたグローバルなデータベースキャッシュで、クエリを高速化します。

`Prisma Pulse` を使用すると、型安全なモデルサブスクリプションを使用してデータベースの変更に対応できます。

## Prisma ORM

数分で Prisma ORM をアプリケーションに追加し、データのモデリング、スキーママイグレーションの実行、データベースへのクエリを開始できます。

SQLite を使用して Prisma を試してみよう！

SQLite は、データベース自体を起動させる必要はありません。

[クイックスタート]()：SQLite データベースを使用して Prisma ORM をゼロから 5 分でセットアップします。

[サンプル]()：お気に入りのフレームワークとライブラリを使用して、すぐに実行できるサンプルを試してみましょう。

## 他のデータベースではじめる

Prisma ORM を SQLite 以外のデータベースに接続することも可能です。

# クイックスタート

このクイックスタートガイドでは、プレーンな **TypeScript** プロジェクトとローカルの `SQLite` データベースファイルを使用して、Prisma ORM を最初から使い始める方法を学習します。**データモデリング**、**マイグレーション**、データベースの **クエリ** について説明します。

PostgreSQL、MySQL、MongoDB、その他サポートされているデータベースで Prisma ORM を使用したい場合は以下をご覧ください。

- Prisma ORM をゼロから始める
- 既存のプロジェクトに Prisma ORM を追加する

動作要件

このガイドには Node.js v16.13.0 以上が必要です（システム要件の詳細については、こちらをご覧ください）。

## 1. TypeScript プロジェクトの作成と Prisma ORM のセットアップ

最初のステップとして、プロジェクトディレクトリを作成し、そこに移動します。

```shell
mkdir hello-prisma
cd hello-prisma
```

次に、npm を使用して TypeScript プロジェクトを初期化します。

```shell
npm init -y
npm install typescript ts-node @types/node --save-dev
```

This creates a `package.json` with an initial setup for your TypeScript app.

> [!NOTE]
> See installation instructions to learn how to install Prisma using a different package manager.

Now, initialize TypeScript:

```shell
npx tsc --init
```

Then, install the Prisma CLI as a development dependency in the project:

```shell
npm install prisma --save-dev
```

Finally, set up Prisma ORM with the `init` command of the Prisma CLI:

```shell
npx prisma init --datasource-provider sqlite
```

This creates a new `prisma` directory with a `prisma.schema` file and configures SQLite as your database. You're now ready to model your data and create your database with some tables.

## 2. Model your data in the Prisma schema

The Prisma schema provides an intuitive way to model data. Add the following models to your `schema.prisma` file:

`prisma/schema.prisma`

```prisma
model User {
  id    Int     @id @default(autoincrement())
  email String  @unique
  name  String?
  posts Post[]
}

model Post {
  id        Int     @id @default(autoincrement())
  title     String
  content   String?
  published Boolean @default(false)
  author    User    @relation(fields: [authorId], references: [id])
  authorId  Int
}
```

Models in the Prisma schema have two main purposes:

- Represent the tables in the underlying database
- Serve as foundation for the generated Prisma Client API

In the next section, you will map these models to database tables using Prisma Migrate.

## 3. Run a migration to create your database tables with Prisma Migrate

At this point, you have a Prisma schema but no database yet. Run the following command in your terminal to create the SQLite database and the `User` and `Post` tables 
represented by your models:

```shell
npx prisma migrate dev --name init
```

This command did three things:

1. It created a new SQL migration file for this migration in the `prisma/migrations` directory.
1. It executed the SQL migration file against the database.
1. It ran `prisma generate` under the hood (which installed the `@prisma/client` package and generated a tailored Prisma Client API based on your models).

Because the SQLite database file didn't exist before, the command also created it inside the `prisma` directory with the name `dev.db` as defined via the environment variable in the `.env` file.

Congratulations, you now have your database and tables ready. Let's go and learn how you can send some queries to read and write data!

## 4. Explore how to send queries to your database with Prisma Client

To send queries to the database, you will need a TypeScript file to execute your Prisma Client queries. Create a new file called script.ts for this purpose:

```shell
touch script.ts
```

Then, paste the following boilerplate into it:

`script.ts`

```ts
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()

async function main() {
  // ... you will write your Prisma Client queries here
}

main()
  .then(async () => {
    await prisma.$disconnect()
  })
  .catch(async (e) => {
    console.error(e)
    await prisma.$disconnect()
    process.exit(1)
  })
```

This code contains a `main` function that's invoked at the end of the script. It also instantiates `PrismaClient` which represents the query interface to your database.

### 4.1. Create a new `User` record

Let's start with a small query to create a new `User` record in the database and log the resulting object to the console. Add the following code to your `script.ts` file:

`script.ts`

```diff
  import { PrismaClient } from '@prisma/client'

  const prisma = new PrismaClient()

  async function main() {
+   const user = await prisma.user.create({
+     data: {
+       name: 'Alice',
+       email: 'alice@prisma.io',
+     },
+   })
+   console.log(user)
  }

  main()
    .then(async () => {
      await prisma.$disconnect()
    })
    .catch(async (e) => {
      console.error(e)
      await prisma.$disconnect()
      process.exit(1)
    })
```

Instead of copying the code, you can type it out in your editor to experience the autocompletion Prisma Client provides. You can also actively invoke the autocompletion by pressing the `CTRL` + `SPACE` keys on your keyboard.

Next, execute the script with the following command:

```shell
npx ts-node script.ts
```

Great job, you just created your first database record with Prisma Client! 🎉

In the next section, you'll learn how to read data from the database.

### 4.2. Retrieve all `User` records

Prisma Client offers various queries to read data from your database. In this section, you'll use the `findMany` query that returns all the records in the database for a given model.

Delete the previous Prisma Client query and add the new `findMany` query instead:

`script.ts`

```diff
  import { PrismaClient } from '@prisma/client'

  const prisma = new PrismaClient()

  async function main() {
+   const users = await prisma.user.findMany()
+   console.log(users)
  }

  main()
    .then(async () => {
      await prisma.$disconnect()
    })
    .catch(async (e) => {
      console.error(e)
      await prisma.$disconnect()
      process.exit(1)
    })
```

Execute the script again:

```shell
npx ts-node script.ts
```

Notice how the single `User` object is now enclosed with square brackets in the console. That's because the `findMany` returned an array with a single object inside.

### 4.3. Explore relation queries with Prisma Client

One of the main features of Prisma Client is the ease of working with [relations](). In this section, you'll learn how to create a `User` and a `Post` record in a nested write query. Afterwards, you'll see how you can retrieve the relation from the database using the `include` option.

First, adjust your script to include the nested query:

`script.ts`

```diff
  import { PrismaClient } from '@prisma/client'

  const prisma = new PrismaClient()

  async function main() {
+   const user = await prisma.user.create({
+     data: {
+       name: 'Bob',
+       email: 'bob@prisma.io',
+       posts: {
+         create: [
+           {
+             title: 'Hello World',
+             published: true
+           },
+           {
+             title: 'My second post',
+             content: 'This is still a draft'
+           }
+         ],
+       },
+     },
+   })
+   console.log(user)
  }

  main()
    .then(async () => {
      await prisma.$disconnect()
    })
    .catch(async (e) => {
      console.error(e)
      await prisma.$disconnect()
      process.exit(1)
    })
```

Run the query by executing the script again:

```shell
npx ts-node script.ts
```

By default, Prisma Client only returns scalar fields in the result objects of a query. That's why, even though you also created a new `Post` record for the new `User` record, the console only printed an object with three scalar fields: `id`, `email` and `name`.

In order to also retrieve the `Post` records that belong to a `User`, you can use the `include` option via the `posts` relation field:

`script.ts`

```diff
  import { PrismaClient } from '@prisma/client'

  const prisma = new PrismaClient()

  async function main() {
+   const usersWithPosts = await prisma.user.findMany({
+     include: {
+       posts: true,
+     },
+   })
+   console.dir(usersWithPosts, { depth: null })
  }

  main()
    .then(async () => {
      await prisma.$disconnect()
    })
    .catch(async (e) => {
      console.error(e)
      await prisma.$disconnect()
      process.exit(1)
    })
```

Run the script again to see the results of the nested read query:

```shell
npx ts-node script.ts
```

This time, you're seeing two `User` objects being printed. Both of them have a `posts` field (which is empty for "`Alice`" and populated with a single `Post` object for "`Bob`") that represents the `Post` records associated with them.

Notice that the objects in the `usersWithPosts` array are fully typed as well. This means you will get autocompletion and the TypeScript compiler will prevent you from accidentally typing them.

## 5. Next steps

In this Quickstart guide, you have learned how to get started with Prisma ORM in a plain TypeScript project. Feel free to explore the Prisma Client API a bit more on your own, e.g. by including filtering, sorting, and pagination options in the `findMany` query or exploring more operations like `update` and `delete` queries.

### Explore the data in Prisma Studio

Prisma ORM comes with a built-in GUI to view and edit the data in your database. You can open it using the following command:

```
npx prisma studio
```

### Set up Prisma ORM with your own database

If you want to move forward with Prisma ORM using your own PostgreSQL, MySQL, MongoDB or any other supported database, follow the Set Up Prisma ORM guides:

- [Start with Prisma ORM from scratch]()
- [Add Prisma ORM to an existing project]()

### Explore ready-to-run Prisma ORM examples

Check out the [`prisma-examples`]() repository on GitHub to see how Prisma ORM can be used with your favorite library. The repo contains examples with Express, NestJS, GraphQL as well as fullstack examples with Next.js and Vue.js, and a lot more.

### Build an app with Prisma ORM

The Prisma blog features comprehensive tutorials about Prisma ORM, check out our latest ones:

- [Build a fullstack app with Next.js]()
- [Build a fullstack app with Remix]() (5 parts, including videos)
- [Build a REST API with NestJS]()

### Prisma コミュニティに参加しましょう 💚

Prisma has a huge [community]() of developers. Join us on [Discord]() or ask questions using [GitHub Discussions]().