"# QuizAppClient" 
## 概要  
クイズアプリ  
- クイズをDBに格納し、http通信でサーバから問題を取得して出題する。  
- 回答は選択形式とする。
- 1ゲームごとに5問の問題を出題する。
- 1ゲームの制限時間は5分とする。
- クイズはランダムに出題する。1ゲーム内での重複は不可。
- 問題はテキストと画像を表示させる。
- 1問につき配点は100点とする。正解の場合、100点から要した時間に応じて点数を減じる。  
- ゲームスコアは、（正解数×１００点）×（１/全体の時間）
- ユーザのDBを保持して成績を振り返ることができる。
- ユーザ全体の成績をランキングで表示する。
- 途中終了した場合は成績に含まない。

## 基本設計  

### 画面一覧  
#### ログイン  
ユーザー名とパスワードを入力させて認証を行う。  
![image](https://user-images.githubusercontent.com/92729088/159005805-5b825d7b-cd26-4aa1-9eec-5730df2a5572.png)
#### アカウント作成  
ユーザー名とパスワードを登録する画面。  
![image](https://user-images.githubusercontent.com/92729088/159006264-c9f3803b-b5d6-4a82-90e5-5a145be6d678.png)
#### ホーム  
ログイン後最初に表示する画面。  
![image](https://user-images.githubusercontent.com/92729088/159006207-d62890e9-04dd-44b5-acea-869ac88918c8.png)
#### ゲーム画面  
問題文を表示して回答を4択選択させる。残り時間を表示する。  
![image](https://user-images.githubusercontent.com/92729088/159007439-79817b4c-b5c4-4233-99e9-d88dd56536a8.png)
#### ゲーム結果  
問題ごとのかかった時間、スコアを表示する。  
![image](https://user-images.githubusercontent.com/92729088/159006293-b6158fc5-e93d-43c7-8b19-7f5970760d22.png)
#### ランキング   
全ユーザーのスコアの良い順番のリストを表示する。  
![image](https://user-images.githubusercontent.com/92729088/159006299-15232c28-b453-4486-b22e-16ae1e91da43.png)
#### マイスコア   
自身のスコアの良い順番のリストを表示する。  
![image](https://user-images.githubusercontent.com/92729088/159006303-327333e4-ea03-4515-9d39-cd7b773437b2.png)

### 画面遷移  
本アプリの画面遷移図を以下に示す。  
![image](https://user-images.githubusercontent.com/92729088/159012712-fae4a50e-bf72-4e57-82ed-6f5cd067c678.png)


### DB設計  
#### ユーザDB  
|  項目名  |  説明  |  備考  |
| :---: | :---: | :---: |
|  ユーザ名  |  ユーザ名  |  重複不可  |
|  パスワード  |  パスワード  |  英数混合、4文字以上8文字以内  |

#### スコアDB  
|  項目名  |  説明  |  備考  |
| :---: | :---: | :---: |
|  ユーザ名  |  ユーザ名  |  重複不可  |  
|  スコア  |  スコア  |  以下のTBLを配列で保持  |
|  日付  |  日付  ||
|  スコア  |  スコア  ||

#### 問題DB  
|  項目名  |  説明  |  備考  |
| :---: | :---: | :---: |
|  問題  |  問題  ||
|  画像有無  |  画像を表示する場合はtrue <br>  画像がない場合はfalse||
|  画像ID  |  画像DBのID  ||
|  回答  |  回答  ||
|  選択肢  |  選択肢  |  4個の選択肢を配列で保持する  |

#### 画像DB  
[gridfs](https://docs.mongodb.com/manual/core/gridfs/#:~:text=GridFS%20is%20a%20specification%20for,chunk%20as%20a%20separate%20document)で画像ファイルを保持する。

## 処理詳細  

### ログイン  
#### ロードイベント  
1. イニファイルを読み込み、サーバのURLを取得する。  
・上記処理の処理シーケンスを以下に示す。  
![image](https://user-images.githubusercontent.com/92729088/159711937-c99ad464-5189-4cd2-8336-72e0baa0e891.png)  
・上記処理のフローチャートを以下に示す。  
![image](https://user-images.githubusercontent.com/92729088/159712362-36dcc2c3-e733-452c-b6a9-dcfd681fd562.png)

#### ログインボタンクリック時  
1. ユーザ名がDBにあるかサーバに問い合わせる。ある場合はユーザ情報を取得する。ない場合はnull。
2. nullの場合、ポップアップでエラーメッセージを表示する。
3. ユーザ情報があった場合は、パスワードをユーザ情報と一致するか確認する。
4. 一致したら、ホーム画面に遷移する。不一致の場合、ポップアップでエラーメッセージを表示する。
#### アカウント作成ボタンクリック時  
1. アカウント作成画面へ遷移する。  

### アカウント作成画面  
#### アカウント作成ボタンクリック時  
1. ユーザ名がDBにあるかサーバに問い合わせる。存在すればエラーメッセージを表示する。  
2. パスワードが条件を満たしているか確認する。満たしていない場合はエラーメッセージを表示する。  
3. 確認用のパスワードが同一か確認する。満たしていない場合はエラーメッセージを表示する。  
4. アカウントを作成するためにユーザ名とパスワードをDBに追加する。

### ホーム画面  
#### ロードイベント  
1. 現在ログイン中のユーザ名を画面に表示する。
#### ゲーム開始ボタンクリック時  
1. ゲーム画面へ遷移する。
#### ランキングボタンクリック時 
1. ランキング画面へ遷移する。
#### マイスコアボタンクリック時  
1. マイスコア画面へ遷移する。  
#### ゲーム終了ボタンクリック時  
1. アプリを終了する。

### ゲーム画面  
#### ロードイベント  
1. ユーザ名を画面に表示する。
2. 問題DBから問題情報を5つ取得する。
3. 問題情報の画像DBが真の場合は画像DBに問い合わせ、画像を取得する。
4. 問題情報から問題と解答を画面に表示する。
5. 何問目かを表示する。
6. 制限時間を表示する。
#### 回答ボタンクリック時  
1. 選択された回答が正解かどうかを保存する。  
2. 経過した時間を保存する。
3. 次の問題情報をもとにゲーム画面を再描画する。この時制限時間は継続してカウントする。
4. 5問目の回答ボタンがクリックされたら、ゲーム結果画面へ遷移する。
#### ゲーム終了ボタンクリック時  
1. ホーム画面へ遷移する。

### ゲーム結果  
#### ロードイベント  
1. ユーザ名を画面に表示する。  
2. 正解不正解のデータとかかった時間をもとにスコアを算出する。
3. 画面にスコアと1問ごとの結果を表示する。
#### ホーム画面に戻るボタンクリック時  
1. スコアDBにユーザ名をキーにして日付とスコアを格納する。  
2. DB追加完了後、ホーム画面に戻る。

### 全体のランキング  
#### ロードイベント  
1. ユーザ名を画面に表示する。 
2. スコアDBから、全ユーザのスコアの良い順に5件取得する。  
3. 画面にスコアとユーザ名をランキングで表示する。
#### ホーム画面に戻るボタンクリック時  
1. ホーム画面に戻る。

### 自分のランキング  
#### ロードイベント  
1. ユーザ名を画面に表示する。 
2. スコアDBから、ログインユーザのスコアの良い順に5件取得する。  
3. 画面にスコアと日付をランキングで表示する。
#### ホーム画面に戻るボタンクリック時  
1. ホーム画面に戻る。
