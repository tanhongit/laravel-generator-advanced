BaseRepository have some functions to use in Repository of each Model.

These functions are created to use for different Models, help to write code more concise and understandable.

You can think of it as a top level repository, which is used to store the common functions of the repository. Instead of each repository of each model having to write the same function, same code, and same logic, you need write it in BaseRepository and use it in each repository.

To apply BaseRepository on the Repository of each Model, you need to extends it.

Example:

```php

namespace App\Repositories;

use App\Repositories\BaseRepository;
use App\Models\User;

class UserRepository extends BaseRepository
{
    public function __construct(User $user)
    {
        $this->model = $user;
    }
}
```

In this example, **UserRepository** extends **BaseRepository** and UserRepository have a property $model is an **instance of** User.

## Functions

### getByAttributes($attributes)

Get data on the table with attributes.

Parameters:

-   $attributes: array of attributes to get data. Example: ['name' => 'John', 'age' => 20]

Example:

```php
# on UserRepository.php

$user = $this->getByAttributes(['name' => 'John Doe']);

# this will result is an instance of User with name is 'John Doe'
```

In this example, **$user** is an instance of User with name is 'John Doe'.

And if there is no data with attributes, it will return null.

> Because this function are using **whereRaw()** to get data from sql string. But we need to use **where()** (or some other functions by Laravel) to get data from array of attributes.

> So some functions are created to help us to use where() to get data with attributes.

### getConditions(object $query, array $conditions)

Get query with condition of columns.

This function is used to get query with condition of columns from parameters. And then, base on the condition by the value of $conditions[1], we can use where(), whereIn(), whereBetween(), whereNotIn(), whereNotBetween(),... of Laravel to get data.

Parameters:

-   $query: query to get data
-   $conditions: array of conditions to get data. Example: ['name', '=', 'John Doe']

Example:

```php
# on UserRepository.php

$user = $this->getConditions($this->model, ['name', '=', 'John Doe'])->all();

# this will result is an instance of User with name is 'John Doe'
```

In this example, **$user** is an instance of User with name is 'John Doe'.

**Note:**

-   $conditions[0] is the column name
-   $conditions[1] is the condition of column
-   $conditions[2] is the value of column
-   $conditions[3] is the operator of condition. Default is 'and'

_This function have special case when **$conditions[3]** is exists and the value of this is **"or"**. It will use orWhere() to get data._

> But we should not use this function on the Repository of each Model. Because this function is used to get data from parameters. And we should use it in BaseRepository.

=> We will call this function on another function in BaseRepository.

### getWhereConditions($conditions)

Get data on the table with query of where conditions.

Parameters:

-   $conditions: array of conditions to get data. Example: ['name', '=', 'John Doe']

Example:

```php
# on UserRepository.php

    /**
     * Get all users
     * @return User[]|\Illuminate\Database\Eloquent\Collection
     */
    public function all($conditions) {
        return $this->model->getWhereConditions($conditions);
    }
```

This function have called **getConditions()** to get data with conditions. Then return all data.

And on the controller, we can call this function to get data.

Example:

```php
# on UserController.php

    /**
     * Get all users
     * @return \Illuminate\Http\JsonResponse
     */
    public function index() {
        $users = $this->userRepository->all(['name', '=', 'John Doe'], ['name', '=', 'John Doe 2', 'or'], ['age', '>=', 20]);
        return response()->json($users);
    }
```

In this example, **$users** is an instance of User with name is 'John Doe' or 'John Doe 2' and age is greater than or equal 20.

### getRelationshipWhereConditions($conditions, $arrayModel)

Get data with relationship another model with query of where conditions.

Parameters:

-   $conditions: array of conditions to get data. Example: ['name', '=', 'John Doe']
-   $arrayModel: array of relationship model. Example: ['orders', 'shops']

Example:

```php
# on UserRepository.php

    /**
     * Get all users
     * @return User[]|\Illuminate\Database\Eloquent\Collection
     */
    public function getData($conditions) {
        $arrayModel = ['orders', 'shops'];
        return $this->model->getRelationshipWhereConditions($conditions, $arrayModel);
    }
```

This function have called **getRelationshipWhereConditions()** to get data with conditions have relationship with another model. Then return all data.

And on the controller, we can call this function to get data.

Example:

```php
# on UserController.php

    /**
     * Get all users
     * @return \Illuminate\Http\JsonResponse
     */
    public function index() {
        $users = $this->userRepository->getData(['age', '>=', 20]);
        return response()->json($users);
    }
```

In this example, **$users** is an instance of User with age is greater than or equal 20 and have relationship with orders and shops.

From this, you can get data on orders and shops of each user.

### getRelationshipWhereConditionsWithPagination($conditions, $arrayModel, $perPage = 15)

Get data with relationship another model with query of where conditions and will be paginated.

Parameters:

-   $conditions: array of conditions to get data. Example: ['name', '=', 'John Doe']
-   $arrayModel: array of relationship model. Example: ['orders', 'shops']
-   $perPage: number of data per page. **Default is 15**

Example:

```php
# on UserRepository.php

    /**
     * Get all users
     * @return User[]|\Illuminate\Database\Eloquent\Collection
     */
    public function getDataPaginate($conditions, $perPage) {
        $arrayModel = ['orders', 'shops'];
        return $this->model->getRelationshipWhereConditionsWithPagination($conditions, $arrayModel, $perPage);
    }
```

This function have called **getRelationshipWhereConditionsWithPagination()** to get data with conditions have relationship with another model. Then return all data with pagination.

And on the controller, we can call this function to get data.

Example:

```php
# on UserController.php

    public const USER_PER_PAGE = 20;

    /**
     * Get all users
     * @return \Illuminate\Http\JsonResponse
     */
    public function getActiceUsers() {
        $users = $this->userRepository->getDataPaginate(['active', '=', 1], self::USER_PER_PAGE);
        return response()->json($users);
    }
```

In this example, **$users** is an instance of User with active is 1 and have relationship with orders and shops. Also, **$users** will be paginated with 20 users per page.

### getRelationshipWhereConditionsFirst($id, $conditions, $arrayModel)

Find data of Object with relationship another model from the query of conditions.

Parameters:

-   $id: id of Object. Example: 1 (int : id of User)
-   $conditions: array of conditions to get data. Example: ['name', '=', 'John Doe']
-   $arrayModel: array of relationship model. Example: ['orders', 'shops']

On this parameter, the $conditions is unnecessary but we should pass it to this function. Because ensure safety of data.

Example:

```php
# on UserRepository.php

    /**
     * Get all users
     * @return User[]|\Illuminate\Database\Eloquent\Collection
     */
    public function getDataFirst($id, $conditions) {
        $arrayModel = ['orders', 'shops'];
        return $this->model->getRelationshipWhereConditionsFirst($id, [], $arrayModel);
    }
```

This function have called **getRelationshipWhereConditionsFirst()** to find data of Object have relationship with another model. Then return data of Object.

---

Currently, I have applied this repository for order manage on Admin page. And it works well. Please review it on Admin/Order/OrderController.php and OrderRepository.php.
