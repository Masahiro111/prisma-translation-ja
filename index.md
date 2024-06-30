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

データベースにクエリを送信するには、Prisma Client のクエリを実行するための TypeScript ファイルが必要です。このために script.ts という新しいファイルを作成します。

```shell
touch script.ts
```

次に、次の定型文を貼り付けます。

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

このコードには、スクリプトの最後に呼び出される `main` 関数が含まれています。また、データベースへのクエリインターフェイスを表す `PrismaClient` のインスタンスを作成します。

### 4.1. 新しい `User` レコードの作成

まず、データベースに新しい `User` レコードを作成し、結果のオブジェクトをコンソールにログ出力する小さなクエリから始めましょう。次のコードを `script.ts` ファイルに追加します。

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

コードをコピーする代わりに、エディタでコードを入力すると、Prisma クライアントが提供するオートコンプリートを体験できます。また、キーボードの `CTRL` + `SPACE` キーを押すことで、アクティブにオートコンプリートを呼び出すこともできます。

次に、以下のコマンドでスクリプトを実行します。

```shell
npx ts-node script.ts
```

これで、Prisma Client で最初のデータベースレコードが作成されました！🎉

次のセクションでは、データベースからデータを読み取る方法を学びます。

### 4.2. すべての `User` レコードを取得

Prisma Client には、データベースからデータを読み込むためのさまざまなクエリが用意されています。このセクションでは、指定されたモデルのデータベース内のすべてのレコードを返す `findMany` クエリを使用します。

以前の Prisma Client クエリを削除し、代わりに新しい `findMany` クエリを追加します。

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

スクリプトを再度実行します。

```shell
npx ts-node script.ts
```

コンソールで、単一の `User` オブジェクトが角括弧で囲まれていることに注目してください。これは、`findMany` が配列の中に１つのオブジェクトを返したためです。

### 4.3. Prisma Client のリレーションクエリ

Prisma Client の主な機能の１つは、[relations]() を簡単に操作できることです。このセクションでは、ネストされた書き込みクエリで `User` レコードと `Post` レコードを作成する方法を学びます。その後、`include` オプションを使用してデータベースからリレーションを取得する方法を説明します。

まず、ネストされたクエリを含めるようにスクリプトを調整します。

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

スクリプトを再度実行してクエリを実行します。

```shell
npx ts-node script.ts
```

デフォルトでは、Prisma Client はクエリの結果オブジェクトにスカラフィールドのみを返します。そのため、新しい `User` レコードに対して新しい `Post` レコードを作成しても、コンソールには `id`、`email`、`name` の３つのスカラフィールドを持つオブジェクトのみが出力されました。

`User` に属する `Post` レコードも取得するには、`posts` リレーションフィールドを介して `include` オプションを使用します。

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

スクリプトを再度実行して、ネストされた読み込みクエリの結果を確認します。

```shell
npx ts-node script.ts
```

今回は、２つの `User` オブジェクトが表示されます。どちらも `posts` フィールド（"`Alice`" の場合は空で、"`Bob`" の場合は1つの `Post` オブジェクトが格納されている）を持っており、このフィールドが `Post` レコードを表しています。

`usersWithPosts` 配列内のオブジェクトも完全に型指定されていることに注意してください。これはオートコンプリート（自動補完）が行われ、TypeScript コンパイラが誤ってタイプしてしまうことを防いでくれます。

## 5. 次のステップ

このクイックスタートガイドでは、プレーンな TypeScript プロジェクトで Prisma ORM を使い始める方法を学びました。例えば、`findMany` クエリにフィルタリング、ソート、ページネーションオプションを追加したり、`update` クエリや `delete` クエリのような操作を追加したりすることで自由に Prisma Client API をより探求することができます。

### Prisma Studio でのデータ取り扱い

Prisma ORM には、データベース内のデータを表示および編集するための GUI が組み込まれています。次のコマンドで開くことができます。

```
npx prisma studio
```

### 独自のデータベースで Prisma ORM を設定

独自の PostgreSQL、MySQL、MongoDB、またはその他のサポートされているデータベースを使用して Prisma ORM を進めたい場合は、Prisma ORM のセットアップガイドに従ってください。

- [Prisma ORM をゼロから始める]()
- [既存のプロジェクトに Prisma ORM を追加する]()

### すぐに実行できる Prisma ORM の例を調べる

GitHub の [`prisma-examples`]() リポジトリをチェックして、Prisma ORM をお気に入りのライブラリでどのように使用できるかを確認してください。リポジトリには、Express、NestJS、GraphQL の例や、Next.js と Vue.js を使用したフルスタックの例など、さまざまな例が含まれています。

### Prisma ORM でアプリを構築する

Prisma ブログには Prisma ORM に関する包括的なチュートリアルが掲載されています。最新のものをご覧ください。

- [Next.js でフルスタックアプリを構築する]()
- [Remix でフルスタックアプリを構築する]() (5 部構成、動画を含む)
- [NestJS で REST API を構築する]()

### Prisma コミュニティに参加しましょう 💚

Prisma には、開発者の巨大な [コミュニティ]() があります。[Discord]() に参加するか、[GitHub Discussions]() を使用して質問してください。

# Prisma ORM のセットアップ

最初から始めるか、既存のプロジェクトに Prisma ORM を追加します。次のチュートリアルでは、Prisma CLI、Prisma Client、および Prisma Migrate について説明します。

# Relational databases

Learn how to create a new Node.js or TypeScript project from scratch by connecting Prisma ORM to your database and generating a Prisma Client for database access. The following tutorial introduces you to the [Prisma CLI](), [Prisma Client](), and [Prisma Migrate]().

## Prerequisites

In order to successfully complete this guide, you need:

- [Node.js]() installed on your machine
- a [PostgreSQL]() database server running

> See [System requirements]() for exact version requirements.

Make sure you have your database [connection URL]() at hand. If you don't have a database server running and just want to explore Prisma ORM, check out the [Quickstart]().

## Create project setup

As a first step, create a project directory and navigate into it:

```shell
mkdir hello-prisma
cd hello-prisma
```

Next, initialize a TypeScript project and add the Prisma CLI as a development dependency to it:

```shell
npm init -y
npm install prisma typescript ts-node @types/node --save-dev
```

This creates a `package.json` with an initial setup for your TypeScript app.

Next, initialize TypeScript:

```shell
npx tsc --init
```

> [!info]
> See [installation instructions]() to learn how to install Prisma using a different package manager.

You can now invoke the Prisma CLI by prefixing it with `npx`:

```shell
npx prisma
```

Next, set up your Prisma ORM project by creating your [Prisma Schema]() file with the following command:

```shell
npx prisma init
```

This command does two things:

- creates a new directory called `prisma` that contains a file called `schema.prisma`, which contains the Prisma schema with your database connection variable and schema models
- creates the [`.env` file]() in the root directory of the project, which is used for defining environment variables (such as your database connection)
