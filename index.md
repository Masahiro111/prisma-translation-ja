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

これにより、TypeScript アプリの初期設定を含む `package.json` が作成されます。

> [!NOTE]
> 別のパッケージマネージャを使用して Prisma をインストールする方法については、インストール手順を参照してください。

次に、TypeScript を初期化します。

```shell
npx tsc --init
```

次に、プロジェクトの開発依存関係として Prisma CLI をインストールします。

```shell
npm install prisma --save-dev
```

最後に、Prisma CLI の `init` コマンドを使用して Prisma ORM をセットアップします。

```shell
npx prisma init --datasource-provider sqlite
```

これにより、新しい `prisma` ディレクトリと `prisma.schema` ファイルが作成され、データベースとして SQLite が設定されます。これでデータをモデル化し、いくつかのテーブルを持つデータベースを作成する準備ができました。

## 2. Prisma スキーマでデータをモデル化する

Prisma スキーマは、データをモデル化する直感的な方法を提供します。次のモデルを `schema.prisma` ファイルに追加します。

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

Prisma スキーマのモデルには、主に２つの目的があります。

- 基礎となるデータベース内のテーブルを表す
- 生成された Prisma クライアント API の基盤として機能する

次のセクションでは、Prisma Migrate を使用してこれらのモデルをデータベーステーブルにマッピングします。

## 3. データベーステーブル作成のため Prisma Migrate でマイグレーションを実行

この時点では、Prisma スキーマはありますが、データベースはまだありません。ターミナルで次のコマンドを実行して、SQLite データベースと、モデルで表される `User` と `Post` テーブルを作成します。

```shell
npx prisma migrate dev --name init
```

このコマンドは次の３つのことを実行します。

1. このマイグレーション用の新しい SQL マイグレーションファイルを `prisma/migrations` ディレクトリに作成します
1. データベースに対して SQL マイグレーションファイルを実行します
1. 内部で `prisma generate` を実行します（これにより `@prisma/client` パッケージがインストールされ、モデルに基づいてカスタマイズされた Prisma Client API が生成されます）

SQLite データベースファイルがないため `.env` ファイルの環境変数で定義された `dev.db` という名前で `prisma` ディレクトリ内に作成されます。

これでデータベースとテーブルの準備ができました。それでは、どのようにクエリを送信してデータを読み書きできるかを学びましょう！

## 4. Prisma Client を使用したデータベースへのクエリ送信方法

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