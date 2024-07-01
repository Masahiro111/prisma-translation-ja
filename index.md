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

# リレーショナルデータベース

Prisma ORM をデータベースに接続し、データベースにアクセスするための Prisma Client を生成して、新しい Node.js または TypeScript プロジェクトをゼロから作成する方法を紹介します。次のチュートリアルでは、[Prisma CLI]()、[Prisma Client]()、[Prisma Migrate]() について説明します。

## 前提条件

このガイドを完了するには、次のものが必要です。

- [Node.js]() をマシンにインストール済であること
- [PostgreSQL]() データベースサーバが稼働していること

> 正確なバージョン要件については、[システム要件]() を参照してください。

データベースの [接続 URL]() が手元にあることを確認してください。データベースサーバーを実行しておらず、Prisma ORM を試してみたいだけの場合は、[クイックスタート]() を確認してください。

## プロジェクト設定の作成

最初のステップとして、プロジェクトディレクトリを作成し、そこに移動します。

```shell
mkdir hello-prisma
cd hello-prisma
```

次に、TypeScript プロジェクトを初期化し、Prisma CLI を開発依存として追加します。

```shell
npm init -y
npm install prisma typescript ts-node @types/node --save-dev
```

これにより、TypeScript アプリの初期設定を含む `package.json` が作成されます。

次に、TypeScript を初期化します。

```shell
npx tsc --init
```

> [!NOTE]
> 別のパッケージマネージャを使用して Prisma をインストールする方法については、[インストール手順]() を参照してください。

`npx` を付け加えて Prisma CLI を呼び出すことができます。

```shell
npx prisma
```

次に、次のコマンドで [Prisma Schema]() ファイルを作成し、Prisma ORM プロジェクトを設定します。

```shell
npx prisma init
```

このコマンドは次の２つのことを行います。

- `prisma` という新しいディレクトリを作成します。このディレクトリには、データベース接続変数とスキーマモデルを記述した Prisma スキーマを含む `schema.prisma` というファイルが含まれます。
- プロジェクトのルートディレクトリに [`.env` ファイル]() を作成します。これは環境変数（データベース接続など）を定義するために使用されます。

# データベースの接続

データベースに接続するには、Prisma スキーマのデータソースブロックの url フィールドをデータベースの [接続 URL]() に設定する必要があります。

```prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}
```

この場合、`url` は .env で定義されている [環境変数を介して設定]() されます。

`.env`

```json
DATABASE_URL="postgresql://johndoe:randompassword@localhost:5432/mydb?schema=public"
```

> [!note]
> 環境変数がコミットされないようにするには、`.gitignore` ファイルに `.env` を追加することをお勧めします。

ここで、接続 URL を調整して、データベースの設定をする必要があります。

データベースの [接続 URL のフォーマット]() は、使用するデータベースによって異なります。PostgreSQL の場合は次のようになります（すべて大文字で表記されている部分は、特定の接続詳細のプレースホルダです）。

```json
postgresql://USER:PASSWORD@HOST:PORT/DATABASE?schema=SCHEMA
```

各コンポーネントの簡単な説明は次のとおりです。

- `USER`: データベースユーザーの名前
- `PASSWORD`: データベースユーザーのパスワード
- `HOST`: ホスト名（ローカル環境の場合は localhost）
- `PORT`: データベースサーバーが稼働しているポート（通常、PostgreSQL の場合は 5432）
- `DATABASE`: [データベース]() の名前
- `SCHEMA`: データベース内の[schema]()の名前

PostgreSQL 接続 URL の `schema` パラメータに何を指定すればよいかわからない場合は、省略してもかまいません。その場合、デフォルトのスキーマ名 `public` が使用されます。

たとえば、Heroku でホストされている PostgreSQL データベースの場合、[接続 URL]() は次のようになります。

`.env`

```env
DATABASE_URL="postgresql://opnmyfngbknppm:XXX@ec2-46-137-91-216.eu-west-1.compute.amazonaws.com:5432/d50rgmkqi2ipus?schema=hello-prisma"
```

macOS 上で PostgreSQL をローカルに実行する場合、ユーザー名とパスワード、およびデータベース名は通常、OS の現在のユーザーに対応します。たとえば、ユーザーが `janedoe` と呼ばれていると仮定します。

`.env`

```env
DATABASE_URL="postgresql://janedoe:janedoe@localhost:5432/janedoe?schema=hello-prisma"
```
