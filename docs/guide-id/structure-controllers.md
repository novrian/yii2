_Controller_
============

_Controller_ adalah salah satu bagian dari arsitektur [MVC](http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller).
_Controller_ adalah objek-objek dari _class_ yang di-_extend_ dari _class_ [[yii\base\Controller]] dan bertanggungjawab untuk memproses _request_ dan
membuat _response_. Secara khusus, setelah diambil alih dari [aplikasi](structure-applications.md),
_controller_ akan menganalisa data _request_ yang masuk, mengirimkannya kepada [model](structure-models.md), memasukkan hasil model
ke dalam [views](structure-views.md), dan terakhir mengirimkan _response_ yang dihasilkan tersebut.


## _Action_ <span id="actions"></span>

_Controller_ terdiri dari beberapa *action* yang merupakan bagian utama yang diketahui dan diminta oleh pengguna untuk
di eksekusi. Sebuah _controller_ dapat memilik satu _action_ atau lebih.

Contoh di bawah ini menampilkan _controller_ `post` dengan dua _action_: `view` dan `create`:

```php
namespace app\controllers;

use Yii;
use app\models\Post;
use yii\web\Controller;
use yii\web\NotFoundHttpException;

class PostController extends Controller
{
    public function actionView($id)
    {
        $model = Post::findOne($id);
        if ($model === null) {
            throw new NotFoundHttpException;
        }

        return $this->render('view', [
            'model' => $model,
        ]);
    }

    public function actionCreate()
    {
        $model = new Post;

        if ($model->load(Yii::$app->request->post()) && $model->save()) {
            return $this->redirect(['view', 'id' => $model->id]);
        } else {
            return $this->render('create', [
                'model' => $model,
            ]);
        }
    }
}
```

Di dalam _action_ `view` (yang dijabarkan oleh _method_ `actionView()`), pertama-tama kode tersebut mengambil [model](structure-models.md)
sesuai dengan ID model yang diminta; Jika model berhasil diambil, _controller_ akan menampilkannya menggunakan
[view](structure-views.md) yang diberi nama `view`. Jika tidak berhasil, _controller_ akan melakukan _throw_ sebuah _exception_.

Di dalam _action_ `create` (yang ditentukan oleh _method_ `actionCreate()`), mengandung kode yang mirip. Pertama-tama _controller_ akan mencoba membuat
sebuah objek [model](structure-models.md) menggunakan data dari _request_ dan menyimpan model tersebut. Jika kedua hal tersebut berhasil, _controller_
akan mengirimkan perintah _redirect_ ke _browser_ untuk menuju ke _action_ `view` dengan ID model yang baru saja dibuat. Jika tidak, _controller_
akan menampilkan _view_ dimana pengguna bisa memasukkan input melalui form.


## Route <span id="routes"></span>

_Action_ yang dituju oleh pengguna disebut dengan *route*. Sebuah _route_ adalah _string_ yang terdiri dari beberapa bagian sebagai berikut:

* ID Module: Ini hanya akan muncul ketika sebuah _controller_ berasal dari [module](structure-modules.md) yang bukan merupakan objek aplikasi;
* [ID Controller](#controller-ids): Sebuah _string_ yang secara unik mengidentifikasi sebuah _controller_ dengan _controller_ lainnya, dalam sebuah aplikasi
  (atau di _module_ yang sama jika _controller_ tersebut berasal dari sebuah _module_);
* [ID Action](#action-ids): sebuah _string_ yang secara unik mengidentifikasi sebuah _action_ dengan _action_ lainnya di dalam _controller_ yang sama.

Berikut ini adalah format dari _Route_:

```
IDController/IDAction
```

jika _controller_ berasail dari sebuah _module_, maka formatnya sebagai berikut:

```php
IDModule/IDController/IDAction
```

Jadi, jika pengguna melakukan _request_ dengan URL `http://hostname/index.php?r=site/index`, berarti _action_ `index` di dalam _controller_ `site`
yang akan dieksekusi. Untuk informasi lebih lanjut mengenai bagaimana _route_ diproses menjadi _action_, silahkan melihat bagian
[Route dan Pembuatan URL](runtime-routing.md).

## Membuat _Controller_ <span id="creating-controllers"></span>

Pada [[yii\web\Application|aplikasi web]], _class controller_ harus di-_extend_ dari _class_ [[yii\web\Controller]] atau
_class_ turunannya. Begitu juga pada [[yii\console\Application|aplikasi konsol]], _class controller_ harus di-_extend_ dari
_class_ [[yii\console\Controller]] atau _class_ turunannya. Kode dibawah ini adalah contoh dari _controller_ `site`:

```php
namespace app\controllers;

use yii\web\Controller;

class SiteController extends Controller
{
}
```


### ID Controller  <span id="controller-ids"></span>

Pada umumnya, sebuah _controller_ diciptakan untuk menangani sebuah _request_ yang berhubungan dengan sumber daya.
Oleh karena itu, ID _controller_ adalah sebuah kata benda yang merepresentasikan sumber daya yang ditangani.
Sebagai contoh, anda boleh menggunakan `article` sebagai ID dari _controller_ yang menangani data artikel.

Secara umum, ID _controller_ harusnya hanya mengandung karakter ini: Huruf latin kecil, angka,
garis bawah _(underscore)_, simbol tanda kurang, dan garis miring _(forward slash)_. Contohnya, `article` dan `post-comment` kedua-duanya adalah ID _controller_ yang valid,
sedangkan `article?`, `PostComment`, `admin\post` adalah contoh yang tidak valid.

Sebuah ID _controller_ juga dapat mengandung prefix dari subdirektori. Sebagai contoh, `admin/article` merupakan ID dari _controller_ `article`
yang berada di dalam subdirektori `admin` yang disimpan di dalam direktori yang ditentukan oleh _property_ [[yii\base\Application::controllerNamespace|_namespace controller_]].
Karakter yang valid untuk prefix subdirektori meliputi: Huruf latin kecil dan kapital, angka, garis bawah _(underscore)_, dan
garis miring _(forward slash)_, dimana garis miring digunakan sebagai separator untuk subdirektori yang bersifat multilevel (contoh: `panels/admin`).


### Penamaan _Class Controller_ <span id="controller-class-naming"></span>

Nama _class_ dari _controller_ dapat diambil dari ID _controller_ dengan mengikut prosedur berikut:

1. Ganti huruf pertama menjadi huruf kapital untuk setiap kata yang dipisahkan oleh tanda kurang. Sebagai catatan, jika ID _controller_
   mengandung garis miring, aturan ini hanya diterapkan untuk kata terakhir setelah garis miring terakhir pada ID _controller_.
2. Menghapus tanda kurang, dan mengganti semua garis miring dengan _backward slash_.
3. Menambahkan suffiks `Controller`.
4. Meletakkan [[yii\base\Application::controllerNamespace|_namespace controller_]] sebagai awalan dari ID.

Berikut adalah beberapa contoh ID _controller_, dengan asumsi bahwa [[yii\base\Application::controllerNamespace|_namespace controller_]]
mengambil nilai default yaitu `app\controllers`:

* `article` menjadi `app\controllers\ArticleController`;
* `post-comment` menjadi `app\controllers\PostCommentController`;
* `admin/post-comment` menjadi `app\controllers\admin\PostCommentController`;
* `adminPanels/post-comment` menjadi `app\controllers\adminPanels\PostCommentController`.

_Class-class Controller_ harus dapat di [autoload](concept-autoloading.md). Karena itulah, di contoh di atas,
_class controller_ `article` harus di simpan pada file dengan [alias](concept-aliases.md)
`@app/controllers/ArticleController.php`; sedangkan _controller_ `admin/post-comment` harus di simpan pada file
`@app/controllers/admin/PostCommentController.php`.

> Info: Contoh terakhir yaitu `admin/post-comment` menunjukkan bahwa anda dapat menyimpan _controller_ ke dalam subdirektori
  dari [[yii\base\Application::controllerNamespace|_namespace controller_]]. Hal ini sangat bermanfaat ketika anda
  ingin memisahkan _controller_ anda ke beberapa kategori namun anda tidak ingin untuk menggunakan [module](structure-modules.md).


### Controller Map <span id="controller-map"></span>

Anda dapat mengatur [[yii\base\Application::controllerMap|_controller map_]] untuk tidak mengikuti batasan-batasan
ID _controller_ dan nama _class_ yang dijelaskan sebelumnya. Hal ini akan sangat berguna ketika anda menggunakan
_controller_ dari pihak ketiga dan anda tidak memiliki wewenang mengganti nama _class_ dari pihak ketiga tersebut.

Anda dapat menentukan [[yii\base\Application::controllerMap|_controller map_]] pada
[konfigurasi aplikasi](structure-applications.md#application-configurations). Sebagai contoh:

```php
[
    'controllerMap' => [
        // menentukan controller "account" menggunakan nama class
        'account' => 'app\controllers\UserController',

        // menentukan controller "article" menggunakan array konfigurasi
        'article' => [
            'class' => 'app\controllers\PostController',
            'enableCsrfValidation' => false,
        ],
    ],
]
```


### Controller Default <span id="default-controller"></span>

Setiap aplikasi memiliki _controller_ default yang ditentukan melalui _property_ [[yii\base\Application::defaultRoute]].
Ketika sebuah _request_ tidak menentukan sebuah [_route_](#routes), maka _route_ yang ditentukan oleh _property_ inilah yang akan digunakan.
Untuk [[yii\web\Application|aplikasi web]], nilainya adalah `'site'`, sedangkan untuk [[yii\console\Application|aplikasi konsol]],
nilainya adalah `help`. Maka, jika URL-nya adalah `http://hostname/index.php`, maka _controller_ `site` yang akan menangani _request_ tersebut.

Anda boleh mengganti _controller_ default menggunakan [konfigurasi aplikasi](structure-applications.md#application-configurations) berikut ini:

```php
[
    'defaultRoute' => 'main',
]
```


## Membuat Action <span id="creating-actions"></span>

Cara membuat _action_ cukup sederhana, sesederhana membuat *method action* di dalam _class controller_. Sebuah _method action_ adalah
sebuah _method public_ yang namanya diawali dengan kata `action`. Nilai yang di-_return_ oleh _method action_ merepresentasikan
data _response_ yang akan dikirim ke pengguna. Contoh kode dibawah ini menjabarkan dua _action_, yaitu `index` dan `hello-world`:

```php
namespace app\controllers;

use yii\web\Controller;

class SiteController extends Controller
{
    public function actionIndex()
    {
        return $this->render('index');
    }

    public function actionHelloWorld()
    {
        return 'Hello World';
    }
}
```


### ID Action <span id="action-ids"></span>

Pada umumnya, sebuah _action_ dirancang untuk melaksanakan tugas manipulasi khusus terhadap sebuah sumber daya. Oleh karena itu,
ID _action_ pada umumnya adalah kata kerja, seperti `view`, `update`, dll.

Secara default, ID _action_ seharusnya hanya berisi karakter berikut ini: Huruf latin kecil, angka,
garis bawah _(underscore)_, dan tanda kurang (anda dapat menggunakan simbol ini untuk memisahkan kata). Sebagai contoh,
`view`, `update2`, dan `comment-post`, semuanya adalah ID _action_ yang valid, sedangkan `view?` dan `Update` merupakan ID yang tidak valid.

Anda dapat membuat _action_ menggunakan dua cara: _action_ sejajar dan _action_ mandiri. Sebuah _action_ sejajar adalah
sebuah _method_ yang ditulis di dalam _class controller_, sedangkan _action_ mandiri adalah sebuah _action_ yang berbentuk _class_ yang di-_extend_
dari _class_ [[yii\base\Action]] atau _class_ turunannya. _Action_ sejajar cukup mudah untuk dibuat dan secara umum direkomendasikan
ketika anda tidak ingin menggunakan _action_ ini di tempat lain. Sedangkan, _Action_ mandiri pada umumnya dibuat
agar bisa digunakan pada _controller_ yang berbeda atau akan didistribusikan sebagai [extension](structure-extensions.md).


### _Action_ Sejajar <span id="inline-actions"></span>

_Action_ sejajar adalah _action_ yang ditulis sebagai _method_ seperti yang barusan dijelaskan.

Nama dari _method_ ini diambil dari ID _action_ yang ditentukan melalui prosedur berikut ini:

1. Ganti huruf pertama menjadi huruf kapital untuk setiap kata dari ID _action_.
2. Hapus tanda kurang.
3. Menambahkan prefix `action`.

Sebagai contoh, _method_ dari _action_ `index` dinamakan dengan `actionIndex`, dan `hello-world` dinamakan dengan `actionHelloWorld`.

> Note: Nama _method_ dari _action_ bersifat *case-sensitive*. Jika anda memiliki sebuah _method_ dengan nama `ActionIndex`,
  maka _method_ tersebut tidak dianggap sebagai _method action_, akibatnya, _request_ untuk _action_ `index`
  akan menghasilkan sebuah _exception_. Harap diingat bahwa _method action_ harus bersifat _public_. _Method action_ yang bersifat _private_ atau _protected_
  tidak dianggap sebagai _action_ sejajar.


_Action_ sejajar adalah jenis _action_ yang umumnya digunakan karena cukup mudah untuk dibuat. Jika
anda bertujuan untuk menggunakan _action_ yang sama di _controller_ yang berbeda, ataukah anda ingin mendistribusikan sebuah _action_,
anda harus mempertimbangkan untuk menggunakan *action mandiri*.


### _Action_ Mandiri <span id="standalone-actions"></span>

_Action_ mandiri adalah _action_ yang berbentuk _class_, dimana _class_ tersebut di-_extend_ dari [[yii\base\Action]] atau _class_ turunannya.
Sebagai contoh, pada rilisan Yii, terdapat _class_ [[yii\web\ViewAction]] dan [[yii\web\ErrorAction]], kedua-duanya merupakan
_action_ mandiri.

Untuk menggunakan _action_ mandiri, anda harus mendeklarasikannya di dalam *action map* dengan menimpa _method_
[[yii\base\Controller::actions()]] di _class controller_ anda seperti yang dicontohkan berikut ini:

```php
public function actions()
{
    return [
        // deklarasi action "error" menggunakan nama class
        'error' => 'yii\web\ErrorAction',

        // deklarasi action "view" menggunakan konfigurasi array
        'view' => [
            'class' => 'yii\web\ViewAction',
            'viewPrefix' => '',
        ],
    ];
}
```

Seperti yang anda lihat, _method_ `actions()` harus mengembalikan nilai dalam bentuk sebuah _array_ dimana _key_-nya adalah ID _Action_ dan _value_-nya adalah
nama _class action_ yang dimaksud atau [konfigurasi array](concept-configurations.md). Berbeda dengan _action_ sejajar, ID _Action_ untuk _action_ mandiri
bisa terdiri dari karakter apa saja, sepanjang ID tersebut di deklarasikan di dalam _method_ `actions()`.

Untuk membuat _class action_ mandiri, anda harus meng-_extend_ [[yii\base\Action]] atau _class_ turunannya, dan mengimplementasikan
sebuah _method public_ dengan nama `run()`. Tujuan dari _method_ `run` disini mirip dengan _method action_ yang dijelaskan pada _action_ sejajar. Sebagai contoh,

```php
<?php
namespace app\components;

use yii\base\Action;

class HelloWorldAction extends Action
{
    public function run()
    {
        return "Hello World";
    }
}
```


### Hasil dari _Action_ <span id="action-results"></span>

Nilai yang di-_return_ oleh _method action_ atau _method_ `run()` dari _action_ mandiri adalah hal yang sangat penting. Karena hal tersebut
merupakan hasil output dari _action_ yang dimaksud.

Nilai yang di-_return_ dapat berupa objek [response](runtime-responses.md) yang akan dikirimkan ke pengguna sebagai _response_.

* Untuk [[yii\web\Application|aplikasi web]], nilai yang di-_return_ bisa berupa data apa saja yang kemudian akan
  ditugaskan ke [[yii\web\Response::data]] dan selanjutnya akan dikonversi menjadi _string_ yang menjadi representasi dari _response body_.
* Untuk [[yii\console\Application|aplikasi konsol]], nilai yang di-_return_ bisa berupa _integer_ yang merepresentasikan
  [[yii\console\Response::exitStatus|status exit]] dari hasil eksekusi perintah.

Pada contoh di atas, hasil dari _action_ merupakan _string_ yang akan diperlakukan sebagai _response body_
dan akan dikirim ke pengguna. Contoh dibawah ini menunjukan bahwa sebuah _action_ dapat mengirimkan perintah _redirect_ ke _browser_ pengguna menuju ke URL baru
dengan me-_return_ objek _response_ (karena _method_ [[yii\web\Controller::redirect()|redirect()]] me-_return_
objek _response_).

```php
public function actionForward()
{
    // redirect browser pengguna menuju http://example.com
    return $this->redirect('http://example.com');
}
```


### Parameter _Action_ <span id="action-parameters"></span>

_Method action_ pada _action_ sejajar dan _method_ `run()` pada _action_ mandiri dapat diberi parameter,
yang disebut dengan *parameter action*. Nilainya didapatkan dari _request_. Pada [[yii\web\Application|aplikasi web]],
nilai dari setiap parameter _action_ didapatkan dari `$_GET` menggunakan nama parameter sebagai _key_;
Pada [[yii\console\Application|aplikasi konsol]], nilainya diambil dari argumen yang dikirim oleh baris perintah.

Pada contoh dibawah ini, _action_ `view` (sebuah _action_ sejajar) mendeklarasikan dua parameter: `$id` dan `$version`.

```php
namespace app\controllers;

use yii\web\Controller;

class PostController extends Controller
{
    public function actionView($id, $version = null)
    {
        // ...
    }
}
```

Parameter _action_ disetiap _request_ akan didapatkan dengan cara seperti ini:

* `http://hostname/index.php?r=post/view&id=123`: parameter `$id` akan diisi dengan nilai
  `'123'`,  sedangkan `$version` masih tetap `null` karena tidak ada _query string_ parameter dengan nama `version`.
* `http://hostname/index.php?r=post/view&id=123&version=2`: parameter `$id` dan `$version` masing-masing akan
  diisi dengan nilai `'123'` dan `'2'.
* `http://hostname/index.php?r=post/view`: _exception_ [[yii\web\BadRequestHttpException]] akan di _throw_
  karena parameter `$id` yang diwajibkan, tidak terdapat pada _request_.
* `http://hostname/index.php?r=post/view&id[]=123`: _exception_ [[yii\web\BadRequestHttpException]] akan di _throw_
  karena parameter `$id` mendapatkan nilai _array_ `['123']` yang tidak diperbolehkan.

Jika anda menginginkan paramter _action_ menerima nilai dalam bentuk _array_, anda harus melakukan _type-hint_ menggunakan `array`, seperti contoh berikut ini:

```php
public function actionView(array $id, $version = null)
{
    // ...
}
```

Sekarang ini, jika _request_-nya adalah `http://hostname/index.php?r=post/view&id[]=123`, parameter `$id` akan mengambil nilai
`['123']`. Jika _request_-nya adalah `http://hostname/index.php?r=post/view&id=123`, parameter `$id` akan tetap
menerima nilai dalam bentuk _array_ yang sama, karena nilai scalar `'123'` akan dirubah menjadi _array_ secara otomatis.

Contoh diatas secara khusus menunjukkan bagaimana parameter _action_ bekerja pada aplikasi web. Untuk aplikasi konsol,
silahkan melihat [Perintah Konsol](tutorial-console.md) untuk informasi lebih lanjut.


### _Action_ Default <span id="default-action"></span>

Setiap _controller_ memiliki sebuah _action_ default yang ditentukan melalui _property_ [[yii\base\Controller::defaultAction]].
Ketika sebuah [route](#routes) hanya terdiri dari ID _controller_, maka _request_ yang dimaksud adalah _action_ default
dari _controller_ yang ditentukan.

Secara default, nilai _action_ default adalah `index`. Jika anda ingin mengganti nilai default, cukup menimpa
_property_ ini pada _class controller_, seperti contoh berikut ini:

```php
namespace app\controllers;

use yii\web\Controller;

class SiteController extends Controller
{
    public $defaultAction = 'home';

    public function actionHome()
    {
        return $this->render('home');
    }
}
```


## Siklus _Controller_ <span id="controller-lifecycle"></span>

Pada saat memproses sebuah _request_, objek [aplikasi](structure-applications.md) akan membuat sebuah _controller_
berdasarkan [route](#routes) yang di-_request_. Kemudian _controller_ akan melalui siklus-siklus dibawah ini
untuk menyelesaikan _request:

1. _Method_ [[yii\base\Controller::init()]] akan dipanggil setelah objek _controller_ dibuat dan dikonfigurasi.
2. _Controller_ membuat objek _action_ berdasarkan ID _action_ yang diminta:
   * Jika ID _action_ tidak ditemukan, maka [[yii\base\Controller::defaultAction|ID action default]] yang akan digunakan.
   * Jika ID _action_ ditemukan di dalam [[yii\base\Controller::actions()|action map]], maka sebuah _action_ mandiri
     akan dibuat;
   * Jika ID _action_ ditemukan dan cocok dengan _method_ yang ada, maka sebuah _action_ sejajar akan dibuat;
   * Jika tidak memenuhi tiga kondisi diatas, maka sebuah [[yii\base\InvalidRouteException]] akan di _throw_.
3. _Controller_ memanggil secara sekuensial _method_ `beforeAction()` dari objek aplikasi, objek modul (jika _controller_ tersebut
   berasal dari sebuah modul), dan _controller_ itu sendiri.
   * Jika salah satu pemanggilan tersebut mengembalikan nilai `false`, maka _event_ `beforeAction`  berikutnya tidak akan dieksekusi dan
     proses _action_ akan dihentikan.
   * Secara default, setiap pemanggilan _method_ `beforeAction` akan menjalankan _event_ `beforeAction` yang anda bisa gunakan untuk menempelkan kode penanganan _event_.
4. _Controller_ menjalankan _action_.
   * Parameter _action_ akan dianalisa dan dibentuk dari data _request_.
5. _Controller_ akan memanggil secara sekuensial _method_ `afterAction()` dari _controller_, _module_ (jika _controller_ tersebut
   berasal dari sebuah _module_), dan aplikasi.
   * Secara default, setiap pemanggilan _method_ `afterAction()` akan menjalankan _event_ `afterAction` yang anda bisa gunakan untuk menempelkan kode penanganan _event_.
6. Aplikasi akan mengambil hasil dari _action_ dan mengirimkannya ke objek [response](runtime-responses.md).


## Kaidah Terbaik <span id="best-practices"></span>

Dalam sebuah aplikasi yang dirancang dengan baik, pada umumnya _controller_ berukuran kecil, dengan setiap _action_ yang terdiri dari beberapa baris kode saja.
Jika _controller_ anda cukup rumit, itu merupakan indikasi bahwa anda seharusnya merancang ulang _controller_ tersebut dan memindahkan beberapa baris kode
ke _class_ yang lain.

Berikut ini adalah beberapa kaidah-kaidah terbaik untuk bekerja dengan _controller_. _Controller_

* boleh mengakses data [request](runtime-requests.md);
* boleh memanggil _method_ dari [models](structure-models.md) dan layanan _component_ lainnya, menggunakan data _request_;
* boleh menggunakan [view](structure-views.md) untuk membuat _response_;
* sebaiknya TIDAK memproses data _request_ - hal ini harusnya dilakukan pada [bagian model](structure-models.md);
* sebaiknya tidak menampilkan HTML atau kode lain yang berhubungan dengan output - hal ini sebaiknya dilakukan pada [views](structure-views.md).
