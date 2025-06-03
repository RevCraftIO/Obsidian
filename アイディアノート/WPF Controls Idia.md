---
作成日: 2025-06-03
tags:
  - CSharp
  - 開発
  - "#アイディア"
---

# WPFカスタムコントロール一覧（全60種）

以下は、実務で役立つWPF向けのカスタムコントロールのアイディアを表形式でまとめたものです。

| No  | コントロール名               | 概要                    | 主な用途             | 特徴                          |
| --- | --------------------- | --------------------- | ---------------- | --------------------------- |
| 1   | PlaceholderTextBox    | プレースホルダー付きTextBox     | 入力補助             | `Hint`属性付き、フォーカス時非表示        |
| 2   | WatermarkPasswordBox  | プレースホルダー付きPasswordBox | ログイン画面など         | `PasswordBox`にヒント表示         |
| 3   | NumericUpDown         | 数値入力専用のTextBox        | 金額、数量、温度など       | `+/-`ボタン、バリデーション対応          |
| 4   | DateRangePicker       | 期間選択UI（From〜To）       | 帳票、フィルターなど       | `DatePicker`2つ＋制約バリデーション    |
| 5   | EnumComboBox          | 列挙型バインド付きComboBox     | 設定画面、ステータス選択     | 自動で`Enum`の項目を列挙             |
| 6   | ToggleIconButton      | アイコン付きトグルボタン          | フラグ設定、表示ON/OFFなど | `IsChecked`対応、切り替えアニメ可      |
| 7   | BusyIndicator         | 非同期処理中を示すインジケータ       | データ読込中、API呼び出し   | オーバーレイ表示、アニメ付き              |
| 8   | StepProgressBar       | 手順や進捗段階を可視化           | ウィザードUI、工程表示     | ステップ数動的対応、現在ステップ強調          |
| 9   | NotificationPanel     | 一時的な通知表示              | エラー通知、成功メッセージ    | タイムアウト・アニメーション付き            |
| 10  | ExpanderList          | 折りたたみ式リスト表示           | FAQ、カテゴリ別リストなど   | `Expander`＋`ItemsControl`連携 |
| 11  | EditableComboBox      | 入力と選択両対応ComboBox      | コード入力＋候補選択       | オートコンプリート付きも可               |
| 12  | MaskedTextBox         | 入力マスク付きTextBox        | 電話番号、日付、郵便番号など   | 正規表現＋マスク指定対応                |
| 13  | MultiSelectComboBox   | 複数選択できるComboBox       | タグ、カテゴリ、複数条件など   | チェックボックス付きドロップダウン           |
| 14  | BreadcrumbControl     | パス階層の表示・選択UI          | フォルダ構造、ページ階層など   | セパレータ付き、ドロップダウン対応           |
| 15  | CollapsibleGroupBox   | 折りたたみ可能なGroupBox      | 設定画面、サブセクション     | タイトルクリックで展開/折りたたみ           |
| 16  | UnitTextBox           | 単位付き入力ボックス            | 長さ、重量、金額など       | フォーカス時のみ単位非表示               |
| 17  | HistoryComboBox       | 履歴保存型ComboBox         | 入力履歴、ファイル履歴など    | ローカル保存、選択補助に有効              |
| 18  | CharacterCountTextBox | 文字数制限＋カウント表示TextBox   | コメント、SNS投稿など     | リアルタイムカウント表示                |
| 19  | InlineValidationBox   | 入力時に即時バリデーション表示       | フォームUI           | アイコン＋エラーメッセージ表示対応           |
| 20  | ThemeToggleSwitch     | ライト/ダーク切替スイッチ         | テーマ設定            | アニメ付き、状態保存対応                |
| 21  | ZoomablePanel         | ズーム対応コンテンツ表示領域        | 地図、図面、画像など       | ホイールズーム、パン対応                |
| 22  | RichToolTip           | 高機能ツールチップ             | ヘルプ、画像・操作補足      | ボタンや画像埋込可能                  |
| 23  | DialogHost            | MVVM対応モーダル表示UI        | 入力ダイアログ、確認画面     | `Task`ベース非同期操作対応            |
| 24  | FilterPanel           | データフィルター条件UI          | 一覧検索、帳票出力など      | 比較演算子＋条件バインド                |
| 25  | SkeletonLoader        | データ読み込み中のスケルトン表示      | ロード中のUX向上        | グレーアウト・アニメ付き                |
| 26  | InfiniteScrollList    | 無限スクロール対応リスト          | SNS風一覧、ログ表示など    | 自動読み込み＋ローディング表示             |
| 27  | LogViewer             | ログ表示UI                | 開発・管理者ツール        | リアルタイム更新、レベル分類対応            |
| 28  | PropertyGrid          | オブジェクトのプロパティ編集UI      | 管理ツール、設定画面       | 型に応じて適切な編集UI生成              |
| 29  | DropZone              | ファイルドロップ対応UI          | アップロード、画像取り込み    | `DragEnter/Leave`ハイライト付き    |
| 30  | ResizablePanel        | サイズ変更可能な領域            | サイドバー、可変パネル      | ドラッグでサイズ調整可                 |
| 31  | MultiColumnComboBox   | 複数列表示のComboBox        | 商品コード＋名称選択など     | DataGrid風、並べ替え可能            |
| 32  | TimelineControl       | イベントを時系列表示            | 履歴、ログ、プロセス表示     | ズーム・スクロール対応                 |
| 33  | TabCloseableControl   | 閉じるボタン付きタブ            | ブラウザ風UI          | タブの動的追加削除対応                 |
| 34  | ImageCropper          | 画像トリミングUI             | 画像加工、プロフィール設定    | 矩形選択、倍率指定可                  |
| 35  | CustomKeyboard        | ソフトウェアキーボード           | タッチパネル操作         | 配列カスタマイズ可能                  |
| 36  | BreadcrumbNavigator   | 階層ナビゲーションUI           | パス選択、履歴表示        | フォルダ構造対応、折りたたみあり            |
| 37  | CodeEditorControl     | シンタックスハイライト付きエディタ     | スクリプト、設定編集       | 行番号、構文色分け、折りたたみ             |
| 38  | GaugeMeter            | 数値ゲージ表示UI             | センサ表示、KPI表示      | 円形・直線、アニメーション対応             |
| 39  | RatingControl         | 星評価入力コントロール           | レビューUI、フィードバック   | 半星、カスタムアイコン対応               |
| 40  | TagInputBox           | タグ入力用TextBox          | 属性指定、キーワード入力     | 削除ボタン付きチップ表示                |
| 41  | CarouselViewer        | 横スクロール型ビューア           | 商品一覧、画像表示        | スナップスクロール、アニメ付              |
| 42  | PasswordStrengthBar   | パスワード強度表示バー           | アカウント作成画面        | 長さ・記号使用でスコア化                |
| 43  | FileSystemTreeView    | ファイル構造表示ツリー           | ファイル操作UI、フォルダ選択  | 遅延読み込み、コンテキストメニュー           |
| 44  | PivotGrid             | クロス集計グリッド             | 帳票、BIダッシュボード     | 行列切替、集計設定可                  |
| 45  | ColorPickerControl    | 色選択ダイアログ              | テーマ設定、ペイントUI     | HSV/RGBスライダー対応              |
| 46  | SpeechToTextInput     | 音声入力TextBox           | 音声メモ、バリアフリー支援    | `System.Speech`／API対応       |
| 47  | HotkeyRecorder        | ホットキー入力UI             | 設定画面、ショートカット割当   | キーコンボ録音・表示対応                |
| 48  | UndoRedoStackViewer   | Undo/Redo履歴表示UI       | 編集履歴ナビゲータ        | ツリー構造、履歴スキップ操作可             |
| 49  | FormGrid              | 項目名と入力欄の整列グリッド        | 登録フォーム、設定画面      | ラベル/入力幅自動調整、バリデーション統合       |
| 50  | NumericTextBlock      | 数値の書式付き表示             | 金額、統計値、センサ値など    | 桁区切り、小数点桁数、単位対応             |
| 51  | SignaturePad          | 手書き署名用キャンバス           | 受付、電子契約、配達確認     | マウス／タッチ対応、PNG出力可            |
| 52  | CommandRecorderPanel  | 実行した操作コマンドを記録・表示      | 操作ログ、学習補助        | `ICommand`呼び出し履歴バインド        |
| 53  | ScrollAwarePanel      | スクロールに応じてUI変更         | ヘッダー固定、スクロール進捗表示 | `ScrollViewer`と連携、進捗バー対応    |
| 54  | ResponsivePanel       | 画面サイズに応じてレイアウト切替      | ウィンドウサイズ変化対応     | カラム／行切替、ブレークポイント定義可         |
| 55  | AdvancedToggleSwitch  | ラベル付きのトグルスイッチ         | 有効/無効、ON/OFF設定   | アイコン付き、状態記憶対応               |
| 56  | QuickAccessToolbar    | ショートカットボタンを集約表示       | よく使う機能、アクションパネル  | ドラッグ並び替え、登録変更可能             |
| 57  | ListDetailsView       | 選択項目の詳細を表示するビュー       | マスタ管理、リスト編集      | 選択→右側詳細表示、`MVVM`適合構造        |
| 58  | VirtualizedGrid       | 大量データに対応したグリッド表示      | マスタ一覧、ログビューア     | 仮想化対応、高速スクロール性能             |
| 59  | ContextAwareTooltip   | 状況依存ツールチップ            | 入力支援、動作ガイド       | 入力内容や状態に応じて切替表示             |
| 60  | LiveChartControl      | ライブで更新されるグラフ          | センサ値、株価、状態監視     | リアルタイム更新、複数系列対応             |
| 61  | SearchableTreeView    | 検索機能付きツリービュー          | 大規模ツリー構造のナビゲーション | インクリメンタル検索、ハイライト表示          |
| 62  | MarkdownViewer        | Markdown表示・編集コントロール   | ドキュメント、README表示  | プレビュー付き、シンタックスハイライト         |
| 63  | ColorPalettePicker    | カラーパレット選択UI           | デザインツール、テーマ設定    | プリセット、カスタムパレット保存            |
| 64  | TimeRangeSlider       | 時間範囲選択スライダー           | タイムライン、スケジュール表示  | ドラッグ操作、範囲制約対応               |
| 65  | FilePreviewControl    | ファイルプレビュー表示           | ファイル選択、プレビュー表示   | サムネイル生成、複数形式対応              |
| 66  | GridSplitterPanel     | 分割可能なパネルレイアウト         | リサイズ可能なUI領域      | ドラッグ分割、最小サイズ制約              |
| 67  | AutoCompleteTextBox   | 高度な自動補完機能付きTextBox    | 検索、コード入力補助       | カスタム補完ロジック、非同期検索対応          |
| 68  | ProgressRing          | 円形プログレスインジケータ         | ローディング表示、進捗表示    | アニメーション、カスタマイズ可能            |
| 69  | CalendarHeatmap       | カレンダーヒートマップ表示         | アクティビティ、統計表示     | 日付ベース、色分け表示                 |
| 70  | KanbanBoard           | カンバンボードUI             | タスク管理、ワークフロー     | ドラッグ&ドロップ、カスタムカラム           |
| 71  | SyntaxHighlightBox    | シンタックスハイライト付きTextBox  | コード入力、ログ表示       | 言語別ハイライト、行番号表示              |
| 72  | ImageGallery          | 画像ギャラリー表示UI           | 写真管理、プレビュー表示     | グリッド/リスト表示切替、ズーム対応          |
| 73  | DataFilterPanel       | データフィルタリングUI          | 一覧検索、条件絞り込み      | 動的条件追加、複合条件対応               |
| 74  | NotificationBadge     | 通知バッジ表示コントロール         | 未読数、アラート表示       | アニメーション、カスタムスタイル            |
| 75  | TreeGrid              | ツリー構造付きデータグリッド        | 階層データ表示、マスタ管理    | 展開/折りたたみ、ソート機能              |
| 76  | ColorGradientPicker   | グラデーション選択UI           | グラデーション設定、デザイン   | カラーポイント編集、プレビュー表示           |
| 77  | FileDropZone          | ファイルドロップエリア           | ファイルアップロード、インポート | ドラッグ&ドロップ、プレビュー表示           |
| 78  | CommandPalette        | コマンドパレットUI            | クイックアクセス、ショートカット | 検索機能、カテゴリ分類                 |
| 79  | TimelineSlider        | タイムラインスライダー           | 動画再生、時系列データ表示    | マーカー表示、範囲選択機能               |
| 80  | RichTextEditor        | リッチテキストエディタ           | 文書編集、メール作成       | フォーマット、画像挿入対応               |
| 81  | DataPivotView         | ピボットテーブル表示UI          | データ分析、集計表示       | ドラッグ&ドロップ、集計方法選択            |
| 82  | ColorSchemeEditor     | カラースキームエディタ           | テーマ設定、デザイン       | プレビュー、エクスポート機能              |
| 83  | FileExplorer          | ファイルエクスプローラーUI        | ファイル管理、ナビゲーション   | ツリービュー、リストビュー統合             |
| 84  | ChartAnnotation       | グラフ注釈コントロール           | データ分析、マーキング      | テキスト、図形、矢印追加機能              |
| 85  | CodeDiffViewer        | コード差分表示UI             | バージョン管理、比較表示     | 行単位比較、ハイライト表示               |
| 86  | DataValidationPanel   | データ検証パネル              | フォーム入力、バリデーション   | リアルタイム検証、エラー表示              |
| 87  | ImageAnnotation       | 画像注釈コントロール            | 画像編集、マーキング       | 図形、テキスト、矢印追加機能              |
| 88  | DataExportPanel       | データエクスポートパネル          | データ出力、レポート作成     | 複数形式対応、プレビュー機能              |
| 89  | ColorBlindSimulator   | 色覚シミュレーター             | アクセシビリティ、デザイン    | 各種色覚タイプシミュレーション             |
| 90  | FileCompareViewer     | ファイル比較ビューア            | 差分確認、マージ支援       | サイドバイサイド表示、ハイライト            |
| 91  | DataImportWizard      | データインポートウィザード         | データ取り込み、変換       | ステップ形式、プレビュー機能              |
| 92  | ImageCropRotate       | 画像トリミング・回転UI          | 画像編集、加工          | 回転、リサイズ、アスペクト比保持            |
| 93  | DataFilterBuilder     | データフィルター構築UI          | 高度な検索条件作成        | ビジュアル構築、条件プレビュー             |
| 94  | ColorAccessibility    | 色のアクセシビリティチェッカー       | デザイン検証、改善        | コントラスト比、WCAG準拠チェック          |
| 95  | FileVersionControl    | ファイルバージョン管理UI         | 履歴管理、差分表示        | バージョン比較、履歴表示                |
| 96  | DataPivotChart        | ピボットチャート表示UI          | データ分析、可視化        | ドラッグ&ドロップ、動的更新              |
| 97  | ImageMetadataEditor   | 画像メタデータエディタ           | 画像情報編集、管理        | EXIF、IPTC、XMP対応             |
| 98  | DataValidationRules   | データ検証ルールエディタ          | バリデーション設定、管理     | ルール構築、テスト機能                 |
| 99  | ColorThemeGenerator   | カラーテーマ生成UI            | デザイン、テーマ作成       | 自動生成、プレビュー機能                |
| 100 | FileSyncStatus        | ファイル同期状態表示UI          | 同期状態、進捗表示        | リアルタイム更新、エラー表示              |

---

