# Flask Signals

Flask的信号是基于blinker实现，其中Flask注册的信号有：

* template_rendered
* before_render_template
* request_finished
* request_tearing_down
* got_request_exception
* appcontext_tearing_down
* appcontext_pushed
* appcontext_popped
* message_flashed

> Note: 
> Blinker provides a fast dispatching system that allows any number of interested parties to subscribe to events, or "signals".


## Signal Test
参考[flask_signal.py](https://bitbucket.org/hotbaby/python/src/f9d4fa8d9090ee9a7d634464ddc18e1af91d3baf/flask/flask_signals.py)