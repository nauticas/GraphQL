# GraphQL

## PENGERTIAN

GraphQL adalah sebuah konsep baru dalam membangun sebuah API dimana kita di beri keleluasaan untuk mendapatkan data yang kita butuhkan. Dalam membangun sebuah API tentu kita ingin memberikan jembatan data untuk aplikasi lain terhadap system yang kita punya, akan tetapi ketika kita pengguna API terkadang mengalami sebuah masalah dimana terkadang kita menerima data yang itu tidak dibutuhkan. Begitu  juga kita akan mendapati masalah dimana biasanya ketika sebuah system yang di tambahkan fitur baru kita perlu menaikan versi API yang kita gunakan, nah dengan menggunakan GraphQL kita tidak lagi perlu mengubah versi atau menaikan versi dari api yang kita buat tiggal saja kita melakukan query terhadap API yang telah kita sediakan.

<img src="https://github.com/nauticas/GraphQL/blob/master/img/graphql.png" width="70%">

## PERBEDAAN

perbedaan dengan REST API adalah 

## Install GraphQL Lumen
Tambahkan depedency berikut ini 
```
{
	"require": {
		"folklore/graphql": "~1.0.0"
	}
}
```
Selanjutnya silahkan jalankan pertinah berikut ini
```
composer update
```
Edit file boostraps/app.php pada baris berikut :
```
$app->instance('path.storage', app()->basePath() . DIRECTORY_SEPARATOR . 'storage');
$app->configure('graphql');
$app->withFacades();
$app->withEloquent();

$app->register(App\Providers\AppServiceProvider::class);
$app->register(App\Providers\AuthServiceProvider::class);
$app->register(Folklore\GraphQL\LumenServiceProvider::class);
```
Sekarang publish config GraphQL dengan menjalankan perintah berikut ini
```
php artisan graphql:publish
```
Selanjutnya tambahkan code berikut ini pada file AppServiceProvider.php
```
<?php
namespace App\Providers;
use Illuminate\Support\ServiceProvider;
class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        $this->app->singleton('Illuminate\Contracts\Routing\ResponseFactory', function ($app) {
            return new \Illuminate\Routing\ResponseFactory(
                $app['Illuminate\Contracts\View\Factory'],
                $app['Illuminate\Routing\Redirector']
            );
        });
    }
}
```
Jika sudah sekarang kita buat file user Query dan user Type pada folder GraphQL
### GraphQL Query
buat sebuah interface Query terhadap data yang kita akan keluarkan
```
<?php 
namespace App\GraphQL\Query;
use GraphQL;
use GraphQL\Type\Definition\Type;
use Folklore\GraphQL\Support\Query;
use App\User;
/**
 * User Query
 */
class UsersQuery extends Query
{
	
	protected $attributes = [
		'name' => 'users'
	];
	public function type()
	{
		return Type::listOf(GraphQL::type('User'));
	}
	public function args()
	{
		return [
			'id' => ['name' => 'id', 'type' => Type::string()],
			'email' => ['name' => 'email', 'type' => Type::string()]
		];
	}
	public function resolve($root, $args)
	{
		if (isset($args['id'])) {
			return User::where('id', $args['id'])->get();
		}else if (isset($args['email'])) {
			return User::where('email', $args['email'])->get();
		}else{
			return User::all();
		}
	}
}
```
### GraphQL Type
Type disini berfungsi untuk mendefinisikan type data dan deskripsi dari model data yang di query kan.
```
<?php 
namespace App\GraphQL\Type;
use GraphQL\Type\Definition\Type;
use Folklore\GraphQL\Support\Type as GraphQLType;
/**
 * User Type
 */
class UserType extends GraphQLType
{
	
	protected $attributes = [
		'name' => 'User',
		'description' => 'A User'
	];
	public function fields()
	{
		return [
			'id' => [
				'type' => Type::nonNull(Type::string()),
				'description' => 'The id of user'
			],
			'email' => [
				'type' => Type::string(),
				'description' => 'The email of user'
			]
		];
	}
	protected function resolveEmailField($root, $args)
	{
		return strtolower($root->email);
	}
}
```
Selanjutnya silahkan pada file config.php seperti berikut ini
```
<?php
return [
    'prefix' => 'graphql',
    'routes' => '{graphql_schema?}',
    'controllers' => \Folklore\GraphQL\GraphQLController::class.'@query',
    'variables_input_name' => 'variables',
    'middleware' => [],
    'headers' => [],
    'json_encoding_options' => 0,
    'graphiql' => [
        'routes' => '/graphiql/{graphql_schema?}',
        'controller' => \Folklore\GraphQL\GraphQLController::class.'@graphiql',
        'middleware' => [],
        'view' => 'graphql::graphiql'
    ],
    'schema' => 'default',
    'schemas' => [
        'default' => [
            'query' => [
                 'users' => 'App\GraphQL\Query\UsersQuery'
            ],
            'mutation' => [
            ]
        ]
    ],
    'types' => [
        'User' => 'App\GraphQL\Type\UserType'
    ],
    'error_formatter' => [\Folklore\GraphQL\GraphQL::class, 'formatError'],
    'security' => [
        'query_max_complexity' => null,
        'query_max_depth' => null,
        'disable_introspection' => false
    ]
];
```

### Testing
<img src="https://github.com/nauticas/GraphQL/blob/master/img/testing.png" width="70%">