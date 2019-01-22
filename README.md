
Push service laravel without PUSHER

for mac
```sh
brew cask install docker
brew install composer
brew install npm

brew uninstall gnu-sed
brew install gnu-sed --with-default-names
echo 'export PATH="/usr/local/opt/gnu-sed/libexec/gnubin:$PATH"' >> ~/.bash_profile 
#reopen terminal
```



```sh
composer create-project --prefer-dist laravel/laravel push
docker run --name host-redis -p 6379:6379 -d redis 
cd push 
composer require predis/predis
sed -i "s/^BROADCAST_DRIVER=.*/BROADCAST_DRIVER=redis/" .env
sed -i "s/^QUEUE_CONNECTION=.*/QUEUE_CONNECTION=redis/" .env
php artisan config:cache
php artisan make:event EventMessagePush
```

edit app/Events/EventMessagePush.php
```php
<?php

namespace App\Events;

use Illuminate\Broadcasting\Channel;
use Illuminate\Queue\SerializesModels;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Broadcasting\PresenceChannel;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;

class EventMessagePush implements ShouldBroadcast
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    public $id;
    public $message;
    /**
     * Create a new event instance.
     *
     * @return void
     */
    public function __construct($id,$message)
    {
        $this->id=$id;
        $this->message=$message;
    }

    /**
     * Get the channels the event should broadcast on.
     *
     * @return \Illuminate\Broadcasting\Channel|array
     */
    public function broadcastOn()
    {
        return new Channel('channel-name');
    }
}
```



```sh
php artisan make:command CommandMessagePush
```

edit app/Console/Commands/CommandMessagePush.php

```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;

use App\Events\EventMessagePush;

class CommandMessagePush extends Command
{
    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'send:message {id} {message}';

    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = 'Send message to browser';

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
     * @return mixed
     */
    public function handle()
    {
        event(new EventMessagePush(
            $this->argument('id'),
            $this->argument('message'))
        );
    }
}

```



ADD TO FILE ./bootstrap.js
```js
//Begin laravel Echo
import Echo from 'laravel-echo'

window.io = require('socket.io-client');

window.Echo = new Echo({
    broadcaster: 'socket.io',
    host: window.location.hostname + ':6001'
});

window.Echo.channel('channel-name')
    .listen('EventMessagePush', (e) => {
        console.log(e);
    });
//End laravel Echo
```

add to file resources/views/welcome.blade.php

```html
    <!-- CSRF Token -->
    <meta name="csrf-token" content="{{ csrf_token() }}">
    <!-- Scripts -->
    <script src="{{ asset('js/app.js') }}" defer></script>

```

```sh
npm i
npm install --save laravel-echo
npm install --save socket.io-client
npm run prod

npm install -g laravel-echo-server

laravel-echo-server init 

laravel-echo-server start 

php artisan queue:listen --tries=1

php artisan serve
```

open http://localhost:8000

```sh
php artisan send:mes 1 "hi it is PUSH mess"
```
OPEN WEB TOOLS CONSOLE 