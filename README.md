# action-cable-react-jwt

Same as [action-cable-react](https://github.com/schneidmaster/action-cable-react), but allows authenticating websockets using JWTs

## Usage

Import ActionCableJwt

```javascript
import {ActionCable} from 'action-cable-jwt.js';
// if you don't use ES6 then use
// const {ActionCable} = require('action-cable-jwt.js');

```

Creating an actioncable websocket

```javascript
let App = {};
App.cable = ActionCable.createConsumer("ws://cable.example.com", jwt) // place your jwt here

```

Subscribing to a Channel for Receiving data

```javascript
this.subscription = App.cable.subscriptions.create({channel: "YourChannel"}, {
      connected: function() { console.log("cable: connected") },             // onConnect
      disconnected: function() { console.log("cable: disconnected") },       // onDisconnect
      received: (data) => { console.log("cable received: ", data); }         // OnReceive
}

```

Send data to a channel

```javascript
this.subscription.send('hello world')

```

Call a method on channel with arguments

```javascript
this.subscription.perform('method_name', arguments)

```

In your `ApplicationCable::Connection` class in Ruby add

```ruby
# app/channels/application_cable/connection.rb
module ApplicationCable
  class Connection < ActionCable::Connection::Base
    identified_by :current_user

    def connect
      self.current_user = find_verified_user
    end

    private

    def find_verified_user
      begin
        header_array = self.request.headers[:HTTP_SEC_WEBSOCKET_PROTOCOL].split(',')
        token = header_array[header_array.length-1]
        decoded_token = JWT.decode token, Rails.application.secrets.secret_key_base, true, { :algorithm => 'HS256' }
        if (current_user = User.find((decoded_token[0])['sub']))
          current_user
        else
          reject_unauthorized_connection
        end
      rescue
        reject_unauthorized_connection
      end
    end

  end
end
```

And in YourChannel.rb

```ruby
# app/channels/you_channel.rb
class LocationChannel < ApplicationCable::Channel

  # calls connect in client
  def subscribed
    stream_from 'location_user_' + current_user.id.to_s
  end

  # calls disconnect in client
  def unsubscribed
    # Any cleanup needed when channel is unsubscribed
  end
  
  # called when send is called in client
  def receive(params)
    print params[:data]
  end
  
  # called when perform is called in client
  def method_name(params)
    print params[:data]
  end
  
end
```


## License

MIT

Copyright (c) 2017 Zeke

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.


