# Spring initializr より作成

## 使用した dependencies

#### Spring Web

#### Spring Data JPA

#### PostgreSQL Server

# プロジェクト内のクラスについて

## Student.java

DB のレコードと紐づく @Entity クラスである。

@Table により DB のテーブルと紐づいている。(明示的に指定しない場合はクラス名を大文字にしたもの"STUDENT"テーブルに紐づく。)

### id 採番時の注意

id の自動採番を行う場合は@GeneratedValue/@SequenceGenerator の設定を行う。

この際@SequenceGenerator(allocationSize = 1)を指定しないと、処理が複数起動した際に正しく採番されない問題が起こる。

### @Transient

@Transient
エンティティクラス内での要素としては存在するが、DB のレコードと直接紐づかないもの。

Student クラスでは現在時刻とレコード内の誕生日(dob)の差を計算して求められる。

## StudentConfig.java

このプロジェクトでは起動時に上記 Entity クラスに紐づくテーブルの DDL 処理が行われる。(resources/application.properties > spring.jpa.hibernate.ddl-auto=create-drop)

その際にこのクラスを@Configuration + @Bean を用いてコンポーネントスキャンの対象にすると、Spring の起動時に commandLineRunner が実行され、テーブル内に初期レコードが登録される。

(ApplicationRunner を使用する方法もある。)

## StudentController.java

@RestController

クライアントとの値の受け渡しを行うためのクラス。

実際のデータ加工処理については@Service を付与したビジネスロジック層へ委譲する。

クラス全体に共通の path を指定できる他、クラス内メソッドにも path を追記できる。

### メソッドの変数に付与されているアノテーションについて

- @pathVariable は api が呼び出された時に URL に使用されている文字列を受け取る。
- @RequestBody はメッセージボディに含まれる POST パラメータを受け取る。
- @RequestParam は URL に含まれるクエリパラメータ、メッセージボディに含まれる POST パラメータを受け取る。

## StudentRepository.java

JpaRepository は@Entity クラスで定義されたテーブル内のレコードの検索について、該当クエリを格納することが出来るクラスである。

ちなみに、findById()などの一部基本クエリは記載しなくても使用可能。

StudentService.java
@Controller から受け渡された値を元に、データ処理層との値の受け渡しを行うためのクラス。
データの更新処理について不正な入力が発見された場合の判定も@Service クラスに実装する。
(例 : email が既に登録されている場合、エラーを返し処理を終了する。)
エラーハンドリングについては本来@ExceptionHandler で行うが、このプロジェクトには実装していない。
@Transactional を付与したメソッドについては、トランザクション制御が行われる。
(例 : レコードの更新処理が行われた場合に、その更新を確定させる。)

# Appendix

## Component と Bean の違い

どちらも Spring の DI コンテナに管理させるが、
@Component は自分で作成しているクラスに適用させる。

- @Controller(API レイヤー)
- @Service(ビジネスロジックレイヤー)
- @Repository(データレイヤー)

@Bean は外部ライブラリに依存しているクラスに適用させる。

(また、@Configuration をつけているクラス内で@Bean を定義しないと登録処理が呼び出されない。)

これは DI コンテナに登録する対象を見つける時に@Configuration をつけたクラスを探しにいくためである。

ただし、@Component に関しては@ComponentScan 処理を用いて別に登録されるため@Configuration 内で定義しなくて良い。

## Hibernate

Hibernate は Java⇔ リレーショナルデータベース間で「O/R マッピング」(Java コードと DB 内レコードを対応付ける処理の事)を行うフレームワークである。

## エラーログについて

resource/application.properties に下記設定を追記すると、エラー発生時に詳しい内容を記載してくれる。

server.error.include-message=always

logging.level.org.springframework.web=DEBUG
