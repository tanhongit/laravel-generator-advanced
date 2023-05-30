# Send Email using Queue with Redis on Laravel

This document shows an example of how to send email using `Queue` with [Redis](https://redis.io/) as a driver on Laravel.

## 1. Config required environment

To send email using `Queue` of laravel, need to set `QUEUE_CONNECTION=redis` environment variable inside the file `.env`.

Define the redis queue driver on the “.env” file as follows:

```env
QUEUE_CONNECTION=redis
```

**Note** The path to the file should be at the root of the project.

Then run the following command to install the `predis/predis` package:

```bash
composer require predis/predis
```

After that, add new environment variable `REDIS_CLIENT` to the file `.env`:

```env
REDIS_CLIENT=predis
```

Generate migration file for the z```failed_jobs` table:

```bash
php artisan queue:failed-table
```

Then run the following command to create these tables:

```bash
php artisan migrate
```

## 2. Create model and migration Order for using of this example

### 2.1 Create model Order
For easy to understand, we will create the the order model, migration, and controller files. These will be used to send email for **_created order action_**.

```bash
php artisan make:model Order -sm
```

### 2.2 Create migration file

Then, we will create the migration file for the `orders` table. This table will be used to store the order data.

Add the following code to the migration file `database/migrations/xxxx_xx_xx_xxxxxx_create_orders_table.php`:

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateOrdersTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('orders', function (Blueprint $table) {
            $table->id();
            $table->string('name', 100);
            $table->unsignedInteger('item_count');
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('orders');
    }
}
```

Now run `php artisan migrate` command to save it into database.

### 2.3 Add seeders for the orders table

Add the following code to the file `database/seeders/OrderSeeder.php`:

```php
<?php

namespace Database\Seeders;

use App\Models\Order;
use Illuminate\Database\Console\Seeds\WithoutModelEvents;
use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\DB;

class OrderSeeder extends Seeder
{
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        $data = array();

        for ($i = 1; $i <= 10; $i++) {
            $data[] = [
                'id' => $i,
                'name' => "Order $i",
                'item_count' => random_int(1, 10),
                'created_at' => now(),
            ];
        }

        DB::beginTransaction();
        try {
            Order::insert($data);
            DB::commit();
        } catch (\Exception $e) {
            DB::rollBack();
            throw $e;
        }
    }
}
```

Then, run the following command to seed the data into the `orders` table:

```bash
php artisan db:seed OrderSeeder
```

## 3. Create the view file layout for email

Create the view file, and it name will be `queueMail.blade.php` on the folder `resources/views/emails/` directory.

And this is the content of the view file:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Send mail Queue Job Laravel 9</title>
</head>
<body>
<p>Hi, Send mail Queue Job</p>
<p>
    Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do
    eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim
    ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut
    aliquip ex ea commodo consequat. Duis aute irure dolor in
    reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla
    pariatur. Excepteur sint occaecat cupidatat non proident, sunt in
    culpa qui officia deserunt mollit anim id est laborum.
</p>

<p>Track order : {{ $order->id }}</p>
<p>Order name : {{ $order->name }}</p>

<strong>Thank you Send mail Queue Job. </strong>
</body>
</html>
```

## 4. Create the mail class

Create the mail class, and it name will be `SendMail` using the artisan command.

```bash
php artisan make:mail SendMailQueueJob
```

And then update the `build` method inside the file `app/Mail/SendMailQueueJob.php` as follows:

```php
<?php

namespace App\Mail;

use Illuminate\Bus\Queueable;
use Illuminate\Mail\Mailable;
use Illuminate\Queue\SerializesModels;

class SendMailQueueJob extends Mailable
{
    use Queueable, SerializesModels;

    public $order;
    /**
     * Create a new message instance.
     *
     * @return void
     */
    public function __construct(Order $order)
    {
        $this->order = $order;
    }

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->view('emails.queueMail');
    }
}
```

On this file, I have added the `order` property to the class. This property will be used to send the order data to the
view file (`queueMail.blade.php`).

## 5. Create the controller for sending email

### 5.1 Create the controller file

Create the controller, and it name will be `SendMailController` using the artisan command.

```bash
php artisan make:controller SendMailController
```

And then add the `sendMail` method inside the file `app/Http/Controllers/SendMailController.php` as follows:

```php
<?php

namespace App\Http\Controllers;

use App\Mail\SendMailQueueJob;
use App\Models\Order;
use Illuminate\Support\Facades\Mail;

class SendMailController extends Controller
{
    public function sendMail() {
        $order = Order::findOrFail(rand(1,10));

        $recipient = 'hello@example.com';

        Mail::to($recipient)->send(new SendMailQueueJob($order));

        return 'Sent order ' . $order->id;
    }
}
```

### 5.2 Create the route for sending email

Add the following code to the file `routes/web.php`:

```php
Route::get('/send-mail', [SendMailController::class, 'sendMail']);
```

> Don't forget to import the `SendMailController` class on web.php file.

Open the browser and go to the URL `http://localhost:8000/send-mail` to send the email.

## 6. Create the queue job for sending email

### 6.1 Create the queue job file

Create the queue job, and it name will be `SendMailQueueJob` using the artisan command.

```bash
php artisan make:job SendMailQueueJob
```

And then update the `handle` method inside the file `app/Jobs/SendMailQueueJob.php` as follows:

```php
<?php

namespace App\Jobs;

use App\Mail\SendMailQueueJob as SendMailQueueJobMail;
use App\Models\Order;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldBeUnique;
use Illuminate\Contracts\Queue\ShouldBeUniqueUntilProcessing;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;
use Illuminate\Support\Facades\Mail;

class SendMailQueueJob implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public $order;
    /**
     * Create a new job instance.
     *
     * @return void
     */
    public function __construct(Order $order)
    {
        $this->order = $order;
        $this->queue = 'send-mail';
    }

    /**
     * Execute the job.
     *
     * @return void
     */
    public function handle()
    {
        $recipient = 'test@msaail.com';
        
        Mail::to($recipient)->send(new SendMailQueueJobMail($this->order));
    }
}
```

On this file, I have added the `order` property to the class. This property will be used to send the order data to the mail class (`SendMailQueueJobMail`).

## 7. Create the queue worker

Create the queue worker using the artisan command.

```bash
php artisan queue:work --queue=send-mail
```

## 8. Update the controller for sending email

Update the `sendMail` method inside the file `app/Http/Controllers/SendMailController.php` as follows:

```php
<?php

namespace App\Http\Controllers;

use App\Jobs\SendMailQueueJob;
use App\Models\Order;
use Illuminate\Support\Facades\Mail;

class SendMailController extends Controller
{
    public function sendMail() {
        $order = Order::findOrFail(rand(1,10));

        SendMailQueueJob::dispatch($order);

        return 'Sent order ' . $order->id;
    }
}
```

Open the browser and go to the URL `http://localhost:8000/send-mail` to send the email.

## 9. Conclusion

In this tutorial, I have shown you how to send email using queue job in Laravel 8.
