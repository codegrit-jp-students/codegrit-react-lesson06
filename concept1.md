# 繰り返しの扱い

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
