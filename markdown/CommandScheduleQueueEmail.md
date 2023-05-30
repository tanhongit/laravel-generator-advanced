Laravel Task Scheduling automatically runs the commands at a given time. This is useful for running scheduled jobs, such as sending emails, running backups, or running database maintenance tasks. Laravel's scheduler is based on the [cron](https://en.wikipedia.org/wiki/Cron) utility, so you should be familiar with cron expressions before using Laravel's scheduler.

## 1. Config required environment
`Laravel` uses the `cron` utility to schedule commands. So, you need to install `cron` on your server. If you are using `Ubuntu`, you can install `cron` by running the following command:

```bash
sudo apt-get install cron
```

Then, add the following line to the file `/etc/crontab`:

```bash
* * * * * root /usr/bin/php /path-to-your-project/artisan schedule:run >> /dev/null 2>&1
```

**Note** The path to the file should be at the root of the project.

## 2. Create send email job

### 2.1 Create job file

For easy to understand, we will create the the job file. This will be used to send email for **_created order action_**.

```bash
php artisan make:job SendEmailJob
```

### 2.2 Create mail file

Then, we will create the mail file for the `orders` table. This table will be used to store the order data.

Add the following code to the mail file `app/Mail/SendEmail.php`:

```php
<?php

namespace App\Mail;

use Illuminate\Bus\Queueable;
use Illuminate\Mail\Mailable;
use Illuminate\Queue\SerializesModels;

class SendEmail extends Mailable
{
    use Queueable, SerializesModels;

    /**
     * Create a new message instance.
     *
     * @return void
     */
    public function __construct()
    {
        //
    }

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->view('emails.send_email');
    }
}
```

### 2.3 Create view file

Then, we will create the view file for the `orders` table. This table will be used to store the order data.

Add the following code to the view file `resources/views/emails/send_email.blade.php`:

```php
<!DOCTYPE html>
<html>
<head>
    <title>Send Email</title>
</head>
<body>
    <h1>Send Email</h1>
    <p>This is a test email.</p>
</body>
</html>
```

### 2.4 Update the job file

Update the job file `app/Jobs/SendEmailJob.php` as follows:

```php
<?php

namespace App\Jobs;

use App\Mail\SendEmail;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;
use Illuminate\Support\Facades\Mail;

class SendEmailJob implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    /**
     * Create a new job instance.
     *
     * @return void
     */
    public function __construct()
    {
        //
    }

    /**
     * Execute the job.
     *
     * @return void
     */
    public function handle()
    {
        Mail::to('the_email_address_you_want_to_send_to')->send(new SendEmail());
    }
}
```

## 3. Create command

### 3.1 Create command

For easy to understand, we will create the command file. This will be used to send email for **_created order action_**.

```bash
php artisan make:command SendEmailOrderCreated
```

Then, add the following code to the command file `app/Console/Commands/SendEmailOrderCreated.php`:

```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use App\Models\Order;

class SendEmailOrderCreated extends Command
{
    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'send:email-order-created';

    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = 'Send email for created order action';

    /**
     * Create a new command instance.
     *
     * @return void
     */
    public function __construct()
    {
        parent::__construct();
    }

    /**
     * Execute the console command.
     *
     * @return int
     */
    public function handle()
    {
        $orders = Order::where('status', 'created')->get();
        foreach ($orders as $order) {
            
            // Send email for order here
            \App\Jobs\SendingEmailOrderCreated::dispatch($order);
            
            $order->status = 'sent';
            $order->save();
        }
    }
}
```

### 3.2 Register command

Then, register the command in the `app/Console/Kernel.php` file:

```php
protected $commands = [
    \App\Console\Commands\SendEmailOrderCreated::class,
];
```

## 4. Schedule command

Then, add the following code to the `app/Console/Kernel.php` file:

```php
protected function schedule(Schedule $schedule)
{
    $schedule->command('send:email-order-created')->everyMinute();
}
```

## 5. Run queue

**Away 1**: Run the following command to run the scheduler:

```bash
php artisan queue:work --queue=THE_QUEUE_NAME
```

**Away 2**: If you want to run the queue in the background, you can run the following command:

```bash
php artisan queue:work --queue=THE_QUEUE_NAME --daemon
```

**Away 3**: If you want to run the queue by add the following line to the file `/etc/crontab`:

Run to open the crontab file:

```bash
crontab -e
```

Or, you can run the following command to open the crontab file (easy to tracking):

```bash
sudo nano /etc/crontab
```

Add the following line to the file `/etc/crontab`:

```bash
* * * * * root /usr/bin/php /path-to-your-project/artisan queue:work --queue=THE_QUEUE_NAME >> /dev/null 2>&1
```

> **Note**: 
> Replace `THE_QUEUE_NAME` with the queue name you want to run. And replace `/path-to-your-project` with the path to your project.
> 
> On the line above, we use `* * * * *` to run the queue every minute. You can change it to run the queue every hour, every day, every week, every month, every year, etc.
> 
> /dev/null 2>&1 is used to hide the output of the command (This is the part that manages the output of the command. You can remove it if you want to see the output of the command).

**Away 4 (Using on this project now)**: If you want to run the queue by add code to the `app/Console/Kernel.php` file:

```php
protected function schedule(Schedule $schedule)
{
    $schedule->command('queue:work --queue=THE_QUEUE_NAME');
}
```

## 6. Add schedule to cron

Add the following line to the file `/etc/crontab`:

First, run command to open the file:

```bash
sudo crontab -e
```

Or you can change the file by using the following command (easy to tracking):

```bash
sudo nano /etc/crontab
```

Then, add the following line to the file:

```bash
* * * * * root /usr/bin/php /path-to-your-project/artisan schedule:run >> /dev/null 2>&1
```

> **Note**: Replace `/path-to-your-project` with the path to your project.

## 7. Note

On the cronjob line, you can change the time to run the command. For example, you can change the `* * * * *` to `*/5 * * * *` to run the command every 5 minutes.

## 8. Conclusion

In this tutorial, we have learned how to send email by using Laravel queue. We have created the `orders` table to store the order data. Then, we have created the command to send email for created order action. Finally, we have scheduled the command to run every minute (* * * * *).

## 9. Reference

[https://laravel.com/docs/8.x/queues](https://laravel.com/docs/8.x/queues)

[https://laravel.com/docs/8.x/scheduling](https://laravel.com/docs/8.x/scheduling)

[https://laravel.com/docs/8.x/mail](https://laravel.com/docs/8.x/mail)

[https://laravel.com/docs/8.x/queues#running-the-queue-worker](https://laravel.com/docs/8.x/queues#running-the-queue-worker)

## 10. Additional

### 10.1 Schedule on Windows

In the Windows operating system, there is a tool Windows Task Scheduler that can be used to schedule tasks. You can use this tool to schedule the command to run every minute.

If you want to run the schedule on Windows, please follow the steps below:

Way 1: Search programs and files box (Windows key + R shortcut) and type `taskschd.msc` to open the Windows Task Scheduler.

Way 2: Right-click My Computer and select Manage. Then, select Task Scheduler in the left pane.

---

Step 1: In the right pane, right-click Task Scheduler Library and select **Create Task**.

Step 2: In the **General** tab, enter the name of the task and select **Run whether user is logged on or not** on the **Security Options** section.

Step 3: In the **Triggers** tab, select **New** and select **Begin the task** on the **Begin the task** section.

Step 4: In the **Actions** tab, select **New** and select **Start a program** on the **Action** section.

Step 5: In the **Program/script** box, enter the following command:

```bash
php C:\path-to-your-project\artisan schedule:run
```

> **Note**: Replace `C:\path-to-your-project` with the path to your project.

**_Here are some notes:_**

Program/script: The path to the php.exe file. For example, `C:\xampp\php\php.exe`.

Add arguments (optional): The path to the artisan file. For example, `C:\xampp\htdocs\laravel-queue\artisan`.
