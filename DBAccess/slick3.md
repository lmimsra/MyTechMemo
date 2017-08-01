# play2.5.Xとslick3で大変だったところのメモ
play2.5のDBアクセスの方法としてslickというものがあるということで、つまったところをコードと一緒にメモしておく  
> 今回はBookという１つのクラスでしか実装してません  
> 試しに実装してみるというのが主旨ですので!

- slickについての資料  
http://slick.lightbend.com/doc/3.1.1/gettingstarted.html
- ProjectRoot/conf/application.conf  
アプリケーションの設定ファイル
```
slick.dbs.default {
  driver="slick.driver.MySQLDriver$"
  db {
    driver=com.mysql.jdbc.Driver
    ↓？の後でエンコードを揃えるので文字化けがしにくくなる
    url="jdbc:mysql://localhost:3306/DB名?useUnicode=true&characterEncoding=utf8"
    user="root"
    password="pass"
    numThreads = 10
    connectionTimeout = 5000
    validationTimeout = 5000
  }
}
//データベースマイグレーションの設定
//本来起動ごとに聞かれるところを全てOKでスキップして実行する
play.evolutions {
  db.default.enabled = true
  autoApply=true
  db.default.autoApply=true
}
```

- ProjectRoot/build.sbt  
ビルド設定ファイル
```  
name := """scala-app"""
version := "1.0-SNAPSHOT"
lazy val root = (project in file(".")).enablePlugins(PlayScala)
scalaVersion := "2.11.7"
libraryDependencies ++= Seq(  
  cache,
  ws,
  "org.scalatestplus.play" %% "scalatestplus-play" % "1.5.1" % Test,
  "mysql" % "mysql-connector-java" % "5.1.34",
  "com.typesafe.play" %% "play-slick" % "2.0.2",
  "com.typesafe.play" %% "play-slick-evolutions" % "2.0.0",
  "com.typesafe.slick" %% "slick" % "3.1.1"
)
fork in run := false
```

- ProjectRoot/app/models/Model.scala  
データベースと紐付けるクラスを作っておく
```
case class Book(id:Long,title:String,author_id:Long)
```

- ProjectRoot/app/dao/BookDAO.scala  
DAOクラス
```
package models
case class Book(id:Long,title:String,author_id:Long)mmr0929:app mmr$ cat dao/BookDAO.scala
package dao
import javax.inject.Inject
import models.Book
import play.api.db.slick.{DatabaseConfigProvider, HasDatabaseConfigProvider}
import slick.driver.JdbcProfile
import scala.concurrent.ExecutionContext.Implicits.global <-これが重要
import scala.concurrent.Future
//クラス名の横の文はDBの定義に必要なのでおまじない
class BookDAO @Inject()(protected val dbConfigProvider:DatabaseConfigProvider)
                              extends HasDatabaseConfigProvider[JdbcProfile]{

  import driver.api._
  private val Books = TableQuery[BooksTable]
  private class BooksTable(tag: Tag) extends Table[Book](tag,"BOOK"){
    def id = column[Long]("id",O.PrimaryKey,O.AutoInc)
    def title = column[String]("title")
    def author_id = column[Long]("author_id")
    def * = (id,title,author_id)<>(Book.tupled,Book.unapply _)
  }
  def all():Future[Seq[Book]] = db.run(Books.result)
  def insert(book: Book):Future[Unit] = db.run(Books += book).map{ _ => () }
}
```

- ProjectRoot/app/controllers/DataController.scala  
コントローラークラス
```
package controllers
import javax.inject._
import dao.BookDAO
import models.Book
import play.api.mvc._
import play.api.data._
import play.api.data.Forms._
import play.api.data.Form
import play.api.Play._
import scala.concurrent.ExecutionContext.Implicits.global <-これ忘れないように
@Singleton
class DataController @Inject()(bookDao:BookDAO) extends Controller {
  //入出力のためのフォームをマッピング
  val bookForm = Form(mapping(
    "id"->longNumber(),
    "title"->text(),
    "author_id"->longNumber()
  )(Book.apply)(Book.unapply)
  )
  //bookのインスタンスを渡して表示
  def indexing = Action.async {
    bookDao.all().map {
      books=>Ok(views.html.database(books))
    }
  }
  //新しくデータを入れる
  def insertData = Action.async { implicit request =>
    val book:Book = bookForm.bindFromRequest.get
    bookDao.insert(book).map(_ => Redirect(routes.DataController.indexing))
  }
}
```

- ProjectRoot/app/views/database.scala.html  
viewファイル  
```
@(books: Seq[Book])
@main("DBAccess") {
	<div>
		<h1>DB接続ページ</h1>
		<h3>Insert a book hare</h3>
		<form action="/insert/book" method="post" name="dbform" onsubmit="inputCheck1()">
			<input name="id" type="number" placeholder="input id" required/>
			<input name="title" type="text" placeholder="input title" required/>
			<input name="author_id" type="number" placeholder="input author id" required/>
			<input type="submit"/><br/>
			<div id="error_ms" style="color: red"></div>
		</form>
		<h3>all tables</h3>
		<table>
			<tr><th>id</th><th>title</th><th>author_id</th></tr>
			@for(b <- books){
				<tr><td>@b.id</td><td>@b.title</td><td>@b.author_id</td></tr>
			}
		</table>
		<button onclick="gotoTop()">トップページに戻る</button>
	</div>
}
```

a
