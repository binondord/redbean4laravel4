# redbean4laravel4
### A Laravel 4 package for RedBeanPHP ORM 4.1

This is a Laravel 4 package to allow the use of [Redbean PHP ORM](http://redbeanphp.com), version 4.1.

### How to install

Add 
	
	"mamift/redbean4-laravel4":"dev-master" 
	
to your composer.json file. Then add this line:

	'Mamift\Redbean4Laravel4\Redbean4Laravel4ServiceProvider'

to your Laravel service provider's array in app.php inside the config/ folder, so RedBeanPHP is setup using Laravel's database settings (inside database.php).

Because RedBeanPHP also includes it's own facade class ("R"), there is no need to add anything into the alias array.

### Usage

Read [RedBeanPHP's documentation](http://redbeanphp.com/crud) for a complete overview of what you can do with RedBean. Because this package includes the full rb.php file unmodified, every programmable interface listed on RedBean's API documentation pages should be usable.

An example:

	$user = R::dispense('user');
	$user['description'] = "Lorem ipsum dolor sit amet, consectetur" + adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.";
	$user->username = "mamift";
	$user->gender = R::enum('gender:male');
	R::store($user);

### Why use RedBean with Laravel?

RedBean is a very flexible way to flesh-out a database without having to worry about foreign-key relations or how your table is structured; it is 'schema-less' in a sense, in that it will build the appropriate database and table structures without you having to worry about the details. For this reason, it is a neat way to rapidly prototype the backend for a Laravel app. You could use RedBean as a lazy substitute for Laravel's Schema Builder and forego seeding in a separate step for example, as you can define the schema and seed the table with values in the one migration step.

In the above example where the following line is:

	$user->gender = R::enum('gender:male');

RedBean will create a separate table, 'gender', and include an appropriate primary key (an **AUTO_INCREMENTING 'ID'** column when you're using MySQL). Whenever you use **R::enum()** again, (like **R::enum('gender:female')** for instance), then RedBean will add another 'female' record inside the 'gender' table. 

Note how it doesn't use the built-in [ENUM] data type as a column type; this enables you to define another bean (table) which can use the same set of values.

RedBean will also determine the appropriate data type depending on the values of the properties of your beans. In the above example, **$user['description']** is stored as TEXT and **$user->username** is stored as VARCHAR(255).

When reverting a Laravel migration, you may need to also use Schema builder methods such as **Schema::drop('user')** or **Schema::drop('gender')**, as RedBean does not provide a way to delete the table schema. Instead it allows you to erase all instances of a bean (equivalent to deleting all rows inside a table) using **R::wipe('user')**. 

However, RedBean does allow you to destroy all tables altogether in a single step using **R::nuke()**, but this will destroy everything inside the database, including the *migrations* table.

### A note on how this package exposes RedBean in Laravel

Due to the way the author of RedBean uses PHP namespaces (it doesn't appear to be PSR-4 compliant), he does not provide his own composer.json and as such, the rb.php file (the file that RedBeanPHP is commonly distributed in) does not appear to be autoloadable by Laravel.

What this package does is load rb.php for each request. Under the "autoload" JSON object inside composer.json, rb.php is specified as part of the "files" array:

	{
	    "autoload": {
    	    "files": [
    	        "src/Mamift/Redbean4Laravel4/rb.php"
    	    ]
    	}
	}
	
A quote from the [Composer documentation](https://getcomposer.org/doc/04-schema.md#files) says:
>If you want to require certain files explicitly on every request then you can use the 'files' autoloading mechanism. This is useful if your package includes PHP functions that cannot be autoloaded by PHP.

Due to rb.php being loaded on each request, there may be a slight performance penalty incurred.