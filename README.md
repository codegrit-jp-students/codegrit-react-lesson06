# レッスン6. 繰り返しや条件付きのコンポーネント呼び出し

## リストを扱う

例えば、レストラン情報を紹介するサイトをイメージしてみましょう。この時に、レストラン一覧ページでは、レストランの情報をリストとしてサーバーからFetch APIを利用して取得します。

```js
const restaurants = [
  {
    id: 100,
    name: レストランA,
    status: "オープン中",
    prefecture: "東京都",
    address: "東京都世田谷区○○"
    opening_time: []
    closing_days: [0, 6], // 曜日を数字で表す。 0は日曜、6は土曜。
    review_count: 10,
    average_rating: 4.5,
    ...省略
  },
  ...省略
];
```

Reactコンポーネントの構成としては、例えば以下のようなものを想定します。

1. RestaurantListContainer -> レストラン一覧を取得するコンテナーコンポーネント
2. RestaurantList -> コンテナーからデータを受け取り表示するためのコンポーネント
3. RestaurantSearchArea -> レストラン一覧のフィルタを設定したり、検索を行うためのエリア
4. Paginator -> レストラン一覧のページネーションを行うためのコンポーネント
5. RestaurantListItem -> レストランごとの概要データを表示する部分、このさらに下に複数のコンポーネントが置かれる。

取得したリストから、RestaurantListItemを呼び出すにはmap()ファンクションを利用します。mapファンクションは、リスト内の要素一つ一つに対してある処理を行いその結果を再度リストとして返します。以下のようにmapファンクション内でコンポーネントを返すことで、リストから繰り返しコンポーネントを呼び出すことができます。

```js
// RestaurantList.js
import React from 'react'
import RestaurantListItem from './RestaurantListItem'

const RestaurantList = ({ restaurants }) => {
  const restaurantListItems = restaurants.map((restaurant) => {
    return (
      <RestaurantListItem
        data={restaurant}
        key={`restaurant-${restaurant.id}`} />
    );
  })
}

export default RestaurantList;
```

### keyについて

さて上記の例でRestaurantListItemコンポーネントのpropsにkeyというものがあることに気づいたかと思います。このkeyというPropsはリストを扱うときに必ずつける必要があり、Reactがコンポーネントの区別をするために利用します。

keyは必ず、他のkeyとは被らないものとなっている必要があります。ただし、グローバルスコープでそうなっている必要はなく、現在扱っているリスト上でのみ被っていなければ大丈夫です。通常は、上記の例のようにデータ名とidを利用することでkeyを作成することができます。


## 条件分岐

リストの他に頻繁に起こることとして、条件によって表示するコンポーネントやReact要素を変えたいという場合があります。典型的な例としては、ログインの有無によって、ユーザーをログインページにリダイレクトしたい、あるいはヘッダーにログインボタンを表示するか、ユーザー向けメニューを表示するか変えたいという場合です。

こうした時にReactで使える方法は以下の2つです。

1. JSX内にインラインで条件分岐を記述する
2. if文によって直接異なるメインのjsxを返す
3. 変数に条件に合わせてコンポーネントやJSXを格納しこれをrenderする。

それぞれの方法を以下では見ていきましょう。

### 1. JSX内にインラインで条件分岐を記述する

インラインで条件分岐を書くには、JSX内で以下のようにして記述します。

- 三項演算子を利用する

三項演算子を使うのは、条件に対してtrueの時とfalseの時いずれの場合も何らかの要素を表示したい場合です。

```js
{条件 ? trueのときの内容 : falseのときの内容}
```

- &&オペレーターを利用する

&&オペレーターはtrueの時のみ何らかの表示をしたい時に利用します。

```js
{
  条件 && 条件が合う場合の内容
}
```

三項演算子を使う場合でも、falseの場合に`null`を返すようにすれば&&オペレーターを利用するのと同じことはできます。

```js
{isLoggedIn ? <a onClick={this.logout}>ログアウト</a> ? null}
```

以下の例では、ログイン中のユーザーが名前を登録している場合は名前を利用し、名前が登録されていない場合はユーザー名を利用して、"こんにちは、○○さん"と表示することを考えてみましょう。この場合以下のように書くことができます。

```js
import React, { Component } from 'react'
import PropTypes from 'prop-types'

export default class extends Component {
  static propTypes = {
    name: PropTypes.string,
    userName: PropTypes.string.isRequired,
  }
  static defaultProps = {
    name: null,
    userName: "ゲスト"
  }
  render() {
    const { 
      name, 
      userName
    } = this.props;
    return (
      <p>
        こんにちは、{name ? name : userName}さん
      </p>
    );
  }
}
```

### 2. 条件に応じてい直接メインのjsxを返す

上記とはことなり、条件に応じて異なるjsxを返すこともできます。例えば、上記の例を書き換えてみましょう。

```js
import React, { Component } from 'react'
import PropTypes from 'prop-types'

export default class extends Component {
  static propTypes = {
    name: PropTypes.string,
    userName: PropTypes.string.isRequired,
  }
  static defaultProps = {
    name: null,
    userName: "ゲスト"
  }
  render() {
    const { 
      name, 
      userName
    } = this.props;
    let helloBase = "こんにちは、"
    if (name) {
      return <p>こんにちは、{name}さん</p>
    }
    return <p>こんにちは、{userName}さん</p>
  }
}
```

### 3. 変数にJSXを格納する

もう少し複雑になると、一つのコンポーネント内で条件に応じて複数のコンポーネントを返すこともあるでしょう。こうした場合、インラインでの条件分岐を利用すると読みづらくなったりするため変数にJSXを格納する方法が楽です。

例えば、ログイン後のダッシュボード上で、ログインの有無によって以下のようにrenderする内容を変更することを考えましょう。

1. ログイン中: ログインユーザー向けのヘッダーと、メインコンテンツを表示する。
2. ログアウト中: ゲストユーザー向けのヘッダーと、ログインを促すコンテンツを表示

この場合、以下のように書くことができます。

```js
import React, { Component } from 'react'
import PropTypes from 'prop-types'
import UserHeader from './UserHeader'
import GuestHeader from './GuestHeader'
import UserDashboard from './UserDashboard'
import ConstraintContent from './ConstraintContent'
import Footer from 'footer'

class Dashboard extends Component {
  state = {
    loadingUser: false // ログインしている場合はcurrentUserを読み込む
  }
  static propTypes = {
    isLoggedIn: PropTypes.bool.isRequired, // ログインしているかどうかの情報
    currentUser: PropTypes.object // ログイン中のユーザー
  }
  static defaultProps = {
    isLoggedIn: false,
    currentUser: null
  }
  render() {
    const { loadingUser } = this.state;
    const { isLoggedIn, currentUser } = this.props
    let header = null
    let mainContent = null
    const { isLoggedIn }
    if (isLoggedIn) { // ログインしているかどうかを判定
      if (loadingUser) { // ユーザー情報を読み込み中かどうかを判定
        mainContent = <p>読み込み中...</p>
      } else { 
        // ログインしていてユーザー情報もあるので、Dashboardを表示する
        header = <UserHeader user={currentUser} />
        mainContent = <UserDashboard user={user} />
      }
    } else {
      // ログインしていないので、Dashboardは表示できない。ゲスト向けのコンテンツを表示する。
      header = <GuestHeader />
      mainContent = <ConstraintContent />
    }
    return (
      <div id="dashboard">
        {header}
        {mainContent}
        <Footer />
      </div>
    );
  }
}
```

## React Fragmentを利用する

さて、上記の例では、return以下の部分が以下のようになっています。

```js
return (
  <div id="dashboard">
    {header}
    {mainContent}
    <Footer />
  </div>
);
```

なぜ次のようにしないのでしょうか？

```js
return (
  {header}
  {mainContent}
  <Footer />
);
```

実際に、自分でも上のように、異なる複数の要素をrender内で返すようにしてみてください。するとエラーが発生することが分かるはずです。これは、Reactにおいて、renderファンクションは一つのReact要素しか返せないようになっているからです。そのため、React16までは、意味のあまりないdiv要素やspan要素を置くなどして複数の要素を囲ってあげる必要がありました。しかしこの方法では、li要素のみを返したいコンポーネントを返したい場合に、`ul > li`のようなスタイリング定義が上手く効かなくなるなど、少し手間が増える場合があります。

React Fragmentを利用することでHTML要素を利用せずに、複数要素を返せるようになりこうした問題が解決できます。

```js
import React, { Component, Fragment } from 'react'

export default class extends Component {
  ...省略

  render() {
    ...省略
    return (
      <Fragment>
        {header}
        {mainContent}
        <Footer />
      </Fragment>
    );
  }
}
```

あるいは以下のようにショートカットを使うこともできます。


```js
import React, { Component, Fragment } from 'react'

export default class extends Component {
  ...省略

  render() {
    ...省略
    return (
      <>
        {header}
        {mainContent}
        <Footer />
      </>
    );
  }
}
```

## 更に学ぼう

- [React公式 - 条件付きレンダー](https://ja.reactjs.org/docs/conditional-rendering.html)
- [React公式 - リストとkey](https://ja.reactjs.org/docs/lists-and-keys.html)
- [React公式 - フラグメント](https://ja.reactjs.org/docs/fragments.html)