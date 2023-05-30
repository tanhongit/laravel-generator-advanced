# Paginating User Manual

## 1. Paginating Query Builder Results
To paginate the results returned in the query builder, you use the paginate method with the following syntax:

    paginate($perPage, $columns, $pageName, $page);
*In which :*

- $perPage- is the number of items that will be retrieved and displayed on each page. The default will be 20 items per page.
- $columns- are the columns to be retrieved in the database. The default will take all ( SELECT *)
- $pageName- is the name of the query string that will contain the page number parameter. Default $pageName = 'page'.
- $page- is the item you want to retrieve is the page number, if the page is null, Laravel will process the data of the page query string. Default $page = null.

*For example display 20 items per page:*

    <?php
    namespace App\Http\Controllers;
    use App\Http\Controllers\Controller;
    use Illuminate\Support\Facades\DB;

    class UserController extends Controller
    {
        /**
        * Show all of the users for the application.
        *
        * @return \Illuminate\Http\Response
        */
        public function index()
        {
            return view('user.index', [
                'users' => DB::table('users')->paginate(20) 
            ]);
        }
    }

## 2. Customizing Pagination URLs:
The paginator's withPath method allows you to customize the URLs used by the paginator when generating links.

*Example:*

    use App\Models\User;
    
    Route::get('/users', function () {
        $users = User::paginate(20);
    
        $users->withPath('/admin/users');
    
        //
    });

## 3. Appending Query String Values: 
You may use the “withQueryString” method if you would like to append all of the current request's query string values to the pagination links:

*Example:*

    $users = User::paginate(20)->withQueryString();

## 4. Displaying Pagination Results

*Example:*

    {{$users->links()}}

If you don't want to use Laravel's default pagination view, you can specify another view or write your own. To change the view you just need to pass the view name into the links.

*Example:*

    {{ $paginator->links('view.name') }}

You can export Laravel's default views to a directory resources/views/vendor for easy reference and editing. To export the view you can use the command:

    php artisan vendor:publish --tag=laravel-pagination