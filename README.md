01.
New request command  :
php artisan make:request RegistrationFormRequest

code : 

*** RegistrationFormRequest.php

<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class RegistrationRequest extends FormRequest
{
    public function rules()
    {
        return [
            'name' => ['required', 'string', 'min:2'],
            'email' => ['required', 'email'],
            'password' => ['required', 'string', 'min:8'],
        ];
    }
}


*** RegistationController.php
<?php

namespace App\Http\Controllers;

use App\Http\Requests\RegistrationRequest;

class RegistrationController extends Controller
{
    public function register(RegistrationRequest $request)
    {
    }
}

*** web.php
<?php

use App\Http\Controllers\RegistrationController;
use Illuminate\Support\Facades\Route;

Route::post('/register', 'RegistrationController@register')->name('register');






02:Create a route /home that redirects to /dashboard using a 302 redirect.
Code :

***web.php
<?php
use Illuminate\Http\RedirectResponse;
use Illuminate\Support\Facades\Route;

Route::get('/home', function () {
    return new RedirectResponse('/dashboard', 302);
});



03:Create a global middleware that logs the request method and URL for every incoming request. Log the information to the Laravel log file.

command : php artisan make:middleware LogRequestMiddleware

***LogRequestMiddleware.php

public function handle($request, Closure $next)
{
    $method = $request->getMethod();
    $url = $request->fullUrl();

    // Log the request method and URL
    \Illuminate\Support\Facades\Log::info("Request Method: $method, URL: $url");

    return $next($request);
}
 
***app/Http/Kernel.php
protected $middleware = [
    // Other middleware...
    \App\Http\Middleware\LogRequestMiddleware::class,
];



04:Create a route group for authenticated users only. This group should include routes for /profile and /settings. Apply a middleware called AuthMiddleware to the route group to ensure only authenticated users can access these routes.

command : php artisan make:middleware AuthMiddleware


***AuthMiddleware.php
code :
public function handle($request, Closure $next)
{
    if (!auth()->check()) {
    
        return redirect()->route('login');
       
    }

    return $next($request);
}

***app/Http/Kernel.php

code:
protected $routeMiddleware = [
    // Other middleware...
    'auth' => \App\Http\Middleware\AuthMiddleware::class,
];


***route/web.php
code:

Route::middleware(['auth'])->group(function () {
    Route::get('/profile', function () {
        // Logic for the profile route
    })->name('profile');

    Route::get('/settings', function () {
        // Logic for the settings route
    })->name('settings');
});


05:Create a controller called ProductController that handles CRUD operations for a resource called Product. Implement the following methods:

command : php artisan make:controller ProductController --resource


***ProductController.php
code:
<?php

namespace App\Http\Controllers;

use App\Models\Product;
use Illuminate\Http\Request;

class ProductController extends Controller
{
    /**
     * Display a listing of the resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function index()
    {
        $products = Product::all();

        return view('products.index', compact('products'));
    }

    /**
     * Show the form for creating a new resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function create()
    {
        return view('products.create');
    }

    /**
     * Store a newly created resource in storage.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function store(Request $request)
    {
        $validatedData = $request->validate([
            'name' => 'required',
            'price' => 'required|numeric',
            // Add validation rules for other fields
        ]);

        Product::create($validatedData);

        return redirect()->route('products.index')
            ->with('success', 'Product created successfully.');
    }

    /**
     * Display the specified resource.
     *
     * @param  \App\Models\Product  $product
     * @return \Illuminate\Http\Response
     */
    public function show(Product $product)
    {
        return view('products.show', compact('product'));
    }

    /**
     * Show the form for editing the specified resource.
     *
     * @param  \App\Models\Product  $product
     * @return \Illuminate\Http\Response
     */
    public function edit(Product $product)
    {
        return view('products.edit', compact('product'));
    }

    /**
     * Update the specified resource in storage.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \App\Models\Product  $product
     * @return \Illuminate\Http\Response
     */
    public function update(Request $request, Product $product)
    {
        $validatedData = $request->validate([
            'name' => 'required',
            'price' => 'required|numeric',
            // Add validation rules for other fields
        ]);

        $product->update($validatedData);

        return redirect()->route('products.index')
            ->with('success', 'Product updated successfully.');
    }

    /**
     * Remove the specified resource from storage.
     *
     * @param  \App\Models\Product  $product
     * @return \Illuminate\Http\Response
     */
    public function destroy(Product $product)
    {
        $product->delete();

        return redirect()->route('products.index')
            ->with('success', 'Product deleted successfully.');
    }
}



06 : Create a single action controller called ContactController that handles a contact form submission. Implement the __invoke() method to process the form submission and send an email to a predefined address with the submitted data.


command:php artisan make:controller ContactController --invokable


code :

file : ContactController.php


<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Mail;

class ContactController extends Controller
{
    /**
     * Handle the contact form submission.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function __invoke(Request $request)
    {
        // Validate the form data
        $validatedData = $request->validate([
            'name' => 'required',
            'email' => 'required|email',
            'message' => 'required',
        ]);

        // Send an email to the predefined address
        Mail::raw($validatedData['message'], function ($message) use ($validatedData) {
            $message->to('your-email@example.com');
            $message->subject('New Contact Form Submission');
            $message->from($validatedData['email'], $validatedData['name']);
        });

        // Redirect or return a response indicating success
        return redirect()->back()->with('success', 'Your message has been sent successfully!');
    }
}


file : web.php
code :
<?php

use Illuminate\Support\Facades\Route;
Route::post('/contact', 'ContactController')->name('contact');


07: Create a resource controller called PostController that handles CRUD operations for a resource called Post. Ensure that the controller provides the necessary methods for the resourceful routing conventions in Laravel.
command : php artisan make:controller PostController --resource

file:PostController.php
code :

<?php

namespace App\Http\Controllers;

use App\Models\Post;
use Illuminate\Http\Request;

class PostController extends Controller
{
    /**
     * Display a listing of the resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function index()
    {
        $posts = Post::all();

        return view('posts.index', compact('posts'));
    }

    /**
     * Show the form for creating a new resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function create()
    {
        return view('posts.create');
    }

    /**
     * Store a newly created resource in storage.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function store(Request $request)
    {
        $validatedData = $request->validate([
            'title' => 'required',
            'content' => 'required',
        ]);

        $post = Post::create($validatedData);

        return redirect()->route('posts.show', $post->id)
            ->with('success', 'Post created successfully.');
    }

    /**
     * Display the specified resource.
     *
     * @param  \App\Models\Post  $post
     * @return \Illuminate\Http\Response
     */
    public function show(Post $post)
    {
        return view('posts.show', compact('post'));
    }

    /**
     * Show the form for editing the specified resource.
     *
     * @param  \App\Models\Post  $post
     * @return \Illuminate\Http\Response
     */
    public function edit(Post $post)
    {
        return view('posts.edit', compact('post'));
    }

    /**
     * Update the specified resource in storage.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \App\Models\Post  $post
     * @return \Illuminate\Http\Response
     */
    public function update(Request $request, Post $post)
    {
        $validatedData = $request->validate([
            'title' => 'required',
            'content' => 'required',
        ]);

        $post->update($validatedData);

        return redirect()->route('posts.show', $post->id)
            ->with('success', 'Post updated successfully.');
    }

    /**
     * Remove the specified resource from storage.
     *
     * @param  \App\Models\Post  $post
     * @return \Illuminate\Http\Response
     */
    public function destroy(Post $post)
    {
        $post->delete();

        return redirect()->route('posts.index')
            ->with('success', 'Post deleted successfully.');
    }
}


file : web.php
code :
<?php

use Illuminate\Support\Facades\Route;
Route::resource('posts', 'PostController');



08: Create a Blade view called welcome.blade.php that includes a navigation bar and a section displaying the text "Welcome to Laravel!".
command : touch resources/views/welcome.blade.php

file : welcome.blade.php
code :
<!DOCTYPE html>
<html>
<head>
    <title>Welcome to Laravel</title>
    <!-- Add your CSS and other head elements here -->
</head>
<body>
    <nav>
        <!-- Add your navigation bar HTML code here -->
    </nav>

    <section>
        <h1>Welcome to Laravel!</h1>
        <!-- Add other content for the section as needed -->
    </section>

    <!-- Add your JavaScript and other scripts here -->
</body>
</html>


file:web.php


use Illuminate\Support\Facades\Route;
Route::get('/', function () {
    return view('welcome');
});



