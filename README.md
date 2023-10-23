# Laraval

Laraval looks extremely complicated, but based on the fact I could pick it up in about an hour, it shouldn't be that much harder for you. Don't feel to intimidated, even though this single `README.md` file is about 1000 words long. This will be much easier to read than the documentation, however, if you are stuck, you should absolutely check out the [documentation online](https://laravel.com/docs/10.x).

## Setup

1. CD into project directory. For me, I'll use the command: `cd ~/Projects`
2. Run the composer setup command. Be sure to change YOUR-APPLICATION-NAME to... your application name of course: `curl -s https://laravel.build/YOUR-APPLICATION-NAME?with=mysql,redis | bash`. This is a script that will automatically create a Laraval project along side docker setup for you. This also creates a MySQL database along side it because we addded `with=mysql,redis` as a URL parameter.
3. Start the project using the following commands:
```
cd YOUR-APPLICATION-NAME
./vendor/bin/sail up
```
4. Application should be at http://localhost

## Directory Structure

### `/app`

This is where your `Models` and `Controllers` will reside in the contect of MVC. 

Models contain data that may be stored in a database, or viewed to the user inisde of your `Views`.

Controllers (inside `/app/Http/Controllers`) is the place where  you call to your database using a Model. An example could be a 'create  account' page. Firstly, a GET method from inside the controller will be called without a model, and load the `View` with the PHP code in it. Then, the user clicks the create account button and it sends a `POST` request with a `User` model which the controller uses to `INSERT` something into the database.

```php
// Example Controller. Routing is done in /routes, so scroll down to see how this actually works!

public function ProductsController()
{
	public function getProducts() 
	{
		$p = array();
		$p[] = 'Test Product 1';
		$p[] = 'Test Product 2';
		$p[] = 'Test Product 3';

		// return view() => This function returns the view
		// 'products'    => Find the `products.blade.php` 
		// $p            => data to insert into the view
   	return view('products', $p);
   }
}
```

### `/bootstrap/app.php`

- Don't worry about `./cache`. 
- `app.php` is the starting point in your application. 
- a 'singleton' is a global reference to an object. For example, you may want to have a SINGLE database connection object which performs your database calls. A singleton makes this possible without having to pass said object into every constructor in the application.

### `/config`

This directory contains a lot. You may want to skim through `app.php`, `auth.php`, and `database.php`. They are short files only containing well-documented variables which you can use to configure how your website runs. The default settings should suffice though. 

### `/database`

This is where the database magic happens. What Laraval does is it takes your PHP class/object, and converts it all into `SQL` queries which it can run on your database for you. For example, you create a `Product.php` class which has a `$name` and `$price` field. The `migrations` folder will then create a new migration and put in a new migration containing a sql query such as `CREATE TABLE PRODUCT(name varchar(100), description text);`

### `/public`

I would not recomend touching anything in this directory as it contains start-up stuff.

### `/resources`

This is where your JavaScript, CSS, and your Views will reside. By default, each view will be of `.php.blade` format

### `/routes`

This contains all of your routing stuff

**`web.php`** contains your `GET`, `POST`, etc... routes. The code bellow will define a route at `/` (in the browser it would be `localhost/ or localhost`. When the user navigates to said route, it will run the function. 
	
```php
// quick way to load view
Route::get('/', function() {
	return view ('welcome');
});

// to load view from Controller (for MVC)
// Route::controller()                     => loading the controller
// ProductsController::class               => The name of the class (ProductsController as string instead of object)
// ->group(function() {})                  => Groups a list of routes
// Route::get('/products', 'getProducts'); => When user performs GET request on /products, call the `getProducts()` function inside the `ProductsController`
Route::controller(ProductsController::class)->group(function() {
	Route::get('/products', 'getProducts');
	Route::post('/products/create', 'postProducts');
});
```

Now that you know this, you should go back and re-read the `app`

## Example Web Page

This example, we will create a page to display a list of products

### Creating Controller

Go into `/app/Http/Controllers`, create `ProductsController` class, make it extend `Controller.php` (php file inside same `Controllers` directory), and create endpoints:

```php
<?php
namespace App\Http\Controllers; // This PHP file is inside this namespace. Everytime you want to use this controller, you must use this namespace

use App\Http\Controllers\Controller; 

class ProductsController extends Controller
{
    // We can turn this function into a proper endpoint in the following instructions. I like to use the naming convention 'ServerMethod EndpointName'. This means method will handle a GET method for the /products endpoint
    public function getProducts()
    {
        $p = array();
        $p[] = 'Test Product 1';
        $p[] = 'Test Product 2';
        $p[] = 'Test Product 3';

        // return view()     => This function returns the view
        // 'products'        => Find the `products.blade.php`
        // ->with('p' => $p) => to insert into the view
        // 'p'                => variable name inside view
            // $p                 => data to populate 'p' variable
        return view('products')
            ->with('p', $p);
    }
}

```

### Creating route for /products endpoint

Create the route (same as example above)

```php
Route::controller(ProductsController::class)->group(function() {
	Route::get('/products', 'getProducts');
	Route::post('/products/create', 'postProducts');
});
```

### Create HTML as View

Create a file called `products.blade.php` inside `/resources/views`. 

```php
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <title>Products</title>
    </head>
    <body>
    {{-- Header --}}
    <div>
        <h1>Products</h1>
    </div>
    <div>
        <table>
            <thead>
            <tr>
                <th>Product Name</th>
            </tr>
            </thead>
            <tbody>
			{{-- You'll notice that $p also exists inside this view. That is because we passed the array in the router! --}}
            @foreach($p as $product)
                <tr>
                    <td>{{ $product }}</td>
                </tr>
            @endforeach
            </tbody>
        </table>
    </div>
    </body>
</html>


```

You may notice that this is just raw HTML code, with extra bullshit. For example, `{{ str_replace('_', '-', app()->getLocale()) }}` just finds the user's language on their operating system / browser. Comments are also done like this: `{{-- Header --}}`.

### Creating Products object

Create the file `/app/Models/Product.php`

### Modifying Products model to fit our needs

Go into `/app/Models/Products.php` which was auto generated previously

Add the following code inside the `Products` class

```php

<?php

	namespace App\Models;

	use Illuminate\Database\Eloquent\Model;

    class Product extends Model
    {
        /**
         * Table name inside the database
         * @var string
         */
        protected $table = 'product';

        /**
         * The column for the primary key
         * @var string
         */
        protected $primaryKey = 'id';

        /**
         * Decides if the $primaryKey is auto-incrementing. We'll set this to true since the database will store IDs as
         * integers for simplicity
         * @var bool
         */
        public $incrementing = true;

        /**
         * Data type to store primary key as
         * @var string
         */
        protected $keyType = 'int';

        /**
         * What database connection to use. Just keep it is as MySQl unless you really want to use multiple different
         * databases.
         * @var string
         */
        protected $connection = 'mysql';
	}


```

### Database
#### Seeding The Database

We want to populate the database by default. We can do this by modifying `DatabaseSeeder.php`

Go into the `public function run() : void` or create it if it does not exist. Add the following lines underneath the comments:

```php
DB::table('products')->insert([
    'name' => 'Some Product', // name column = 'Some Product`
    'description' => Str::random(100),
    'price' => 20
]);
```

#### Creating Database Migration

We now want to create the table. We can do this automatically using database migrations... is what I wolud say if Laraval was actually good. As before, go into the terminal interface in your docker container and run `php artisan make:migration create_product_table`. If you are curious as to what that did, you can go into `database/migrations/{most recent migration}`, and have a look! If something looks not right, roll back the migration using `php artisan migrate:rollback`, and find out what happened.

I personally don't understand how migrations work. Aparently they are supposed to automatically load the columns as you need them, but that clearly is not the case as it has not with `DatabaseSeeder.php`. In other words; **Fuck Laraval Database Migrations**. There is probably a way to do it somewhere, but I'm not sure at the moment

To get around this issue, go into the most recent migration and modify the `up()` function. You want to replace the current `product` table creation because you'll notice how NONE of the columns we want are inside of it. 

Add them as such:

```php
Schema::create('product', function (Blueprint $table) {
    $table->id();                  // Id Column
    $table->string('name');        // Name Column
    $table->string('description'); // Description Column
    $table->float('price');        // Price Column
});
```


#### Updating The Database

Once you have created your Models and Migrations, we now need to set them in stone. 

Go into the docker container once again and run `php artisan migrate`.

I hate Laraval Databases if you can't tell already

### Getting products from the database

Now that we finally have a database with some stuff in it, we can populate our little web page

Go back into `ProductsController` and lets start creating a query using Query Bulider. You may want to view the documentation for this as it contains more information about queries you can create [here](https://laravel.com/docs/10.x/queries)

```php
// select * from product
// DB::table('product') => Load the product table
// ->get()              => Get everything from the product table
$p = DB::table('product')->get();
```

`DB::table` is a static method, hence you won't need an object reference. 	DON'T TELL CHELTON.

That new array we have is very special. Each element inside the array is a record in the database. We can access the column values by accessing it's property. By that, I mean you'd do `$p->{ColumnName}`. We could do `$p->name`, `$p->description`, and `$p->price`.

With that information, you should change your `@foreach` loop inside the view to reflect these new changes

```php
@foreach($p as $product)
    <tr>
        <td>{{ $product->name }}</td>
        <td>{{ $product->description }}</td>
        <td>${{ $product->price }}</td>
    </tr>
@endforeach
```

(The description will be very large because we set it equal to random text in the `ProductFactory`. You can obviously change it to something more sensible, but I'm lazy).
