了解しました！リポジトリのREADMEを解析し、Expo + TypeScript を使用したモックアプリの設計を行います。ディレクトリ構成の詳細や、それぞれのファイルの役割・実装方針も明記します。設計がまとまり次第、お知らせします。

# モックアプリの設計方針

日本語学習アプリの要件に基づき、最新のExpo（React Native）とTypeScriptでモックアプリを設計します。データはすべてアプリ内に保持し、ネットワーク通信は行わず**モックAPI**で代用します。これによりオフラインでも動作可能で、複雑なバックエンド実装を省略できます。 ([japanese-app/README.md at main · FriendlyTech-Inc/japanese-app · GitHub](https://github.com/FriendlyTech-Inc/japanese-app/blob/main/README.md#:~:text=%E3%83%AC%E3%83%83%E3%82%B9%E3%83%B3%E5%8D%98%E4%BD%8D%E3%81%A7%E3%81%AE%E3%81%BE%E3%81%A8%E3%82%81%E3%82%AF%E3%82%A4%E3%82%BA%E3%82%84%E3%80%81%E3%82%B9%E3%83%94%E3%83%BC%E3%83%89%E5%BD%A2%E5%BC%8F%E3%81%AE%E9%80%A3%E7%B6%9A%E3%82%AF%E3%82%A4%E3%82%BA%E3%81%AA%E3%81%A9%E8%A4%87%E6%95%B0%E3%83%A2%E3%83%BC%E3%83%89%E3%81%AB%E5%AF%BE%E5%BF%9C%204)) た、React Nativeのコンポーネント指向設計とTypeScriptの型定義を活かし、**可読性**・**保守性**・**パフォーマンス**に優れた構成を目指します。

## ディレクトリ構成

プロジェクトのディレクトリ構成は関心ごとに整理し、各フォルダに明確な責務を持たせます。主なフォルダは `components/`、`screens/`、`services/`、`state/` で、それ以外にもナビゲーションやデータ定義用のフォルダを設けます。以下は詳細な構成例です。

```
project-root/
├── App.tsx                # エントリーポイント（プロバイダやナビゲーションの設定）
├── assets/                # 画像・音声などのアセット類（ローカル音声ファイル等）
├── data/                  # アプリ内コンテンツの静的データ(JSON形式)
│   ├── lessons.json       # レッスン/フレーズ等のコンテンツ定義 ([japanese-app/README.md at main · FriendlyTech-Inc/japanese-app · GitHub](https://github.com/FriendlyTech-Inc/japanese-app/blob/main/README.md#:~:text=%E3%83%AC%E3%83%83%E3%82%B9%E3%83%B3%E5%8D%98%E4%BD%8D%E3%81%A7%E3%81%AE%E3%81%BE%E3%81%A8%E3%82%81%E3%82%AF%E3%82%A4%E3%82%BA%E3%82%84%E3%80%81%E3%82%B9%E3%83%94%E3%83%BC%E3%83%89%E5%BD%A2%E5%BC%8F%E3%81%AE%E9%80%A3%E7%B6%9A%E3%82%AF%E3%82%A4%E3%82%BA%E3%81%AA%E3%81%A9%E8%A4%87%E6%95%B0%E3%83%A2%E3%83%BC%E3%83%89%E3%81%AB%E5%AF%BE%E5%BF%9C%204)) │   └── ...               
├── src/                   # ソースコード（TypeScript）
│   ├── components/        # 再利用可能なUIコンポーネント群
│   │   ├── common/        # 汎用的なUI部品（ボタン、カードなど）
│   │   │   └── Button.tsx        # カスタムボタンコンポーネント
│   │   ├── lesson/        # 学習用特化コンポーネント
│   │   │   └── PhraseCard.tsx   # フレーズ表示・録音UIカード ([japanese-app/README.md at main · FriendlyTech-Inc/japanese-app · GitHub](https://github.com/FriendlyTech-Inc/japanese-app/blob/main/README.md#:~:text=Flutter%E3%82%84React%20Native%E3%81%A7%E9%96%8B%E7%99%BA%E3%81%99%E3%82%8B%E3%81%93%E3%81%A8%E3%81%A7iOS%2FAndroid%E4%B8%A1%E5%AF%BE%E5%BF%9C%E3%82%92%E5%8A%B9%E7%8E%87%E5%8C%96%E3%80%82%20,%E9%9F%B3%E5%A3%B0%E8%AA%8D%E8%AD%98%E9%83%A8%E5%88%86%E3%81%AF%E3%82%AA%E3%83%B3%E3%83%A9%E3%82%A4%E3%83%B3%E5%BF%85%E9%A0%88%E3%81%A0%E3%81%8C%E3%80%81%E3%82%AA%E3%83%95%E3%83%A9%E3%82%A4%E3%83%B3%E5%AD%A6%E7%BF%92%E7%94%A8%E3%81%AB%E3%82%AF%E3%82%A4%E3%82%BA%E3%82%84%E5%8D%98%E8%AA%9E%E5%B8%B3%E3%82%92%E3%82%AD%E3%83%A3%E3%83%83%E3%82%B7%E3%83%A5%E3%81%99%E3%82%8B%E4%BB%95%E7%B5%84%E3%81%BF%E3%82%82%E6%A4%9C%E8%A8%8E%E5%8F%AF)) │   │   └── quiz/          # クイズ関連コンポーネント
│   │       └── QuizOption.tsx   # クイズの選択肢ボタンなど
│   ├── screens/           # 画面コンポーネント（各画面のコンテナ）
│   │   ├── HomeScreen.tsx      # ホーム画面（レッスン一覧やモード選択）
│   │   ├── LessonScreen.tsx    # レッスン画面（フレーズ一覧・学習画面）
│   │   ├── PracticeScreen.tsx  # 発音練習画面（インプット/アウトプット統合）
│   │   ├── QuizScreen.tsx      # クイズ出題画面（問題表示と解答入力）
│   │   └── QuizResultScreen.tsx # クイズ結果画面（スコア表示など）
│   ├── navigation/        # 画面遷移（ナビゲーション）設定
│   │   └── AppNavigator.tsx    # StackやTabナビゲーションの定義
│   ├── services/          # ビジネスロジック・データ取得（モックAPI）
│   │   ├── contentService.ts   # 学習コンテンツ提供（JSON読み込み/検索）
│   │   ├── quizService.ts      # クイズ問題生成ロジック（モックAPI）
│   │   └── speechService.ts    # 発音評価ロジック（モックAPI；ダミー評価) ([japanese-app/README.md at main · FriendlyTech-Inc/japanese-app · GitHub](https://github.com/FriendlyTech-Inc/japanese-app/blob/main/README.md#:~:text=1.%20%E3%82%A4%E3%83%B3%E3%83%97%E3%83%83%E3%83%88%EF%BC%88Input%EF%BC%89%20,%E3%82%AF%E3%82%A4%E3%82%BA%EF%BC%88Review%20%2F%20Assessment%EF%BC%89)) │   ├── state/             # グローバル状態管理
│   │   ├── ProgressContext.tsx # 学習進捗や設定を管理するContextとProvider
│   │   └── index.ts         # Contextの初期化・フック定義など
│   ├── types/             # 型定義（TypeScriptのインターフェース等）
│   │   └── content.ts       # LessonやPhrase、Quiz問題などの型定義
│   └── utils/             # 共通のユーティリティ関数・フック
│       ├── useAudio.ts      # 音声再生や録音を扱うカスタムフック
│       └── formatFurigana.ts # ふりがな付きテキストの表示整形関数
└── package.json
```

上記のようにフォルダを分離することで、関心ごと（UI、画面ロジック、データ処理、状態管理）が明確になり、コードの見通しが良くなります。特に`src/`以下にアプリの主要ロジックをまとめ、UIとビジネスロジックを分離する構成としています。

### `components/` ディレクトリ

`components/` にはUIの再利用可能なパーツをまとめます。デザインの一貫性と開発効率のため、ボタンやカード、モーダルといった汎用コンポーネントを`components/common/`に配置し、学習機能特有のコンポーネント（例えば発音練習用のマイクボタンやフレーズ表示カードなど）を機能別サブフォルダに整理します ([japanese-app/README.md at main · FriendlyTech-Inc/japanese-app · GitHub](https://github.com/FriendlyTech-Inc/japanese-app/blob/main/README.md#:~:text=Flutter%E3%82%84React%20Native%E3%81%A7%E9%96%8B%E7%99%BA%E3%81%99%E3%82%8B%E3%81%93%E3%81%A8%E3%81%A7iOS%2FAndroid%E4%B8%A1%E5%AF%BE%E5%BF%9C%E3%82%92%E5%8A%B9%E7%8E%87%E5%8C%96%E3%80%82%20,%E9%9F%B3%E5%A3%B0%E8%AA%8D%E8%AD%98%E9%83%A8%E5%88%86%E3%81%AF%E3%82%AA%E3%83%B3%E3%83%A9%E3%82%A4%E3%83%B3%E5%BF%85%E9%A0%88%E3%81%A0%E3%81%8C%E3%80%81%E3%82%AA%E3%83%95%E3%83%A9%E3%82%A4%E3%83%B3%E5%AD%A6%E7%BF%92%E7%94%A8%E3%81%AB%E3%82%AF%E3%82%A4%E3%82%BA%E3%82%84%E5%8D%98%E8%AA%9E%E5%B8%B3%E3%82%92%E3%82%AD%E3%83%A3%E3%83%83%E3%82%B7%E3%83%A5%E3%81%99%E3%82%8B%E4%BB%95%E7%B5%84%E3%81%BF%E3%82%82%E6%A4%9C%E8%A8%8E%E5%8F%AF)) これらコンポーネントはできるだけ**純粋（presentational）**に保ち、受け取ったプロップスに応じて表示を行います。例えば:

- **Button.tsx**: カスタムデザインのボタンコンポーネント。押下時の動作をプロップス経由で受け取り、画面ごとに再利用します。  
- **PhraseCard.tsx**: レッスン内で使用するフレーズ表示カード。日本語のフレーズとふりがな・翻訳・音声再生ボタンを表示し、ユーザーが発音録音できるUIも含みます ([japanese-app/README.md at main · FriendlyTech-Inc/japanese-app · GitHub](https://github.com/FriendlyTech-Inc/japanese-app/blob/main/README.md#:~:text=1.%20%E5%8D%98%E8%AA%9E%E3%83%BB%E3%83%95%E3%83%AC%E3%83%BC%E3%82%BA%E8%A1%A8%E7%A4%BA%20,%E5%8D%98%E8%AA%9E%E3%81%AE%E5%88%86%E6%9B%B8%EF%BC%88%E5%85%AC%E5%9C%92%EF%BC%9Dpark%20%E3%81%AA%E3%81%A9%EF%BC%89%E3%82%92%E4%BD%B5%E8%A8%98)) 録音ボタンが押された際の処理（モック評価呼び出し）は親からコールバックで受け取ります。

このようにコンポーネントを細分化することで、同様のUI要素を複数画面で一貫して使い回せます。共通コンポーネントを増やすことで、見た目と操作性を統一しつつ、修正も一箇所で済むため保守性が向上します。

### `screens/` ディレクトリ

`screens/` には各画面（ページ）に相当するコンテナコンポーネントを配置します。スクリーンはナビゲーション単位のコンポーネントであり、内部で上記の汎用`components`を組み合わせてUIを構成します。**スクリーンの責務**は主に以下の通りです:
- 必要なデータを`services`経由で取得し（モックAPI呼び出し）、コンポーネントにプロップスとして渡す。  
- ユーザーの入力や操作（ボタン押下、解答選択など）に応じて状態を更新し、必要に応じて画面遷移を行う。  
- コンテキストやグローバル状態（後述の`state`）が必要な場合はそれを読み込んでUIに反映する。

主要なスクリーン例:
- **HomeScreen.tsx**: レッスン一覧や学習モード選択を表示するトップ画面。例えば「通常レッスン」「スピードクイズ」などモードを選ぶメニューと、各レッスンのリストを表示します。ユーザーがレッスンやモードを選択すると対応する画面にナビゲートします ([japanese-app/README.md at main · FriendlyTech-Inc/japanese-app · GitHub](https://github.com/FriendlyTech-Inc/japanese-app/blob/main/README.md#:~:text=,%E3%82%AA%E3%83%AA%E3%82%B8%E3%83%8A%E3%83%AB%E3%83%9F%E3%83%8B%E3%83%AC%E3%83%83%E3%82%B9%E3%83%B3%20%E3%83%A6%E3%83%BC%E3%82%B6%E3%83%BC%E3%81%8C%E8%8B%A6%E6%89%8B%E3%83%95%E3%83%AC%E3%83%BC%E3%82%BA%E3%81%A0%E3%81%91%E3%82%92%E3%83%94%E3%83%83%E3%82%AF%E3%82%A2%E3%83%83%E3%83%97%E3%81%97%E3%80%81%E7%8B%AC%E8%87%AA%E3%83%AA%E3%82%B9%E3%83%88%E3%82%92%E4%BD%9C%E6%88%90%20%E2%86%92%20%E3%82%A4%E3%83%B3%E3%83%97%E3%83%83%E3%83%88%E3%83%BB%E3%82%A2%E3%82%A6%E3%83%88%E3%83%97%E3%83%83%E3%83%88%E3%83%BB%E3%82%AF%E3%82%A4%E3%82%BA%E3%82%92%E8%87%AA%E7%94%B1%E3%81%AB%E7%B5%84%E3%81%BF%E5%90%88%E3%82%8F%E3%81%9B%E3%81%A6%E5%86%8D%E5%AD%A6%E7%BF%92))   
- **LessonScreen.tsx**: レッスン内の学習画面。選択されたレッスンに含まれるフレーズ一覧を表示し、各フレーズに対して**インプット**（意味・音声の確認）と**アウトプット**（発話練習）のUIを提供します ([japanese-app/README.md at main · FriendlyTech-Inc/japanese-app · GitHub](https://github.com/FriendlyTech-Inc/japanese-app/blob/main/README.md#:~:text=1.%20%E3%82%A4%E3%83%B3%E3%83%97%E3%83%83%E3%83%88%EF%BC%88Input%EF%BC%89%20,%E3%82%AF%E3%82%A4%E3%82%BA%EF%BC%88Review%20%2F%20Assessment%EF%BC%89)) `PhraseCard`コンポーネントをリスト表示し、ユーザーが各カード上で発音録音→モック評価結果を確認できるようにします。全フレーズ学習後、クイズ画面へのナビゲーションボタンを表示します。  
- **PracticeScreen.tsx**: （オプション）アウトプット特化モード等で使用する発話練習専用画面。選択したフレーズや会話文の発音練習を連続して行うUIで、音声入力→評価のサイクルを繰り返せます。  
- **QuizScreen.tsx**: クイズ出題画面。現在のレッスンや選択モードに応じて問題を出題します。4択問題などの場合は`QuizOption`コンポーネントを利用して選択肢を表示し、ユーザーの選択に応じて正誤判定・スコア集計を行います ([japanese-app/README.md at main · FriendlyTech-Inc/japanese-app · GitHub](https://github.com/FriendlyTech-Inc/japanese-app/blob/main/README.md#:~:text=,%E5%AD%A6%E7%BF%92%E3%83%A2%E3%83%BC%E3%83%89%E3%81%AB%E3%82%88%E3%81%A3%E3%81%A6%E3%81%AF%E3%80%81%E3%83%AC%E3%83%83%E3%82%B9%E3%83%B3%E3%81%AE%E6%9C%80%E5%BE%8C%E3%81%AB%E3%81%BE%E3%81%A8%E3%82%81%E3%81%A6%E3%82%AF%E3%82%A4%E3%82%BA%E3%82%92%E8%A1%8C%E3%81%A3%E3%81%9F%E3%82%8A%E3%80%81%E3%82%B9%E3%83%94%E3%83%BC%E3%83%89%E9%80%A3%E7%B6%9A%E5%87%BA%E9%A1%8C%E3%82%92%E3%81%97%E3%81%9F%E3%82%8A%E3%81%99%E3%82%8B)) 問題文や選択肢の生成には`quizService`を用い、表示上のタイマーや進捗バーなども実装します（パフォーマンスのため`FlatList`で問題リストをレンダリングし、必要に応じてページング）。  
- **QuizResultScreen.tsx**: クイズ終了後の結果表示画面。スコアや正答率を表示し、復習すべきフレーズの提案など簡易なフィードバックを行います。ここでは`state`に保存したユーザーの成績データを参照し表示します。

各スクリーンは必要なときにのみマウントされるため、画面間でデータを共有する場合は`state`（Context経由のグローバル状態）やナビゲーションパラメータを使います。スクリーンコンポーネント自体は見通しを良くするためできるだけUI構築と画面遷移ロジックに専念させ、細かなロジック（データ処理や判定など）は`services`に委譲します。これによりスクリーンのコードが肥大化せず、可読性が保たれます。

### `navigation/` ディレクトリ

React Navigationを用いて画面遷移を管理します。`AppNavigator.tsx`ではスタックナビゲーションやタブナビゲーションを定義し、各スクリーンを登録します。例えばホーム画面をタブのひとつに配置し、別タブに「設定」画面（必要なら）やプロフィール画面を置くこともできます。通常は**Stack Navigator**でレッスン→クイズといった遷移を管理しつつ、全体をTab Navigatorでモード別に分ける構成が考えられます。Expoの場合は`App.tsx`で`NavigationContainer`をラップする必要があるため、`AppNavigator`を`App.tsx`から呼び出し、Contextプロバイダもその上位で包むようにします。

ナビゲーションのベストプラクティスとして、画面間パラメータやナビゲーションロジックは集中管理し、各画面コンポーネントは自身がどの画面から来たかを意識しない実装にします。これにより画面の再利用性が上がり、ナビゲーション構造を変更する際も各画面の修正が不要になります。

### `services/` ディレクトリ（モックAPI実装）

`services/`にはビジネスロジックやデータ取得処理を担当するモジュールを配置します。ここでは実際のAPIの代わりに**モックAPI**を実装し、静的なコンテンツデータ(JSON)を提供します。アプリ要件である「音声認識APIによる発音評価」「多彩なクイズ形式」などの機能は、この層でモック実装します ([japanese-app/README.md at main · FriendlyTech-Inc/japanese-app · GitHub](https://github.com/FriendlyTech-Inc/japanese-app/blob/main/README.md#:~:text=1.%20%E3%82%A4%E3%83%B3%E3%83%97%E3%83%83%E3%83%88%EF%BC%88Input%EF%BC%89%20,%E3%82%AF%E3%82%A4%E3%82%BA%EF%BC%88Review%20%2F%20Assessment%EF%BC%89))  ([japanese-app/README.md at main · FriendlyTech-Inc/japanese-app · GitHub](https://github.com/FriendlyTech-Inc/japanese-app/blob/main/README.md#:~:text=,%E5%AD%A6%E7%BF%92%E3%83%A2%E3%83%BC%E3%83%89%E3%81%AB%E3%82%88%E3%81%A3%E3%81%A6%E3%81%AF%E3%80%81%E3%83%AC%E3%83%83%E3%82%B9%E3%83%B3%E3%81%AE%E6%9C%80%E5%BE%8C%E3%81%AB%E3%81%BE%E3%81%A8%E3%82%81%E3%81%A6%E3%82%AF%E3%82%A4%E3%82%BA%E3%82%92%E8%A1%8C%E3%81%A3%E3%81%9F%E3%82%8A%E3%80%81%E3%82%B9%E3%83%94%E3%83%BC%E3%83%89%E9%80%A3%E7%B6%9A%E5%87%BA%E9%A1%8C%E3%82%92%E3%81%97%E3%81%9F%E3%82%8A%E3%81%99%E3%82%8B)) UI側（スクリーンやコンポーネント）は`services`から提供される関数を呼び出すだけでデータ取得や評価処理が行われるため、将来的に本物のAPI接続に差し替える場合も`services`内を修正すれば対応できます（疎結合な設計）。

主なサービスモジュール例:
- **contentService.ts**: アプリ内学習コンテンツ（例文・フレーズ情報）の提供を行います。`data/lessons.json`などに定義されたローカルJSONを読み込み、レッスン一覧やフレーズ詳細を取得する関数を持ちます。例えば`getLessons()`でレッスン一覧（タイトルや概要）を返し、`getLessonContent(lessonId)`で特定レッスン内のフレーズ配列を返す、といった実装です。JSON形式でコンテンツを管理することで、フレーズを最小単位として様々な学習モードに再利用できる柔軟性を確保します ([japanese-app/README.md at main · FriendlyTech-Inc/japanese-app · GitHub](https://github.com/FriendlyTech-Inc/japanese-app/blob/main/README.md#:~:text=%E3%83%AC%E3%83%83%E3%82%B9%E3%83%B3%E5%8D%98%E4%BD%8D%E3%81%A7%E3%81%AE%E3%81%BE%E3%81%A8%E3%82%81%E3%82%AF%E3%82%A4%E3%82%BA%E3%82%84%E3%80%81%E3%82%B9%E3%83%94%E3%83%BC%E3%83%89%E5%BD%A2%E5%BC%8F%E3%81%AE%E9%80%A3%E7%B6%9A%E3%82%AF%E3%82%A4%E3%82%BA%E3%81%AA%E3%81%A9%E8%A4%87%E6%95%B0%E3%83%A2%E3%83%BC%E3%83%89%E3%81%AB%E5%AF%BE%E5%BF%9C%204))   
- **quizService.ts**: クイズ問題の生成と採点ロジックを担います。指定したレッスンやモードに応じて出題内容を構築する`generateQuizQuestions()`などの関数を提供します。たとえば通常レッスン後のクイズではそのレッスンの範囲から問題を出し、スピードクイズモードでは全データからランダムに4択問題を一定数生成する、といった処理を行います ([japanese-app/README.md at main · FriendlyTech-Inc/japanese-app · GitHub](https://github.com/FriendlyTech-Inc/japanese-app/blob/main/README.md#:~:text=%E7%89%B9%E5%AE%9A%E3%81%AE%E3%83%AC%E3%83%83%E3%82%B9%E3%83%B3%EF%BC%88%E4%BE%8B%EF%BC%9A%E3%81%82%E3%81%84%E3%81%95%E3%81%A4%EF%BC%89%E3%82%92%E9%81%B8%E6%8A%9E%20%E2%86%92%20%E5%90%84%E3%83%95%E3%83%AC%E3%83%BC%E3%82%BA%E3%81%AE%E3%82%A4%E3%83%B3%E3%83%97%E3%83%83%E3%83%88%E3%81%A8%E3%82%A2%E3%82%A6%E3%83%88%E3%83%97%E3%83%83%E3%83%88%20%E2%86%92%20%E3%83%AC%E3%83%83%E3%82%B9%E3%83%B3%E7%B5%82%E4%BA%86%E6%99%82%E3%81%AB%E3%81%BE%E3%81%A8%E3%82%81%E3%82%AF%E3%82%A4%E3%82%BA,%E3%82%AA%E3%83%AA%E3%82%B8%E3%83%8A%E3%83%AB%E3%83%9F%E3%83%8B%E3%83%AC%E3%83%83%E3%82%B9%E3%83%B3%20%E3%83%A6%E3%83%BC%E3%82%B6%E3%83%BC%E3%81%8C%E8%8B%A6%E6%89%8B%E3%83%95%E3%83%AC%E3%83%BC%E3%82%BA%E3%81%A0%E3%81%91%E3%82%92%E3%83%94%E3%83%83%E3%82%AF%E3%82%A2%E3%83%83%E3%83%97%E3%81%97%E3%80%81%E7%8B%AC%E8%87%AA%E3%83%AA%E3%82%B9%E3%83%88%E3%82%92%E4%BD%9C%E6%88%90%20%E2%86%92%20%E3%82%A4%E3%83%B3%E3%83%97%E3%83%83%E3%83%88%E3%83%BB%E3%82%A2%E3%82%A6%E3%83%88%E3%83%97%E3%83%83%E3%83%88%E3%83%BB%E3%82%AF%E3%82%A4%E3%82%BA%E3%82%92%E8%87%AA%E7%94%B1%E3%81%AB%E7%B5%84%E3%81%BF%E5%90%88%E3%82%8F%E3%81%9B%E3%81%A6%E5%86%8D%E5%AD%A6%E7%BF%92)) 選択肢の誤答候補もコンテンツデータからピックアップし、正答率の計算もこのサービス内で完結させます。UI側は問題と選択肢を受け取って表示し、ユーザーの解答を`quizService`に渡して正誤判定・スコア計算をしてもらいます。  
- **speechService.ts**: 発音練習（アウトプット）の評価を行うサービスです。本来はクラウドの音声認識APIを呼び出し、ユーザーの発話音声をテキスト化・解析してスコアを返すところですが ([japanese-app/README.md at main · FriendlyTech-Inc/japanese-app · GitHub](https://github.com/FriendlyTech-Inc/japanese-app/blob/main/README.md#:~:text=1.%20%E7%99%BA%E9%9F%B3%E5%85%A5%E5%8A%9B%E3%83%BB%E8%A9%95%E4%BE%A1%20,%E8%AA%A4%E5%B7%AE%E3%81%AE%E3%81%82%E3%82%8B%E7%99%BA%E9%9F%B3%E7%AE%87%E6%89%80%E3%81%AE%E7%B0%A1%E6%98%93%E3%83%95%E3%82%A3%E3%83%BC%E3%83%89%E3%83%90%E3%83%83%E3%82%AF%EF%BC%88%E3%81%A9%E3%81%AE%E9%9F%B3%E3%81%8C%E5%BC%B1%E3%81%84%E3%83%BB%E3%81%A9%E3%81%93%E3%82%92%E5%BC%B7%E8%AA%BF%E3%81%99%E3%81%B9%E3%81%8D%E3%81%AA%E3%81%A9%EF%BC%89%202.%20%E5%A4%9A%E6%A7%98%E3%81%AA%E5%87%BA%E9%A1%8C%E5%BD%A2%E5%BC%8F%EF%BC%88%E3%82%BF%E3%82%B9%E3%82%AF%E3%83%86%E3%83%B3%E3%83%97%E3%83%AC%E3%83%BC%E3%83%88%EF%BC%89)) モック実装ではランダムもしくは固定ロジックでスコア評価を行います。例えば`evaluatePronunciation(audioClip, phrase)`関数で擬似的に発音の一致率スコアを返し、常に80点程度のスコアを返すか、あるいはフレーズごとに用意した模擬結果を返すようにします。簡易フィードバック（「〇〇の発音が弱い」など）もダミーのメッセージを返し、UI側で表示します。これによりネットワークに繋がずとも発音評価の流れをアプリ上体験できるようにします。

サービス層では`async`関数やPromiseを利用して**非同期API風**のインターフェースを提供します。例えばモックでも`await getLessons()`でデータ取得できるようにし、適度に`setTimeout`を使ってレスポンス遅延を再現することで、本物のAPIに近い使用感をテストできます。TypeScriptの型定義により、サービス関数の返却値（例: `Promise<Lesson[]>`）や引数型を明示し、UI側での誤使用を防ぎます。さらに、将来本物のAPIと接続する際は、このサービス層をREST APIやGraphQLクライアントに差し替えるだけで済むよう、**UIとデータ取得ロジックの分離**を徹底しています。

### `state/` ディレクトリ

`state/`にはグローバルな状態管理のコードを配置します。小規模なモバイルアプリであるため、Reactの**Context API** ([What are the best practices for handling state in React Native? - Reddit](https://www.reddit.com/r/reactnative/comments/1cjmax1/what_are_the_best_practices_for_handling_state_in/#:~:text=Reddit%20www,Context%20API%2C%20or%20something%20else)) 用いてシンプルに実装します（必要に応じてRedux等への移行も可能な構成）。例えば**ProgressContext**を用意し、ユーザーの学習進捗や設定（例: レッスン完了状況、累計スコア、UIの言語設定など）をグローバルに保持します。`ProgressContext.tsx`ではContextとProviderコンポーネントを定義し、`useProgress()`カスタムフック経由でどのコンポーネントからでも現在の進捗データや更新関数にアクセスできるようにします。

具体的な状態例:
- ユーザーが最後に学習したレッスンIDや日時
- 各レッスンの達成度合い（完了済みフラグや平均クイズスコア）
- 現在のアプリ設定（UI表示言語、日本語/英語翻訳の切替設定など）

Contextの初期値は`lessons.json`に基づき未完了状態を生成するか、または初回起動時にデフォルト値をセットします。進捗更新（例: クイズ結果の保存）は`ProgressContext`内で状態更新関数を通じて行い、必要なら端末ストレージ（AsyncStorageなど）にも保存します。こうしたグローバル状態管理により、画面間でデータを手渡す煩雑さ（プロップスの引き回し）を解消し、また状態が集中管理されることでロジックの見通しも良くなります。状態管理コードは`state/`以下にまとめてあるため、将来的に状態管理ライブラリを変更する場合もこのフォルダ内だけで対応可能です。

### その他のディレクトリ

- **assets/**: 画像や音声ファイルを配置します。特に本アプリではネイティブ発音の音声データが重要になるため、各フレーズの音声ファイル（mp3等）や効果音を`assets/`に含めます ([japanese-app/README.md at main · FriendlyTech-Inc/japanese-app · GitHub](https://github.com/FriendlyTech-Inc/japanese-app/blob/main/README.md#:~:text=1.%20%E5%8D%98%E8%AA%9E%E3%83%BB%E3%83%95%E3%83%AC%E3%83%BC%E3%82%BA%E8%A1%A8%E7%A4%BA%20,%E5%8D%98%E8%AA%9E%E3%81%AE%E5%88%86%E6%9B%B8%EF%BC%88%E5%85%AC%E5%9C%92%EF%BC%9Dpark%20%E3%81%AA%E3%81%A9%EF%BC%89%E3%82%92%E4%BD%B5%E8%A8%98)) Expoでは`import`や`require`でアセットを使用でき、事前に読み込んでキャッシュすることで再生遅延を減らします。  
- **data/**: アプリ内コンテンツを定義したJSONファイルを置きます。例えば`lessons.json`ではレッスン一覧と各レッスンに属するフレーズIDを定義し、`phrases.json`で全フレーズの詳細（日本語テキスト、ふりがな、翻訳、音声ファイル名、例文など）を持つ構造にします ([japanese-app/README.md at main · FriendlyTech-Inc/japanese-app · GitHub](https://github.com/FriendlyTech-Inc/japanese-app/blob/main/README.md#:~:text=1.%20%E5%8D%98%E8%AA%9E%E3%83%BB%E3%83%95%E3%83%AC%E3%83%BC%E3%82%BA%E8%A1%A8%E7%A4%BA%20,%E5%8D%98%E8%AA%9E%E3%81%AE%E5%88%86%E6%9B%B8%EF%BC%88%E5%85%AC%E5%9C%92%EF%BC%9Dpark%20%E3%81%AA%E3%81%A9%EF%BC%89%E3%82%92%E4%BD%B5%E8%A8%98)) JSON構造にしておくことで、新しいレッスンやフレーズの追加・編集が容易になり、非エンジニアでもコンテンツを更新しやすくなります。アプリ起動時に`contentService`がこれらJSONをパースしてメモリに保持するか、必要なときに都度読み込む方針とします（データ量によって調整）。  
- **types/**: アプリ全体で用いる型定義を管理します。TypeScriptのインターフェースや型エイリアスを用い、例として`Lesson`型（id, タイトル, フレーズ配列など）、`Phrase`型（日本語本文, 読み仮名, 翻訳, 音声パス, 例文 等 ([japanese-app/README.md at main · FriendlyTech-Inc/japanese-app · GitHub](https://github.com/FriendlyTech-Inc/japanese-app/blob/main/README.md#:~:text=1.%20%E5%8D%98%E8%AA%9E%E3%83%BB%E3%83%95%E3%83%AC%E3%83%BC%E3%82%BA%E8%A1%A8%E7%A4%BA%20,%E5%8D%98%E8%AA%9E%E3%81%AE%E5%88%86%E6%9B%B8%EF%BC%88%E5%85%AC%E5%9C%92%EF%BC%9Dpark%20%E3%81%AA%E3%81%A9%EF%BC%89%E3%82%92%E4%BD%B5%E8%A8%98)) 、`QuizQuestion`型（質問文, 正解, 選択肢一覧, 種別）などを定義します。型を明確にすることで、モックAPIから返るデータとUIで想定するデータ構造を常に一致させ、開発中に型エラーで不整合に気付けるメリットがあります。  
- **utils/**: 共通処理やカスタムフックを配置します。例えばテキストにふりがなルビを振るための関数、文字列から音声アセットURIを組み立てるヘルパー、あるいは`useAudio`フック（Expoの`Audio` APIをラップし再生/録音を簡潔に行う）などです。ユーティリティは純粋関数や汎用的なフックとして実装し、副作用を持たないか最小限にとどめます。これによりテストが容易で、他のプロジェクトへ流用もしやすくなります。

## 実装方針

上記構成に沿って、各ファイルを**単一責任の原則**に従い実装します。コンポーネントは表示専担、サービスはロジック専担、というように役割を明確に分離します。React Hooksと関数型コンポーネントを活用し、状態管理や副作用処理を適切に行います。

- **データフロー**: アプリ内のデータは基本的に`state`(Context)に集約し、一部の一時的なUI状態のみコンポーネント内部の`useState`で扱います。画面が表示されるときに`services`から必要データを取得し（例えば`useEffect`内で`contentService`を呼ぶ）、結果をコンポーネントのstateやContextに保存してUIを更新します。ユーザー操作→イベントハンドラ→`services`呼び出し→結果をstate更新→UI反映、という一連の流れを確立します。  
- **ナビゲーション**: React Navigationのパターンに従い、スタック/タブ構成を実装します。例えば通常レッスン→クイズ→結果の流れはスタックでプッシュし、ホームから各モードへの切替はタブで行う形です。`AppNavigator`で画面遷移図を一元管理することで、画面間連携を把握しやすくします。  
- **TypeScriptの活用**: すべてのコンポーネントと関数に適切な型を付与します。PropsやStateの型定義、サービスの返却値の型などを厳密にし、型エラー＝バグの芽として早期に発見します。たとえば`PhraseCard`のpropsには`phrase: Phrase`型（自前定義）と`onEvaluate: (result: Score) => void`のように型を定義します。型情報によりエディタでの補完が効き、可読性も向上します。  
- **コンポーネント設計**: Atomic Designを参考にレイヤー分けし、基本的なUI部品（Atoms/Molecules）は`components/common/`に集約、画面固有の組み合わせ(UI Templates/Organisms相当)は`screens/`や`components/lesson/`などに配置します。これにより**デザインの一貫性**を保ちつつ、新たな画面追加時も既存コンポーネントを組み合わせるだけで実装可能になります。  
- **スタイルとレイアウト**: Expo標準のスタイルシート（`StyleSheet.create`}や、必要に応じてCSS-in-JSライブラリ（例: Styled Components）を用いてスタイルを定義します。テーマやカラーパレットは共通の定数として管理し、ダークモード対応や多言語対応（翻訳テキストの外部化）も見据えます。特にテキストは翻訳ファイル（例えば`translations/en.json`, `ja.json`）に入れ、将来的な多言語対応を容易にします ([japanese-app/README.md at main · FriendlyTech-Inc/japanese-app · GitHub](https://github.com/FriendlyTech-Inc/japanese-app/blob/main/README.md#:~:text=1.%20%E5%8D%98%E8%AA%9E%E3%83%BB%E3%83%95%E3%83%AC%E3%83%BC%E3%82%BA%E8%A1%A8%E7%A4%BA%20,%E9%9F%B3%E5%A3%B0%E5%86%8D%E7%94%9F%EF%BC%88%E3%83%8D%E3%82%A4%E3%83%86%E3%82%A3%E3%83%96%E7%99%BA%E9%9F%B3%EF%BC%89))   

以上の実装方針により、UIとロジックの分離、型安全な開発、再利用性の高いコンポーネント設計が実現できます。コード規約やLint設定も適用し、プロジェクト全体でスタイル統一とバグの早期発見に努めます。

## モックAPIの設計

モックAPIは**非同期関数**として実装し、本物のAPIと同じインターフェースを模倣します。例えば`contentService`内で`export async function getLessons(): Promise<Lesson[]>`のように定義し、関数内でローカルJSONを読み込んで`Lesson`オブジェクト配列を返します（実際には`import lessonsData from '../data/lessons.json'`で静的インポート可能） ([japanese-app/README.md at main · FriendlyTech-Inc/japanese-app · GitHub](https://github.com/FriendlyTech-Inc/japanese-app/blob/main/README.md#:~:text=%E3%83%AC%E3%83%83%E3%82%B9%E3%83%B3%E5%8D%98%E4%BD%8D%E3%81%A7%E3%81%AE%E3%81%BE%E3%81%A8%E3%82%81%E3%82%AF%E3%82%A4%E3%82%BA%E3%82%84%E3%80%81%E3%82%B9%E3%83%94%E3%83%BC%E3%83%89%E5%BD%A2%E5%BC%8F%E3%81%AE%E9%80%A3%E7%B6%9A%E3%82%AF%E3%82%A4%E3%82%BA%E3%81%AA%E3%81%A9%E8%A4%87%E6%95%B0%E3%83%A2%E3%83%BC%E3%83%89%E3%81%AB%E5%AF%BE%E5%BF%9C%204)) 実行時にはPromise.resolveで即座に結果を返すか、開発テスト目的で`await new Promise(res => setTimeout(res, 500));`のように人工的な遅延を加えてからデータを返すこともできます。

**データ構造設計**については、JSONで以下のようなモデルを想定します:
- `Lesson`: { id, title, description, phraseIds: number[] }  
- `Phrase`: { id, japanese: string, furigana: string, translation: string, audio: string, example?: { ja: string, translation: string, audio: string } }  
- `QuizQuestion`: { id, question: string, options: string[], correctIndex: number, type: 'meaning'|'context' } など  

フレーズ情報は上記のように詳細項目を持ち、日本語表示とふりがな、翻訳（母国語訳）を含みます ([japanese-app/README.md at main · FriendlyTech-Inc/japanese-app · GitHub](https://github.com/FriendlyTech-Inc/japanese-app/blob/main/README.md#:~:text=1.%20%E5%8D%98%E8%AA%9E%E3%83%BB%E3%83%95%E3%83%AC%E3%83%BC%E3%82%BA%E8%A1%A8%E7%A4%BA%20,%E5%8D%98%E8%AA%9E%E3%81%AE%E5%88%86%E6%9B%B8%EF%BC%88%E5%85%AC%E5%9C%92%EF%BC%9Dpark%20%E3%81%AA%E3%81%A9%EF%BC%89%E3%82%92%E4%BD%B5%E8%A8%98)) 必要に応じて例文もオプションで含め、例文にも音声ファイルを用意します。Lessonはフレーズのリストを参照する形で、Quiz問題はフレーズから動的生成する場合はこの型を使わず都度組み立てます。

**モックAPI関数の例**:
- `getLessons()`: 全レッスンの基本情報を返す（学習モード選択画面用）。  
- `getLessonContent(lessonId)`: レッスン内の全フレーズ詳細を返す。必要ならここでフレーズごとの音声ファイルを事前ロードする処理も挟みます。  
- `evaluatePronunciation(phraseId, recordedAudio)`: 音声認識API呼び出しの代わりに、事前に用意した模擬結果を返します。例えば対応するPhraseに期待されるテキストがあるとして、recordedAudioは実際には使わず、常に「80点・Good」といった固定評価を返すか、ランダムでスコアを変える実装です ([japanese-app/README.md at main · FriendlyTech-Inc/japanese-app · GitHub](https://github.com/FriendlyTech-Inc/japanese-app/blob/main/README.md#:~:text=1.%20%E7%99%BA%E9%9F%B3%E5%85%A5%E5%8A%9B%E3%83%BB%E8%A9%95%E4%BE%A1%20,%E8%AA%A4%E5%B7%AE%E3%81%AE%E3%81%82%E3%82%8B%E7%99%BA%E9%9F%B3%E7%AE%87%E6%89%80%E3%81%AE%E7%B0%A1%E6%98%93%E3%83%95%E3%82%A3%E3%83%BC%E3%83%89%E3%83%90%E3%83%83%E3%82%AF%EF%BC%88%E3%81%A9%E3%81%AE%E9%9F%B3%E3%81%8C%E5%BC%B1%E3%81%84%E3%83%BB%E3%81%A9%E3%81%93%E3%82%92%E5%BC%B7%E8%AA%BF%E3%81%99%E3%81%B9%E3%81%8D%E3%81%AA%E3%81%A9%EF%BC%89%202.%20%E5%A4%9A%E6%A7%98%E3%81%AA%E5%87%BA%E9%A1%8C%E5%BD%A2%E5%BC%8F%EF%BC%88%E3%82%BF%E3%82%B9%E3%82%AF%E3%83%86%E3%83%B3%E3%83%97%E3%83%AC%E3%83%BC%E3%83%88%EF%BC%89)) 返却型は{ score: number, feedback: string }などとし、UIでスコア表示やフィードバックメッセージ表示に使います。  
- `generateQuizQuestions(mode, lessonId?)`: 引数によりクイズモードを判定し、問題リストを生成します。lessonIdがあればそのレッスン範囲内から出題し、なければ全範囲・カテゴリ指定などに応じて出題します ([japanese-app/README.md at main · FriendlyTech-Inc/japanese-app · GitHub](https://github.com/FriendlyTech-Inc/japanese-app/blob/main/README.md#:~:text=,%E3%82%AA%E3%83%AA%E3%82%B8%E3%83%8A%E3%83%AB%E3%83%9F%E3%83%8B%E3%83%AC%E3%83%83%E3%82%B9%E3%83%B3%20%E3%83%A6%E3%83%BC%E3%82%B6%E3%83%BC%E3%81%8C%E8%8B%A6%E6%89%8B%E3%83%95%E3%83%AC%E3%83%BC%E3%82%BA%E3%81%A0%E3%81%91%E3%82%92%E3%83%94%E3%83%83%E3%82%AF%E3%82%A2%E3%83%83%E3%83%97%E3%81%97%E3%80%81%E7%8B%AC%E8%87%AA%E3%83%AA%E3%82%B9%E3%83%88%E3%82%92%E4%BD%9C%E6%88%90%20%E2%86%92%20%E3%82%A4%E3%83%B3%E3%83%97%E3%83%83%E3%83%88%E3%83%BB%E3%82%A2%E3%82%A6%E3%83%88%E3%83%97%E3%83%83%E3%83%88%E3%83%BB%E3%82%AF%E3%82%A4%E3%82%BA%E3%82%92%E8%87%AA%E7%94%B1%E3%81%AB%E7%B5%84%E3%81%BF%E5%90%88%E3%82%8F%E3%81%9B%E3%81%A6%E5%86%8D%E5%AD%A6%E7%BF%92)) 選択肢の誤答は同じJSONデータ内から類似の意味を持つフレーズやランダムな他フレーズを抽出します。例えば「こんにちは」の英訳を問う問題なら、正解「Hello」に対し、他の選択肢は「Goodbye」「Thank you」など別のフレーズの訳を流用する、といった具合です。

モックAPI設計において重要なのは、**実際のAPIとのインターフェース互換性**と**簡潔なスタブロジック**です。前者により、後で本物のAPI実装に差し替える際もUI側のコード変更を最小限にできます。後者により、複雑なロジックを避けて開発スピードを上げつつ、アプリの流れ全体を検証できます。

## 開発ベストプラクティスの考慮

本設計には、パフォーマンス・可読性・保守性のベストプラクティスが随所に織り込まれています。

- **パフォーマンス**: リスト表示には仮想化対応の`FlatList`または`FlashList`を用いて大量のフレーズや問題を効率的にレンダリングします（大量データでもスクロール性能が維持されます） ([Project Structure | React Native / Expo Starter](https://starter.obytes.com/getting-started/project-structure/#:~:text=)) また、画面遷移間で必要なデータのみを取得し、不要になったコンポーネントはアンマウントしてメモリを解放します。重い処理（例: クイズ問題の生成やスコア計算）は`services`側で行い、必要ならWebWorker的な仕組み（現在のReact Nativeでは直接のWebWorkerは使えませんが、重い処理は分割する工夫）でUIスレッドをブロックしないよう配慮します。Expoの`Asset`機能で音声や画像を事前読み込みし、再生時の遅延を抑えます。アニメーションには`react-native-reanimated`などを利用してネイティブドライバで実行し、カクつきを防ぎます。  
- **可読性**: プロジェクトの構造を論理的に分割し、ファイル/フォルダ命名もその責務がひと目で分かるようにしています。例えば`QuizScreen.tsx`を開けばクイズ画面の全体像が把握でき、細かな選択肢UIは`QuizOption.tsx`に委ねるなど、コードの見通しを良くしています。さらに、コード中に適切なコメントを添え、関数やコンポーネントの意図が明確になるようにします。ファイルが肥大化しそうな場合は早めに分割し、一つのファイルが数百行を超えないよう留意します。  
- **保守性**: 型定義を集中管理し、変更に強い設計にしています。例えばデータ構造に変更（フレーズに新項目追加等）があった場合でも、型定義を更新すればコンパイラが影響箇所を指摘してくれるため、安全に修正を加えられます。コンポーネントとロジックの分離により、UIデザインの変更やAPI仕様変更にも局所的な修正で対応可能です。プロジェクト構成は一般的なReact Nativeのベストプラクティスに沿っており、他の開発者も理解しやすい形です ([Project Structure | React Native / Expo Starter](https://starter.obytes.com/getting-started/project-structure/#:~:text=Note))  ([Project Structure | React Native / Expo Starter](https://starter.obytes.com/getting-started/project-structure/#:~:text=We%20use%20absolute%20imports%20to,code%20cleaner%20and%20more%20readable)) 例えば**絶対パスインポート**の導入（`babel-plugin-module-resolver`利用）により、`import { QuizScreen } from '@/screens/QuizScreen';`のようにシンプルにモジュールを参照できます ([Project Structure | React Native / Expo Starter](https://starter.obytes.com/getting-started/project-structure/#:~:text=We%20use%20absolute%20imports%20to,code%20cleaner%20and%20more%20readable)) これによりリファクタリングやファイル移動時もインポートパスの修正が少なく済みます。

- **UXと機能拡張性**: モックアプリとはいえ、できるだけ実際のユーザー体験に近づけます。レスポンスの遅延やアニメーション、トースト通知など細部も作り込みます。また、将来的な機能追加（例えばAIによる学習者へのフレーズ推薦や他ユーザーとの進捗共有 ([japanese-app/README.md at main · FriendlyTech-Inc/japanese-app · GitHub](https://github.com/FriendlyTech-Inc/japanese-app/blob/main/README.md#:~:text=1.%20%E5%AD%A6%E7%BF%92%E3%82%A2%E3%83%AB%E3%82%B4%E3%83%AA%E3%82%BA%E3%83%A0%E3%81%AE%E9%AB%98%E5%BA%A6%E5%8C%96%20,%E5%BA%83%E5%91%8A%E3%83%A2%E3%83%87%E3%83%AB%E3%81%A8%E3%81%AE%E3%83%90%E3%83%A9%E3%83%B3%E3%82%B9%E3%82%92%E8%80%83%E6%85%AE%E3%81%97%E3%80%81%E5%AD%A6%E7%BF%92%E4%BD%93%E9%A8%93%E3%82%92%E6%90%8D%E3%81%AA%E3%82%8F%E3%81%AA%E3%81%84%E5%BD%A2%E3%81%A7%E3%81%AE%E5%B0%8E%E5%85%A5%E3%82%92%E6%A4%9C%E8%A8%8E)) を見据え、コードの拡張性を意識します。コンテンツの柔軟な再利用設計 ([japanese-app/README.md at main · FriendlyTech-Inc/japanese-app · GitHub](https://github.com/FriendlyTech-Inc/japanese-app/blob/main/README.md#:~:text=%E6%9C%AC%E3%82%A2%E3%83%97%E3%83%AA%E3%81%AF%E3%80%81%E3%82%A4%E3%83%B3%E3%83%97%E3%83%83%E3%83%88%E3%81%A8%E3%82%A2%E3%82%A6%E3%83%88%E3%83%97%E3%83%83%E3%83%88%E3%82%92%E4%B8%80%E4%BD%93%E7%9A%84%E3%81%AB%E5%AD%A6%E3%81%B9%E3%82%8B%E3%82%88%E3%81%86%E3%81%AA%E5%AD%A6%E7%BF%92%E8%A8%AD%E8%A8%88%E3%82%92%E6%A0%B8%E3%81%A8%E3%81%97%E3%80%81%E7%89%B9%E3%81%AB%E9%9F%B3%E5%A3%B0%E7%99%BA%E8%A9%B1%E7%B7%B4%E7%BF%92%E3%82%92%E5%A4%9A%E5%BD%A9%E3%81%AA%E5%BD%A2%E3%81%A7%E3%82%B5%E3%83%9D%E3%83%BC%E3%83%88%E3%81%99%E3%82%8B%E3%81%93%E3%81%A8%E3%81%8C%E5%A4%A7%E3%81%8D%E3%81%AA%E7%89%B9%E5%BE%B4%E3%81%A7%E3%81%99%E3%80%82)) より、新しい学習モードの追加も容易で、プロトタイプから本実装への発展もスムーズです。

以上の設計により、Expo + TypeScriptを用いた日本語学習モックアプリは、要件を満たしつつ高品質なコードベースとなります。ディレクトリ構成と責務分離により開発チームでの協業もしやすく、将来的な機能拡張や本物のAPI接続への移行もスムーズに行えるよう配慮しています。各種ベストプラクティスを踏襲したこのアプリケーション設計をベースに、実装を進めていきます。 ([japanese-app/README.md at main · FriendlyTech-Inc/japanese-app · GitHub](https://github.com/FriendlyTech-Inc/japanese-app/blob/main/README.md#:~:text=%E3%83%AC%E3%83%83%E3%82%B9%E3%83%B3%E5%8D%98%E4%BD%8D%E3%81%A7%E3%81%AE%E3%81%BE%E3%81%A8%E3%82%81%E3%82%AF%E3%82%A4%E3%82%BA%E3%82%84%E3%80%81%E3%82%B9%E3%83%94%E3%83%BC%E3%83%89%E5%BD%A2%E5%BC%8F%E3%81%AE%E9%80%A3%E7%B6%9A%E3%82%AF%E3%82%A4%E3%82%BA%E3%81%AA%E3%81%A9%E8%A4%87%E6%95%B0%E3%83%A2%E3%83%BC%E3%83%89%E3%81%AB%E5%AF%BE%E5%BF%9C%204)) 14†L513-L518】
