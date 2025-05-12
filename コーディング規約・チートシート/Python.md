# Python コーディング規約

## 目次

1. [はじめに](#はじめに)
2. [コードレイアウト](#コードレイアウト)
3. [命名規則](#命名規則)
4. [インポート](#インポート)
5. [コメント](#コメント)
6. [文字列](#文字列)
7. [式と文](#式と文)
8. [関数](#関数)
9. [クラス](#クラス)
10. [例外処理](#例外処理)
11. [テスト](#テスト)
12. [型ヒント](#型ヒント)

## はじめに

この規約は、読みやすく一貫性のあるPythonコードを書くためのガイドラインです。主にPEP 8（Python Enhancement Proposal 8）に基づいていますが、いくつかの社内独自の規約も含まれています。

### 基本原則

- 可読性が最も重要です。
- 一貫性を保ちましょう。
- 複雑さよりも明快さを優先しましょう。
- コードは書く時間よりも読まれる時間の方が長いことを認識しましょう。

## コードレイアウト

### インデント

- **インデントには4つのスペースを使用** します。タブは使用しません。
- 行の継続にはインデントを揃えるか、ぶら下げインデント（hanging indent）を使用します。

```python
# 良い例: インデントを揃える
foo = long_function_name(var_one, var_two,
                         var_three, var_four)

# 良い例: ぶら下げインデント
foo = long_function_name(
    var_one, var_two,
    var_three, var_four)
```

### 行の長さ

- 1行の長さは最大**88文字**に制限します。
- 長い行は適切に改行します。
- バックスラッシュ(`\`)を使った行の継続よりも、括弧内の暗黙の行継続を優先します。

```python
# 良い例: 括弧内で改行
income = (gross_wages
          + taxable_interest
          + (dividends - qualified_dividends)
          - ira_deduction
          - student_loan_interest)

# 避けるべき例: バックスラッシュを使用
income = gross_wages \
         + taxable_interest \
         + (dividends - qualified_dividends) \
         - ira_deduction \
         - student_loan_interest
```

### 空行

- トップレベルの関数とクラスの定義は2行の空行で区切ります。
- クラス内のメソッド定義は1行の空行で区切ります。
- 関連する一群のプロパティ定義を区切るために追加の空行を使用することができます。

### 空白

- 以下の場所では空白を避けます:
  - 括弧、大括弧、角括弧の内側: `spam(ham[1], {eggs: 2})`
  - カンマ、セミコロン、コロンの前: `if x == 4: print(x, y); x, y = y, x`
  - 関数呼び出しの開き括弧の前: `spam(1)`
  - インデックスやスライスの角括弧の前: `dct['key'] = lst[index]`

- 以下の場所では空白を使用します:
  - 二項演算子の前後: `x == 1`
  - 代入演算子の前後: `x = 1`
  - カンマの後: `a, b = 1, 2`
  - コロンの後（ただし辞書のキーを指定する場合を除く）: `def func(x): ...`

```python
# 良い例：適切な空白の使用
def complex(real, imag=0.0):
    return magic(r=real, i=imag)

# 避けるべき例：不適切な空白
def complex ( real, imag = 0.0 ):
    return magic( r = real, i = imag )
```

## 命名規則

### 一般的な命名規則

- **モジュール名**: 短い、全て小文字、アンダースコアなし: `utils.py`, `email_validator.py`
- **パッケージ名**: 短い、全て小文字、アンダースコアなし: `django`, `numpy`
- **クラス名**: CapWords (PascalCase): `Person`, `EmailValidator`
- **例外名**: CapWords、例外は「Error」で終わる: `ValueError`, `ConnectionError`
- **関数名**: snake_case: `send_email()`, `get_user_by_id()`
- **メソッド名**: snake_case: `save()`, `get_absolute_url()`
- **変数名**: snake_case: `user_email`, `item_count`
- **定数名**: 大文字とアンダースコア: `MAX_RETRY_COUNT`, `PI`
- **型変数名**: CapWords、通常は短い名前: `T`, `AnyStr`, `ResponseT`

### 特別な命名規則

- **プライベートメンバー**: アンダースコア1つをプレフィックスとして使用: `_private_method()`
- **"内部利用"変数**: アンダースコア2つをプレフィックスとして使用: `__internal_value`
- **マジックメソッド**: アンダースコア2つで囲む: `__init__()`, `__str__()`

### 命名のベストプラクティス

- 意味のある名前を選びましょう。`data`, `info`, `value`などの一般的すぎる名前は避けましょう。
- 略語よりも完全な単語を使用しましょう（ただし広く使われている略語は例外）。
- 意味が明確な場合のみ単一文字の変数名（`i`, `j`, `k`）を使用しましょう（ループカウンタなど）。
- booleansには `is_`, `has_`, `can_` などのプレフィックスを使用しましょう。

```python
# 良い例
def is_valid_email(email):
    """メールアドレスのバリデーションを行います。"""
    return re.match(EMAIL_PATTERN, email) is not None

# 避けるべき例
def valid(e):
    """不明確な名前"""
    return re.match(EMAIL_PATTERN, e) is not None
```

## インポート

### インポートの順序

インポートは以下の順に記述し、各グループ間に空行を入れます:

1. 標準ライブラリのインポート
2. サードパーティのライブラリのインポート
3. ローカルアプリケーション/ライブラリのインポート

各グループ内ではアルファベット順に並べます。

```python
# 標準ライブラリのインポート
import os
import sys
from datetime import datetime

# サードパーティのライブラリのインポート
import numpy as np
import pandas as pd
import requests

# ローカルアプリケーションのインポート
from myapp.models import User
from myapp.utils import validate_email
```

### インポートのスタイル

- `import x` または `from x import y` の形式を使用します。
- `from x import *` は避けましょう (名前空間が汚染されます)。
- 長いモジュール名には省略名を使用します: `import numpy as np`
- 関連するインポートはグループ化できますが、それ以外は1行に1つのインポートを記述します。

```python
# 良い例
import os
import sys

# 関連するインポートはグループ化できる
from django.db import models
from django.conf import settings

# 長いモジュール名には省略名を使用
import matplotlib.pyplot as plt
import numpy as np

# 避けるべき例
import os, sys  # 複数のモジュールを1行にまとめない
from module import *  # ワイルドカードインポートを避ける
```

## コメント

### ドキュメンテーション文字列（Docstrings）

- モジュール、関数、クラス、メソッドにはdocstringsを記述します。
- docstringsには三重引用符（`"""`）を使用します。
- 一行docstringsは一行で完結させます。
- 複数行docstringsは、概要行の後に空行を入れ、詳細を記述します。

```python
def calculate_tax(amount, rate):
    """税金を計算して返します。

    Args:
        amount (float): 元の金額
        rate (float): 税率（例: 0.1 で 10%）

    Returns:
        float: 計算された税額
    
    Raises:
        ValueError: 金額または税率が負の場合
    """
    if amount < 0 or rate < 0:
        raise ValueError("金額と税率は0以上である必要があります")
    return amount * rate
```

### インラインコメント

- コードの難解な部分を説明するためにコメントを使用します。
- コメントは完全な文で書き、最初の単語を大文字で始めます。
- コードとコメントを区別するために少なくとも2つのスペースを入れます。
- 明らかなことをコメントで説明することは避けましょう。

```python
# 良い例
x = x + 1  # 境界条件を補正

# 避けるべき例
x = x + 1  # xをインクリメント（自明）
```

### TODO コメント

- 一時的なコード、将来の開発のためのプレースホルダー、または警告には "TODO" コメントを使用します。
- 標準形式を使用します: `# TODO: 説明`

```python
# TODO: このメソッドをユーザー設定に対応させる
def process_data(data):
    """データを処理します。"""
    return data.transform()
```

## 文字列

### 文字列のクォート

- 一貫性があれば、シングルクォート（`'`）とダブルクォート（`"`）のどちらを使用しても構いません。
- 文字列内にクォートが含まれる場合は、エスケープを避けるためにもう一方のクォートを使用します。
- 三重引用符（`"""`）はdocstringsと複数行の文字列に使用します。

```python
# シングルクォートの使用例
name = 'John Doe'

# 文字列内にシングルクォートが含まれる場合はダブルクォートを使用
message = "It's a beautiful day"

# 複数行の文字列には三重引用符を使用
query = """
SELECT *  
FROM users
WHERE email = 'user@example.com'
"""
```

### 文字列フォーマット

- フォーマット済み文字列リテラル（f-strings）を優先して使用します。
- `.format()` メソッドも許容されます。
- 古いスタイルの `%` フォーマットは避けましょう。

```python
name = "Alice"
age = 30

# 良い例：f-strings
message = f"{name} is {age} years old"

# 許容される例：str.format()
message = "{} is {} years old".format(name, age)

# 避けるべき例：% フォーマット
message = "%s is %d years old" % (name, age)
```

## 式と文

### 比較

- `None`, `True`, `False` との比較には `is`, `is not` を使用します: `if x is None:`
- 空のシーケンス（文字列、リスト、タプルなど）の検査には暗黙のbooleanを使用します: `if not items:`
- Pythonの暗黙の真偽値変換を活用しましょう。
- 明示的な比較が明確さを向上させる場合は使用しましょう: `if length == 0:`

```python
# 良い例
if not users:  # ユーザーリストが空かどうか
    print("No users found")

if user is None:  # None との比較
    print("User not set")

# 避けるべき例
if len(users) == 0:  # 冗長
    print("No users found")

if user == None:  # is を使うべき
    print("User not set")
```

### 条件式

- シンプルな条件式は1行で記述できます。
- 複雑なロジックは複数行に分けて記述しましょう。
- 三項演算子は短い式にのみ使用しましょう。

```python
# 良い例：シンプルな三項演算子
status = "active" if user.is_logged_in else "inactive"

# 良い例：複雑なロジックは複数行に分ける
if (user.is_authenticated and 
        user.has_permission('edit') and 
        not article.is_locked):
    article.update(request.data)
```

### 代入式（:= ウォルラス演算子）

- 代入式は値を割り当てると同時に式を評価できる場合に使用します。
- 読みやすさを損なう場合は使用を避けましょう。

```python
# 良い例：代入と条件チェックを組み合わせる
if (match := pattern.search(text)) is not None:
    print(f"Found: {match.group()}")

# 良い例：繰り返し実行される関数の結果を保存
while (chunk := file.read(8192)):
    process(chunk)
```

## 関数

### 関数定義

- 関数名は動詞または動詞句を使用します。
- 関数は一つのタスクを実行するように設計します。
- 引数と戻り値には型ヒントを使用します。

```python
def calculate_average(numbers: list[float]) -> float:
    """数値リストの平均を計算します。

    Args:
        numbers: 平均を計算する数値のリスト

    Returns:
        リストの数値の平均値

    Raises:
        ValueError: リストが空の場合
    """
    if not numbers:
        raise ValueError("空のリストの平均は計算できません")
    return sum(numbers) / len(numbers)
```

### 引数

- デフォルト値を持つ引数はデフォルト値を持たない引数の後に配置します。
- ミュータブル型（リスト、辞書など）をデフォルト引数として使用しないでください。

```python
# 良い例：デフォルト値の適切な使用
def create_user(name, email, is_active=True, roles=None):
    if roles is None:
        roles = ["user"]
    # 実装

# 避けるべき例：ミュータブルなデフォルト引数
def append_to_list(item, items=[]):  # バグの原因になる
    items.append(item)
    return items
```

### 戻り値

- 複数の値を返す場合はタプルまたは適切なデータ構造を使用します。
- 関数が何も返さない場合は明示的に `return None` と書く必要はありません。

```python
# 良い例：複数の値を返す
def get_user_stats(user_id: int) -> tuple[int, int, float]:
    """ユーザーの統計情報を取得します。"""
    posts_count = count_posts(user_id)
    followers_count = count_followers(user_id)
    engagement_rate = calculate_engagement(user_id)
    return posts_count, followers_count, engagement_rate

# 使用例
posts, followers, engagement = get_user_stats(user_id)
```

## クラス

### クラス定義

- クラス名はCapWords (PascalCase) を使用します。
- クラスは `object` を明示的に継承する必要はありません（Python 3）。
- クラス変数は慎重に使用し、インスタンス変数と混同しないようにします。

```python
class User:
    """ユーザーを表すクラス。"""
    
    # クラス変数
    all_users = []
    
    def __init__(self, name: str, email: str):
        """新しいユーザーを初期化します。"""
        self.name = name  # インスタンス変数
        self.email = email  # インスタンス変数
        self.is_active = True  # インスタンス変数
        User.all_users.append(self)
    
    def deactivate(self) -> None:
        """ユーザーを非アクティブ化します。"""
        self.is_active = False
```

### メソッド

- インスタンスメソッドの最初のパラメータは `self` とします。
- クラスメソッドの最初のパラメータは `cls` とし、`@classmethod` デコレータを使用します。
- スタティックメソッドには `@staticmethod` デコレータを使用します。

```python
class Order:
    """注文を表すクラス。"""
    
    TAX_RATE = 0.1
    
    def __init__(self, customer_id: str, items: list[dict]):
        self.customer_id = customer_id
        self.items = items
        self.total = self.calculate_total()
    
    def calculate_total(self) -> float:
        """注文の合計金額を計算します（税込）。"""
        subtotal = sum(item["price"] * item["quantity"] for item in self.items)
        return subtotal * (1 + self.TAX_RATE)
    
    @classmethod
    def from_json(cls, json_data: str) -> "Order":
        """JSON文字列から注文オブジェクトを作成します。"""
        data = json.loads(json_data)
        return cls(data["customer_id"], data["items"])
    
    @staticmethod
    def validate_item(item: dict) -> bool:
        """注文項目が有効かどうかを検証します。"""
        return (isinstance(item, dict) and
                "price" in item and
                "quantity" in item and
                item["price"] > 0 and
                item["quantity"] > 0)
```

### プロパティ

- ゲッターとセッターの代わりにプロパティを使用します。
- プロパティは計算コストの低い属性アクセスに使用し、重い処理はメソッドとして実装します。

```python
class Rectangle:
    """長方形を表すクラス。"""
    
    def __init__(self, width: float, height: float):
        self._width = width
        self._height = height
    
    @property
    def width(self) -> float:
        """長方形の幅を取得します。"""
        return self._width
    
    @width.setter
    def width(self, value: float) -> None:
        """長方形の幅を設定します。"""
        if value <= 0:
            raise ValueError("幅は正の値でなければなりません")
        self._width = value
    
    @property
    def height(self) -> float:
        """長方形の高さを取得します。"""
        return self._height
    
    @height.setter
    def height(self, value: float) -> None:
        """長方形の高さを設定します。"""
        if value <= 0:
            raise ValueError("高さは正の値でなければなりません")
        self._height = value
    
    @property
    def area(self) -> float:
        """長方形の面積を計算します。"""
        return self.width * self.height
```

## 例外処理

### 例外の使用

- 例外は例外的な状況にのみ使用し、通常のフロー制御には使用しません。
- 具体的な例外クラスをキャッチします。`except Exception:` や `except:` を避けます。
- `try/except` ブロックの範囲をできるだけ小さくします。

```python
# 良い例：具体的な例外を処理
try:
    user = User.objects.get(id=user_id)
except User.DoesNotExist:
    logger.warning(f"User with ID {user_id} not found")
    raise Http404("User not found")

# 避けるべき例：一般的すぎる例外処理
try:
    # 多くの異なる例外が発生する可能性のある長いコードブロック
    result = complex_operation(data)
except Exception as e:  # すべての例外をキャッチするのは避ける
    logger.error(f"処理に失敗しました: {e}")
```

### 例外の再発生

- 例外をキャッチして処理した後に再発生させる必要がある場合は、元の例外情報を保持します。
- コンテキスト情報を追加するために例外をラップすることを検討します。

```python
def process_file(filename: str) -> dict:
    """ファイルを処理します。"""
    try:
        with open(filename, 'r') as file:
            return json.load(file)
    except FileNotFoundError:
        logger.error(f"ファイルが見つかりません: {filename}")
        raise  # 元の例外を再発生
    except json.JSONDecodeError as e:
        logger.error(f"JSONの解析エラー: {filename}")
        raise ValueError(f"無効なJSON形式のファイル: {filename}") from e  # 新しい例外を元の例外とともに発生
```

### カスタム例外

- アプリケーション固有の例外にはカスタム例外クラスを定義します。
- わかりやすい名前を付け、通常は「Error」で終わります。
- 関連する例外クラスで階層構造を作成します。

```python
# 基本例外クラス
class AppError(Exception):
    """アプリケーション固有の基本例外。"""
    pass

# 派生例外クラス
class ConfigurationError(AppError):
    """設定エラーを表す例外。"""
    pass

class APIConnectionError(AppError):
    """API接続エラーを表す例外。"""
    def __init__(self, api_name: str, original_error=None):
        self.api_name = api_name
        self.original_error = original_error
        message = f"{api_name} APIに接続できませんでした"
        if original_error:
            message += f": {original_error}"
        super().__init__(message)
```

## テスト

### テスト構造

- テストファイルには `test_` プレフィックスを付けます。
- テストクラスには `Test` プレフィックスを付けます。
- テスト関数/メソッドには `test_` プレフィックスを付けます。

```python
# test_user.py
import unittest
from app.models import User

class TestUser(unittest.TestCase):
    """Userクラスのテスト。"""
    
    def setUp(self):
        """各テストの前に実行される。"""
        self.user = User("test@example.com", "password123")
    
    def test_email_validation(self):
        """メールアドレスのバリデーションをテストします。"""
        with self.assertRaises(ValueError):
            User("invalid-email", "password123")
    
    def test_password_hashing(self):
        """パスワードがハッシュ化されることをテストします。"""
        self.assertNotEqual(self.user.password, "password123")
        self.assertTrue(self.user.verify_password("password123"))
```

### アサーション

- 適切なアサーションメソッドを使用します。
- テストの失敗時に意味のあるメッセージを提供します。

```python
# unittest を使用したテスト
def test_calculate_average(self):
    """calculate_average関数をテストします。"""
    # 基本的なケース
    self.assertEqual(calculate_average([1, 2, 3]), 2.0)
    
    # エッジケース
    self.assertEqual(calculate_average([0]), 0.0)
    
    # 空のリストは例外をスローする
    with self.assertRaises(ValueError):
        calculate_average([])

# pytest を使用したテスト
def test_calculate_discount():
    """calculate_discount関数をテストします。"""
    # 通常のケース
    assert calculate_discount(100, 20) == 80
    
    # 100%割引
    assert calculate_discount(100, 100) == 0
    
    # 無効な割引率
    with pytest.raises(ValueError):
        calculate_discount(100, 110)
```

### モック

- 外部依存をモックするには `unittest.mock` または `pytest-mock` を使用します。
- モックはテスト対象の実際の動作に焦点を当てるために使用します。

```python
from unittest.mock import patch, MagicMock

def test_send_email():
    """send_email関数をテストします。"""
    with patch('app.utils.smtp_lib.send') as mock_send:
        result = send_email("user@example.com", "Hello")
        
        # 関数が呼び出されたことを確認
        mock_send.assert_called_once()
        
        # 正しい引数で呼び出されたことを確認
        mock_send.assert_called_with(
            to="user@example.com",
            subject="Hello",
            body=mock_send.call_args[1]['body']  # 動的に生成される部分はチェックしない
        )
        
        # 戻り値を確認
        assert result is True
```

## 型ヒント

### 基本的な型ヒント

- 関数の引数と戻り値には型ヒントを使用します。
- 複雑な型には `typing` モジュールを使用します。
- 変数の型ヒントは必要な場合にのみ使用します。

```python
from typing import List, Dict, Optional, Union, Tuple, Any

def process_users(
    users: List[Dict[str, Any]],
    fields: Optional[List[str]] = None
) -> Dict[str, List[Any]]:
    """ユーザーデータを処理します。

    Args:
        users: ユーザー辞書のリスト
        fields: 処理するフィールドのリスト、Noneの場合はすべてのフィールド

    Returns:
        フィールド名をキーとし、すべてのユーザーの値をリストで持つ辞書
    """
    result: Dict[str, List[Any]] = {}
    fields = fields or list(users[0].keys()) if users else []
    
    for field in fields:
        result[field] = [user.get(field) for user in users]
    
    return result
```

### ジェネリクス

- コレクションには具体的な型パラメータを指定します。
- 複雑な型定義には型エイリアスを使用します。

```python
from typing import TypeVar, List, Dict, Callable, TypedDict

T = TypeVar('T')  # ジェネリック型変数

# 型エイリアス
UserId = int
UserData = Dict[str, Union[str, int, bool]]

# TypedDict の使用
class UserDict(TypedDict):
    id: int
    name: str
    email: str
    is_active: bool
    meta: Dict[str, Any]

def filter_list(items: List[T], predicate: Callable[[T], bool]) -> List[T]:
    """条件に一致する項目だけをリストから取得します。"""
    return [item for item in items if predicate(item)]

def get_active_users(users: List[UserDict]) -> List[UserDict]:
    """アクティブなユーザーのみを返します。"""
    return filter_list(users, lambda user: user['is_active'])
```

