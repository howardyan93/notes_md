django源代码阅读之clickjacking
========================

最近打算开始学习django，然后通过梳理一些django的功能的方式来读一下django的源代码。

首先是clickjacking这个中间件。在所有中间件里面，算是比较简单好梳理的。

clickjacking的原理见[wiki](https://en.wikipedia.org/wiki/Clickjacking#Server_and_client)

在web http协议里，防护clickjacking的主要是通过在header里面添加X-Frame-Options来禁止加载伪装成透明的iframe。

首先直接看代码django/middleware/clickjacking.py:

    class XFrameOptionsMiddleware(MiddlewareMixin):
        """
        Middleware that sets the X-Frame-Options HTTP header in HTTP responses.

        Does not set the header if it's already set or if the response contains
        a xframe_options_exempt value set to True.

        By default, sets the X-Frame-Options header to 'SAMEORIGIN', meaning the
        response can only be loaded on a frame within the same site. To prevent the
        response from being loaded in a frame in any site, set X_FRAME_OPTIONS in
        your project's Django settings to 'DENY'.

        Note: older browsers will quietly ignore this header, thus other
        clickjacking protection techniques should be used if protection in those
        browsers is required.

        https://en.wikipedia.org/wiki/Clickjacking#Server_and_client
        """
        def process_response(self, request, response):
            # Don't set it if it's already in the response
            if response.get('X-Frame-Options') is not None:
                return response

            # Don't set it if they used @xframe_options_exempt
            if getattr(response, 'xframe_options_exempt', False):
                return response

            response['X-Frame-Options'] = self.get_xframe_options_value(request,
                                                                        response)
            return response

教程里面，要实现自己定制的中间件，一般要实现process_request，process_view，process_response，process_exception这几个函数。但是clickjacking只需要对返回response的X-Frame-Options进行控制，所以这里只需要实现process_response函数。

然后在process_response里面调用的get_xframe_options_value的也没做什么事情, 只是从settings里面读取默认的X_FRAME_OPTIONS配置：

    def get_xframe_options_value(self, request, response):
        """
        Gets the value to set for the X_FRAME_OPTIONS header.

        By default this uses the value from the X_FRAME_OPTIONS Django
        settings. If not found in settings, defaults to 'SAMEORIGIN'.

        This method can be overridden if needed, allowing it to vary based on
        the request or response.
        """
        return getattr(settings, 'X_FRAME_OPTIONS', 'SAMEORIGIN').upper()

由于clickjacking中间件跟其他通用中间件作为全局的配置，所有reqeust的请求都会经过其处理，如果想要某个视图函数不进行clickjacking，则可以这样做：

    from django.views.decorators.clickjacking import xframe_options_exempt

    @xframe_options_exempt
    def xxx_view(request):
        do_something
        return render(request, 'xxx.html')

而xframe_options_exempt的代码则位于/django/views/decorators下：

    def xframe_options_exempt(view_func):
        """
        Modifies a view function by setting a response variable that instructs
        XFrameOptionsMiddleware to NOT set the X-Frame-Options HTTP header.
        
        e.g.

        @xframe_options_exempt
        def some_view(request):
            ...
        """
        def wrapped_view(*args, **kwargs):
            resp = view_func(*args, **kwargs)
            resp.xframe_options_exempt = True
            return resp
        return wraps(view_func, assigned=available_attrs(view_func))(wrapped_view)

可以看到，要移除clickjacking的防护，只需要简单的在设置resp的xframe_options_exempt即可。回想最开始的代码段，里面只要发现resp的这个字段为True，就直接跳过:

    # Don't set it if they used @xframe_options_exempt
    if getattr(response, 'xframe_options_exempt', False):
        return response

在同一文件里面，我们还可以看到对clickjacking还有两个装饰器函数xframe_options_sameorigin和xframe_options_deny。这两个函数可以在全局不设置clickjacking中间件的情况下对指定的视图函数进行wrap。

最后我们在看一下wrap装饰器：

    return wraps(view_func, assigned=available_attrs(view_func))(wrapped_view)

默认assigned的参数为:

    wraps(wrapped, assigned=('__module__', '__name__', '__doc__'), updated=('__dict__',))
        Decorator factory to apply update_wrapper() to a wrapper function
        
        Returns a decorator that invokes update_wrapper() with the decorated
        function as the wrapper argument and the arguments to wraps() as the
        remaining arguments. Default arguments are as for update_wrapper().
        This is a convenience function to simplify applying partial() to
        update_wrapper().

但是django为了考虑python2和python3的兼容性，所以写了一个available_attrs，其实干的事情就是获取('__module__', '__name__', '__doc__')，代码位于：

    from functools import WRAPPER_ASSIGNMENTS, update_wrapper, wraps
    
    def available_attrs(fn):
        """
        Return the list of functools-wrappable attributes on a callable.
        This is required as a workaround for http://bugs.python.org/issue3445
        under Python 2.
        """
        if six.PY3:
            return WRAPPER_ASSIGNMENTS
        else:
            return tuple(a for a in WRAPPER_ASSIGNMENTS if hasattr(fn, a))

看多了就会发现，django的中间件都会采用设置某一个参数为True的来控制是否对request和respone进行处理，属于基本套路。
